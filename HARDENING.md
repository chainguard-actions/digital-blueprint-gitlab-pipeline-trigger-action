<!-- markdownlint-disable -->

# Hardening Report: digital-blueprint--gitlab-pipeline-trigger-action/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **digital-blueprint--gitlab-pipeline-trigger-action/v1.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference GitHub Actions using mutable tags instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: main.yml — `actions/checkout@v5`; format-check.yml — `actions/checkout@v5`, `cachix/install-nix-action@v31`, `cachix/cachix-action@v16`; test.yml — `actions/checkout@v5`.

Locations:

- `.github/workflows/main.yml:16`
- `.github/workflows/format-check.yml:18`
- `.github/workflows/format-check.yml:20`
- `.github/workflows/format-check.yml:23`
- `.github/workflows/test.yml:13`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/format-check.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Three `run:` steps in main.yml directly interpolate `${{ steps.test.outputs.* }}` expressions inside shell command strings (sub-rule a). These expressions flow through YAML template substitution before the shell processes them, allowing a malicious pipeline output to inject arbitrary shell commands. Offending lines: `run: echo "The status was ${{ steps.test.outputs.status }}"`, `run: echo "The web_url was ${{ steps.test.outputs.web_url }}"`, `run: echo "The artifacts_downloaded was ${{ steps.test.outputs.artifacts_downloaded }}"`.

Locations:

- `.github/workflows/main.yml:40`
- `.github/workflows/main.yml:43`
- `.github/workflows/main.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/main.yml, format-check.yml, and test.yml:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs (actions/checkout@v5 → fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09, cachix/install-nix-action@v31 → 630ae543ea3a38a9a4166f03376c02c50f408342, cachix/cachix-action@v16 → 3ba601ff5bbb07c7220846facfa2cd81eeee15a1). Original tags preserved as inline comments.

2. missing-permissions: Added `permissions: {}` top-level block to all three workflow files.

3. script-injection: In main.yml, moved all three `${{ steps.test.outputs.* }}` expressions out of `run:` shell strings into step-level `env:` blocks (STATUS, WEB_URL, ARTIFACTS_DOWNLOADED), referencing them as plain env vars in the shell commands.

