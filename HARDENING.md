<!-- markdownlint-disable -->

# Hardening Report: digital-blueprint--gitlab-pipeline-trigger-action/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **digital-blueprint--gitlab-pipeline-trigger-action/v1.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v4`, which is a mutable tag reference rather than a pinned full 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the action will silently execute different code. Pin to a specific commit SHA instead, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:15`
- `.github/workflows/test.yml:12`

### script-injection (severity: high)

Two `run:` steps in main.yml directly interpolate `${{ steps.test.outputs.status }}` and `${{ steps.test.outputs.web_url }}` inside shell command strings (sub-rule a). These expressions are expanded by the GitHub Actions template engine before the shell ever sees the string, so a malicious value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) returned by the upstream pipeline could execute arbitrary commands. Replace with environment variables: set `STATUS: ${{ steps.test.outputs.status }}` in an `env:` block and reference `"$STATUS"` in the shell script instead.

Locations:

- `.github/workflows/main.yml:24`
- `.github/workflows/main.yml:26`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no job within them declares job-level `permissions:`. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` on `contents`). Add a top-level `permissions: {}` block and grant only the minimum scopes required.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:
1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` (with `# v4` comment for readability) in both `.github/workflows/main.yml` and `.github/workflows/test.yml`.
2. **script-injection**: In `main.yml`, moved `${{ steps.test.outputs.status }}` and `${{ steps.test.outputs.web_url }}` out of `run:` shell strings into `env:` blocks as `STATUS` and `WEB_URL` respectively, then referenced them as plain shell variables (`$STATUS`, `$WEB_URL`).
3. **missing-permissions**: Added `permissions: {}` top-level block to both `main.yml` and `test.yml` to enforce least-privilege by default.

