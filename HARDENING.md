<!-- markdownlint-disable -->

# Hardening Report: peter-murray--workflow-application-token-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-murray--workflow-application-token-action/v3.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

The file `.github_application` at the action root contains a hardcoded RSA private key. The JSON field `"privateKey"` holds a base64-encoded value (`LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQ...`) that decodes to a PEM RSA private key (`-----BEGIN RSA PRIVATE KEY-----`). This is a real private key committed to the repository and is not a GitHub Actions secret expression. This file is not under `tests/`, `dist/`, `vendor/`, or `node_modules/` — it is part of the distributed action root and is in scope for security checks.

Locations:

- `.github_application:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Removed the hardcoded RSA private key from hardened/action/.github_application. The 'test' section's 'privateKey' field contained a real base64-encoded RSA private key (decoding to a PEM RSA private key). It was replaced with the placeholder string 'REPLACE_WITH_BASE64_ENCODED_PRIVATE_KEY'. The 'test-ghes' section already used a placeholder value and was left unchanged.

