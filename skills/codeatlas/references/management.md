# codeatlas management

`codeatlas` indexes Java/Spring architecture facts locally from source ASTs.

## Installation

```bash
uv tool install codeatlas
env -u SSL_CERT_FILE uvx --from huggingface_hub hf download \
  jinaai/jina-code-embeddings-1.5b \
  --local-dir ~/models/jina-code-embeddings-1.5b
```

The local embedding model is optional for architecture extraction.

## Project initialization

```bash
codeatlas init
codeatlas doctor
codeatlas index
```

If `.codeatlas/config.yml` already exists, keep it and run `codeatlas index`; do not
recreate it silently.

## MCP setup

The MCP server reads the project from its process working directory. Initialize
and index the project first, then start the coding agent from that repository.

```bash
codex mcp add codeatlas -- codeatlas mcp
codex mcp get codeatlas
```

For another MCP-compatible client:

```json
{"mcpServers": {"codeatlas": {"command": "codeatlas", "args": ["mcp"]}}}
```

## Refreshing the index

- After code changes: run `codeatlas index`.
- After a broad refactor or extractor upgrade: run `codeatlas index --full`.
- Use `codeatlas microservices`, `codeatlas topics`, `codeatlas apis`, `codeatlas mongodb`,
  `codeatlas modules` and `codeatlas analyze audit` for architecture questions.

## Troubleshooting

- `codeatlas` missing: install `codeatlas`.
- Missing configuration: run `codeatlas init` from the target repository.
- Absent index: run `codeatlas index`.
- Embedding-model issue: extraction still works; configure a valid local model
  only when an embedding feature is needed.
