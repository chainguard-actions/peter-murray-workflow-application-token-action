<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference actions using mutable version tags (@v4, @v7) instead of pinned 40-character SHA commits. This exposes the workflows to supply-chain attacks if the upstream action tags are moved or compromised. Affected references include: actions/checkout@v4 and actions/github-script@v7.

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:18`
- `.github/workflows/test_failure_repository_not_installed.yml:18`
- `.github/workflows/test_organization_installed.yml:22`
- `.github/workflows/test_organization_installed.yml:27`
- `.github/workflows/test_organization_installed.yml:38`
- `.github/workflows/test_organization_installed_revocation.yml:22`
- `.github/workflows/test_organization_installed_revocation.yml:27`
- `.github/workflows/test_organization_installed_revocation.yml:38`
- `.github/workflows/test_repository_installed.yml:22`
- `.github/workflows/test_repository_installed.yml:27`
- `.github/workflows/test_repository_installed.yml:38`
- `.github/workflows/test_repository_installed_limited.yml:14`
- `.github/workflows/test_repository_installed_limited.yml:22`
- `.github/workflows/test_repository_installed_proxy.yml:29`
- `.github/workflows/test_repository_installed_proxy.yml:34`
- `.github/workflows/test_repository_installed_proxy.yml:55`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:33`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:38`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:57`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:96`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:101`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:120`

### missing-permissions (severity: medium)

None of the workflow files declare a top-level `permissions:` key, and none of the individual jobs declare job-level `permissions:` keys. Without explicit permissions, workflows run with the default (often broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

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

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands (rule a), allowing an attacker to inject arbitrary shell commands. Specific violations: (1) test_repository_installed_proxy.yml 'Start Squid Proxy container' uses `mkdir ${{ github.workspace }}/squid` and similar; (2) test_repository_installed_proxy.yml 'Show Squid Logs' uses `ls -la ${{ github.workspace }}/squid`; (3) test_repository_installed_proxy_using_no_proxy.yml has the same patterns in both jobs; (4) test_failure_organization_not_installed.yml 'Validate failure' uses `echo "Outcome: ${{ needs.test_failure.outputs.action_step_outcome }}"`; (5) test_failure_repository_not_installed.yml has the same pattern. All ${{ ... }} expressions inside run: blocks are interpolated before the shell sees them, enabling injection.

Locations:

- `.github/workflows/test_repository_installed_proxy.yml:39`
- `.github/workflows/test_repository_installed_proxy.yml:40`
- `.github/workflows/test_repository_installed_proxy.yml:41`
- `.github/workflows/test_repository_installed_proxy.yml:53`
- `.github/workflows/test_repository_installed_proxy.yml:54`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:44`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:45`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:46`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:58`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:59`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:107`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:108`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:109`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:121`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:122`
- `.github/workflows/test_failure_organization_not_installed.yml:34`
- `.github/workflows/test_failure_organization_not_installed.yml:35`
- `.github/workflows/test_failure_repository_not_installed.yml:34`
- `.github/workflows/test_failure_repository_not_installed.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 8 workflow files. (1) unpinned-uses: Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b across all workflow files; original tags preserved as inline comments. (2) missing-permissions: Added `permissions: {}` at the top level of all 8 workflow files. (3) script-injection: Moved all ${{ ... }} expressions out of run: blocks into step-level env: blocks — needs.test_failure.outputs.* moved to ACTION_STEP_OUTCOME/ACTION_STEP_CONCLUSION in the two failure test files; github.workspace moved to WORKSPACE env var in the proxy test files (both jobs in test_repository_installed_proxy_using_no_proxy.yml and the single job in test_repository_installed_proxy.yml).

