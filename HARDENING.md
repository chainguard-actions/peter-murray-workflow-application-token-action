<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag-based refs for external actions instead of pinned full-length SHA commits. Specifically, `actions/checkout@v4` and `actions/github-script@v7` are used across all 9 workflow files. These tags can be moved by the upstream repository owner, enabling supply-chain attacks. Each should be pinned to a full 40-character commit SHA (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/test_failure_organization_not_installed.yml:19`
- `.github/workflows/test_failure_repository_not_installed.yml:19`
- `.github/workflows/test_organization_installed.yml:19`
- `.github/workflows/test_organization_installed.yml:30`
- `.github/workflows/test_organization_installed.yml:40`
- `.github/workflows/test_organization_installed_revocation.yml:19`
- `.github/workflows/test_organization_installed_revocation.yml:30`
- `.github/workflows/test_organization_installed_revocation.yml:40`
- `.github/workflows/test_repository_installed.yml:19`
- `.github/workflows/test_repository_installed.yml:30`
- `.github/workflows/test_repository_installed.yml:40`
- `.github/workflows/test_repository_installed_limited.yml:14`
- `.github/workflows/test_repository_installed_limited.yml:24`
- `.github/workflows/test_repository_installed_proxy.yml:27`
- `.github/workflows/test_repository_installed_proxy.yml:34`
- `.github/workflows/test_repository_installed_proxy.yml:57`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:27`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:34`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:60`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:32`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:39`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:66`

### missing-permissions (severity: medium)

None of the 9 workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions. Each workflow should declare minimal required permissions (e.g., `permissions: contents: read`).

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

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell commands, violating rule (a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever parses the command.

(1) `test_repository_installed_proxy.yml` — 'Start Squid Proxy container' step interpolates `${{ github.workspace }}` directly in shell commands: `mkdir ${{ github.workspace }}/squid`, `docker run ... -v ${{ github.workspace }}/...`. 'Show Squid Logs' step also uses `${{ github.workspace }}`.

(2) `test_repository_installed_proxy_explict_ignore.yml` — same pattern in 'Start Squid Proxy container' and 'Show Squid Logs' steps.

(3) `test_repository_installed_proxy_using_no_proxy.yml` — same pattern in both jobs.

(4) `test_failure_organization_not_installed.yml` — 'Validate failure' step interpolates `${{ needs.test_failure.outputs.action_step_outcome }}` and `${{ needs.test_failure.outputs.action_step_conclusion }}` directly in `echo` commands inside a `run:` block. These are step outputs from a prior job and are workflow-controllable.

(5) `test_failure_repository_not_installed.yml` — same pattern as (4).

Fix: replace `${{ github.workspace }}` with the environment variable `$GITHUB_WORKSPACE`, and route `needs.*.outputs.*` values through `env:` variables with double-quoted shell expansions.

Locations:

- `.github/workflows/test_repository_installed_proxy.yml:38`
- `.github/workflows/test_repository_installed_proxy.yml:50`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:38`
- `.github/workflows/test_repository_installed_proxy_explict_ignore.yml:53`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:43`
- `.github/workflows/test_repository_installed_proxy_using_no_proxy.yml:55`
- `.github/workflows/test_failure_organization_not_installed.yml:39`
- `.github/workflows/test_failure_repository_not_installed.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 9 workflow files:

1. unpinned-uses: Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b across all 9 workflow files. Preserved the tag as a comment (# v4, # v7) for readability.

2. missing-permissions: Added 'permissions: contents: read' top-level block to all 9 workflow files.

3. script-injection: (a) In test_failure_organization_not_installed.yml and test_failure_repository_not_installed.yml, moved needs.test_failure.outputs.* expressions into env: variables (ACTION_STEP_OUTCOME, ACTION_STEP_CONCLUSION) and referenced them as plain shell variables in the run: block. (b) In the three proxy workflow files (test_repository_installed_proxy.yml, test_repository_installed_proxy_explict_ignore.yml, test_repository_installed_proxy_using_no_proxy.yml), replaced all ${{ github.workspace }} interpolations in run: blocks with the $GITHUB_WORKSPACE environment variable (double-quoted), which is always available in GitHub Actions runners without template substitution.

