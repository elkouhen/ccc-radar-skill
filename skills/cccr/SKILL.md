---
name: cccr
description: "Use this skill for Java/Spring architecture exploration with cccr: initialize and index a project, inspect microservices, Kafka topics, HTTP APIs, MongoDB collections and modules, analyze dependencies, trace architecture flows, or export architecture graphs. Trigger phrases include 'audit', 'microservice', 'Kafka', 'REST', 'MongoDB', 'cccr', and 'architecture graph'."
---

# cccr Architecture Explorer

`cccr` derives Java/Spring architecture facts locally from source ASTs. Use its
object commands before reading implementation files: they describe services,
modules, HTTP APIs, Kafka topics, MongoDB collections and their relationships.

## Documentation language

Write user-facing documentation, generated report text and examples in English.
Preserve exact CLI output, source snippets and user-provided text when quoting
them.

## Ownership

The agent owns the `cccr` lifecycle for the current project.

- If an architecture command reports an absent index, run `cccr init`, then
  `cccr index`, and retry.
- Run `cccr index` after relevant source changes, renamed modules or a major
  refactor. Read-only queries do not require a refresh.
- Run `cccr doctor` before drawing broad architectural conclusions.
- `cccr` operates from local Java/Spring ASTs; no rules directory or external
  source-analysis tool is required.

## Setup

For a fresh project:

```bash
cccr init
cccr doctor
cccr index
```

Do not overwrite an existing `.cccr/config.yml`. It controls include/exclude
scope, the optional local embedding model and index behavior.

## Architecture workflow

Start with the indexed architecture and retrieve source only when needed:

1. `cccr microservices` — discover services and main integrations.
2. `cccr microservices show <name>` — inspect one service.
3. `cccr microservices topics <name>`, `apis <name>` or `mongodb <name>` —
   follow linked objects.
4. `cccr topics`, `cccr apis` or `cccr mongodb` — inspect a shared object from
   the other direction.
5. `cccr analyze audit` — assess topology before interpreting a dense graph.
6. `cccr analyze microservices impact <name>` or `path <source> <target>` —
   inspect dependencies and impact paths.

```bash
cccr microservices show order-service
cccr topics consumers orders.created
cccr apis consumers 'POST /payments'
cccr mongodb services orders
cccr modules show shared-domain
cccr analyze audit
```

Kafka message types are shown only when explicit in the source. A missing type
is unknown, not an invitation to infer it from a topic name or serializer.

When the repository follows the documented `getTopics()` and
`${kafka.topics.*.name}` conventions, use the explicit strategy:

```bash
cccr index --topic-strategy strategy1
```

It may derive convention-based Kafka and configured HTTP dependencies. Treat
these as convention-derived facts and inspect their source evidence.

## Exports and MCP

```bash
cccr export microservices --html architecture.html
cccr export microservices --c4 likec4-project
cccr export modules --html module-dependencies.html
cccr mcp
```

The MCP server exposes the same indexed architecture. Start the agent from the
initialized repository so it can locate `.cccr/config.yml` and its local index.

## References

- [settings.md](references/settings.md): project configuration.
- [management.md](references/management.md): installation, refresh and
  troubleshooting.
