# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Standards reference**: <https://github.com/wphillipmoore/standards-and-conventions>
— active standards documentation lives in the standard-tooling repository under `docs/`.
Repository profile: `standard-tooling.toml`.

## Memory management

Memory is allowed with human approval. The authoritative policy is in
the user's global `~/.claude/CLAUDE.md` — agents must propose memory
writes and suggest a destination (repo memory, global CLAUDE.md, or
plugin/skill issue) before writing. See that file for the full
workflow.

Available skills:
- `/standard-tooling:memory-init` — set up or update the policy header
  in a project's `MEMORY.md`.
- `/standard-tooling:memory-audit` — structured collaborative review
  of memory files.

## Parallel AI agent development

This repository supports running multiple Claude Code agents in parallel via
git worktrees. The convention keeps parallel agents' working trees isolated
while preserving shared project memory (which Claude Code derives from the
session's starting CWD).

**Canonical spec:**
[`standard-tooling/docs/specs/worktree-convention.md`](https://github.com/wphillipmoore/standard-tooling/blob/develop/docs/specs/worktree-convention.md)
— full rationale, trust model, failure modes, and memory-path implications.
The canonical text lives in `standard-tooling`; this section is the local
on-ramp.

### Structure

```text
~/dev/github/mq-rest-admin-common/          ← sessions ALWAYS start here
  .git/
  CLAUDE.md, docs/, …                       ← main worktree (usually `develop`)
  .worktrees/                               ← container for parallel worktrees
    issue-162-adopt-worktree-convention/    ← worktree on feature/162-...
    …
```

### Rules

1. **Sessions always start at the project root.**
   `cd ~/dev/github/mq-rest-admin-common && claude` — never from inside
   `.worktrees/<name>/`. This keeps the memory-path slug stable and shared.
2. **Each parallel agent is assigned exactly one worktree.** The session
   prompt names the worktree (see Agent prompt contract below).
   - For Read / Edit / Write tools: use the worktree's absolute path.
   - For Bash commands that touch files: `cd` into the worktree first,
     or use absolute paths.
3. **The main worktree is read-only.** All edits flow through a worktree
   on a feature branch — the logical endpoint of the standing
   "no direct commits to `develop`" policy.
4. **One worktree per issue.** Don't stack in-flight issues. When a
   branch lands, remove the worktree before starting the next.
5. **Naming: `issue-<N>-<short-slug>`.** `<N>` is the GitHub issue
   number; `<short-slug>` is 2–4 kebab-case tokens.

### Agent prompt contract

When launching a parallel-agent session, use this template (fill in the
placeholders):

```text
You are working on issue #<N>: <issue title>.

Your worktree is: /Users/pmoore/dev/github/mq-rest-admin-common/.worktrees/issue-<N>-<slug>/
Your branch is:   feature/<N>-<slug>

Rules for this session:
- Do all git operations from inside your worktree:
    cd <absolute-worktree-path> && git <command>
- For Read / Edit / Write tools, use the absolute worktree path.
- For Bash commands that touch files, cd into the worktree first
  or use absolute paths.
- Do not edit files at the project root. The main worktree is
  read-only — all changes flow through your worktree on your
  feature branch.
```

All fields are required.

## Project Overview

This is the shared common repository for the mq-rest-admin project family, serving two roles:

1. **Documentation fragments**: Language-neutral documentation fragments consumed by the per-language repos (Java, Python, Go) via their documentation toolchains
2. **Canonical mapping data**: Single source of truth for `mapping-data.json`, the MQ REST API attribute mapping definitions consumed by all language implementations

**Project name**: mq-rest-admin-common

**Status**: Active

**Canonical Standards**: This repository follows standards at https://github.com/wphillipmoore/standards-and-conventions (local path: `../standards-and-conventions` if available)

## Development Commands

This is a documentation-only repository. There are no build or test commands.

### Environment Setup

```bash
git config core.hooksPath ../standard-tooling/scripts/lib/git-hooks  # Enable git hooks
```

Standard-tooling CLI tools (`st-commit`, `st-validate`, etc.) are
pre-installed in the dev container images. No local setup required.

### Validation

```bash
st-docker-run -- st-validate   # Full validation (runs in dev container)
```

## Architecture

### Fragment Organization

```text
fragments/
  architecture/           # Core architecture concepts
  mapping-pipeline/       # Attribute mapping pipeline
  design/                 # Design decisions and rationale
  concepts/               # High-level patterns (ensure, sync)
```

Fragments contain language-neutral concept explanations, diagrams, and tables.
They do NOT contain language-specific code examples, documentation tool syntax,
or references to specific library names.

### mapping-data.json

The root-level `mapping-data.json` is the canonical source for MQ REST API
attribute mapping definitions. It contains:

- **commands**: MQSC command → qualifier mapping (e.g., `"ALTER CHANNEL"` → `"channel"`)
- **qualifiers**: Per-qualifier mapping tables with four map types:
  - `request_key_map`: Friendly name → MQSC parameter (outbound)
  - `request_value_map`: Friendly value → MQSC value (outbound)
  - `response_key_map`: MQSC parameter → friendly name (inbound)
  - `response_value_map`: MQSC value → friendly value (inbound)
  - `request_key_value_map`: Composite key-value transforms (e.g., replace/noreplace)
  - `response_parameter_macros`: Parameter groups expanded at runtime
- **version**: Schema version (currently `1`)

### How Consuming Repos Integrate

- **Java**: Copies `mapping-data.json` into `src/main/resources/` (via CI or manual sync)
- **Python**: References via local clone or CI checkout
- **Go**: Uses `go:embed` or symlink from local clone
- **Documentation sites**: Each language repo clones this repository and uses its
  documentation tool's include mechanism (`pymdownx.snippets`, MyST `{include}`)

## Local MQ Environment

Each language repo (Python, Java, Go, Ruby, Rust) includes thin wrapper
scripts for managing a local MQ container environment. The actual Docker
Compose configuration is owned by the
[mq-rest-admin-dev-environment](https://github.com/wphillipmoore/mq-rest-admin-dev-environment)
repository, which must be cloned as a sibling directory.

### Prerequisite

```bash
git clone https://github.com/wphillipmoore/mq-rest-admin-dev-environment.git ../mq-rest-admin-dev-environment
```

### Lifecycle scripts (in each language repo)

```bash
./scripts/dev/mq_start.sh    # Start containerized MQ queue managers
./scripts/dev/mq_seed.sh     # Seed deterministic test objects (DEV.* prefix)
./scripts/dev/mq_verify.sh   # Verify REST-based MQSC responses
./scripts/dev/mq_stop.sh     # Stop the queue managers
./scripts/dev/mq_reset.sh    # Reset to clean state (removes data volumes)
```

Each language repo sets its own `COMPOSE_PROJECT_NAME` and unique port
assignments to avoid collisions when running multiple environments
simultaneously. Override the dev-environment path with `MQ_DEV_ENV_PATH`.

### Integration test gate

Integration tests are gated by the `MQ_REST_ADMIN_RUN_INTEGRATION`
environment variable. When unset, integration tests are skipped. CI sets
this automatically; for local runs, start MQ first, then export the variable:

```bash
./scripts/dev/mq_start.sh
./scripts/dev/mq_seed.sh
export MQ_REST_ADMIN_RUN_INTEGRATION=true
# Run integration tests (language-specific command)
```

### Container details

- Queue managers: `QM1` and `QM2`
- Admin credentials: `mqadmin` / `mqadmin`
- Read-only credentials: `mqreader` / `mqreader`
- Object prefix: `DEV.*`
- Ports are language-specific (see each repo's `scripts/dev/mq_start.sh`)

## Key References

**Sibling repositories**:
- `../mq-rest-admin-python` — Python implementation
- `../mq-rest-admin-java` — Java implementation
- `../mq-rest-admin-go` — Go implementation

**External Documentation**:
- IBM MQ 9.4 administrative REST API
- MQSC command reference
