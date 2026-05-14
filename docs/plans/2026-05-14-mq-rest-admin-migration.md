# MQ REST Admin Migration — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use
> superpowers:subagent-driven-development (recommended) or
> superpowers:executing-plans to implement this plan task-by-task. Steps
> use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transfer all 8 mq-rest-admin repositories from `wphillipmoore`
to `mq-rest-admin-project`, update all 7 active repos to post-VERGIL
tooling names, and verify everything works.

**Architecture:** Batch transfer in dependency order, then per-repo
migration using a shared template with repo-specific deltas. Each repo
gets one feature branch and one PR.

**Spec:** `docs/specs/2026-05-14-mq-rest-admin-migration-design.md`

**Relationship to org governance plan:** This is Plan B (migration).
The org governance plan
(`docs/plans/2026-05-14-mq-rest-admin-org-governance-setup.md`) is
Plan A. Plan A Phase 1 (Tasks 1–3) must complete before this plan
begins. Plan A Phase 2 (Tasks 4–10) executes after Task 2 of this plan
completes.

**Prerequisites:**
- VERGIL rename complete (all tooling at post-VERGIL names)
- `mq-rest-admin-project` org governance Phase 1 complete (Tasks 1–3)
- No open PRs on any of the 8 repos
- Clean working state on `develop` for all active repos

---

## Task 1: Pre-Flight Checks

**Files:** None (verification only)

- [ ] **Step 1: Verify no open PRs on any repo**

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment mq-rest-admin-template; do
    count=$(gh pr list --repo wphillipmoore/$repo --state open --json number --jq 'length')
    echo "$repo: $count open PRs"
  done
  ```

  Expected: all repos show 0 open PRs. If any exist, merge or close
  them first.

- [ ] **Step 2: Verify clean working state for all active repos**

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment; do
    echo "=== $repo ==="
    cd ~/dev/github/$repo
    git status --short
    git log --oneline -1
    cd -
  done
  ```

  Expected: all on `develop`, clean working trees, up to date with
  origin.

- [ ] **Step 3: Verify org governance Phase 1 is complete**

  ```bash
  gh api orgs/mq-rest-admin-project --jq '.login'
  # Expected: mq-rest-admin-project

  security find-generic-password -s "mq-rest-admin/app-private-key" -w > /dev/null \
    && echo "app-private-key: OK"
  ```

- [ ] **Step 4: Verify vergil-actions v2.0 exists**

  ```bash
  gh release view v2.0.0 --repo vergil-project/vergil-actions \
    --json tagName --jq '.tagName'
  ```

  Expected: `v2.0.0`. If this fails, the VERGIL rename is not
  complete — do not proceed.

---

## Task 2: Transfer Sequence

**Files:** None (GitHub API operations)

Transfer repos one at a time in dependency order. After each transfer,
verify success before proceeding. If any transfer fails, stop and
diagnose before continuing.

- [ ] **Step 1: Transfer `mq-rest-admin-dev-environment`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-dev-environment/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-dev-environment \
    --jq '.full_name'
  ```

  Expected: `mq-rest-admin-project/mq-rest-admin-dev-environment`

- [ ] **Step 2: Transfer `mq-rest-admin-common`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-common/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-common \
    --jq '.full_name'
  ```

- [ ] **Step 3: Transfer `mq-rest-admin-python`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-python/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-python \
    --jq '.full_name'
  ```

- [ ] **Step 4: Transfer `mq-rest-admin-java`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-java/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-java \
    --jq '.full_name'
  ```

- [ ] **Step 5: Transfer `mq-rest-admin-go`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-go/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-go \
    --jq '.full_name'
  ```

- [ ] **Step 6: Transfer `mq-rest-admin-ruby`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-ruby/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-ruby \
    --jq '.full_name'
  ```

