<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

CI.yml uses `julia-actions/setup-julia@v2` in 6 places — a mutable tag reference rather than a pinned 40-character commit SHA. This allows supply-chain attacks if the tag is moved or the repository is compromised. All occurrences should be replaced with a full SHA pin (e.g. `julia-actions/setup-julia@<40-hex-sha> # v2`).

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:139`
- `.github/workflows/CI.yml:308`
- `.github/workflows/CI.yml:332`
- `.github/workflows/CI.yml:367`
- `.github/workflows/CI.yml:391`

### script-injection (severity: high)

Sub-rule (a): Five `run:` steps in CI.yml directly interpolate GitHub Actions expressions inside shell command strings. Specifically, `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` are embedded directly in `echo` commands. Per the check rules, any `${{ ... }}` expression inside a `run:` block is a script-injection finding regardless of which context it reads from. The offending pattern in each step is: `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"`. These values should be moved to `env:` variables and the shell expansions double-quoted.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:186`
- `.github/workflows/CI.yml:255`
- `.github/workflows/CI.yml:307`
- `.github/workflows/CI.yml:366`

### github-env-injection (severity: high)

Five `run:` steps write values derived from `${{ needs.generate-prefix.outputs.cache-prefix }}` (a `needs.*.outputs.*` value) and `${{ github.job }}` (a `github.*` value) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The pattern `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"` appears in the 'Set cache-name' steps of jobs: test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, and test-no-save-on-failure. A newline injected into either context value could poison subsequent steps that read from GITHUB_OUTPUT.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:186`
- `.github/workflows/CI.yml:255`
- `.github/workflows/CI.yml:307`
- `.github/workflows/CI.yml:366`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all findings in .github/workflows/CI.yml:
1. unpinned-uses: Replaced all 6 occurrences of `julia-actions/setup-julia@v2` with the pinned SHA `julia-actions/setup-julia@4c0cb0fce8556fdb04a90347310e5db8b1f98fb9 # v2`.
2. script-injection + github-env-injection: In all 5 'Set cache-name' steps (jobs: test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, test-no-save-on-failure), moved `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` into `env:` block variables (`CACHE_PREFIX` and `JOB_NAME`), then sanitized each with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

