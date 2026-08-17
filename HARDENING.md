<!-- markdownlint-disable -->

# Hardening Report: digital-blueprint--gitlab-pipeline-trigger-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **digital-blueprint--gitlab-pipeline-trigger-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in main.yml directly interpolate `steps.*.outputs.*` expressions inside shell commands (sub-rule a). The expressions `${{ steps.test.outputs.status }}` and `${{ steps.test.outputs.web_url }}` are substituted by the Actions template engine before the shell ever sees the string, allowing an attacker who can influence step outputs to inject arbitrary shell commands. Fix: move the values into `env:` variables and reference them as quoted shell variables, e.g. `env:\n  STATUS: ${{ steps.test.outputs.status }}\nrun: echo "The status was $STATUS"`.

Locations:

- `.github/workflows/main.yml:23`
- `.github/workflows/main.yml:25`

### missing-permissions (severity: medium)

Neither `.github/workflows/main.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` key, and neither of their jobs declares a job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (often `write-all` for private repos), granting unnecessarily broad token access. Add a top-level `permissions: {}` or minimal specific scopes (e.g. `contents: read`) to each workflow.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script injection in main.yml by moving `${{ steps.test.outputs.status }}` and `${{ steps.test.outputs.web_url }}` out of `run:` shell strings into `env:` blocks (STATUS and WEB_URL), referenced as plain shell variables. Added `permissions: {}` at the top level of both main.yml and test.yml to enforce least-privilege token access.