- [ ] **Step 7: Transfer `mq-rest-admin-rust`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-rust/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-rust \
    --jq '.full_name'
  ```

- [ ] **Step 8: Transfer `mq-rest-admin-template`**

  ```bash
  gh api repos/wphillipmoore/mq-rest-admin-template/transfer \
    -f new_owner=mq-rest-admin-project --silent
  sleep 10
  gh api repos/mq-rest-admin-project/mq-rest-admin-template \
    --jq '.full_name'
  ```

  This repo is archived. Transfer only — no per-repo migration.

- [ ] **Step 9: Verify all transfers succeeded**

  ```bash
  for repo in mq-rest-admin-dev-environment mq-rest-admin-common \
    mq-rest-admin-python mq-rest-admin-java mq-rest-admin-go \
    mq-rest-admin-ruby mq-rest-admin-rust mq-rest-admin-template; do
    result=$(gh api repos/mq-rest-admin-project/$repo --jq '.full_name' 2>/dev/null)
    echo "$repo: ${result:-FAILED}"
  done
  ```

  Expected: all 8 repos show `mq-rest-admin-project/<name>`.

- [ ] **Step 10: Update local git remotes**

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment; do
    cd ~/dev/github/$repo
    git remote set-url origin git@github.com:mq-rest-admin-project/$repo.git
    git fetch origin
    echo "$repo: $(git remote get-url origin)"
    cd -
  done
  ```

- [ ] **Step 11: Proceed to governance Phase 2**

  All transfers are complete. Run Plan A Phase 2 (Tasks 4–10) now
  before continuing with per-repo migrations.

---

## Task 3: Per-Repo Migration Template

This is the canonical task sequence for migrating one repo. It is not
a standalone task — Tasks 4–10 each execute this template with
repo-specific adjustments.

The template is included here as a reference. Each per-repo task
below lists only its **delta** (repo-specific additions or
modifications to the template steps).

### Template Steps

**T1. Pre-flight checks**

```bash
cd ~/dev/github/<repo>
git status
git log --oneline -1
git remote get-url origin
# Verify: on develop, clean, remote points to mq-rest-admin-project
```

**T2. Create feature branch**

```bash
git checkout -b feature/<issue>-org-migration
```

**T3. Rename config file**

```bash
git mv standard-tooling.toml vergil.toml
```

**T4. Update vergil.toml contents**

Replace the full file contents. The key changes:
- `[dependencies]`: `standard-tooling = "v1.4"` → `vergil = "v2.0"`
- `[project.co-authors]`: Replace `claude` and `codex` entries with
  single `agent` entry using `wphillipmoore-agent` noreply email

Get the agent user ID:
```bash
gh api users/wphillipmoore-agent --jq '.id'
```

**T5. Update CI/CD workflows**

In `.github/workflows/ci.yml` and `.github/workflows/cd.yml`:

```bash
sed -i '' \
  -e 's|wphillipmoore/standard-actions|vergil-project/vergil-actions|g' \
  -e 's|@v1\.5|@v2.0|g' \
  -e 's|wphillipmoore/mq-rest-admin-dev-environment|mq-rest-admin-project/mq-rest-admin-dev-environment|g' \
  .github/workflows/ci.yml .github/workflows/cd.yml
```

**T5a. Replace `secrets: inherit` with explicit secrets in CD workflow**

`secrets: inherit` does not work cross-org. Replace it with explicit
secret passing in `.github/workflows/cd.yml`:

```yaml
# Before:
    secrets: inherit

# After:
    secrets:
      APP_CLIENT_ID: ${{ secrets.APP_CLIENT_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

**T5b. Configure repository secrets**

Each repo needs `APP_CLIENT_ID` and `APP_PRIVATE_KEY` as repository
secrets (the App client ID was recorded during governance setup
Task 2 Step 3; the private key is in Keychain):

```bash
APP_CLIENT_ID="<client-id-from-governance-setup>"
APP_PRIVATE_KEY=$(security find-generic-password -s "mq-rest-admin/app-private-key" -w)

gh secret set APP_CLIENT_ID --repo mq-rest-admin-project/<repo> --body "$APP_CLIENT_ID"
gh secret set APP_PRIVATE_KEY --repo mq-rest-admin-project/<repo> --body "$APP_PRIVATE_KEY"
```

Verify:
```bash
gh secret list --repo mq-rest-admin-project/<repo>
```

**T6. Update issue templates**

```bash
sed -i '' \
  's|wphillipmoore/standard-tooling|vergil-project/vergil-tooling|g' \
  .github/ISSUE_TEMPLATE/config.yml \
  .github/ISSUE_TEMPLATE/issue.yml
