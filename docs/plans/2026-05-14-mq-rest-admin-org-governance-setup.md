# MQ REST Admin Org Governance Setup — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use
> superpowers:subagent-driven-development (recommended) or
> superpowers:executing-plans to implement this plan task-by-task. Steps
> use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Configure the `mq-rest-admin-project` GitHub org with the same
governance model as `vergil-project` and `diogenes-project` (identity
separation, branch protection, credential management, GitHub App) so
that the MQ REST Admin migration (separate plan) can proceed into a
properly configured org.

**Architecture:** Two-phase approach — pre-transfer infrastructure
setup (org settings, GitHub App) and post-transfer governance activation
(credentials, rulesets, agent invitation). Fine-grained PATs and
rulesets require repos to exist under the resource owner, so they must
wait until after the transfer.

**Spec:** `docs/specs/2026-05-14-mq-rest-admin-org-governance-design.md`

**Relationship to migration plan:** This is Plan A (org setup). The
migration plan (`docs/plans/2026-05-14-mq-rest-admin-migration.md`) is
Plan B. Plan A Phase 1 must complete before Plan B begins. Plan A
Phase 2 executes after Plan B Task 2 (transfer sequence) completes.

**Prerequisites:**
- VERGIL rename complete
- `wphillipmoore-agent` GitHub account exists (created during VERGIL
  setup)

---

## Phase 1: Pre-Transfer Setup

These tasks are performed once, mostly via the GitHub web UI and `gh`
CLI. They establish the org configuration that the transfer depends on.

### Task 1: Verify Org Exists and Configure Security Settings

**Files:** None (`gh` CLI)

The `mq-rest-admin-project` org must exist before proceeding. This
task verifies it and applies security settings.

- [ ] **Step 1: Verify org exists**

  ```bash
  gh api orgs/mq-rest-admin-project --jq '.login'
  ```

  Expected: `mq-rest-admin-project`

  If the org does not exist, create it via the GitHub web UI at
  <https://github.com/organizations/plan>.

- [ ] **Step 2: Require 2FA for all org members**

  ```bash
  gh api orgs/mq-rest-admin-project \
    -X PATCH \
    -f two_factor_requirement_enabled=true
  ```

- [ ] **Step 3: Set default repository permission to Write**

  ```bash
  gh api orgs/mq-rest-admin-project \
    -X PATCH \
    -f default_repository_permission=write
  ```

- [ ] **Step 4: Disable forking of private repos**

  ```bash
  gh api orgs/mq-rest-admin-project \
    -X PATCH \
    -F members_can_fork_private_repositories=false
  ```

- [ ] **Step 5: Verify settings**

  ```bash
  gh api orgs/mq-rest-admin-project \
    --jq '{two_factor: .two_factor_requirement_enabled, default_perm: .default_repository_permission, fork_private: .members_can_fork_private_repositories}'
  ```

  Expected:
  ```json
  {
    "two_factor": true,
    "default_perm": "write",
    "fork_private": false
  }
  ```

### Task 2: Register `mq-rest-admin-release` GitHub App

**Files:** None (GitHub web UI + Keychain)

- [ ] **Step 1: Register the App**

  Go to <https://github.com/organizations/mq-rest-admin-project/settings/apps/new>

  - App name: `mq-rest-admin-release`
  - Homepage URL: `https://github.com/mq-rest-admin-project`
  - Webhook: uncheck "Active" (not needed)
  - Permissions:
    - Repository permissions:
      - Contents: Read and write
      - Pull requests: Read and write
      - Metadata: Read-only
    - No organization permissions
    - No account permissions
  - Where can this app be installed: Only on this account

- [ ] **Step 2: Generate a private key**

  On the App settings page, under "Private keys", click
  "Generate a private key". Download the `.pem` file.

- [ ] **Step 3: Record the App client ID**

  Note the client ID from the App settings page. This is not a
  secret — it will be recorded in the CI/CD workflow files as an
  argument to the reusable release workflow.

- [ ] **Step 4: Store the App private key in Keychain**

  ```bash
  security add-generic-password \
    -a "mq-rest-admin" \
    -s "mq-rest-admin/app-private-key" \
    -w "$(cat /path/to/downloaded-key.pem)" \
    -T "" \
    -U
  ```

- [ ] **Step 5: Verify Keychain retrieval**

  ```bash
  security find-generic-password -s "mq-rest-admin/app-private-key" -w > /dev/null \
    && echo "app-private-key: OK"
  ```

