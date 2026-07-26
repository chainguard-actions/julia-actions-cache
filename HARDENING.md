<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Five steps in CI.yml write `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` directly into shell run: blocks that pipe to $GITHUB_OUTPUT. These expressions are evaluated by the YAML template engine before the shell sees them, allowing injection of shell metacharacters. Offending line pattern: echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:168`
- `.github/workflows/CI.yml:212`
- `.github/workflows/CI.yml:258`
- `.github/workflows/CI.yml:314`

### github-env-injection (severity: high)

Five run: steps in CI.yml write values derived from ${{ needs.generate-prefix.outputs.cache-prefix }} and ${{ github.job }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). These are workflow-controlled values (steps outputs and job context) that can contain newlines, enabling environment injection attacks. Offending pattern: echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:168`
- `.github/workflows/CI.yml:212`
- `.github/workflows/CI.yml:258`
- `.github/workflows/CI.yml:314`

### unpinned-uses (severity: high)

The action julia-actions/setup-julia is referenced using a mutable version tag @v3 instead of a full 40-character commit SHA. This means the action could be silently updated to a malicious version without any change to the workflow file, creating a supply-chain attack vector. All 6 occurrences in CI.yml use julia-actions/setup-julia@v3.

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:133`
- `.github/workflows/CI.yml:259`
- `.github/workflows/CI.yml:285`
- `.github/workflows/CI.yml:315`
- `.github/workflows/CI.yml:345`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 script-injection and github-env-injection occurrences in CI.yml by moving ${{ needs.generate-prefix.outputs.cache-prefix }} and ${{ github.job }} into step env: blocks (as CACHE_PREFIX and GITHUB_JOB_NAME), then sanitizing with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Fixed all 6 unpinned julia-actions/setup-julia@v3 references by pinning to full SHA fa02766e078afaaf09b14210362cee14137e6a32 with # v3 comment preserved.

