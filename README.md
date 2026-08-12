# codeatlas-skill

Coding-agent skill for `codeatlas`, a local Java/Spring architecture explorer based
on source ASTs.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of microservices, HTTP APIs, Kafka topics, MongoDB
collections, modules, dependencies and topology risks.

## Install

```bash
npx skills add elkouhen/ccc-radar-skill
uv tool install codeatlas
```

In the Java/Spring repository to inspect:

```bash
codeatlas init
codeatlas doctor
codeatlas index
codeatlas analyze audit
```

No rule packs or external code-analysis tool are required.

## Contents

- [`skills/codeatlas/SKILL.md`](skills/codeatlas/SKILL.md) — architecture-first workflow.
- [`skills/codeatlas/references/settings.md`](skills/codeatlas/references/settings.md) —
  project configuration.
- [`skills/codeatlas/references/management.md`](skills/codeatlas/references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `codeatlas mcp` from an initialized repository. For Codex:

```bash
codex mcp add codeatlas -- codeatlas mcp
```

## License

[Apache License 2.0](LICENSE).
