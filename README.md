# systemlens-skill

Coding-agent skill for `systemlens`, a local Java/Spring architecture explorer based
on source ASTs.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of microservices, HTTP APIs, Kafka topics, MongoDB
collections, modules, dependencies and topology risks.

## Install

```bash
npx skills add elkouhen/ccc-radar-skill
uv tool install systemlens
```

In the Java/Spring repository to inspect:

```bash
systemlens init
systemlens doctor
systemlens index
systemlens analyze audit
```

No rule packs or external code-analysis tool are required.

## Contents

- [`skills/systemlens/SKILL.md`](skills/systemlens/SKILL.md) — architecture-first workflow.
- [`skills/systemlens/references/settings.md`](skills/systemlens/references/settings.md) —
  project configuration.
- [`skills/systemlens/references/management.md`](skills/systemlens/references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `systemlens mcp` from an initialized repository. For Codex:

```bash
codex mcp add systemlens -- systemlens mcp
```

## License

[Apache License 2.0](LICENSE).