- [ ] **Step 6: Delete the downloaded `.pem` file**

  ```bash
  rm /path/to/downloaded-key.pem
  ```

  The key is now stored in the Keychain only.

### Task 3: Pre-Transfer Gate

**Files:** None (verification only)

- [ ] **Step 1: Verify all Phase 1 tasks are complete**

  ```bash
  # Org exists and configured
  gh api orgs/mq-rest-admin-project \
    --jq '{two_factor: .two_factor_requirement_enabled, default_perm: .default_repository_permission}'

  # App private key in Keychain
  security find-generic-password -s "mq-rest-admin/app-private-key" -w > /dev/null \
    && echo "app-private-key: OK"
  ```

- [ ] **Step 2: Record completion**

  Phase 1 is complete. Plan B (the migration) can now proceed with
  the transfer sequence (Task 2). Plan A Phase 2 resumes after all
  repos are transferred.

---

## Phase 2: Post-Transfer Governance Activation

These tasks execute after Plan B Task 2 (transfer sequence) is
complete and all 8 repos are at `mq-rest-admin-project/*`.

### Task 4: Generate and Store Human PAT

**Files:** None (GitHub web UI + Keychain)

- [ ] **Step 1: Generate the human PAT**

  Go to <https://github.com/settings/personal-access-tokens/new>
  (logged in as `wphillipmoore`).

  - Token name: `mq-rest-admin-human`
  - Expiration: 1 year
  - Resource owner: `mq-rest-admin-project`
  - Repository access: All repositories
  - Permissions:
    - Administration: Read and write
    - Contents: Read and write
    - Issues: Read and write
    - Pull requests: Read and write
    - Actions: Read and write
    - Metadata: Read (auto-granted)

  Copy the token value.

- [ ] **Step 2: Verify PAT scope is correct**

  ```bash
  GH_TOKEN=<human-pat> gh api user --jq '.login'
  ```

  Expected: `wphillipmoore`

- [ ] **Step 3: Store the human PAT in Keychain**

  ```bash
  security add-generic-password \
    -a "mq-rest-admin" \
    -s "mq-rest-admin/human-pat" \
    -w "<paste-human-pat-here>" \
    -T "" \
    -U
  ```

- [ ] **Step 4: Verify Keychain retrieval**

  ```bash
  security find-generic-password -s "mq-rest-admin/human-pat" -w
  ```

### Task 5: Install GitHub App and Invite Agent Account

**Files:** None (`gh` CLI + GitHub web UI)

- [ ] **Step 1: Install the App on the org**

  Go to the App settings page → "Install App" → Install on
  `mq-rest-admin-project` → All repositories.

- [ ] **Step 2: Verify the installation**

  ```bash
  gh api orgs/mq-rest-admin-project/installations --jq '.[].app_slug'
  ```

  Expected: `mq-rest-admin-release`

- [ ] **Step 3: List all repos in the org**

  ```bash
  gh repo list mq-rest-admin-project --json name --jq '.[].name'
  ```

- [ ] **Step 4: Invite `wphillipmoore-agent` as outside collaborator**

  ```bash
  for repo in $(gh repo list mq-rest-admin-project --json name --jq '.[].name'); do
    gh api repos/mq-rest-admin-project/$repo/collaborators/wphillipmoore-agent \
      -X PUT \
      -f permission=push
  done
  ```

- [ ] **Step 5: Accept the invitation**

  Log in as `wphillipmoore-agent` in the GitHub web UI and accept
  the collaboration invitation.

- [ ] **Step 6: Verify access**

  Confirm the org's repos are visible at
  `https://github.com/orgs/mq-rest-admin-project/repositories`.

### Task 6: Generate and Store Agent PAT

**Files:** None (GitHub web UI + Keychain)

- [ ] **Step 1: Generate the agent PAT**

  Go to <https://github.com/settings/personal-access-tokens/new>
  (logged in as `wphillipmoore-agent`).

  - Token name: `mq-rest-admin-agent`
  - Expiration: 1 year
  - Resource owner: `mq-rest-admin-project`
  - Repository access: All repositories
  - Permissions:
    - Contents: Read and write
    - Issues: Read and write
    - Pull requests: Read and write
    - Metadata: Read (auto-granted)
  - **Not granted:** Administration, Actions, org settings, secrets,
    deployments

  Copy the token value.

- [ ] **Step 2: Verify agent PAT identity**

  ```bash
  GH_TOKEN=<agent-pat> gh api user --jq '.login'
  ```

  Expected: `wphillipmoore-agent`

