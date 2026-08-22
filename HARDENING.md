<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v5.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v5.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell commands. In test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml, the 'Validate failure' step echoes ${{ needs.test_failure.outputs.action_step_outcome }} and ${{ needs.test_failure.outputs.action_step_conclusion }} directly into shell commands — these step outputs are workflow-controllable and flow through YAML template substitution before the shell sees them. In test_repository_installed_proxy.yml, test_repository_installed_proxy_explict_ignore.yml, and test_repository_installed_proxy_using_no_proxy.yml, the 'Start Squid Proxy container' and 'Show Squid Logs' run: steps interpolate ${{ github.workspace }} directly into shell commands (e.g. 'mkdir ${{ github.workspace }}/squid', 'ls -la ${{ github.workspace }}/squid', 'sudo cat ${{ github.workspace }}/squid/access.log'). All of these should be replaced with the corresponding environment variable references ($GITHUB_WORKSPACE, $RUNNER_TEMP, etc.) or moved into env: blocks and referenced as quoted shell variables.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:41`
- `.github/workflows/test_failure_repository_not_installed.yml:38`
- `.github/workflows/test_repository_installed_proxy.yml:41`
- `.github/workflows/test_repository_installed_proxy.yml:57`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:41`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:57`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:44`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:60`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions by mutable version tags rather than full 40-character commit SHAs. Specifically: 'actions/checkout@v7' and 'actions/github-script@v9' are used across all 9 workflow files. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. Each reference should be pinned to a full SHA, e.g. 'actions/checkout@<40-hex-sha> # v7'.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:23`
- `.github/workflows/test_failure_repository_not_installed.yml:22`
- `.github/workflows/test_organization_installed.yml:21`
- `.github/workflows/test_organization_installed_revocation.yml:21`
- `.github/workflows/test_repository_installed.yml:21`
- `.github/workflows/test_repository_installed_limited.yml:16`
- `.github/workflows/test_repository_installed_proxy.yml:32`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:32`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 9 workflow files: (1) Pinned actions/checkout@v7 to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and actions/github-script@v9 to SHA 3a2844b7e9c422d3c10d287c895573f7108da1b3 across all workflow files. (2) Fixed script injection in test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml by moving needs.test_failure.outputs.* expressions into env: blocks and referencing them as $ACTION_STEP_OUTCOME/$ACTION_STEP_CONCLUSION in shell commands. (3) Fixed script injection in the three proxy workflow files by replacing ${{ github.workspace }} in run: steps with the $GITHUB_WORKSPACE environment variable (properly quoted). Remaining ${{ github.workspace }} occurrences are in commented-out YAML lines and are not executed.

