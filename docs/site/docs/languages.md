# Language Libraries

The mq-rest-admin project provides implementations in three languages.
Each library wraps the IBM MQ administrative REST API with the same
design — qualifier-based attribute mapping, method-per-command API,
and shared `mapping-data.json` — adapted to idiomatic conventions
for its language.

## Java

**Repository**: [mq-rest-admin-java](https://github.com/mq-rest-admin-project/mq-rest-admin-java)
| **Documentation**: [mq-rest-admin-project.github.io/mq-rest-admin-java](https://mq-rest-admin-project.github.io/mq-rest-admin-java/1.1/)

- Maven coordinates: `io.github.wphillipmoore:mq-rest-admin`
- Zero runtime dependencies beyond Gson
- `java.net.http.HttpClient` transport
- camelCase method names (`displayQueue()`, `defineQlocal()`)

## Python

**Repository**: [mq-rest-admin-python](https://github.com/mq-rest-admin-project/mq-rest-admin-python)
| **Documentation**: [mq-rest-admin-project.github.io/mq-rest-admin-python](https://mq-rest-admin-project.github.io/mq-rest-admin-python/1.1/)

- PyPI package: `pymqrest`
- `httpx` transport with async support
- snake_case method names (`display_queue()`, `define_qlocal()`)

## Go

**Repository**: [mq-rest-admin-go](https://github.com/mq-rest-admin-project/mq-rest-admin-go)
| **Documentation**: [mq-rest-admin-project.github.io/mq-rest-admin-go](https://mq-rest-admin-project.github.io/mq-rest-admin-go/1.1/)

- Zero external dependencies (Go standard library only)
- `context.Context` integration for all I/O methods
- PascalCase method names (`DisplayQueue()`, `DefineQlocal()`)
