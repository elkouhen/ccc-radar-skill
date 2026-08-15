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
```

`include` and `exclude` define the source perimeter. Maven/Gradle test source
sets are excluded automatically. `min_severity` remains accepted for index
compatibility but does not change AST endpoint extraction.

After changing project configuration, run `systemlens index`. Explicit Kafka
manifests can be indexed with `systemlens index --manifest FILE`; they must be
Markdown or JSON files inside the indexed repository.
