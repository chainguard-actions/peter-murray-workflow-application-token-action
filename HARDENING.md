<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 8 workflow files reference actions using mutable tag-based refs instead of pinned 40-character commit SHAs. This exposes the workflows to supply-chain attacks if the upstream action tags are moved or compromised. Affected refs: `actions/checkout@v4` and `actions/github-script@v7` appear across all workflow files.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:20`
- `.github/workflows/test_failure_repository_not_installed.yml:20`
- `.github/workflows/test_organization_installed.yml:20`
- `.github/workflows/test_organization_installed.yml:33`
- `.github/workflows/test_organization_installed_revocation.yml:20`
- `.github/workflows/test_organization_installed_revocation.yml:34`
- `.github/workflows/test_repository_installed.yml:20`
- `.github/workflows/test_repository_installed.yml:33`
- `.github/workflows/test_repository_installed_limited.yml:14`
- `.github/workflows/test_repository_installed_limited.yml:27`
- `.github/workflows/test_repository_installed_proxy.yml:27`
- `.github/workflows/test_repository_installed_proxy.yml:31`
- `.github/workflows/test_repository_installed_proxy.yml:56`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:30`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:34`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:59`

### missing-permissions (severity: medium)

None of the 8 workflow files define a top-level `permissions:` key, and no job within any of these files defines a job-level `permissions:` block. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad), violating the principle of least privilege.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:1`
- `.github/workflows/test_failure_repository_not_installed.yml:1`
- `.github/workflows/test_organization_installed.yml:1`
- `.github/workflows/test_organization_installed_revocation.yml:1`
- `.github/workflows/test_repository_installed.yml:1`
- `.github/workflows/test_repository_installed_limited.yml:1`
- `.github/workflows/test_repository_installed_proxy.yml:1`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (rule a). Expression values are substituted into the shell command string before the shell parses it, allowing shell metacharacters in the value to be executed. Violations: (1) test_failure_organization_not_installed.yml lines 37-38: `echo "Outcome: ${{ needs.test_failure.outputs.action_step_outcome }}"` and `echo "Conclusion: ${{ needs.test_failure.outputs.action_step_conclusion }}"`. (2) test_failure_repository_not_installed.yml lines 34-35: same needs.*.outputs.* pattern. (3) test_repository_installed_proxy.yml line 33: `mkdir ${{ github.workspace }}/squid` and line 51: `ls -la ${{ github.workspace }}/squid`. (4) test_repository_installed_proxy_using_no_proxy.yml lines 38 and 56 (job 1) and lines 93 and 111 (job 2): same `${{ github.workspace }}` pattern. Fix: assign the expression to an `env:` variable and reference it double-quoted in the shell script.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:37`
- `.github/workflows/test_failure_organization_not_installed.yml:38`
- `.github/workflows/test_failure_repository_not_installed.yml:34`
- `.github/workflows/test_failure_repository_not_installed.yml:35`
- `.github/workflows/test_repository_installed_proxy.yml:33`
- `.github/workflows/test_repository_installed_proxy.yml:51`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:38`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:56`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:93`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:111`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 8 workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to SHA `34e114876b0b11c390a56381ad16ebd13914f8d5` and `actions/github-script@v7` to SHA `f28e40c7f34bde8b3046d885e986cb6290c5673b` across all 8 files, preserving the tag as a comment.

2. **missing-permissions**: Added `permissions: {}` at the top level of all 8 workflow files to enforce least privilege.

3. **script-injection**: 
   - In `test_failure_organization_not_installed.yml` and `test_failure_repository_not_installed.yml`: moved `${{ needs.test_failure.outputs.action_step_outcome }}` and `${{ needs.test_failure.outputs.action_step_conclusion }}` into `env:` blocks as `ACTION_STEP_OUTCOME` and `ACTION_STEP_CONCLUSION`, then referenced them as `$ACTION_STEP_OUTCOME` and `$ACTION_STEP_CONCLUSION` in the shell script.
   - In `test_repository_installed_proxy.yml`: moved `${{ github.workspace }}` into an `env:` block as `WORKSPACE` for both the 'Start Squid Proxy container' and 'Show Squid Logs and stop container' steps, then referenced it as `"$WORKSPACE/..."` (double-quoted) in the shell scripts.
   - In `test_repository_installed_proxy_using_no_proxy.yml`: applied the same `WORKSPACE` env fix to both `test_no_proxy_ignored` and `test_no_proxy_acknowledged` jobs (4 run steps total).

