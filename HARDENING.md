<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action `julia-actions/setup-julia` is referenced with a mutable version tag `@v2` in 6 places in CI.yml instead of a pinned 40-character commit SHA. This allows the action to be silently updated to a different (potentially malicious) version without any change to the workflow file. All occurrences should be pinned to a full SHA, e.g. `julia-actions/setup-julia@<40-char-sha> # v2`.

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:139`
- `.github/workflows/CI.yml:228`
- `.github/workflows/CI.yml:261`
- `.github/workflows/CI.yml:296`
- `.github/workflows/CI.yml:323`

### script-injection (severity: high)

Sub-rule (a): Five `run:` steps in CI.yml directly interpolate GitHub Actions expressions inside shell command strings. The pattern `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >> "$GITHUB_OUTPUT"` embeds `${{ needs.generate-prefix.outputs.cache-prefix }}` (a `needs.*.outputs.*` value) and `${{ github.job }}` (a `github.*` value) directly into the shell command before the shell ever sees it. If either value contains shell metacharacters, they will be interpreted by the shell. Both `needs.*.outputs.*` and `github.*` are in the untrusted-input list and must be passed via an `env:` block and then double-quoted in the shell script.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:148`
- `.github/workflows/CI.yml:227`
- `.github/workflows/CI.yml:260`
- `.github/workflows/CI.yml:295`

### github-env-injection (severity: high)

Five `run:` steps write values derived from untrusted GitHub Actions expressions directly to `$GITHUB_OUTPUT` without sanitization. The pattern `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >> "$GITHUB_OUTPUT"` writes `needs.*.outputs.*` and `github.*` context values directly into the special environment file. A value containing a newline could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs. The required sanitization step (`safe=$(printf '%s' "$VAR" | tr -d '\n\r')`) must be applied before every write to `$GITHUB_OUTPUT`, `$GITHUB_ENV`, or `$GITHUB_PATH` when the source is an untrusted input.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:148`
- `.github/workflows/CI.yml:227`
- `.github/workflows/CI.yml:260`
- `.github/workflows/CI.yml:295`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 6 occurrences of unpinned `julia-actions/setup-julia@v2` by pinning to SHA `4c0cb0fce8556fdb04a90347310e5db8b1f98fb9 # v2`. Fixed all 5 script-injection and github-env-injection issues in 'Set cache-name' steps across test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, and test-no-save-on-failure jobs: moved `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` into `env:` blocks as `CACHE_PREFIX` and `GITHUB_JOB_NAME`, then sanitized with `printf '%s' | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

