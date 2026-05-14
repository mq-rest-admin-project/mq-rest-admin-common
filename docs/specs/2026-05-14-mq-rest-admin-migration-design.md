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

2. **`mq-rest-admin-project` org governance Phase 1 complete.** The org
   exists with security settings configured and GitHub App registered
   per governance spec Section 5. Phase 2 (credentials, rulesets,
   agent invitation) runs after the batch transfer.

## Scope

This is a multi-repo migration — 8 repositories transferred to the
same org in sequence, then 7 active repos updated repo-by-repo (the
archived `mq-rest-admin-template` is transferred but not migrated).
Unlike the
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
| `mq-rest-admin-template` | None | Archived — transfer only, no per-repo migration |

### What changes (all repos, uniform)

| Category | Before | After |
|---|---|---|
| GitHub location | `wphillipmoore/mq-rest-admin-*` | `mq-rest-admin-project/mq-rest-admin-*` |
| Config file | `standard-tooling.toml` | `vergil.toml` |
| Tooling dependency | `standard-tooling = "v1.4"` | `vergil = "v2.0"` |
| Co-authors | `claude`, `codex` (per-harness) | `agent` (single entry) |
| CI workflows | `wphillipmoore/standard-actions@v1.5` | `vergil-project/vergil-actions@v2.0` |
| CD secrets | `secrets: inherit` | Explicit `secrets:` block (`APP_CLIENT_ID`, `APP_PRIVATE_KEY`) — `secrets: inherit` does not work cross-org |
| Issue templates | `wphillipmoore/standard-tooling` source comments | `vergil-project/vergil-tooling` |
| CLAUDE.md | `wphillipmoore`, `standard-tooling`, `st-*` | `vergil-project`, `vergil-tooling`, `vrg-*` |
| Standards reference | `wphillipmoore/standards-and-conventions` | `vergil-project/vergil-tooling` (active docs) |

### What changes (repo-specific)

| Repo | Additional changes |
|---|---|
| **common** | None beyond the uniform set |
| **python** | `pyproject.toml` URLs (Homepage, Repository, Issues) |
| **java** | `pom.xml` URLs (SCM, project URL, issue management) |
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

## Section 1: Transfer Sequence

Repos are transferred one at a time via the GitHub API
(`POST /repos/{owner}/{repo}/transfer`). There is no atomic batch
transfer — each call operates independently.

### Transfer order

Transfer repos in dependency order so that CI-referenced repos move
first:

1. `mq-rest-admin-dev-environment` (referenced by language repo CI)
2. `mq-rest-admin-common` (referenced by language repos for data/docs)
3. `mq-rest-admin-python`
4. `mq-rest-admin-java`
5. `mq-rest-admin-go`
6. `mq-rest-admin-ruby`
7. `mq-rest-admin-rust`
8. `mq-rest-admin-template` (archived — transfer only, no further work)

### Pre-flight checks

- No open PRs on any of the 8 repos
- Clean working state on `develop` for all active repos
- All CI passing
- Governance Phase 1 complete (org exists, security settings
  configured, GitHub App registered — see governance spec Section 5)

### Transfer procedure

For each repo in order:

1. Transfer via GitHub API
2. Verify the transfer succeeded:
   `gh api repos/mq-rest-admin-project/{repo} --jq .full_name`
3. Update local git remote:
   `git remote set-url origin git@github.com:mq-rest-admin-project/{repo}.git`
4. Proceed to the next repo

If a transfer fails, stop. Do not continue transferring remaining
repos. Diagnose the failure, resolve it, then resume from the failed
repo.

### Partial failure recovery

If transfers partially complete (some repos moved, some not):

- **Push forward** (preferred): Fix the failure cause and resume
  transfers from where they stopped. GitHub redirects cover
  cross-references from already-transferred repos to
  not-yet-transferred repos.
- **Roll back**: Transfer already-moved repos back to `wphillipmoore`
  via the same API. Reset local remotes. This is safe because no
  per-repo migration branches have been pushed yet at this stage.

### Post-transfer

