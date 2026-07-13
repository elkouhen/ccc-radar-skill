# ccc-findings-skill

Claude Code skill for `cccf` (CocoIndex Code + Semgrep findings): semantic
code search, index management, and Semgrep findings lookup, driven
automatically by the agent.

- [`skills/cccf/SKILL.md`](skills/cccf/SKILL.md) — the skill itself:
  ownership rules, searching, filtering, pagination.
- [`skills/cccf/references/settings.md`](skills/cccf/references/settings.md) —
  embedding model, include/exclude patterns, language overrides.
- [`skills/cccf/references/management.md`](skills/cccf/references/management.md) —
  installation, initialization, daemon management, troubleshooting.
- [`skills/cccf/rules/default/`](skills/cccf/rules/default/) — bundled
  Semgrep rule pack (Java) for bounded file streaming, Kafka
  claim-check/delivery guarantees, object storage as source of truth, and
  safe PDF/archive handling, run by default on `cccf init` (see
  **Default Rules** in `SKILL.md`).
- [`skills/cccf/rules/liveness/`](skills/cccf/rules/liveness/) — bundled
  Semgrep rule pack (Python + Java) for distributed-system blocking points:
  missing HTTP timeouts, blocking waits, synchronous REST calls inside
  Kafka consumers, network calls under a lock — also run by default on
  `cccf init` (see **Default Rules** in `SKILL.md`).

## Installation

```bash
npx skills add elkouhen/ccc-findings-skill
```

This installs the `cccf` skill (`skills/cccf/`) for your coding agent. It
still requires the `cccf` CLI itself — see
[Installation in `ccc-findings`](https://github.com/elkouhen/ccc-findings#installation).

## Related projects

- [`ccc-findings`](https://github.com/elkouhen/ccc-findings) (`cccf`) — the
  CLI and MCP server this skill drives. It indexes Semgrep findings locally
  and joins them to code search results from `ccc`.
- [`cocoindex-code`](https://github.com/cocoindex-io/cocoindex-code) (`ccc`)
  — the underlying AST-based semantic code search tool that `cccf` extends
  as a companion package (no fork, no internal import at the CLI/MCP level —
  see `ccc-findings`'s ADR-1).

## Provenance

This skill started as an adaptation of cocoindex-code's own
[`skills/ccc/`](https://github.com/cocoindex-io/cocoindex-code/tree/main/skills/ccc)
skill (Apache-2.0): `SKILL.md` is renamed to `cccf` and extended to cover
Semgrep findings, while `references/settings.md` and
`references/management.md` are carried over unmodified since they document
`ccc` itself, which `cccf` relies on unchanged. Each file links back to its
source. See [LICENSE](LICENSE) for the terms this carries over.

## License

[Apache License 2.0](LICENSE), matching the upstream
[`cocoindex-code`](https://github.com/cocoindex-io/cocoindex-code) project.
