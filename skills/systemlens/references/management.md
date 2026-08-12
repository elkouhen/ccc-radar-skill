# systemlens management

`systemlens` indexes Java/Spring architecture facts locally from source ASTs.

## Installation

```bash
uv tool install systemlens
env -u SSL_CERT_FILE uvx --from huggingface_hub hf download \
  jinaai/jina-code-embeddings-1.5b \
  --local-dir ~/models/jina-code-embeddings-1.5b
```

The local embedding model is optional for architecture extraction.

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
  `systemlens modules` and `systemlens analyze audit` for architecture questions.

## Troubleshooting

- `systemlens` missing: install `systemlens`.
- Missing configuration: run `systemlens init` from the target repository.
- Absent index: run `systemlens index`.
- Embedding-model issue: extraction still works; configure a valid local model
  only when an embedding feature is needed.
