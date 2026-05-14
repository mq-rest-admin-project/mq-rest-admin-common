# MQ REST Admin Migration Design

**Date:** 2026-05-14
**Status:** Draft

## Problem

The mq-rest-admin repository family (7 repos) needs to move from
`wphillipmoore` to `mq-rest-admin-project`. The repos also need to
adopt the post-VERGIL tooling names (`vergil.toml`, `vergil-actions`,
`vrg-commit`, etc.), which they have not yet picked up. Both changes
are combined into a single migration pass per repo.

## Prerequisites

1. **VERGIL rename complete.** All references target post-VERGIL names:
   `vergil-project/vergil-actions`, `vergil.toml`, `vrg-commit`, etc.

2. **`mq-rest-admin-project` org governance complete.** The org exists
   with security settings, credentials, and GitHub App configured per
   `docs/specs/2026-05-14-mq-rest-admin-org-governance-design.md`.

## Scope

This is a multi-repo migration — 7 repositories transferred to the
same org in a single batch, then updated repo-by-repo. Unlike the
Diogenes migration (single repo with plugin namespace and directory
restructuring), these repos have no plugin changes, no renames, and no
directory restructuring. The changes are limited to org transfer and
tooling reference updates.

### Repositories in scope

| Repository | Primary language | Notes |
|---|---|---|
| `mq-rest-admin-common` | None (docs/data) | Hub repo, mapping data, doc fragments |
| `mq-rest-admin-python` | Python | Language implementation |
| `mq-rest-admin-java` | Java | Language implementation |
| `mq-rest-admin-go` | Go | Language implementation |
| `mq-rest-admin-ruby` | Ruby | Language implementation |
| `mq-rest-admin-rust` | Rust | Language implementation |
| `mq-rest-admin-dev-environment` | Shell | Docker Compose, reusable GH Action |

### Excluded

| Repository | Reason |
|---|---|
| `mq-rest-admin-template` | Archived, no longer active |

### What changes (all repos, uniform)

| Category | Before | After |
|---|---|---|
| GitHub location | `wphillipmoore/mq-rest-admin-*` | `mq-rest-admin-project/mq-rest-admin-*` |
| Config file | `standard-tooling.toml` | `vergil.toml` |
| Tooling dependency | `standard-tooling = "v1.4"` | `vergil = "v2.0"` |
| Co-authors | `claude`, `codex` (per-harness) | `agent` (single entry) |
| CI workflows | `wphillipmoore/standard-actions@v1.5` | `vergil-project/vergil-actions@v2.0` |
| Issue templates | `wphillipmoore/standard-tooling` source comments | `vergil-project/vergil-tooling` |
| CLAUDE.md | `wphillipmoore`, `standard-tooling`, `st-*` | `vergil-project`, `vergil-tooling`, `vrg-*` |
| Standards reference | `wphillipmoore/standards-and-conventions` | `vergil-project/vergil-tooling` (active docs) |

### What changes (repo-specific)

| Repo | Additional changes |
|---|---|
| **common** | None beyond the uniform set |
| **python** | `pyproject.toml` URLs (Homepage, Repository, Issues) |
| **java** | Build config URLs (pom.xml or Gradle SCM/URL sections) |
| **go** | `go.mod` module path if it references the GitHub org |
| **ruby** | Gemspec URLs (homepage, source_code_uri, metadata) |
| **rust** | `Cargo.toml` URLs (repository, homepage) |
| **dev-environment** | Reusable action internal references; language repos' CI references to this action |

### What does NOT change

- Repository names (all keep `mq-rest-admin-*`)
- Branch names (`develop`, `main`)
- Internal code logic, tests, mapping data
- License
- Directory structure (no plugin restructuring)
- Local sibling path references (`../mq-rest-admin-python`, etc.)

## Section 1: Batch Transfer

All 7 repos are transferred to `mq-rest-admin-project` in a single
batch via the GitHub API. GitHub creates automatic redirects from the
old URLs, so CI and cross-references continue working during the
migration window.

**Pre-flight checks:**
- No open PRs on any of the 7 repos
- Clean working state on `develop` for all repos
- All CI passing

**Post-transfer:**
- Update local git remotes for all 7 repos
- Verify all transfers succeeded via GitHub API

## Section 2: Per-Repo Migration Template

Each repo follows the same task sequence. This is the canonical
template — repo-specific additions are documented in Section 3.

1. **Pre-flight checks** — clean state, on develop, up to date
2. **Create feature branch** — `feature/<issue-number>-org-migration`
3. **Rename config file** — `git mv standard-tooling.toml vergil.toml`
4. **Update vergil.toml contents** — dependency `vergil = "v2.0"`,
   co-author consolidated to single `agent` entry
5. **Update CI/CD workflows** — all `uses:` references from
   `wphillipmoore/standard-actions@v1.5` to
   `vergil-project/vergil-actions@v2.0`; also update any references
   to `wphillipmoore/mq-rest-admin-dev-environment/` to
   `mq-rest-admin-project/mq-rest-admin-dev-environment/`
