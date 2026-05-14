# Alignment Review: MQ REST Admin Org Migration

**Date:** 2026-05-14
**Commit:** 2cd1c7b775c744baa30d63079c50b5c6808ce964

## Documents Reviewed

- **Intent:**
  - `docs/specs/2026-05-14-mq-rest-admin-org-governance-design.md`
  - `docs/specs/2026-05-14-mq-rest-admin-migration-design.md`
- **Action:**
  - `docs/plans/2026-05-14-mq-rest-admin-org-governance-setup.md`
  - `docs/plans/2026-05-14-mq-rest-admin-migration.md`
- **Design:** None (specs serve as both requirements and design)

## Source Control Conflicts

None — no conflicts with recent changes (verified during prior
pushback review).

## Issues Reviewed

### [1] Plan batches all transfers before per-repo migrations — contradicts spec's tight-sequence requirement
- **Category:** Design gap
- **Severity:** Critical
- **Documents:** Migration spec Section 2 vs. migration plan Tasks 2 + 4–10
- **Issue:** The spec said "do not leave a gap where CI could trigger
  against stale workflow references." The plan batches all 8 transfers
  in Task 2 before any per-repo migrations (Tasks 4–10), creating
  exactly that gap.
- **Resolution:** Dropped — single-developer project with repos frozen
  during migration. No risk of surprise CI triggers. Softened spec
  language to note this applies in multi-developer environments.

### [2] `main` branch merge restriction to org owners missing from governance plan ruleset
- **Category:** Missing coverage
- **Severity:** Important
- **Documents:** Governance spec Section 2 vs. governance plan Task 7
- **Issue:** Spec said "restricts merge to org owners only (release
  gate)." Plan's `main` ruleset was identical to `develop` with no
  ownership restriction.
- **Resolution:** Clarified in governance spec that the release gate is
  credential-based, not ruleset-based. The release tooling authenticates
  as the GitHub App using the human's Keychain credentials, so the human
  controls when releases happen. The existing review requirement plus
  credential isolation is sufficient.

### [3] Deferred work issue mismatch between spec and plan
- **Category:** Scope compliance
- **Severity:** Important
- **Documents:** Governance spec Section 7 vs. governance plan Task 9
- **Issue:** Plan created issues for "Claude Code permission model"
  (not in spec, already has its own spec) and ".github profile repo"
  (already exists). Missing "Cross-human review CI check" from spec
  (also already exists).
- **Resolution:** Removed `.github` profile repo and Claude Code
  permission model steps from plan (both already have existing issues).
  Kept credential audit tooling and merge queue (don't exist yet).

### [4] Cross-org `secrets: inherit` doesn't work — App credentials not in migration plan
- **Category:** Missing coverage (new finding during review)
- **Severity:** Important
- **Documents:** Governance spec Section 3 + migration spec + migration
  plan T5 + governance plan
- **Issue:** `secrets: inherit` in CD workflows only works within the
  same org. After migration, repos in `mq-rest-admin-project` call
  reusable workflows in `vergil-project` — cross-org, so explicit
  secret passing is required. Discovered during Diogenes migration.
- **Resolution:** Updated all four documents:
  - Governance spec: added "Repository Secrets for CD Workflows"
    subsection documenting `APP_CLIENT_ID` and `APP_PRIVATE_KEY` as
    per-repo secrets
  - Migration spec: added CD secrets row to "What changes" table,
    updated template step 5, added `secrets: inherit` to reference sweep
  - Migration plan: added T5a (replace `secrets: inherit` with explicit
    block) and T5b (configure repository secrets via `gh secret set`)
  - Both sweep steps: added `secrets:.*inherit` to grep patterns

### [5] Partial failure recovery options from spec not in migration plan
- **Category:** Missing coverage
- **Severity:** Minor
- **Documents:** Migration spec Section 1 vs. migration plan Task 2
- **Issue:** Spec documents push-forward and roll-back options; plan
  says "stop and diagnose" without reproducing recovery options.
- **Resolution:** Dropped — operator can consult the spec for recovery
  options if needed.

### [6] First post-migration publish checklist has no plan task
- **Category:** Missing coverage
- **Severity:** Minor
- **Documents:** Migration spec Section 3.1 vs. migration plan
- **Issue:** Spec's 4-item Go/Java publish checklist had no
  corresponding plan task.
- **Resolution:** Added Task 11 (Verify Module Identity for Go and
  Java) to the migration plan — checks Go module proxy, Maven Central,
  new path resolution, and release notes template. Renumbered
  subsequent tasks.

## Unresolved Issues

None — all issues addressed.

## Alignment Summary

- **Requirements:** 28 distinct items across both specs, 27 covered, 1 dropped (tight-sequence — not applicable to single-developer context)
- **Tasks:** 23 tasks across both plans (governance: 10, migration: 13), 22 in scope, 1 had out-of-scope items removed (governance Task 9)
- **New coverage added:** 3 items (cross-org secrets, module identity verification, release gate clarification)
- **Status:** Aligned — ready for implementation
