---
name: cccf
description: Interroger et corriger la dette sécurité/qualité d'un repo via l'index Semgrep de ccc-findings. Déclencheurs — vulnérabilité, sécurité, semgrep, finding, dette, audit.
---

# ccc-findings (cccf)

Index Semgrep local, interrogeable en langage naturel, joint au code via `ccc`.
Le skill doit aider l'agent à répondre vite, avec peu de bruit, sans relancer un
scan complet dès qu'une question sécurité apparaît.

## Règle d'or UX

Commencer par la requête la moins coûteuse qui répond à la question, puis
demander plus de contexte seulement quand il faut agir :

1. Vue globale → `findings_summary()`.
2. Question ou fichier précis → `search_findings(...)`.
3. Exploration de code avec dette associée → `search_code_with_findings(...)`.
4. Après patch → `reindex_findings()` puis la même recherche qu'avant patch.

Ne jamais noyer l'utilisateur sous le JSON brut : résumer les findings utiles,
leurs fichiers/lignes, et l'action proposée ou réalisée.

## Choisir le bon tool

| Besoin utilisateur | Tool à utiliser | Sortie attendue côté agent |
|---|---|---|
| « Fais un état des lieux sécurité » | `findings_summary()` | 3-5 lignes : sévérités, règles principales, zones chaudes. |
| « Y a-t-il une injection SQL ? » | `search_findings(query="injection sql", limit=5)` | Findings pertinents, triés, sans contexte détaillé au premier appel. |
| « Je vais modifier ce fichier » | `search_findings(path_glob="<fichier>*", limit=10)` | Prévenir les findings connus sur le fichier avant patch. |
| « Montre le code de session avec ses problèmes » | `search_code_with_findings(query="gestion session")` | Résultats code annotés par findings et sévérité max. |
| « Corrige ce finding » | `search_findings(..., include_context=true)` puis patch | Lire le contexte avant toute modification. |
| « Vérifie après correction » | `reindex_findings()` puis `search_findings(...)` | Confirmer disparition ou expliquer le blocage. |

## Parcours agréable par défaut

Quand la demande est large ou ambiguë :

1. Appeler `findings_summary()` pour mesurer le volume.
2. Si des `ERROR` existent, chercher d'abord les problèmes critiques :
   `search_findings(query="sécurité critique", severity="ERROR", limit=5)`.
3. Proposer ou appliquer une correction seulement après avoir récupéré le
   contexte d'un finding précis avec `include_context: true`.

Quand la demande cible un fichier :

1. Appeler `search_findings(path_glob="<fichier>*", limit=10)`.
2. Si aucun finding n'est connu, le dire simplement et continuer la tâche.
3. Si des findings existent, les prendre en compte avant de modifier le fichier.

Quand la demande demande une correction :

1. Rechercher le finding avec les filtres les plus précis disponibles.
2. Relire le contexte (`include_context: true`) ou le fichier source.
3. Patcher le code en respectant `fix` si Semgrep en fournit un.
4. Si le MCP Semgrep officiel est disponible, scanner seulement le fichier
   modifié pour une vérification fraîche.
5. Appeler `reindex_findings()`.
6. Relancer la même recherche qu'au départ pour confirmer la disparition.
7. Après 2 tentatives infructueuses, arrêter et expliquer le blocage.

## Installation

1. **Semgrep** (dépendance requise par `cccf`) : `pipx install semgrep` (ou
   `brew install semgrep`).
2. **cccf** : `uv tool install ccc-findings` (ou `pipx install ccc-findings`).
3. **Modèle d'embedding** : téléchargé automatiquement au premier `cccf index`
   (`sentence-transformers`, modèle `Snowflake/snowflake-arctic-embed-xs` par
   défaut, ~100 Mo) — accès réseau nécessaire une seule fois, les exécutions
   suivantes réutilisent le cache local (`~/.cache/huggingface`).
4. **Initialiser et indexer le repo cible** :
   ```bash
   cd <votre-repo>
   cccf init                # détecte .semgrep.yml/semgrep.yml/.semgrep ;
                              # sinon, --rules <chemin-ou-pack>, sinon repli
                              # automatique sur le pack p/security-audit
   cccf index
   ```
