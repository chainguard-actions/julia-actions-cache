<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v3.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `julia-actions/setup-julia@v2` — a mutable tag reference — in 6 steps across multiple jobs. If the tag is moved to a different commit, the action could execute arbitrary code. All `uses:` references should be pinned to a full 40-character commit SHA.

Locations:

- `.github/workflows/CI.yml:87`
- `.github/workflows/CI.yml:107`
- `.github/workflows/CI.yml:279`
- `.github/workflows/CI.yml:310`
- `.github/workflows/CI.yml:349`
- `.github/workflows/CI.yml:375`

### script-injection (severity: high)

Sub-rule (a): Five `run:` blocks in CI.yml directly interpolate GitHub Actions expressions into shell commands. The offending pattern is: `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"`. The expressions `${{ needs.generate-prefix.outputs.cache-prefix }}` (a step output) and `${{ github.job }}` are substituted into the shell string before the shell parses it, allowing an attacker who can influence these values to inject arbitrary shell commands.

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:218`
- `.github/workflows/CI.yml:278`
- `.github/workflows/CI.yml:348`

### github-env-injection (severity: high)

Five `run:` blocks write values derived from untrusted GitHub Actions expressions directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The pattern `echo "cache-name=${{ needs.generate-prefix.outputs.cache-prefix }}-${{ github.job }}" >>"$GITHUB_OUTPUT"` writes `steps.*.outputs.*` and `github.*` context values unsanitized. A newline embedded in these values could inject additional key=value pairs into GITHUB_OUTPUT, poisoning downstream step outputs.

Locations:

- `.github/workflows/CI.yml:86`
- `.github/workflows/CI.yml:155`
- `.github/workflows/CI.yml:218`
- `.github/workflows/CI.yml:278`
- `.github/workflows/CI.yml:348`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 6 occurrences of `julia-actions/setup-julia@v2` by pinning to SHA `4c0cb0fce8556fdb04a90347310e5db8b1f98fb9 # v2`. Fixed all 5 `Set cache-name` steps by moving `${{ needs.generate-prefix.outputs.cache-prefix }}` and `${{ github.job }}` expressions into step `env:` blocks as `CACHE_PREFIX` and `JOB_NAME`, then sanitizing with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`. The README.md reference to `julia-actions/setup-julia@v2` was left as-is since it is documentation, not an executable workflow.

