# archlens-skill

Coding-agent skill for `archlens`, a local Java/Spring architecture explorer based
on source ASTs.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of microservices, HTTP APIs, Kafka topics, MongoDB
collections, modules, dependencies and topology risks.

## Install

```bash
npx skills add elkouhen/ccc-radar-skill
uv tool install archlens
```

In the Java/Spring repository to inspect:

```bash
archlens init
archlens doctor
archlens index
archlens analyze audit
```

No rule packs or external code-analysis tool are required.

## Contents

- [`skills/archlens/SKILL.md`](skills/archlens/SKILL.md) — architecture-first workflow.
- [`skills/archlens/references/settings.md`](skills/archlens/references/settings.md) —
  project configuration.
- [`skills/archlens/references/management.md`](skills/archlens/references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `archlens mcp` from an initialized repository. For Codex:

```bash
codex mcp add archlens -- archlens mcp
```

## License

[Apache License 2.0](LICENSE).