5. **Enregistrer le serveur MCP `cccf`** auprès du client (ex. `.mcp.json` à
   la racine du projet pour Claude Code, ou l'équivalent de votre client) :
   ```json
   {"mcpServers": {"cccf": {"command": "cccf", "args": ["mcp"]}}}
   ```
6. **(Recommandé) Enregistrer aussi le MCP Semgrep officiel**, utilisé à
   l'étape 4 du Workflow 3 pour la vérification fraîche post-patch :
   ```json
   {"mcpServers": {"semgrep": {"command": "uvx", "args": ["semgrep-mcp"]}}}
   ```
   Sans lui, cette étape est simplement sautée : `reindex_findings` +
   `search_findings` (étapes 5-6 du Workflow 3) suffisent à vérifier la
   disparition du finding, avec une confiance moindre qu'un scan frais.

## Configurer les règles Semgrep à analyser

Le champ `rules` de `.cccf/config.yml` est le point de contrôle de ce qui
est analysé. Sans rien de spécifié, `cccf init` utilise déjà le pack
registry `p/security-audit` par défaut (voir Installation, étape 4) — cette
section sert à le personnaliser. Sources valides (mélangeables dans une
même liste) :

- **Fichier de règles local**, ex. `rules/rules.yml`, au format Semgrep
  standard (`rules: [{id, pattern, message, severity, languages,
  metadata: {cwe, owasp}}, ...]`).
- **Dossier de règles** : chemin vers un dossier contenant plusieurs YAML,
  chargés ensemble.
- **Pack registry Semgrep**, ex. `p/security-audit`, `p/python`,
  `p/owasp-top-ten` — accès réseau nécessaire au premier usage (Semgrep
  télécharge/cache le pack).

Définir à l'init (`--rules` est répétable) :
```bash
cccf init --rules rules/rules.yml --rules p/security-audit
```
ou éditer directement `.cccf/config.yml` a posteriori :
```yaml
rules:
  - rules/rules.yml
  - p/security-audit
```
Puis appliquer le changement à tout le repo avec un scan complet :
```bash
cccf index --full
```
**Piège pour un agent** : le tool MCP `reindex_findings` est toujours
incrémental (il ne re-scanne que les fichiers modifiés). Après un changement
de `rules`, `reindex_findings` seul ne réanalyse PAS les fichiers déjà
indexés avec l'ancienne config — il faut passer par `cccf index --full` en
CLI (pas d'équivalent `--full` côté MCP).

`min_severity` (`INFO`/`WARNING`/`ERROR`, même fichier) filtre ce qui est
conservé en base parmi ce que les règles trouvent — un réglage indépendant
du choix des règles elles-mêmes.

## Workflows de référence

### Workflow 1 — Explorer les problèmes connus

1. Appeler `search_findings(query="<description du problème>", limit=5)`.
2. Sur le finding retenu, rappeler `search_findings` avec le même filtre et
   `include_context: true` pour obtenir le code entourant le finding.

### Workflow 2 — Avant de modifier un fichier

1. Appeler `search_findings(path_glob="<fichier>*")` pour lister les
   findings connus sur ce fichier avant de le patcher.

### Workflow 3 — Boucle de correction

1. `search_findings(query="...", severity="ERROR", path_glob="...")` pour
   lister les findings à corriger.
2. Relire le contexte de chaque finding (`include_context: true`).
3. Patcher le code, en respectant le champ `fix` du finding s'il est fourni.
4. Si le MCP Semgrep officiel est disponible, appeler son tool
   `semgrep_scan` sur le seul fichier modifié pour une vérification fraîche
   immédiate (ne pas scanner tout le repo).
5. Appeler `reindex_findings` pour mettre à jour l'index `cccf`.
6. Rappeler `search_findings` avec le même filtre qu'à l'étape 1 pour
   confirmer la disparition du finding.
7. Si le finding persiste après 2 tentatives de correction, ne pas
   réessayer une 3e fois : rapporter le blocage à l'utilisateur.

### Workflow 4 — Vue d'ensemble

1. Appeler `findings_summary()` pour un état agrégé (sévérités, top
   règles) à faible coût de tokens.

## Recherche croisée code + findings

Pour explorer du code en tenant compte de sa dette sécurité, préférer
`search_code_with_findings(query="...")` à une recherche `ccc` seule : il
annote chaque résultat de code des findings Semgrep connus qui le
recouvrent.

## Anti-patterns

- Ne pas scanner tout le repo via le MCP Semgrep officiel (`semgrep_scan`
  sans cible précise) : utiliser l'index `cccf` (`search_findings`,
  `findings_summary`) pour tout ce qui est déjà indexé, et ne réserver le
  MCP Semgrep officiel qu'à la vérification post-patch d'un fichier précis.
- Ne pas corriger un finding sans avoir lu son contexte
  (`include_context: true` ou lecture directe du fichier).
- Ne pas supprimer un commentaire `# nosemgrep` existant : il reflète une
  décision déjà prise sur un faux positif.
- Ne pas afficher la réponse JSON brute si l'utilisateur n'a pas demandé du
  JSON : transformer les résultats en synthèse actionnable.
- Ne pas bloquer si `ccc` est indisponible : `search_code_with_findings`
  retourne un fallback findings, à utiliser pour poursuivre l'analyse.

## Format de réponse recommandé

Répondre court et orienté décision :

```text
J'ai trouvé 2 findings ERROR dans src/payments/.
- src/payments/db.py:42 — injection SQL possible (CWE-89), corrigé et réindexé.
- src/payments/token.py:18 — génération de token faible, reste à corriger.
```

Si rien n'est trouvé :

```text
Aucun finding Semgrep connu sur src/payments/*. L'index est à jour après reindex_findings.
```
