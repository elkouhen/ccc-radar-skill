# ccc-radar-skill

Coding-agent skill for `cccr`, a local Java/Spring architecture explorer based
on source ASTs.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of microservices, HTTP APIs, Kafka topics, MongoDB
collections, modules, dependencies and topology risks.

## Install

```bash
npx skills add elkouhen/ccc-radar-skill
uv tool install ccc-radar
```

In the Java/Spring repository to inspect:

```bash
cccr init
cccr doctor
cccr index
cccr analyze audit
```

No rule packs or external code-analysis tool are required.

## Contents

- [`skills/cccr/SKILL.md`](skills/cccr/SKILL.md) — architecture-first workflow.
- [`skills/cccr/references/settings.md`](skills/cccr/references/settings.md) —
  project configuration.
- [`skills/cccr/references/management.md`](skills/cccr/references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `cccr mcp` from an initialized repository. For Codex:

```bash
codex mcp add cccr -- cccr mcp
```

## License

[Apache License 2.0](LICENSE).