- [ ] **Step 3: Store the agent PAT in Keychain**

  ```bash
  security add-generic-password \
    -a "mq-rest-admin" \
    -s "mq-rest-admin/agent-pat" \
    -w "<paste-agent-pat-here>" \
    -T "" \
    -U
  ```

- [ ] **Step 4: Verify Keychain retrieval**

  ```bash
  security find-generic-password -s "mq-rest-admin/agent-pat" -w
  ```

### Task 7: Configure Org-Level Rulesets

**Files:** None (`gh` CLI)

- [ ] **Step 1: Create the branch protection ruleset for `develop`**

  ```bash
  gh api orgs/mq-rest-admin-project/rulesets \
    -X POST \
    --input - <<'JSON'
  {
    "name": "Branch protection (develop)",
    "target": "branch",
    "enforcement": "active",
    "conditions": {
      "ref_name": {
        "include": ["refs/heads/develop"],
        "exclude": []
      }
    },
    "rules": [
      { "type": "pull_request",
        "parameters": {
          "required_approving_review_count": 1,
          "dismiss_stale_reviews_on_push": true,
          "require_code_owner_review": false,
          "require_last_push_approval": true,
          "required_review_thread_resolution": true
        }
      },
      { "type": "required_status_checks",
        "parameters": {
          "strict_status_checks_policy": true,
          "status_checks": []
        }
      },
      { "type": "deletion" },
      { "type": "non_fast_forward" }
    ],
    "bypass_actors": []
  }
  JSON
  ```

  **Note:** `bypass_actors: []` means no one can bypass — not even
  org owners. `status_checks` is empty at the org level because CI
  check names vary by repo.

- [ ] **Step 2: Create the branch protection ruleset for `main`**

  ```bash
  gh api orgs/mq-rest-admin-project/rulesets \
    -X POST \
    --input - <<'JSON'
  {
    "name": "Branch protection (main)",
    "target": "branch",
    "enforcement": "active",
    "conditions": {
      "ref_name": {
        "include": ["refs/heads/main"],
        "exclude": []
      }
    },
    "rules": [
      { "type": "pull_request",
        "parameters": {
          "required_approving_review_count": 1,
          "dismiss_stale_reviews_on_push": true,
          "require_code_owner_review": false,
          "require_last_push_approval": true,
          "required_review_thread_resolution": true
        }
      },
      { "type": "required_status_checks",
        "parameters": {
          "strict_status_checks_policy": true,
          "status_checks": []
        }
      },
      { "type": "deletion" },
      { "type": "non_fast_forward" }
    ],
    "bypass_actors": []
  }
  JSON
  ```

- [ ] **Step 3: Verify rulesets are active**

  ```bash
  gh api orgs/mq-rest-admin-project/rulesets \
    --jq '.[] | {name: .name, enforcement: .enforcement}'
  ```

  Expected:
  ```json
  {"name": "Branch protection (develop)", "enforcement": "active"}
  {"name": "Branch protection (main)", "enforcement": "active"}
  ```

### Task 8: Test Rulesets

**Files:** None (manual verification)

- [ ] **Step 1: Verify direct push is blocked**

  Attempt to push directly to `develop` on any repo:

  ```bash
  cd /tmp && git clone git@github.com:mq-rest-admin-project/mq-rest-admin-common.git mq-test
  cd mq-test
  git checkout develop
  echo "test" > test-rulesets.txt
  git add test-rulesets.txt
  git commit -m "test: verify branch protection"
  git push origin develop
  ```

  Expected: push rejected with a message about branch protection.

- [ ] **Step 2: Verify PR without review is blocked**

  ```bash
  git checkout -b test/verify-rulesets
  git push -u origin test/verify-rulesets

  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    gh pr create \
      --repo mq-rest-admin-project/mq-rest-admin-common \
      --title "test: verify branch protection" \
      --body "Testing governance rulesets. Will delete." \
      --head test/verify-rulesets \
      --base develop

  # Attempt to merge without approval — should fail
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    gh pr merge <PR-NUMBER> \
      --repo mq-rest-admin-project/mq-rest-admin-common \
      --merge
  ```

  Expected: merge rejected — required reviews not satisfied.

- [ ] **Step 3: Verify human approval enables merge**

  ```bash
  # Approve as human
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh pr review <PR-NUMBER> \
      --repo mq-rest-admin-project/mq-rest-admin-common \
      --approve

  # Merge as human
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh pr merge <PR-NUMBER> \
      --repo mq-rest-admin-project/mq-rest-admin-common \
      --merge
  ```

  Expected: merge succeeds.

