## Overview

Every mq-rest-admin repository enforces a layered quality gate system.
Local git hooks catch issues before code leaves the developer's machine,
and CI pipelines enforce the same standards (plus additional checks) on
every pull request. Pull requests cannot merge until all required jobs
pass.

## Claude Code hook guard

The `.claude/hooks/guard.sh` PreToolUse hook blocks raw `git` and
`gh` commands in AI agent sessions — all operations must go through
the `vrg-git` / `vrg-gh` wrappers.

### pre-commit

The `pre-commit` hook runs before each commit and enforces:

**Protected branch blocking** — Commits directly to `main`, `develop`,
`release/*`, or a detached HEAD are rejected. All work must go through
short-lived feature branches and pull requests.

**Branch naming enforcement** — The hook reads the `branching_model`
attribute from `docs/repository-standards.md` and enforces the
corresponding naming convention:

| Branching model | Allowed prefixes |
|---|---|
| `docs-single-branch` | `feature/*`, `bugfix/*` |
| `library-release` | `feature/*`, `bugfix/*`, `hotfix/*` |
| `application-promotion` | `feature/*`, `bugfix/*`, `hotfix/*`, `promotion/*` |

### commit-msg

The `commit-msg` hook runs after the commit message is written and
validates two things:

**Conventional Commits format** — The subject line must match:

```
<type>(optional-scope): <description>
```

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

**Co-author identity validation** — If the commit includes
`Co-Authored-By` trailers, each trailer is checked against the approved
identity list in `docs/repository-standards.md`. Unapproved identities
are rejected. Commits without co-author trailers (human-only commits)
are always valid.

## CI pipeline

The CI pipeline (`ci.yml`) runs on every pull request and on pushes to
`develop` and `release/**`. It consists of several independent jobs, all
of which must pass for a PR to merge.

### ci: docs-only detection

A preliminary job detects whether the changeset is docs-only (as defined
by the repository). When docs-only is detected, the test-intensive jobs
(test-and-validate, CodeQL, Trivy, Semgrep, integration tests) are
skipped, while standards-compliance and dependency-audit still run.

### ci: standards-compliance

Validates repository standards using a shared composite action:

- **Conventional Commits** — All commits in the PR are validated against
  the `type(scope): description` format.
- **Co-author trailers** — All `Co-Authored-By` trailers are checked
  against the approved list.
- **PR issue linkage** — The PR body must include `Fixes #N` or `Ref #N`
  linking to a tracking issue. Other GitHub keywords (`Closes`,
  `Resolves`) are rejected.
- **Markdown standards** — Documentation files are checked for exactly
  one H1, no heading level skips, presence of a `## Table of Contents`
  section, and markdownlint compliance.
- **Repository profile** — The `docs/repository-standards.md` file is
  validated for required attributes: `repository_type`,
  `versioning_scheme`, `branching_model`, `release_model`, and
  `supported_release_lines`.

### dependency-audit

Audits runtime dependencies for security vulnerabilities and license
compliance:

| Language | Vulnerability scanner | License checker |
|---|---|---|
| Java | `mvn dependency:tree` (resolution verification) | `license-maven-plugin` with allow-list |
| Python | `pip-audit` against requirements files | `pip-licenses` with allow-list |
| Go | `govulncheck ./...` | `go-licenses check` with allow-list |

Allowed licenses: Apache-2.0, MIT, BSD-2-Clause, BSD-3-Clause, ISC,
MPL-2.0, GPL-3.0-or-later, and PSF-2.0.

### release-gates

Validates version metadata on PRs targeting protected branches:

**PRs targeting main** (release PRs):

- Version must be plain semver (`x.y.z`) with no pre-release suffix
- Version must not already exist on the package registry
- Changelog must include an entry for the release version

**PRs targeting develop**:

- Version must differ from the version on `main`, preventing accidental
  version collisions

### test-and-validate

Runs the full validation pipeline across a matrix of language/runtime
versions:

| Language | Versions tested | Validation command |
|---|---|---|
| Java | 17, 21, 25-ea | `./mvnw verify` |
| Python | 3.12, 3.13, 3.14 | ruff + mypy + ty + pytest |
| Go | 1.25, 1.26 | go vet + golangci-lint + go test |

The validation pipeline includes (in order):

1. **Formatting check** — Unformatted code fails the build
2. **Lint and style checks** — Code smell and style rule enforcement
3. **Type checking** — Strict type analysis (where applicable)
4. **Unit tests** — Full test suite execution
5. **Coverage enforcement** — 100% line and branch coverage required
6. **Static analysis** — Bug pattern and vulnerability detection

### CodeQL

GitHub's semantic code analysis runs on every non-docs PR. CodeQL
performs deep data flow and taint tracking analysis to detect injection
flaws, insecure data handling, and other vulnerability classes specific
to each language.

### Trivy

Aqua Security's Trivy runs a filesystem vulnerability scan on every
non-docs PR. It detects known CVEs in dependencies, lock files, and
configuration files across all language ecosystems, complementing the
language-specific scanners in the dependency-audit job with broader
cross-ecosystem coverage.

### Semgrep

Semgrep runs pattern-based Static Application Security Testing (SAST)
on every non-docs PR. It complements CodeQL by providing broader
language coverage and support for custom security rules. Each language
repo configures Semgrep with language-specific rulesets.

### integration-tests

End-to-end tests run against containerized IBM MQ queue managers
provisioned by the `mq-rest-admin-dev-environment` repository. These tests issue
real MQSC commands through the REST API and verify that:

- Mapping data correctly translates between friendly names and MQ
  attribute names
- Command methods produce valid REST API requests
- Response parsing handles real MQ response structures

## Documentation deployment

A separate `docs.yml` workflow deploys documentation on pushes to `main`
and `develop` using mike for versioned deployment. Pushes to `develop`
deploy the `dev` version; pushes to `main` deploy a numbered version
(read from `VERSION` or the package metadata) with a `latest` alias.
