# mq-rest-admin-common

Shared documentation fragments and canonical mapping data for the
mq-rest-admin project family.

## What this repository provides

- **Documentation fragments** — language-neutral explanations of architecture,
  mapping pipeline, design decisions, and high-level patterns (ensure, sync)
  that are composed into per-language documentation sites.
- **Canonical mapping data** — the single source of truth for
  `mapping-data.json`, which defines the bidirectional attribute mappings
  between developer-friendly names and native MQSC parameters.
- **Qualifier mapping reference** — auto-generated documentation for all
  41 MQSC qualifier mappings.

## Documentation sections

- [Design](design/index.md) — design rationale, runCommandJSON endpoint,
  nested object flattening
- [Mappings](mappings/index.md) — per-qualifier attribute mapping reference
  (41 qualifiers)
- [AI Engineering](ai-engineering.md) — transparency statement and
  AI-assisted development practices
- [Development](development/local-mq-container.md) — local MQ container
  setup for integration testing

## Language implementations

- [mq-rest-admin-java](https://mq-rest-admin-project.github.io/mq-rest-admin-java/1.1/) — Java
- [mq-rest-admin-python](https://mq-rest-admin-project.github.io/mq-rest-admin-python/1.1/) — Python
- [mq-rest-admin-go](https://mq-rest-admin-project.github.io/mq-rest-admin-go/1.1/) — Go
- [mq-rest-admin-ruby](https://github.com/mq-rest-admin-project/mq-rest-admin-ruby) — Ruby (Work in Progress)
- [mq-rest-admin-rust](https://github.com/mq-rest-admin-project/mq-rest-admin-rust) — Rust (Work in Progress)
