# systemlens management

`systemlens` indexes Java/Spring architecture facts locally from source ASTs.

## Installation

```bash
uv tool install systemlens
systemlens version
```

Architecture extraction is local and does not require a model download, rule
pack or external code-analysis service.

## Project initialization

```bash
systemlens init
systemlens doctor
systemlens index
```

If `.systemlens/config.yml` already exists, keep it and run `systemlens index`; do not
recreate it silently.

## MCP setup

The MCP server reads the project from its process working directory. Initialize
and index the project first, then start the coding agent from that repository.

```bash
codex mcp add systemlens -- systemlens mcp
codex mcp get systemlens
```

For another MCP-compatible client:

```json
{"mcpServers": {"systemlens": {"command": "systemlens", "args": ["mcp"]}}}
```

## Refreshing the index

- After code changes: run `systemlens index`.
- After a broad refactor or extractor upgrade: run `systemlens index --full`.
- Use `systemlens microservices`, `systemlens topics`, `systemlens apis`, `systemlens mongodb`,
  `systemlens dtos`, `systemlens modules`, `systemlens analyze coverage`,
  `systemlens analyze indexing-issues` and `systemlens analyze audit` for architecture questions.
- Use `systemlens index --topic-strategy strategy1` only for repositories that
  follow the documented Strategy1 Kafka and REST conventions.

## Troubleshooting

- `systemlens` missing: install `systemlens`.
- Missing configuration: run `systemlens init` from the target repository.
- Absent index: run `systemlens index`.
- Unresolved facts: run `systemlens analyze indexing-issues --json` and inspect
  the recorded source evidence; do not replace them with guessed dependencies.
