# cccr settings

`cccr init` creates the project configuration consumed by `cccr index`:

```yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".cccr/**"
min_severity: INFO
embedding_model: ~/models/jina-code-embeddings-1.5b
```

`include` and `exclude` define the source perimeter. Maven/Gradle test source
sets are excluded automatically. `embedding_model` is used only for local
embedding features; AST endpoint extraction is independent of it.

After changing project configuration, run `cccr index`. There is no separate
user-level configuration for a source-analysis or code-search dependency.