```

**T7. Update CLAUDE.md**

Apply the substitution table from the migration design spec
(Section 2). All substitutions are mechanical text replacements:

```bash
sed -i '' \
  -e 's|https://github.com/wphillipmoore/standards-and-conventions|https://github.com/vergil-project/vergil-tooling|g' \
  -e 's|standard-tooling\.toml|vergil.toml|g' \
  -e 's|/standard-tooling:memory-init|/vergil:memory-init|g' \
  -e 's|/standard-tooling:memory-audit|/vergil:memory-audit|g' \
  -e 's|standard-tooling/docs/specs/worktree-convention\.md|vergil-tooling/docs/specs/worktree-convention.md|g' \
  -e 's|https://github.com/wphillipmoore/standard-tooling/blob/develop/|https://github.com/vergil-project/vergil-tooling/blob/develop/|g' \
  -e 's|The canonical text lives in `standard-tooling`|The canonical text lives in `vergil-tooling`|g' \
  -e 's|\.\./standard-tooling/scripts/lib/git-hooks|../vergil-tooling/scripts/lib/git-hooks|g' \
  -e 's|st-docker-run -- st-validate|vrg-docker-run -- vrg-validate|g' \
  -e 's|st-docker-run|vrg-docker-run|g' \
  -e 's|st-validate|vrg-validate|g' \
  -e 's|st-commit|vrg-commit|g' \
  -e 's|ST_COMMIT_CONTEXT|VRG_COMMIT_CONTEXT|g' \
  -e 's|Standard-tooling CLI tools|VERGIL CLI tools|g' \
  -e 's|https://github.com/wphillipmoore/mq-rest-admin-dev-environment|https://github.com/mq-rest-admin-project/mq-rest-admin-dev-environment|g' \
  -e 's|wphillipmoore/standards-and-conventions|vergil-project/vergil-tooling|g' \
  CLAUDE.md
```

**Manual review required** after sed — verify that substitutions
did not corrupt any context-sensitive references. In particular,
check that the worktree paths and agent prompt contract are intact.

**T8. Update repo-specific files**

Per-repo delta (see Tasks 4–10 below). Skip for repos with no
additional files.

**T9. Final reference sweep**

```bash
grep -rn "wphillipmoore\|standard-tooling\|standard-actions\|st-commit\|st-validate\|st-docker\|ST_COMMIT\|standard-tooling:\|secrets:.*inherit" \
  --include="*.py" --include="*.md" --include="*.toml" \
  --include="*.yml" --include="*.yaml" --include="*.json" \
  --include="*.sh" --include="*.go" --include="*.rs" \
  --include="*.rb" --include="*.java" --include="*.xml" \
  --include="*.gradle" --include="*.kts" . \
  | grep -v CHANGELOG.md | grep -v .git/
```

Expected: zero matches outside of historical files. Fix any
remaining references.

**T10. Validate**

Run the repo-specific validation command if one exists:

```bash
vrg-docker-run -- vrg-validate
```

If `vrg-docker-run` is not yet available, use repo-specific commands
(test suite, linting, etc.).

**T11. Commit and push**

Stage all changes. Commit with logically grouped commits:

1. Config rename and update (vergil.toml)
2. CI/CD workflow updates
3. Documentation updates (CLAUDE.md, issue templates)
4. Repo-specific file updates (if any)
5. Any fixes from validation

Push and open PR:

```bash
git push -u origin feature/<issue>-org-migration

gh pr create \
  --repo mq-rest-admin-project/<repo> \
  --title "chore: migrate to mq-rest-admin-project org and vergil tooling" \
  --body "$(cat <<'EOF'
## Summary

Migrate from wphillipmoore to mq-rest-admin-project org and adopt
post-VERGIL tooling names.

- Config: standard-tooling.toml → vergil.toml (vergil v2.0)
- CI: wphillipmoore/standard-actions@v1.5 → vergil-project/vergil-actions@v2.0
- Co-authors: consolidated to single agent entry
- Documentation: all references updated

## Test plan

- [ ] CI passes on PR
- [ ] No remaining references to wphillipmoore or standard-tooling
- [ ] Validation passes

