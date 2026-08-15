<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in CI.yml directly interpolate `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` expressions inside shell commands. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, allowing an attacker to inject shell metacharacters. The offending pattern appears in five separate jobs:
- test-save (Set cache-name step): `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"`
- test-save-nomatrix (Set cache-name step): same pattern
- test-save-cloned-registry (Set cache-name step): same pattern
- test-save-on-failure (Set cache-name step): same pattern
- test-no-save-on-failure (Set cache-name step): same pattern
Fix: move the values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:220`
- `.github/workflows/CI.yml:285`
- `.github/workflows/CI.yml:355`

### github-env-injection (severity: high)

Multiple `run:` steps write values derived from untrusted inputs (`needs.*.outputs.*` and `github.*`) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The pattern `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"` appears in five jobs (test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, test-no-save-on-failure). A value containing newlines could inject additional key=value pairs into the output file, potentially overwriting other outputs. Fix: sanitize the values before writing, e.g. `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` then `echo "cache-name=$safe" >>"$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:220`
- `.github/workflows/CI.yml:285`
- `.github/workflows/CI.yml:355`

### unpinned-uses (severity: high)

The action `julia-actions/setup-julia` is referenced 6 times in CI.yml using the mutable tag `@v3` instead of a full 40-character commit SHA. A mutable tag can be silently moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. All occurrences should be pinned to a specific SHA, e.g. `julia-actions/setup-julia@<40-char-sha> # v3`.

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:156`
- `.github/workflows/CI.yml:286`
- `.github/workflows/CI.yml:356`
- `.github/workflows/CI.yml:392`
- `.github/workflows/CI.yml:430`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 'Set cache-name' steps in CI.yml (test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, test-no-save-on-failure) by: (1) moving ${{ needs.generate-prefix.outputs.cache-prefix }} and ${{ github.job }} into env: variables to prevent script injection, (2) sanitizing values with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent env injection. Also pinned all 6 occurrences of julia-actions/setup-julia@v3 to the full commit SHA fa02766e078afaaf09b14210362cee14137e6a32.

