<!-- markdownlint-disable -->

# Hardening Report: digital-blueprint--gitlab-pipeline-trigger-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **digital-blueprint--gitlab-pipeline-trigger-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in `.github/workflows/main.yml` directly interpolate `${{ ... }}` expressions inside shell commands (sub-rule a). Line 29: `run: echo "The status was ${{ steps.test.outputs.status }}"` and line 31: `run: echo "The web_url was ${{ steps.test.outputs.web_url }}"`. The `steps.*.outputs.*` context is workflow-controllable and is substituted by the Actions template engine before the shell ever sees the string, enabling command injection if the output value contains shell metacharacters. These values should be passed via an `env:` variable and the shell expansion double-quoted instead.

Locations:

- `.github/workflows/main.yml:29`
- `.github/workflows/main.yml:31`

### missing-permissions (severity: medium)

Neither `.github/workflows/main.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` key, and no individual job within either file declares a `permissions:` key. Without explicit permissions, workflows run with the repository's default token permissions (which may be `write-all` on some repositories), granting broader access than necessary. A minimal `permissions:` block (e.g. `permissions: {}` or only the specific scopes required) should be added at the top level of each workflow file.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/main.yml by moving ${{ steps.test.outputs.status }} and ${{ steps.test.outputs.web_url }} into env: blocks (STATUS and WEB_URL) and referencing them as plain shell variables. Added top-level `permissions: {}` to both .github/workflows/main.yml and .github/workflows/test.yml to address missing-permissions finding.

