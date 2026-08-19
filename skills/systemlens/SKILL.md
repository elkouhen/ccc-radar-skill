---
name: systemlens
description: "Guide architecture-first exploration of a Java/Spring repository with SystemLens. Use when asked to use SystemLens or to inspect an indexed project's microservices, HTTP APIs, Kafka topics, MongoDB collections, build modules, dependency impact, architecture coverage, unresolved facts, message flows, or architecture exports."
---

# systemlens Architecture Explorer

Use `systemlens` to query a local, persisted Java/Spring architecture inventory.
Start with the inventory before inspecting implementation files. It describes
services, modules, HTTP APIs, Kafka topics, MongoDB collections and their
evidenced relationships.

## Documentation language

Write user-facing documentation, generated report text and examples in English.
Preserve exact CLI output, source snippets and user-provided text when quoting
them.

## Ownership

Work from the target repository root. Own the `systemlens` lifecycle for that
project.

- Run `systemlens doctor` before drawing broad architectural conclusions.
- If configuration is missing, run `systemlens init`; never overwrite an existing
  `.systemlens/config.yml`.
- If the index is absent or source/configuration changed, run `systemlens index`
  before querying. Use `systemlens index --full` after a broad refactor or an
  extractor upgrade.
- Treat `systemlens` as static, source-evidenced analysis. Keep dynamic,
  ambiguous and unresolved integrations explicit; do not infer a dependency from
  a coincidental route or a topic name.

## Setup

For a fresh project:

```bash
systemlens init
systemlens doctor
systemlens index
```

The configuration controls the indexed source perimeter. See
[settings.md](references/settings.md) only when the perimeter must change.

## Architecture workflow

Start with the indexed architecture and retrieve source evidence only when needed:

1. `systemlens microservices` — discover services and main integrations.
2. `systemlens microservices show <name>` — inspect one service.
3. `systemlens microservices topics <name>`, `apis <name>` or `mongodb <name>` —
   follow linked objects.
4. `systemlens topics`, `systemlens apis`, `systemlens dtos` or
   `systemlens mongodb` — inspect a shared object from the other direction.
5. `systemlens analyze coverage` and `systemlens analyze indexing-issues` —
   identify inventory limits before relying on a conclusion.
6. `systemlens analyze audit` — assess static topology risks.
7. `systemlens analyze microservices impact <name>` or `path <source> <target>` —
   inspect dependencies and impact paths.

```bash
systemlens microservices show order-service
systemlens topics consumers orders.created
systemlens apis consumers 'POST /payments'
systemlens mongodb services orders
systemlens modules show shared-domain
systemlens analyze coverage
systemlens analyze indexing-issues
systemlens analyze audit
```

Kafka message types are shown only when explicit in the source. A missing type
is unknown, not an invitation to infer it from a topic name or serializer.

Use `--json` when a downstream step needs structured results. For a Kafka topic,
use `systemlens topics trace <topic>`; for a Kafka topic or HTTP route, use the
MCP `trace_message_flow` tool to follow indexed source sites.

When the repository follows the documented `getTopics()` and
`${kafka.topics.*.name}` conventions, use the explicit strategy:

```bash
systemlens index --topic-strategy strategy1
```

It may derive convention-based Kafka and configured HTTP dependencies. Treat
these as convention-derived facts and inspect their source evidence. Do not
enable this strategy solely to make an expected relation appear.

For the Strategy1 OpenAPI publication convention, a declaration at
`src/main/resources/openapi/xxx.rest` can publish valid same-named contracts
anywhere in the repository. It also publishes every valid YAML or JSON OpenAPI
contract under `model-xxx/src/main/resources/openapi/`; contract file names do
not need to match `xxx`. Reindex with `--full --topic-strategy strategy1` after
adding or moving a declaration or shared contract.

Every build module independently inventories all valid YAML or JSON OpenAPI
documents under its own `src/main/resources/openapi/` directory. Contract file
names do not need to be `openapi.*` or `swagger.*`.

## Runtime APM investigation

When read-only Elasticsearch access is configured and the user needs observed
runtime service dependencies, errors, or latency, export a bounded APM digest:

```bash
systemlens apm doctor --json
systemlens apm export --since 1h --environment production --out apm-digest.json
```

Configure `SYSTEMLENS_ELASTICSEARCH_URL` and
`SYSTEMLENS_ELASTICSEARCH_API_KEY` in the shell; do not put credentials in a
command, report, or prompt. The digest contains `service_destination` metric
aggregates only, never raw spans or request data. It is a one-shot observed
dataset: it is not persisted in the SystemLens index, surfaced through MCP, or
evidence for a static source relationship. Check `coverage` before treating an
absent relation as conclusive, because the aggregation and export are bounded.

For a human performance investigation, produce the explicit self-contained
runtime report instead of asking an agent to interpret raw APM events:

```bash
systemlens apm report --since 1h --environment production --html apm-runtime.html
```

The report ranks service and transaction tail latency with average and P95,
provides a filterable service-to-transaction ownership graph, shows dependency
volume, average latency and aggregate failures, and provides a source/target
flow filter. The graph does not claim that a transaction called a dependency;
that would require a separately approved sampled-span aggregate. Dependency P95
is intentionally unavailable in this first pass.
The report contains aggregates only, not raw spans, trace IDs, request data,
or error messages. It is observed runtime context, not evidence of a static
source relationship; review its window and coverage before drawing conclusions.

## Exports and MCP

```bash
systemlens export microservices --html architecture.html
systemlens export microservices --c4 likec4-project
systemlens export modules --html module-dependencies.html
systemlens mcp
```

Exports consume the persisted snapshot; refresh the index deliberately when
source changes must be reflected. Use `--root-path` only to resolve local source
links at HTML export time.

The MCP server exposes the same indexed architecture. Start the agent from the
initialized repository so it can locate `.systemlens/config.yml` and its local
index. Query it before an edit, and call `reindex_architecture` after relevant
changes before making further architecture claims.

## References

- [settings.md](references/settings.md): project configuration.
- [management.md](references/management.md): installation, refresh and
  troubleshooting.
