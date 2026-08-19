<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v5.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use tag-based (non-SHA-pinned) `uses:` references. `actions/checkout@v4` and `actions/github-script@v7` are mutable tags that can be silently updated to point to different code, enabling supply-chain attacks. All refs must be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:19`
- `.github/workflows/test_failure_repository_not_installed.yml:19`
- `.github/workflows/test_organization_installed.yml:22`
- `.github/workflows/test_organization_installed.yml:35`
- `.github/workflows/test_organization_installed_revocation.yml:22`
- `.github/workflows/test_organization_installed_revocation.yml:36`
- `.github/workflows/test_repository_installed.yml:17`
- `.github/workflows/test_repository_installed.yml:21`
- `.github/workflows/test_repository_installed.yml:32`
- `.github/workflows/test_repository_installed_limited.yml:14`
- `.github/workflows/test_repository_installed_limited.yml:24`
- `.github/workflows/test_repository_installed_proxy.yml:28`
- `.github/workflows/test_repository_installed_proxy.yml:32`
- `.github/workflows/test_repository_installed_proxy.yml:55`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:28`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:32`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:57`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:33`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:37`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:60`

### missing-permissions (severity: medium)

None of the 9 workflow files declare a top-level `permissions:` block, and no individual job within any of these files declares a job-level `permissions:` block. Without explicit permissions, workflows run with the default (often write-all) token permissions, violating the principle of least privilege.

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

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). This allows template substitution to inject arbitrary shell metacharacters before the shell parses the command.

1. `test_failure_organization_not_installed.yml` (line 36): `echo "Outcome: ${{ needs.test_failure.outputs.action_step_outcome }}"` and `echo "Conclusion: ${{ needs.test_failure.outputs.action_step_conclusion }}"` — `needs.*.outputs.*` values are workflow-controllable.

2. `test_failure_repository_not_installed.yml` (line 34): same `needs.test_failure.outputs.*` pattern.

3. `test_repository_installed_proxy.yml` (line 36): `mkdir ${{ github.workspace }}/squid`, `sudo chown proxy:proxy ${{ github.workspace }}/squid`, `docker run ... -v ${{ github.workspace }}/...` — `github.workspace` is interpolated directly into shell commands.

4. `test_repository_installed_proxy_explict_ignore.yml` (line 36): same `github.workspace` pattern in `run:` blocks.

5. `test_repository_installed_proxy_using_no_proxy.yml` (line 41): same `github.workspace` pattern in multiple `run:` blocks across two jobs.

Fix: move all context values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$GITHUB_WORKSPACE"` which is already available as a pre-set env var, or use the built-in `$GITHUB_WORKSPACE` env var instead of `${{ github.workspace }}`).

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:36`
- `.github/workflows/test_failure_repository_not_installed.yml:34`
- `.github/workflows/test_repository_installed_proxy.yml:36`
- `.github/workflows/test_repository_installed_proxy.yml:50`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:36`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:51`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:41`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:55`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:97`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:111`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 9 workflow files:

1. unpinned-uses: Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b across all workflow files, preserving the tag in a comment.

2. missing-permissions: Added `permissions: {}` top-level block to all 9 workflow files to enforce least-privilege.

3. script-injection: (a) In test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml, moved needs.test_failure.outputs.* expressions into env: variables (ACTION_STEP_OUTCOME, ACTION_STEP_CONCLUSION) and referenced them as shell variables. (b) In test_repository_installed_proxy.yml, test_repository_installed_proxy_explict_ignore.yml, and test_repository_installed_proxy_using_no_proxy.yml, replaced all ${{ github.workspace }} interpolations in run: blocks with the pre-set $GITHUB_WORKSPACE environment variable (double-quoted), which is already available as a standard GitHub Actions environment variable and avoids template injection.

