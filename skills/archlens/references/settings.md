# archlens settings

`archlens init` creates the project configuration consumed by `archlens index`:

```yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".archlens/**"
min_severity: INFO
embedding_model: ~/models/jina-code-embeddings-1.5b
```

`include` and `exclude` define the source perimeter. Maven/Gradle test source
sets are excluded automatically. `embedding_model` is used only for local
embedding features; AST endpoint extraction is independent of it.

After changing project configuration, run `archlens index`. There is no separate
user-level configuration for a source-analysis or code-search dependency.
