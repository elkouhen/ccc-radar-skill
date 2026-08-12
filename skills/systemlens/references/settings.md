# systemlens settings

`systemlens init` creates the project configuration consumed by `systemlens index`:

```yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".systemlens/**"
min_severity: INFO
embedding_model: ~/models/jina-code-embeddings-1.5b
```

`include` and `exclude` define the source perimeter. Maven/Gradle test source
sets are excluded automatically. `embedding_model` is used only for local
embedding features; AST endpoint extraction is independent of it.

After changing project configuration, run `systemlens index`. There is no separate
user-level configuration for a source-analysis or code-search dependency.
