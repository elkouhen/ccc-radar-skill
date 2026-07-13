# Règles de conception technique

Règles opposables en revue de code pour des services Java qui manipulent des
fichiers volumineux et des flux d'événements Kafka. Chaque règle a un
identifiant (R1, R2…) pour être citée dans les PR et les ADR. Les identifiants
sont repris dans `metadata.regle` et `metadata.reference` des règles Semgrep
de ce pack (`a-memoire-fichiers.yaml`, `b-kafka.yaml`, `c-donnees.yaml`,
`d-pdf-archives.yaml`), qui n'en couvrent que le sous-ensemble détectable
statiquement — R4, R9, R11, R14, R15, R16 restent des règles de conception
non outillées ici.

---

## A. Gestion mémoire et fichiers

### R1 — Aucun fichier chargé entièrement en mémoire

Interdits sur tout flux de fichier volumineux (document, pièce jointe,
archive) :

- `byte[] content = inputStream.readAllBytes()` / `Files.readAllBytes(...)`
- `ByteArrayOutputStream` accumulant un fichier complet
- `String xml = new String(bytes)` sur un document entier
- Parsing DOM (`DocumentBuilder`) ou unmarshalling JAXB d'un document non borné

Obligatoire :

- Traitement par flux : `InputStream` → traitement → `OutputStream`, buffers
  bornés (8–64 Ko).
- XML : **StAX** (`XMLStreamReader`) ou SAX pour la validation et
  l'extraction. JAXB acceptable uniquement sur des sous-arbres bornés extraits
  via StAX (ex. un élément métier unitaire), jamais sur le document racine
  d'un flux entrant.
- PDF : extraction d'un contenu embarqué (XML, pièce jointe) via lecture
  incrémentale (PDFBox en mode `RandomAccessRead` sur fichier temporaire
  local — spool depuis l'object storage, voir R17 —, pas en `byte[]`).
- Empreintes (SHA-256) calculées au fil de l'eau (`DigestInputStream`)
  pendant l'upload, pas dans une passe séparée en mémoire.

### R2 — Upload et download en streaming pur

- HTTP entrant : multipart/streaming consommé au fil de l'eau et **écrit
  directement vers l'object storage** (multipart upload). Le corps de
  requête ne transite jamais par un buffer complet en heap.
- Taille max de requête imposée au niveau du serveur ET vérifiée au fil de
  l'eau (compteur d'octets) — ne pas faire confiance au `Content-Length`.
- Download/restitution : proxy en streaming object storage → client, ou URL
  présignée.

### R3 — Bornes explicites partout

Tout ce qui est non borné finit par faire tomber un pod. Imposer :

- taille max de fichier acceptée (et par type de contenu) ;
- profondeur et nombre d'éléments max en parsing XML (protection XXE et
  « billion laughs » : `XMLInputFactory` avec `SUPPORT_DTD=false` et
  `IS_SUPPORTING_EXTERNAL_ENTITIES=false`) ;
- taille max des collections chargées depuis la DB (pagination obligatoire,
  pas de `findAll()`) ;
- pools bornés (connexions DB, clients object storage, threads).

Si une interface externe (partenaire, régulateur) impose ses propres limites
(taille de flux, taille par fichier, quotas), les documenter et les
reprendre comme point de départ pour dimensionner les bornes internes du
service — elles ne dispensent pas de bornes propres, potentiellement plus
strictes, sur les flux internes non contraints par cette interface.

### R4 — Dimensionnement JVM standardisé