- [ ] **Step 4: Clean up test branch**

  ```bash
  gh api repos/mq-rest-admin-project/mq-rest-admin-common/git/refs/heads/test/verify-rulesets \
    -X DELETE
  ```

- [ ] **Step 5: Clean up test clone**

  ```bash
  rm -rf /tmp/mq-test
  ```

### Task 9: Create Deferred Work Issues

**Files:** None (`gh` CLI)

Create issues in `mq-rest-admin-project/mq-rest-admin-common` for
deferred governance work. Issues for `.github` profile repo and
cross-human review CI check already exist — do not duplicate.

- [ ] **Step 1: Create issue for credential audit tooling**

  ```bash
  gh issue create --repo mq-rest-admin-project/mq-rest-admin-common \
    --title "feat: extend credential audit tooling for mq-rest-admin-project" \
    --body "Extend VERGIL's vrg-credential-audit (once built) to cover the mq-rest-admin/ credential namespace in macOS Keychain. Track alongside the same effort in vergil-project and diogenes-project."
  ```

- [ ] **Step 2: Create issue for merge queue**

  ```bash
  gh issue create --repo mq-rest-admin-project/mq-rest-admin-common \
    --title "feat: enable merge queue for mq-rest-admin-project" \
    --body "Enable GitHub merge queue for the mq-rest-admin-project org when the org moves to a paid GitHub plan. Coordinate with the same effort in vergil-project and diogenes-project."
  ```

- [ ] **Step 3: Verify issues are created**

  ```bash
  gh issue list --repo mq-rest-admin-project/mq-rest-admin-common \
    --json number,title \
    --jq '.[] | "\(.number): \(.title)"'
  ```

### Task 10: End-to-End Verification

**Files:** None (manual verification)

- [ ] **Step 1: Verify identity separation**

  ```bash
  # Agent can push a branch
  cd /tmp && git clone git@github.com:mq-rest-admin-project/mq-rest-admin-common.git mq-verify
  cd mq-verify
  git checkout -b test/verify-identity
  echo "test" > test-identity.txt
  git add test-identity.txt
  git commit -m "test: identity verification"
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    git push -u origin test/verify-identity

  # Agent cannot merge without approval
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    gh pr create \
      --repo mq-rest-admin-project/mq-rest-admin-common \
      --title "test: identity verification" \
      --body "Verifying governance model." \
      --head test/verify-identity \
      --base develop

  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    gh pr merge <PR> --merge
  # Expected: FAIL — review required

  # Human approves and merges
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh pr review <PR> --approve
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh pr merge <PR> --merge
  # Expected: SUCCESS
  ```

- [ ] **Step 2: Verify credential isolation**

  ```bash
  # Agent PAT cannot administer
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/agent-pat" -w) \
    gh api orgs/mq-rest-admin-project \
      -X PATCH \
      -f description="test"
  # Expected: 403 Forbidden

  # Human PAT can administer
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh api orgs/mq-rest-admin-project \
      -X PATCH \
      -f description="MQ REST Admin — multi-language client libraries for the IBM MQ administrative REST API"
  # Expected: 200 OK
  ```

- [ ] **Step 3: Verify GitHub App**

  ```bash
  GH_TOKEN=$(security find-generic-password -s "mq-rest-admin/human-pat" -w) \
    gh api orgs/mq-rest-admin-project/installations \
      --jq '.[].app_slug'
  # Expected: mq-rest-admin-release
  ```

- [ ] **Step 4: Verify all credentials in Keychain**

  ```bash
  security find-generic-password -s "mq-rest-admin/human-pat" -w > /dev/null \
    && echo "human-pat: OK"
  security find-generic-password -s "mq-rest-admin/agent-pat" -w > /dev/null \
    && echo "agent-pat: OK"
  security find-generic-password -s "mq-rest-admin/app-private-key" -w > /dev/null \
    && echo "app-private-key: OK"
  ```

- [ ] **Step 5: Clean up test branches and PRs**

  Remove any test branches and close any test PRs created during
  verification.

  ```bash
  gh api repos/mq-rest-admin-project/mq-rest-admin-common/git/refs/heads/test/verify-identity \
    -X DELETE
  rm -rf /tmp/mq-verify
  ```

- [ ] **Step 6: Record completion**

  The org governance setup is complete. The migration is verified
  and the org is ready for production use.
