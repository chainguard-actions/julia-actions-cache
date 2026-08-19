<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in CI.yml directly interpolate GitHub Actions expressions inside shell commands. Specifically, `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` are interpolated directly into the shell string: `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"`. Any expression inside `${{ }}` is substituted by the Actions runner before the shell sees the command, allowing an attacker who controls the output of `generate-prefix` to inject arbitrary shell metacharacters. This pattern appears in 5 separate jobs (test-save, test-save-nomatrix, test-save-cloned-registry, test-save-on-failure, test-no-save-on-failure).

Locations:

- `.github/workflows/CI.yml:79`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:222`
- `.github/workflows/CI.yml:290`
- `.github/workflows/CI.yml:358`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted expression sources (`${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}`) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending pattern is: `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"`. A newline embedded in the cache-prefix output could inject additional key=value pairs into the GitHub output environment. This appears in 5 jobs.

Locations:

- `.github/workflows/CI.yml:79`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:222`
- `.github/workflows/CI.yml:290`
- `.github/workflows/CI.yml:358`

### unpinned-uses (severity: high)

The action `julia-actions/setup-julia` is referenced 6 times using the mutable tag `@v2` instead of a full 40-character commit SHA. A mutable tag can be silently moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. All references should be pinned to a specific SHA, e.g. `julia-actions/setup-julia@<40-char-sha> # v2`.

Locations:

- `.github/workflows/CI.yml:80`
- `.github/workflows/CI.yml:120`
- `.github/workflows/CI.yml:291`
- `.github/workflows/CI.yml:331`
- `.github/workflows/CI.yml:359`
- `.github/workflows/CI.yml:399`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 instances of script-injection and github-env-injection in CI.yml by moving ${{ needs.generate-prefix.outputs.cache-prefix }} and ${{ github.job }} expressions into step-level env: blocks (as CACHE_PREFIX and JOB_NAME), then sanitizing with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Pinned all 6 occurrences of julia-actions/setup-julia@v2 to the full commit SHA 4c0cb0fce8556fdb04a90347310e5db8b1f98fb9 with # v2 comment preserved.

