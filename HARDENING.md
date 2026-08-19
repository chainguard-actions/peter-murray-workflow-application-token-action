<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v5.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple workflow files interpolate ${{ ... }} expressions directly inside run: shell blocks (sub-rule a). In test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml, `${{ needs.test_failure.outputs.action_step_outcome }}` and `${{ needs.test_failure.outputs.action_step_conclusion }}` are echoed directly in shell commands — these step outputs could contain attacker-influenced content. In test_repository_installed_proxy.yml, test_repository_installed_proxy_explict_ignore.yml, and test_repository_installed_proxy_using_no_proxy.yml, `${{ github.workspace }}` is interpolated directly into shell commands (mkdir, docker run -v, ls -la, sudo cat). Any ${{ }} expression inside a run: block is a script-injection risk as YAML template substitution occurs before the shell ever sees the value.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:39`
- `.github/workflows/test_failure_organization_not_installed.yml:40`
- `.github/workflows/test_failure_repository_not_installed.yml:36`
- `.github/workflows/test_failure_repository_not_installed.yml:37`
- `.github/workflows/test_repository_installed_proxy.yml:34`
- `.github/workflows/test_repository_installed_proxy.yml:47`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:35`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:50`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:38`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:51`

### missing-permissions (severity: medium)

None of the 9 workflow files under .github/workflows/ declare a top-level `permissions:` key, and none of the individual jobs declare a job-level `permissions:` key. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions. Each workflow file should declare minimal required permissions.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:1`
- `.github/workflows/test_failure_repository_not_installed.yml:1`
- `.github/workflows/test_organization_installed.yml:1`
- `.github/workflows/test_organization_installed_revocation.yml:1`
- `.github/workflows/test_repository_installed.yml:1`
- `.github/workflows/test_repository_installed_limited.yml:1`
- `.github/workflows/test_repository_installed_proxy.yml:1`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:1`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:1`

### unpinned-uses (severity: high)

All `uses:` references in every workflow file use mutable version tags (@v4, @v7) rather than immutable 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if the referenced action tags are moved or compromised. Affected references include: `actions/checkout@v4` (appears in all 9 workflow files) and `actions/github-script@v7` (appears in 8 workflow files).

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:20`
- `.github/workflows/test_failure_repository_not_installed.yml:20`
- `.github/workflows/test_organization_installed.yml:19`
- `.github/workflows/test_organization_installed_revocation.yml:19`
- `.github/workflows/test_repository_installed.yml:19`
- `.github/workflows/test_repository_installed_limited.yml:13`
- `.github/workflows/test_repository_installed_proxy.yml:24`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:24`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 9 workflow files under .github/workflows/:

1. unpinned-uses: Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b across all 9 files.

2. missing-permissions: Added `permissions: {}` at the top level of all 9 workflow files.

3. script-injection: (a) In test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml, moved needs.test_failure.outputs.action_step_outcome and action_step_conclusion into env: blocks (ACTION_STEP_OUTCOME, ACTION_STEP_CONCLUSION) and referenced them as plain env vars in the run: block. (b) In test_repository_installed_proxy.yml, test_repository_installed_proxy_explict_ignore.yml, and test_repository_installed_proxy_using_no_proxy.yml, moved github.workspace into env: blocks (WORKSPACE) and referenced it as "$WORKSPACE" (properly quoted) in all shell commands (mkdir, docker run -v, ls -la, sudo cat).

