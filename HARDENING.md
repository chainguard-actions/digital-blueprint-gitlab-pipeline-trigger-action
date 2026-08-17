<!-- markdownlint-disable -->

# Hardening Report: digital-blueprint--gitlab-pipeline-trigger-action/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **digital-blueprint--gitlab-pipeline-trigger-action/v1.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved.

- .github/workflows/format-check.yml: `actions/checkout@v6`, `cachix/install-nix-action@v31`, `cachix/cachix-action@v16`
- .github/workflows/main.yml: `actions/checkout@v6`
- .github/workflows/test.yml: `actions/checkout@v6`

Locations:

- `.github/workflows/format-check.yml:10`
- `.github/workflows/format-check.yml:12`
- `.github/workflows/format-check.yml:15`
- `.github/workflows/main.yml:14`
- `.github/workflows/test.yml:11`

### script-injection (severity: high)

Sub-rule (a): Three `run:` steps in main.yml directly interpolate `${{ steps.test.outputs.* }}` expressions inside shell commands. Although `steps.*.outputs.*` values are set by the action under test, they are still workflow-controlled context values that flow through YAML template substitution before the shell sees them. If the action ever sets an output containing shell metacharacters, this would result in command injection.

Offending lines:
  - `run: echo "The status was ${{ steps.test.outputs.status }}"`
  - `run: echo "The web_url was ${{ steps.test.outputs.web_url }}"`
  - `run: echo "The artifacts_downloaded was ${{ steps.test.outputs.artifacts_downloaded }}"`

Fix: Move the values into env vars and reference them as `"$STATUS"` etc.

Locations:

- `.github/workflows/main.yml:38`
- `.github/workflows/main.yml:41`
- `.github/workflows/main.yml:44`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default token permissions, which may be broader than necessary (e.g., write access to contents and packages on some repository configurations).

Affected files:
- .github/workflows/format-check.yml
- .github/workflows/main.yml
- .github/workflows/test.yml

Locations:

- `.github/workflows/format-check.yml:1`
- `.github/workflows/main.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/format-check.yml, main.yml, and test.yml:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v6 → SHA d23441a..., cachix/install-nix-action@v31 → SHA 630ae54..., cachix/cachix-action@v16 → SHA 3ba601f...

2. script-injection: Moved the three ${{ steps.test.outputs.* }} expressions in main.yml into step-level env: blocks (STATUS, WEB_URL, ARTIFACTS_DOWNLOADED) and updated the run: commands to reference plain env vars.

3. missing-permissions: Added `permissions: {}` at the top level of all three workflow files to enforce least-privilege token access.

