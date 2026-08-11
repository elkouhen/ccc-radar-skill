# cccr management

`cccr` indexes Java/Spring architecture facts locally from source ASTs.

## Installation

```bash
uv tool install ccc-radar
env -u SSL_CERT_FILE uvx --from huggingface_hub hf download \
  jinaai/jina-code-embeddings-1.5b \
  --local-dir ~/models/jina-code-embeddings-1.5b
```

The local embedding model is optional for architecture extraction.

## Project initialization

```bash
cccr init
cccr doctor
cccr index
```

If `.cccr/config.yml` already exists, keep it and run `cccr index`; do not
recreate it silently.

## MCP setup

The MCP server reads the project from its process working directory. Initialize
and index the project first, then start the coding agent from that repository.

```bash
codex mcp add cccr -- cccr mcp
codex mcp get cccr
```

For another MCP-compatible client:

```json
{"mcpServers": {"cccr": {"command": "cccr", "args": ["mcp"]}}}
```

## Refreshing the index

- After code changes: run `cccr index`.
- After a broad refactor or extractor upgrade: run `cccr index --full`.
- Use `cccr microservices`, `cccr topics`, `cccr apis`, `cccr mongodb`,
  `cccr modules` and `cccr analyze audit` for architecture questions.

## Troubleshooting

- `cccr` missing: install `ccc-radar`.
- Missing configuration: run `cccr init` from the target repository.
- Absent index: run `cccr index`.
- Embedding-model issue: extraction still works; configure a valid local model
  only when an embedding feature is needed.
