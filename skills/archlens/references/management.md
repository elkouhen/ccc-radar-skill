# archlens management

`archlens` indexes Java/Spring architecture facts locally from source ASTs.

## Installation

```bash
uv tool install archlens
env -u SSL_CERT_FILE uvx --from huggingface_hub hf download \
  jinaai/jina-code-embeddings-1.5b \
  --local-dir ~/models/jina-code-embeddings-1.5b
```

The local embedding model is optional for architecture extraction.

## Project initialization

```bash
archlens init
archlens doctor
archlens index
```

If `.archlens/config.yml` already exists, keep it and run `archlens index`; do not
recreate it silently.

## MCP setup

The MCP server reads the project from its process working directory. Initialize
and index the project first, then start the coding agent from that repository.

```bash
codex mcp add archlens -- archlens mcp
codex mcp get archlens
```

For another MCP-compatible client:

```json
{"mcpServers": {"archlens": {"command": "archlens", "args": ["mcp"]}}}
```

## Refreshing the index

- After code changes: run `archlens index`.
- After a broad refactor or extractor upgrade: run `archlens index --full`.
- Use `archlens microservices`, `archlens topics`, `archlens apis`, `archlens mongodb`,
  `archlens modules` and `archlens analyze audit` for architecture questions.

## Troubleshooting

- `archlens` missing: install `archlens`.
- Missing configuration: run `archlens init` from the target repository.
- Absent index: run `archlens index`.
- Embedding-model issue: extraction still works; configure a valid local model
  only when an embedding feature is needed.
