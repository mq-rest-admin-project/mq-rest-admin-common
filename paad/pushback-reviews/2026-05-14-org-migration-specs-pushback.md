# Pushback Review: MQ REST Admin Org Migration Specs

**Date:** 2026-05-14
**Specs reviewed:**
- `docs/specs/2026-05-14-mq-rest-admin-org-governance-design.md`
- `docs/specs/2026-05-14-mq-rest-admin-migration-design.md`
**Commit:** 601cf023411ca5ae0b018d817c27bec5e7fb0630

## Source Control Conflicts

None — no conflicts with recent changes. The current codebase state
(pre-VERGIL tooling names, `wphillipmoore` org references, `claude`/`codex`
co-author entries) matches what both specs assume.

## Issues Reviewed

### [1] Batch transfer has no atomicity and no rollback plan
- **Category:** Feasibility + Omissions
- **Severity:** Critical
- **Issue:** GitHub's transfer API operates one repo at a time. A mid-batch
  failure leaves repos split between orgs with no documented recovery.
- **Resolution:** Replaced "single batch" with explicit sequential transfer
  procedure including dependency-ordered transfer, per-repo verification,
  stop-on-failure policy, and rollback instructions.

### [2] CI breaks during migration window
- **Category:** Feasibility
- **Severity:** Serious
- **Issue:** GitHub Actions `uses:` resolution does not reliably follow
  redirects for transferred repos. Any CI trigger between transfer and
  workflow file update could fail.
- **Resolution:** Added requirement for tight transfer-then-update sequence
  per repo to eliminate the window where stale references could trigger CI.

### [3] CLAUDE.md changes underspecified
- **Category:** Omissions
- **Severity:** Serious
- **Issue:** Step 7 ("Update CLAUDE.md") was too vague — at least 12 distinct
  reference categories needed updating, including skill names, git hooks
  path, worktree convention URLs, dev environment URLs, and CLI command names.
- **Resolution:** Added a concrete before/after substitution table to the
  per-repo migration template making step 7 mechanically executable.

### [4] "Plan A Phase 2" is an undefined reference
- **Category:** Ambiguity
- **Severity:** Serious
- **Issue:** Post-migration Section 4 referenced "Plan A Phase 2" for
  governance activation without defining it or pointing to a source.
- **Resolution:** Replaced with inlined governance activation steps —
  credential creation, App installation, agent invitation, ruleset
  configuration, and verification checks.

### [5] Go/Java module identity changes buried as one-liners
- **Category:** Scope imbalance
- **Severity:** Moderate
- **Issue:** Go and Java both expose the GitHub org path in their
  module/package identity (unlike Python/Ruby/Rust). The org transfer
  changes how the community discovers and imports these libraries.
  This was listed as a one-liner alongside trivial URL updates.
- **Resolution:** Added Section 3.1 covering both languages — Go module
  proxy caching, retraction directives, Maven coordinate implications,
  deprecation of old published versions, internal import path updates,
  and a first-post-migration-publish checklist. Pinned Java to Maven
  (`pom.xml`) per actual build system.

### [6] No verification step for vergil-actions@v2.0 prerequisite
- **Category:** Omissions
- **Severity:** Moderate
- **Issue:** No concrete check to verify the VERGIL rename prerequisite
  was met before starting migration.
- **Resolution:** Dropped — the VERGIL rename is confirmed complete and
  the `v2.0` tag is published.

### [7] Java build system ambiguous
- **Category:** Ambiguity
- **Severity:** Moderate
- **Issue:** Spec said "Maven pom.xml or Gradle" — should state which
  build system the repo actually uses.
- **Resolution:** Pinned to Maven (`pom.xml`) throughout the spec and
  file map.

### [8] Credential creation timing relative to transfer unspecified
- **Category:** Omissions
- **Severity:** Moderate
- **Issue:** Fine-grained PATs scoped to `mq-rest-admin-project` need
  repos to exist under that org. The governance spec treated setup as a
  single "complete before migration" block, but credentials and rulesets
  can only be created/verified after repos are transferred.
- **Resolution:** Split governance into Phase 1 (pre-transfer: org
  creation, security settings, App registration) and Phase 2
  (post-transfer: credentials, rulesets, agent invitation). Added
  Section 5 to the governance spec documenting the phasing and rationale.
  Updated migration spec prerequisites and post-migration steps to
  reference the phases.

### [9] Archived mq-rest-admin-template left behind in wphillipmoore
- **Category:** Omissions
- **Severity:** Minor
- **Issue:** The archived template repo would remain as an orphan in
  `wphillipmoore` after the other 7 repos move. GitHub allows
  transferring archived repos without unarchiving.
- **Resolution:** Added to the transfer list (8 repos total, transferred
  archived, no per-repo migration needed). Updated scope, repo count
  references, and risk table.

## Unresolved Issues

None — all issues addressed.

## Summary

- **Issues found:** 9
- **Issues resolved:** 8
- **Dropped:** 1 (finding #6 — prerequisite confirmed met)
- **Spec status:** Both specs updated and ready for implementation