- Java 21+, G1 par défaut ; ZGC pour les services sensibles à la latence.
- Heap plafonnée par service et cohérente avec les limites mémoire du
  conteneur : `-Xmx` explicite, ou à défaut `MaxRAMPercentage` ≤ 75 % de la
  limite du pod (les deux sont exclusifs — `-Xmx` prime s'il est posé).
- Les I/O bloquantes (object storage, DB, HTTP) tournent sur **virtual
  threads** — pas de réactif complexe pour compenser des lectures en mémoire
  qu'on n'aurait pas dû faire.
- Tout service traitant des fichiers doit avoir un test de charge prouvant
  une consommation heap **constante** quelle que soit la taille du fichier
  (fichier de 1 Ko vs 100 Mo → même profil mémoire).

---

## B. Kafka et flux événementiels

### R5 — Claim check : jamais de payload fichier dans Kafka

Un événement porte : identifiants métier, référence(s) vers l'object storage
(bucket/key — fichier brut, et le cas échéant jeu de données extrait),
`sha256`, taille, format, horodatage. Jamais le contenu. Taille max de
message applicative : 256 Ko (marge sous la limite broker de 1 Mo).

### R6 — Partitionnement par clé métier, ordre garanti par entité

- Clé de partition : un identifiant métier stable (ex. l'identifiant de
  l'entité concernée, ou une clé de regroupement pour un topic de lot) pour
  garantir l'ordre des événements relatifs à une même entité.
- Ne jamais dépendre d'un ordre global inter-entités.
- Nombre de partitions dimensionné dès la création (ex. 48–96 sur les topics
  chauds) : le repartitionnement casse les garanties d'ordre.

### R7 — At-least-once + idempotence, pas de « exactly-once » magique

- Producteurs : `enable.idempotence=true`, `acks=all` (défauts depuis
  Kafka 3.0 — les poser explicitement en config pour qu'aucun réglage
  voisin ne les désactive silencieusement).
- Consommateurs : commit après traitement ; tout handler doit être
  **rejouable** — déduplication par clé métier (contrainte unique en DB ou
  table `processed_events`).
- Transactions Kafka réservées aux topologies Kafka→Kafka (Streams) ; pour
  DB+Kafka, c'est l'outbox (R8).

### R8 — Outbox pattern pour toute écriture DB + événement

Écriture de l'état et de l'événement dans la même transaction PostgreSQL
(table outbox), publication vers Kafka par un relais (Debezium ou poller).
La double écriture directe DB puis Kafka est interdite.

### R9 — Contrats d'événements versionnés

- Schema Registry + Avro (ou Protobuf), compatibilité `BACKWARD` imposée.
- Un topic = un contrat ; convention de nommage
  `<contexte>.<agrégat>.<fait>` (ex. `commande.paiement.effectue`).
- Les événements sont des **faits passés** immuables, nommés au participe
  passé — jamais des commandes déguisées.

### R10 — DLQ et rejeu systématiques

- Chaque consumer group a sa DLQ (`<topic>.<groupe>.dlq` — le groupe fait
  partie du nom : plusieurs groupes peuvent consommer le même topic) avec
  l'erreur et les en-têtes d'origine.
- Retries : d'abord in-process avec backoff (erreurs transitoires), puis DLQ.
  Jamais de retry infini qui bloque une partition.
- Procédure de rejeu documentée et testée (le rejeu est un cas nominal
  d'exploitation, pas un incident).

### R11 — Backpressure par le consommateur

Le débit d'entrée (dépôts) ne doit jamais imposer son rythme aux traitements
aval : Kafka est le tampon. Dimensionner par le **lag** (alerte + autoscaling
des consumers sur le lag), pas par des files en mémoire.

---

## C. Données, archivage, exploitation

### R12 — L'object storage est la source de vérité des fichiers

La base de données ne stocke jamais de contenu de fichier (pas de
`bytea`/LOB). Métadonnées en DB, contenu en object storage, lien par clé +
empreinte.

### R13 — Archivage immuable

- Bucket dédié avec verrouillage d'objet (mode compliance/WORM), durée de
  rétention explicite et alignée sur les obligations réglementaires
  applicables au contexte (à déterminer projet par projet — retenir la durée
  la plus contraignante s'il y en a plusieurs).
- Scellement : empreinte SHA-256, horodatage qualifié (RFC 3161),
  journal de preuve chaîné (chaque entrée référence l'empreinte de la
  précédente).
- Toute restitution vérifie l'empreinte avant livraison.

### R14 — Idempotence des dépôts

Un même fichier redéposé (même émetteur + même empreinte) ne crée pas de
doublon : réponse au client avec la référence existante. Contrainte unique
`(emetteur, sha256)` dès l'acquisition.

### R15 — Observabilité orientée volumétrie

- Métriques obligatoires par service : lag Kafka par consumer group, débit
  (unités métier/s), p99 de traitement, heap/GC pause, taille des DLQ.
- Traçabilité : un identifiant métier (et un identifiant de lot le cas
  échéant) propagés en header Kafka et en contexte de log (MDC) sur toute la
  chaîne — une entité doit être traçable de bout en bout, un lot
  reconstituable.
- SLO explicites par flux (ex. dépôt→statut « déposé » < 5 s au p99,
  dépôt→traitement aval < 15 min).

### R16 — Tests de charge comme critère d'acceptation

Avant toute mise en production d'un contexte : test de charge au débit cible
(dimensionner pour le pic, ex. périodes de clôture = 5–10× le débit moyen),
avec vérification du profil mémoire (R4) et du lag (R11).

---

## D. Contenus non fiables : PDF et archives compressées

Tout fichier déposé est une **entrée hostile** jusqu'à preuve du contraire
(bombe de décompression, PDF malformé, Zip Slip). Les règles A s'appliquent,
avec les adaptations suivantes.

### R17 — PDF : accès aléatoire sur disque, jamais en heap

Un PDF n'est **pas streamable** : sa table xref est en fin de fichier, le
parsing exige un accès aléatoire. R1 s'adapte donc ainsi :

- **Spool sur disque local** (volume temporaire du pod), jamais en `byte[]` :
  object storage → fichier temporaire en streaming, puis parsing en accès
  aléatoire.
- PDFBox 3.x : `Loader.loadPDF(new RandomAccessReadBufferedFile(file))` avec
  cache de streams **sur disque** (`IOUtils.createTempFileOnlyStreamCache()`),
  jamais le cache mémoire par défaut sur des fichiers non bornés.
- Contenu embarqué (pièce jointe PDF/A-3, XML) : l'extraction ne nécessite
  aucun rendu de page — extraire la pièce, la streamer vers l'object
  storage/le parseur StAX, fermer le document.
- Bornes obligatoires : taille max, nombre de pages max, timeout de parsing,
  nombre max de pièces embarquées.
- **Isolation** : le parsing PDF tourne dans des workers dédiés (consumers
  Kafka séparés, mémoire plafonnée). Un PDF piégé peut tuer un worker — il
  ne doit jamais faire tomber le service de dépôt. Le fichier fautif part en
  DLQ (R10), le worker redémarre.
- Nettoyage systématique des fichiers temporaires (`try-with-resources` +
  répertoire temporaire purgé au démarrage du pod).

### R18 — Décompression défensive, en streaming

S'applique à toute archive compressée en entrée, en particulier aux lots
multi-fichiers. `tar.gz` est un choix de référence pour ce type de flux
(Apache Commons Compress : `TarArchiveInputStream` +
`GzipCompressorInputStream`) ; les mêmes bornes s'appliquent à ZIP si ce
format doit être accepté. Le risque n'est pas le format, c'est l'absence de
bornes :

- **Streaming entrée par entrée** : décompression → compteur d'octets →
  object storage/disque, buffers bornés. Jamais d'entrée décompressée en
  `byte[]`.
- **La bombe de décompression est au niveau de la compression globale du
  flux (gzip), pas de l'archivage (tar)** : les en-têtes tar déclarent des
  tailles *non compressées* que le parseur borne à la lecture (contrairement
  à ZIP où le flux DEFLATE d'une entrée peut dépasser la taille déclarée).
  Un en-tête hostile peut néanmoins déclarer une entrée de 100 Go que le
  flux livrera docilement : compter les octets réellement lus au fil de
  l'eau et couper au dépassement reste obligatoire.
- Plafonds obligatoires (configuration par type de flux) :
  - octets décompressés max par entrée et au total ;
  - ratio de compression max **global** (octets décompressés cumulés /
    taille compressée de l'archive — le ratio par entrée n'a pas de sens
    pour du tar.gz, la compression étant globale), ex. 100:1 — au-delà,
    rejet = bombe présumée ;
  - nombre d'entrées max ;
  - **archives imbriquées interdites** (une archive dans une archive = rejet).
- **Zip Slip** (path traversal générique à tout format d'archive, tar inclus,
  malgré le nom) : rejeter toute entrée dont le nom contient `..` ou un
  chemin absolu — de toute façon, ne jamais utiliser le nom d'entrée comme
  chemin d'écriture ; écrire sous une clé/un nom généré par la plateforme.
  Spécifique à tar : rejeter aussi les entrées de type lien
  symbolique/lien physique (`TarArchiveEntry.isSymbolicLink()` /
  `isLink()`), absentes du format ZIP mais capables du même contournement.
- Même isolation que R17 : workers dédiés, DLQ, mémoire plafonnée.
