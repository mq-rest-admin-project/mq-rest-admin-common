## Transparency statement

The mq-rest-admin project is an AI-assisted engineering effort. Significant
portions of each language implementation were developed collaboratively between
a human engineer and AI coding agents. This page documents what was
AI-generated, what was human-designed, and how quality was maintained.

### Tools used

- **Claude Code** (Anthropic) — interactive pair-programming agent
- **Codex** (OpenAI) — autonomous task execution agent

## What was AI-generated

**Mapping data bootstrap**: The canonical `mapping-data.json` was originally
bootstrapped from IBM MQ 9.4 documentation using an AI-assisted extraction
pipeline. The pipeline parsed MQSC and PCF command references to propose
`snake_case` attribute names, value mappings, and command metadata for all
48 qualifiers. The automated output was then reviewed, customized, and
rationalized by hand. `mapping-data.json` is now maintained directly as the
sole authoritative source — the extraction pipeline is archived in the Python
repository at `docs/archive/extraction/`.

**Command method generation**: The ~144 MQSC command wrapper methods in each
language implementation are generated from the command definitions in
`mapping-data.json`. Each method is a thin wrapper with the correct verb,
qualifier, and return type.

**Mapping documentation**: The 48 qualifier mapping pages in the documentation
were generated from the same mapping data source.

**Implementation code**: Much of the session, mapping, transport, and test
code was written through AI-assisted pair programming, with the AI agent
proposing implementations and the human engineer reviewing, refining, and
directing the design.

## What was human-designed

**Architecture and API design**: The overall design — single-endpoint via
`runCommandJSON`, qualifier-based mapping, method naming conventions,
return shape rules, error handling strategy — was conceived and directed
by the human engineer.

**Quality standards**: The requirement for comprehensive test coverage and
the validation gate pipeline were human-defined standards. Each language
implementation enforces its own quality tooling as a CI hard gate.

**Design decisions**: Key choices like the mapping opt-out mechanism,
strict vs lenient modes, the transport abstraction, and error handling
patterns were human decisions refined through discussion.

## Quality assurance

AI-generated code is held to the same standards as human-written code.
Every guardrail listed below is enforced as a CI hard gate — pull
requests that violate any check cannot merge.

### Test coverage

All production code is covered by unit tests with 100% line and branch
coverage enforced at the CI level. Coverage is measured per-build and
any drop below 100% fails the pipeline. Each language uses its
ecosystem's standard coverage tool (JaCoCo, pytest-cov, go tool cover).

### Static analysis and linting

Each implementation runs language-appropriate linters and static analysis
tools as CI hard gates:

- **Formatting**: Automatic formatting enforcement ensures consistent
  style regardless of author (human or AI). Formatting is checked, not
  just applied, so unformatted code fails CI.
- **Lint rules**: Comprehensive lint rulesets catch code smells, unused
  imports, naming violations, and anti-patterns.
- **Bug analysis**: Static analysis tools detect potential bugs, null
  pointer risks, concurrency issues, and security vulnerabilities.
- **Type checking**: Where applicable, strict type checking ensures type
  safety across the entire source tree.

### Security scanning

- **CodeQL**: GitHub's semantic code analysis runs on every pull request,
  detecting injection flaws, insecure data handling, and other
  vulnerability classes via deep data flow and taint tracking.
- **Semgrep**: Pattern-based SAST provides custom security rule coverage
  across all languages, complementing CodeQL with broader language
  support and project-specific rule authoring.
- **Trivy**: Filesystem vulnerability scanning detects known CVEs in
  dependencies, lock files, and configuration files across all
  language ecosystems.
- **Dependency auditing**: Every build audits runtime dependencies for
  known CVEs using language-native tools (pip-audit, govulncheck,
  Maven dependency verification). Vulnerable dependencies fail the
  pipeline.
- **License compliance**: Dependency licenses are checked against an
  allow-list of approved open-source licenses. Unapproved licenses
  fail the build.

### Validation pipeline

Each repository provides a single command that runs the same checks as
CI — formatting, linting, type checking, tests, coverage enforcement,
and static analysis. This ensures developers (human and AI) can verify
changes locally before pushing.

### Integration testing

Containerized IBM MQ queue managers provide real MQSC command validation.
Integration tests verify that mapping assumptions hold against actual MQ
responses, catching mismatches between the mapping data and real queue
manager behavior.

### Git hooks

Local git hooks enforce standards before code reaches CI:

- **Conventional Commits**: The `commit-msg` hook validates that every
  commit message follows the `type(scope): description` format.
- **Co-author validation**: The `commit-msg` hook checks that
  `Co-Authored-By` trailers match the repository's approved identity
  list — unapproved AI identities are rejected.
- **Branch protection**: The `pre-commit` hook blocks commits to
  protected branches (`main`, `develop`, `release/*`) and enforces
  the repository's branching model naming convention.

### CI gate structure

Pull requests must pass all of the following independent CI jobs before
merging:

1. **ci: standards-compliance** — Conventional Commit format, co-author
   validation, PR issue linkage, markdown standards, repository profile
2. **dependency-audit** — Vulnerability scanning and license compliance
3. **release-gates** — Version format and divergence checks for
   release-targeting PRs
4. **test-and-validate** — Full validation pipeline across multiple
   language/runtime versions
5. **codeql** — GitHub semantic security analysis
6. **trivy** — Filesystem vulnerability scanning across all ecosystems
7. **semgrep** — Pattern-based SAST with language-specific rulesets
8. **integration-tests** — End-to-end tests against containerized MQ

A **ci: docs-only** detection job allows documentation changes to skip
test-and-validate, CodeQL, Trivy, Semgrep, and integration tests while
still requiring standards compliance and dependency audit.

## The `Co-Authored-By` convention

Commits that involved AI assistance include a `Co-Authored-By` trailer
identifying the agent used:

```text
Co-Authored-By: wphillipmoore-agent <284101533+wphillipmoore-agent@users.noreply.github.com>
```

This provides a transparent, auditable record of AI involvement in the
project's git history.
