<!-- markdownlint-disable -->

# Hardening Report: NathanVaughn--actions-cloudflare-purge/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **NathanVaughn--actions-cloudflare-purge/v3.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/tests.yml references two GitHub Actions using mutable version tags instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tags are moved or compromised. Failing references: `actions/checkout@v2` (line 16) and `actions/setup-python@v2` (line 19). These should be pinned to their full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/tests.yml:16`
- `.github/workflows/tests.yml:19`

### missing-permissions (severity: medium)

The workflow file .github/workflows/tests.yml has no top-level `permissions:` key and the `tests` job also has no `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or on every job.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed .github/workflows/tests.yml: (1) Pinned actions/checkout@v2 to full SHA ee0669bd1cc54295c223e0bb666b733df41de1c5 and actions/setup-python@v2 to full SHA e9aba2c848f5ebd159c070c61ea2c4e2b122355e, preserving the version tags as comments. (2) Added top-level `permissions: contents: read` block to restrict the workflow token to the minimum necessary permissions.