- Verify all 8 transfers succeeded via GitHub API
- Run governance Phase 2 (credentials, rulesets, agent invitation —
  see governance spec Section 5)

## Section 2: Per-Repo Migration Template

Each repo follows the same task sequence. This is the canonical
template — repo-specific additions are documented in Section 3.

**Note:** In a multi-developer environment, each repo's migration
should be executed as a tight sequence immediately after transfer to
prevent CI from triggering against stale workflow references. In a
single-developer environment where repos are frozen during migration,
the transfer and per-repo migration phases can be batched separately.

1. **Pre-flight checks** — clean state, on develop, up to date
2. **Create feature branch** — `feature/<issue-number>-org-migration`
3. **Rename config file** — `git mv standard-tooling.toml vergil.toml`
4. **Update vergil.toml contents** — dependency `vergil = "v2.0"`,
   co-author consolidated to single `agent` entry
5. **Update CI/CD workflows** — all `uses:` references from
   `wphillipmoore/standard-actions@v1.5` to
   `vergil-project/vergil-actions@v2.0`; update any references
   to `wphillipmoore/mq-rest-admin-dev-environment/` to
   `mq-rest-admin-project/mq-rest-admin-dev-environment/`; replace
   `secrets: inherit` in CD workflows with explicit `secrets:` block
   passing `APP_CLIENT_ID` and `APP_PRIVATE_KEY` (required for
   cross-org reusable workflow calls)
6. **Update issue templates** — source comment references from
   `wphillipmoore/standard-tooling` to `vergil-project/vergil-tooling`
7. **Update CLAUDE.md** — see CLAUDE.md substitution table below
8. **Update repo-specific files** — per Section 3 delta
9. **Final reference sweep** — grep for any remaining `wphillipmoore`,
   `standard-tooling`, `standard-actions`, `st-commit`, `st-validate`,
   `st-docker`, `ST_COMMIT`, `standard-tooling:`, `secrets: inherit`
   references
10. **Validate** — repo-specific validation command if available
11. **Push and open PR** — targeting `develop`

### CLAUDE.md substitution table

Step 7 requires the following concrete substitutions:

| Before | After | Location in CLAUDE.md |
|---|---|---|
| `https://github.com/wphillipmoore/standards-and-conventions` | `https://github.com/vergil-project/vergil-tooling` | Standards reference header |
| `standard-tooling.toml` | `vergil.toml` | Repository profile reference |
| `/standard-tooling:memory-init` | `/vergil:memory-init` | Memory management skills |
| `/standard-tooling:memory-audit` | `/vergil:memory-audit` | Memory management skills |
| `standard-tooling/docs/specs/worktree-convention.md` | `vergil-tooling/docs/specs/worktree-convention.md` | Worktree convention link |
| `https://github.com/wphillipmoore/standard-tooling/blob/develop/` | `https://github.com/vergil-project/vergil-tooling/blob/develop/` | Worktree convention URL |
| `The canonical text lives in \`standard-tooling\`` | `The canonical text lives in \`vergil-tooling\`` | Worktree section prose |
| `../standard-tooling/scripts/lib/git-hooks` | `../vergil-tooling/scripts/lib/git-hooks` | Git hooks path |
| `Standard-tooling CLI tools (\`st-commit\`, \`st-validate\`, etc.)` | `VERGIL CLI tools (\`vrg-commit\`, \`vrg-validate\`, etc.)` | Environment setup |
| `st-docker-run -- st-validate` | `vrg-docker-run -- vrg-validate` | Validation command |
| `https://github.com/wphillipmoore/mq-rest-admin-dev-environment` | `https://github.com/mq-rest-admin-project/mq-rest-admin-dev-environment` | Dev environment clone URL |
| `wphillipmoore/standards-and-conventions` | `vergil-project/vergil-tooling` | Canonical standards reference |

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
- `pom.xml` (Maven): Update SCM URLs, project URL, issue management
  URL to `https://github.com/mq-rest-admin-project/mq-rest-admin-java`
- Grep `src/` for any `wphillipmoore` references in Java source
- See Section 3.1 for Maven coordinate and publication implications

### mq-rest-admin-go