Closes #<issue>
EOF
)"
```

---

## Task 4: Migrate mq-rest-admin-common

**Delta:** None — template only. This is a docs-and-data repo with no
language-specific metadata.

- [ ] **Step 1: Execute template steps T1–T11**

  No repo-specific files to update (T8 is a no-op).

---

## Task 5: Migrate mq-rest-admin-python

**Delta:** `pyproject.toml` URL updates.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Update pyproject.toml (T8)**

  Update the `[project.urls]` section:

  ```toml
  [project.urls]
  Homepage = "https://github.com/mq-rest-admin-project/mq-rest-admin-python"
  Repository = "https://github.com/mq-rest-admin-project/mq-rest-admin-python"
  Issues = "https://github.com/mq-rest-admin-project/mq-rest-admin-python/issues"
  ```

- [ ] **Step 3: Grep Python source for old references**

  ```bash
  grep -rn "wphillipmoore" src/ --include="*.py"
  ```

  Fix any matches found.

- [ ] **Step 4: Execute template steps T9–T11**

---

## Task 6: Migrate mq-rest-admin-java

**Delta:** `pom.xml` URL updates.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Update pom.xml (T8)**

  Update SCM, project URL, and issue management sections:

  ```bash
  sed -i '' \
    's|wphillipmoore/mq-rest-admin-java|mq-rest-admin-project/mq-rest-admin-java|g' \
    pom.xml
  ```

  Verify the changes cover:
  - `<url>` (project URL)
  - `<scm><url>` and `<scm><connection>` and `<scm><developerConnection>`
  - `<issueManagement><url>` (if present)

- [ ] **Step 3: Check Maven coordinates**

  ```bash
  grep -n "groupId\|artifactId" pom.xml | head -10
  ```

  If `groupId` references `wphillipmoore`, update it. If no versions
  have been published to a public Maven repository, no
  deprecation/retraction is needed.

- [ ] **Step 4: Grep Java source for old references**

  ```bash
  grep -rn "wphillipmoore" src/ --include="*.java"
  ```

  Fix any matches found.

- [ ] **Step 5: Execute template steps T9–T11**

---

## Task 7: Migrate mq-rest-admin-go

**Delta:** `go.mod` module path and internal import updates.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Update go.mod module path (T8)**

  ```bash
  sed -i '' \
    's|github.com/wphillipmoore/mq-rest-admin-go|github.com/mq-rest-admin-project/mq-rest-admin-go|g' \
    go.mod
  ```

- [ ] **Step 3: Update internal Go imports**

  ```bash
  find . -name '*.go' -exec sed -i '' \
    's|github.com/wphillipmoore/mq-rest-admin-go|github.com/mq-rest-admin-project/mq-rest-admin-go|g' \
    {} +
  ```

- [ ] **Step 4: Check Go module proxy status**

  ```bash
  curl -s "https://proxy.golang.org/github.com/wphillipmoore/mq-rest-admin-go/@v/list"
  ```

  If versions have been published, add a `retract` directive for all
  published versions in the old module's `go.mod` before proceeding.
  If no versions are published (empty response), skip retraction.

- [ ] **Step 5: Verify Go module integrity**

  ```bash
  go mod tidy
  go build ./...
  ```

- [ ] **Step 6: Execute template steps T9–T11**

---

## Task 8: Migrate mq-rest-admin-ruby

**Delta:** Gemspec URL updates.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Update gemspec (T8)**

  Find the gemspec file and update URLs:

  ```bash
  sed -i '' \
    's|wphillipmoore/mq-rest-admin-ruby|mq-rest-admin-project/mq-rest-admin-ruby|g' \
    *.gemspec
  ```

  Verify the changes cover:
  - `spec.homepage`
  - `spec.metadata["source_code_uri"]`
  - `spec.metadata["changelog_uri"]` (if present)
  - `spec.metadata["bug_tracker_uri"]` (if present)

- [ ] **Step 3: Grep Ruby source for old references**

  ```bash
  grep -rn "wphillipmoore" lib/ --include="*.rb"
  ```

  Fix any matches found.

- [ ] **Step 4: Execute template steps T9–T11**

---

## Task 9: Migrate mq-rest-admin-rust

**Delta:** `Cargo.toml` URL updates.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Update Cargo.toml (T8)**

  ```bash
  sed -i '' \
    's|wphillipmoore/mq-rest-admin-rust|mq-rest-admin-project/mq-rest-admin-rust|g' \
    Cargo.toml
  ```

  Verify the changes cover:
  - `repository`
  - `homepage` (if present)
  - `documentation` (if present)

- [ ] **Step 3: Grep Rust source for old references**

  ```bash
  grep -rn "wphillipmoore" src/ --include="*.rs"
  ```

  Fix any matches found.

- [ ] **Step 4: Execute template steps T9–T11**

---

## Task 10: Migrate mq-rest-admin-dev-environment

**Delta:** Reusable action internal references.

- [ ] **Step 1: Execute template steps T1–T7**

- [ ] **Step 2: Check reusable action for org references (T8)**

  ```bash
  grep -rn "wphillipmoore" .github/actions/ --include="*.yml"
  ```

  Update any matches to `mq-rest-admin-project`.

- [ ] **Step 3: Check Docker Compose and scripts for org references**

  ```bash
  grep -rn "wphillipmoore" docker-compose*.yml scripts/ \
    --include="*.yml" --include="*.yaml" --include="*.sh" 2>/dev/null
  ```

  Fix any matches found.

- [ ] **Step 4: Execute template steps T9–T11**

---

## Task 11: Verify Module Identity for Go and Java

After per-repo migrations are complete, verify that the module
identity changes for Go and Java are clean before the first release.

- [ ] **Step 1: Check Go module proxy for old path**

  ```bash
  curl -s "https://proxy.golang.org/github.com/wphillipmoore/mq-rest-admin-go/@v/list"
  ```

  If any versions are listed, confirm retraction was handled in
  Task 7. If empty, no action needed.

- [ ] **Step 2: Check Maven Central for old Java coordinates**

  Verify whether any versions of the Java library have been published
  under the old `groupId`. If so, confirm deprecation was handled in
  Task 6.

- [ ] **Step 3: Verify new Go module resolves**

  ```bash
  GONOSUMCHECK=* go list -m github.com/mq-rest-admin-project/mq-rest-admin-go@latest 2>&1
  ```

  This will fail until the first version is published under the new
  path — that's expected. The purpose is to confirm the module path
  is syntactically valid and the proxy is reachable.

- [ ] **Step 4: Document in release notes template**

  For both Go and Java, ensure the first post-migration release notes
  include:
  - The module/artifact identity change
  - Retraction/deprecation of old identities (if any were published)
  - Instructions for consumers to update import paths

---

## Task 12: Cross-Repo Reference Sweep

- [ ] **Step 1: Search for stale references across wphillipmoore repos**

  ```bash
  gh search code "mq-rest-admin" --owner wphillipmoore \
    --json repository,path \
    --jq '.[] | "\(.repository.fullName): \(.path)"'
  ```

  Any remaining references to `wphillipmoore/mq-rest-admin-*` in
  other repos should be updated.

- [ ] **Step 2: Search for stale references in vergil-project**

  ```bash
  gh search code "wphillipmoore/mq-rest-admin" --owner vergil-project \
    --json repository,path \
    --jq '.[] | "\(.repository.fullName): \(.path)"'
  ```

- [ ] **Step 3: Search for stale references in diogenes-project**

  ```bash
  gh search code "wphillipmoore/mq-rest-admin" --owner diogenes-project \
    --json repository,path \
    --jq '.[] | "\(.repository.fullName): \(.path)"'
  ```

- [ ] **Step 4: Update any cross-references found**

  For each stale reference, open a PR in the affected repo or note
  it for a future cleanup pass.

---

## Task 13: Local Cleanup

- [ ] **Step 1: Verify all local remotes point to new org**

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment; do
    cd ~/dev/github/$repo
    echo "$repo: $(git remote get-url origin)"
    cd -
  done
  ```

  All should show `git@github.com:mq-rest-admin-project/<repo>.git`.

- [ ] **Step 2: Clean up worktrees**

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment; do
    cd ~/dev/github/$repo
    git worktree prune
    rm -rf .worktrees/ 2>/dev/null
    cd -
  done
  ```

- [ ] **Step 3: Migrate Claude Code memory**

  The memory directory slug is derived from the CWD path. If local
  directories are not renamed, the slug remains the same and no
  migration is needed. If directories are renamed, move relevant
  memories:

  ```
  Old: ~/.claude/projects/-Users-pmoore-dev-github-mq-rest-admin-<repo>/memory/
  New: (same if directories unchanged)
  ```

- [ ] **Step 4: Smoke test end-to-end**

  For each repo, verify basic operations work:

  ```bash
  for repo in mq-rest-admin-common mq-rest-admin-python mq-rest-admin-java \
    mq-rest-admin-go mq-rest-admin-ruby mq-rest-admin-rust \
    mq-rest-admin-dev-environment; do
    cd ~/dev/github/$repo
    echo "=== $repo ==="
    git fetch origin
    git log --oneline -1
    cd -
  done
  ```

- [ ] **Step 5: Record completion**

  The migration is complete. All repos are at
  `mq-rest-admin-project/*`, using VERGIL tooling, with org
  governance active.