6. **Update issue templates** — source comment references from
   `wphillipmoore/standard-tooling` to `vergil-project/vergil-tooling`
7. **Update CLAUDE.md** — org references, tooling references, command
   references (`st-*` to `vrg-*`), standards URL, skills references
8. **Update repo-specific files** — per Section 3 delta
9. **Final reference sweep** — grep for any remaining `wphillipmoore`,
   `standard-tooling`, `standard-actions`, `st-commit`, `st-validate`,
   `st-docker`, `ST_COMMIT` references
10. **Validate** — repo-specific validation command if available
11. **Push and open PR** — targeting `develop`

### Commit strategy

Each repo's migration is a single feature branch with logically
grouped commits:

- Config rename and update (vergil.toml)
- CI/CD workflow updates
- Documentation updates (CLAUDE.md, issue templates)
- Repo-specific file updates (if any)
- Any fixes from validation

## Section 3: Per-Repo Deltas

### mq-rest-admin-common

No additional changes beyond the template. This is a docs-and-data
repo with no language-specific metadata.

### mq-rest-admin-python

**Additional files:**
- `pyproject.toml`: Update `[project.urls]` section — Homepage,
  Repository, Issues URLs to
  `https://github.com/mq-rest-admin-project/mq-rest-admin-python`
- Grep `src/` for any `wphillipmoore` references in Python source

### mq-rest-admin-java

**Additional files:**
- Build configuration (Maven `pom.xml` or Gradle): Update SCM URLs,
  project URL, issue management URL to
  `https://github.com/mq-rest-admin-project/mq-rest-admin-java`
- Grep `src/` for any `wphillipmoore` references in Java source

### mq-rest-admin-go

**Additional files:**
- `go.mod`: If the module path uses
  `github.com/wphillipmoore/mq-rest-admin-go`, update to
  `github.com/mq-rest-admin-project/mq-rest-admin-go`
- Grep `*.go` files for any `wphillipmoore` import paths or references

**Note:** A Go module path change is technically a breaking change for
external consumers. There are no known external consumers of this
module.

### mq-rest-admin-ruby

**Additional files:**
- Gemspec: Update `homepage`, `source_code_uri`, and metadata URLs to
  `https://github.com/mq-rest-admin-project/mq-rest-admin-ruby`
- Grep `lib/` for any `wphillipmoore` references in Ruby source

### mq-rest-admin-rust

**Additional files:**
- `Cargo.toml`: Update `repository` and `homepage` URLs to
  `https://github.com/mq-rest-admin-project/mq-rest-admin-rust`
- Grep `src/` for any `wphillipmoore` references in Rust source

### mq-rest-admin-dev-environment

**Additional files:**
- `.github/actions/setup-mq/action.yml`: Check for any internal
  references to `wphillipmoore` org
- Note: language repos reference this action as
  `wphillipmoore/mq-rest-admin-dev-environment/.github/actions/setup-mq@main`
  — those references are updated as part of each language repo's CI
  workflow sweep (the template step covers this)

## Section 4: Post-Migration

After all 7 repos are migrated:

1. **Org governance activation** — Plan A Phase 2: invite agent
   account, configure org-level rulesets, verify governance model
2. **Create deferred work issues** — `.github` profile repo,
   cross-human review CI check, credential audit tooling, merge queue
3. **Cross-repo reference sweep** — search all repos in `wphillipmoore`
   and other orgs for stale references to the old locations
4. **Local cleanup** — update local directory names if desired, update
   Claude Code memory paths

## Risks

| Risk | Mitigation |
|---|---|
| CI breaks during migration window | GitHub redirects cover the gap; sweep all repos promptly |
| Go module path change breaks external consumers | No known external consumers; document in release notes |
| Language repos reference dev-environment action at old path | Redirects cover it; each repo's workflow sweep updates the reference |
| Credential proliferation (3 more Keychain entries) | Manageable at current scale; credential audit tooling in backlog |
| 7 repos means 7 PRs to review and merge | Template-driven changes are mechanical and parallelizable |
| Org-level misconfiguration affects all 7 repos | Verify governance settings before any repo migration begins |
| Stale cross-references in repos outside this project | Post-migration sweep covers this |

## File Map (per-repo template)

### Renamed
- `standard-tooling.toml` -> `vergil.toml`

### Modified (all repos)
- `vergil.toml` (post-rename contents)
- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`
- `.github/ISSUE_TEMPLATE/config.yml`
- `.github/ISSUE_TEMPLATE/issue.yml`
- `CLAUDE.md`

### Modified (repo-specific, where applicable)
- `pyproject.toml` (Python)
- `pom.xml` or Gradle config (Java)
- `go.mod` (Go)
- Gemspec (Ruby)
- `Cargo.toml` (Rust)
- `.github/actions/setup-mq/action.yml` (dev-environment)

### Not modified
- Source code (no org references in application logic)
- Tests
- `mapping-data.json` (common)
- `CHANGELOG.md` (historical record)
- License files
