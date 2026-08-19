<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v2.1.0** was hardened automatically. 13 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (rule a), allowing script injection. Affected steps:

1. `paths` step: interpolates `${{ inputs.depot }}` directly in shell (e.g. `if [ -n "${{ inputs.depot }}" ]` and `depot="${{ inputs.depot }}"`), and `${{ inputs.cache-artifacts }}`, `${{ inputs.cache-packages }}`, `${{ inputs.cache-registries }}`, `${{ inputs.cache-compiled }}`, `${{ inputs.cache-scratchspaces }}`, `${{ inputs.cache-logs }}` in conditional expressions.

2. `Generate Keys` step: interpolates `${{ inputs.include-matrix }}`, `${{ inputs.cache-name }}`, `${{ runner.os }}`, `${{ github.run_id }}`, `${{ github.run_attempt }}` directly in shell (e.g. `restore_key="${{ inputs.cache-name }};os=${{ runner.os }};..."`).

3. `make depot if not restored` step: interpolates `${{ steps.paths.outputs.depot }}` directly in shell commands (`mkdir -p ${{ steps.paths.outputs.depot }}` and `du -shc ${{ steps.paths.outputs.depot }}/*`).

4. `Update any cached registries` step: interpolates `${{ steps.paths.outputs.depot }}` directly in shell (`if [ -d "${{ steps.paths.outputs.depot }}/registries" ]`).

All of these should be moved to `env:` variables and then referenced as quoted `"$VAR"` in the shell.

Locations:

- `action.yml:61`
- `action.yml:79`
- `action.yml:96`
- `action.yml:107`

### github-env-injection (severity: high)

Two `run:` blocks in action.yml write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. `paths` step: `depot` is set from `${{ inputs.depot }}` (attacker-controlled) and then written unsanitized: `echo "depot=$depot" | tee -a "$GITHUB_OUTPUT"`. A newline in `inputs.depot` could inject additional key=value pairs into GITHUB_OUTPUT.

2. `Generate Keys` step: `restore_key` is built from `${{ inputs.cache-name }}` (attacker-controlled) and written unsanitized: `echo "restore-key=${restore_key}" >> $GITHUB_OUTPUT` and `echo "key=${key}" >> $GITHUB_OUTPUT`. A newline in `inputs.cache-name` could inject additional entries into GITHUB_OUTPUT.

Locations:

- `action.yml:61`
- `action.yml:79`

### unpinned-uses (severity: high)

The workflow file references `julia-actions/setup-julia@v2` using a mutable tag (`v2`) instead of a full 40-character commit SHA. This means the action could be silently updated to a malicious version without any change to the workflow file. The reference appears twice (in the `test-save` and `test-restore` jobs).

Locations:

- `.github/workflows/CI.yml:65`
- `.github/workflows/CI.yml:108`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.depot }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:67`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.depot }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:68`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-artifacts }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-packages }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-registries }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:89`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-compiled }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:97`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-scratchspaces }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:99`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-logs }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:101`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.include-matrix }}" appears directly in run: block of step "Generate Keys"; move to env: map

Locations:

- `action.yml:116`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-name }}" appears directly in run: block of step "Generate Keys"; move to env: map

Locations:

- `action.yml:119`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yml and .github/workflows/CI.yml:

1. **script-injection / static-inline-injection** (action.yml): Moved all ${{ inputs.* }} expressions out of run: blocks into env: blocks. Specifically:
   - `paths` step: INPUT_DEPOT, INPUT_CACHE_ARTIFACTS, INPUT_CACHE_PACKAGES, INPUT_CACHE_REGISTRIES, INPUT_CACHE_COMPILED, INPUT_CACHE_SCRATCHSPACES, INPUT_CACHE_LOGS all moved to env: and referenced as $VAR in shell.
   - `Generate Keys` step: INPUT_INCLUDE_MATRIX, INPUT_CACHE_NAME moved to env:; ${{ runner.os }} moved to RUNNER_OS env var; ${{ github.run_id }} and ${{ github.run_attempt }} replaced with built-in $GITHUB_RUN_ID and $GITHUB_RUN_ATTEMPT env vars.
   - `make depot if not restored` step: ${{ steps.paths.outputs.depot }} moved to STEP_DEPOT env var.
   - `Update any cached registries` step: ${{ steps.paths.outputs.depot }} moved to STEP_DEPOT env var.

2. **github-env-injection** (action.yml): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT for both the `depot` value (paths step) and `restore-key`/`key` values (Generate Keys step).

3. **unpinned-uses** (CI.yml): Pinned both occurrences of `julia-actions/setup-julia@v2` to full SHA `julia-actions/setup-julia@4c0cb0fce8556fdb04a90347310e5db8b1f98fb9 # v2`.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. action.yml non-Windows pyTooling/with-post-step: Moved github.repository, steps.keys.outputs.restore-key, github.ref, and inputs.delete-old-caches expression into env vars (STEP_REPOSITORY, STEP_RESTORE_KEY, STEP_REF, STEP_DELETE_OLD); post: command now references $STEP_* env vars instead of ${{ }} expressions.
2. action.yml Windows pyTooling/with-post-step: Same fix using %STEP_*% Windows cmd syntax.
3. action.yml hit step: Added sanitization (printf '%s' "$CACHE_HIT" | tr -d '\n\r') before writing cache-hit to $GITHUB_OUTPUT.
4. CI.yml three Set cache-name steps (test-save, test-save-nomatrix, test-save-cloned-registry): Moved ${{ needs.generate-prefix.outputs.cache-prefix }} and ${{ github.job }} into env vars (CACHE_PREFIX, JOB_NAME); sanitized the combined value with printf | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in the `paths` step of action.yml. Changed `depot=$(echo $JULIA_DEPOT_PATH | cut -d$PATH_DELIMITER -f1)` to `depot=$(echo "$JULIA_DEPOT_PATH" | cut -d"$PATH_DELIMITER" -f1)`. Both `$JULIA_DEPOT_PATH` (an inherited process env var that could be workflow-controllable) and `$PATH_DELIMITER` (set from a runner context expression) are now properly double-quoted, preventing shell metacharacters in their values from being interpreted by the shell.