**Additional files:**
- `go.mod`: If the module path uses
  `github.com/wphillipmoore/mq-rest-admin-go`, update to
  `github.com/mq-rest-admin-project/mq-rest-admin-go`
- Grep `*.go` files for any `wphillipmoore` import paths or references
- See Section 3.1 for Go module path and publication implications

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

### Section 3.1: Module Identity Changes (Go and Java)

Go and Java both expose the GitHub org path in their module/package
identity, unlike Python (where PyPI is the namespace) or Ruby/Rust
(where the gem/crate name is independent of the source location).
The org transfer changes how the community discovers and imports
these libraries.

There are no known external consumers of either module today. This
makes the migration low-risk but the first post-migration publish
must handle the identity change cleanly.

#### Go

- **Module path change:** `github.com/wphillipmoore/mq-rest-admin-go`
  becomes `github.com/mq-rest-admin-project/mq-rest-admin-go`
- **Internal imports:** All `*.go` files with cross-package imports
  using the old module path must be updated
- **Go module proxy:** `proxy.golang.org` caches the old module path
  permanently — it does not follow redirects. The old path will
  continue to resolve to the last version published under it.
- **Retraction:** Add a `retract` directive in the old module's
  `go.mod` (before transfer) for all published versions, directing
  users to the new path. If no versions have been published to the
  proxy, this step can be skipped.
- **Verification:** `go mod tidy` must succeed after all path updates

#### Java (Maven)

- **Maven coordinates:** If the `groupId` or artifact metadata in
  `pom.xml` references `wphillipmoore`, update to reflect the new org
- **Maven Central:** If any versions have been published, the old
  coordinates remain cached in Maven Central permanently
- **Deprecation:** Mark old published versions as deprecated in the
  repository metadata. If no versions have been published to a public
  repository, this step can be skipped.

#### First post-migration publish checklist

For both Go and Java, the first release after migration should:

1. Verify the new module/artifact identity resolves correctly
2. Confirm old versions are retracted/deprecated (if any were published)
3. Document the identity change in release notes
4. Verify downstream resolution — `go get` / Maven dependency
   resolution pulls the new identity correctly

## Section 4: Post-Migration

After all 8 repos are transferred and per-repo migrations are
complete:

1. **Governance Phase 2 activation** (if not already done after
   transfer — see governance spec Section 5):
   - Create Human PAT and Agent PAT scoped to `mq-rest-admin-project`
   - Store credentials in Keychain under `mq-rest-admin/` namespace
   - Install the `mq-rest-admin-release` GitHub App org-wide
   - Invite `wphillipmoore-agent` as outside collaborator
   - Configure org-level branch protection rulesets
   - Verify: agent can push branches, human can approve/merge PRs,
     direct pushes to `develop`/`main` are blocked, CI runs on PRs
2. **Create deferred work issues** — `.github` profile repo,
   cross-human review CI check, credential audit tooling, merge queue
3. **Cross-repo reference sweep** — search all repos in `wphillipmoore`
   and other orgs for stale references to the old locations
4. **Local cleanup** — update local directory names if desired, update
   Claude Code memory paths

## Risks

| Risk | Mitigation |
|---|---|
| Transfer fails mid-sequence (partial state) | Sequential transfer with verification; stop on failure, push forward or roll back (Section 1) |
| CI triggers between transfer and workflow update | Repos are frozen during migration (single-developer project); in a multi-developer environment, use a tight transfer-then-update sequence per repo |
| Go module path change breaks external consumers | No known external consumers; retract old versions if published; document in release notes (Section 3.1) |
| Java Maven coordinates change | No known external consumers; deprecate old versions if published (Section 3.1) |
| Language repos reference dev-environment action at old path | Dev-environment transfers first (dependency order); each repo's workflow sweep updates the reference |
| Credential proliferation (3 more Keychain entries) | Manageable at current scale; credential audit tooling in backlog |
| 8 repos means 7 PRs to review and merge (template excluded) | Template-driven changes are mechanical and parallelizable |
| Org-level misconfiguration affects all 8 repos | Verify governance settings before any repo migration begins |
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
- `pom.xml` (Java)
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
