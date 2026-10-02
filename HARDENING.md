<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **julia-actions--cache/v2.1.0** was hardened automatically. 16 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The `paths` step directly interpolates `${{ inputs.depot }}`, `${{ inputs.cache-artifacts }}`, `${{ inputs.cache-packages }}`, `${{ inputs.cache-registries }}`, `${{ inputs.cache-compiled }}`, `${{ inputs.cache-scratchspaces }}`, and `${{ inputs.cache-logs }}` inside `run:` shell commands. These expressions are substituted by the Actions runner before the shell sees them, allowing an attacker-controlled input to inject arbitrary shell metacharacters. Example offending lines: `if [ -n "${{ inputs.depot }}" ]; then` and `depot="${{ inputs.depot }}"`.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Rule (a): The `Generate Keys` step directly interpolates `${{ inputs.include-matrix }}`, `${{ inputs.cache-name }}`, `${{ runner.os }}`, `${{ github.run_id }}`, and `${{ github.run_attempt }}` inside `run:` shell commands. Example offending lines: `if [ "${{ inputs.include-matrix }}" == "true" ]`, `restore_key="${{ inputs.cache-name }};os=${{ runner.os }};${matrix_key}"`, and `key="${restore_key}run_id=${{ github.run_id }};run_attempt=${{ github.run_attempt }}"`.

Locations:

- `action.yml:107`

### script-injection (severity: high)

Rule (a): The `make depot if not restored, then list depot directory sizes` step directly interpolates `${{ steps.paths.outputs.depot }}` inside `run:` shell commands. Offending lines: `mkdir -p ${{ steps.paths.outputs.depot }}` and `du -shc ${{ steps.paths.outputs.depot }}/* || true`. The `steps.*.outputs.*` context is workflow-controllable and must not be interpolated directly into shell.

Locations:

- `action.yml:148`

### script-injection (severity: high)

Rule (a): The `Update any cached registries` step directly interpolates `${{ steps.paths.outputs.depot }}` inside `run:` shell commands. Offending line: `if [ -d "${{ steps.paths.outputs.depot }}/registries" ] && [ -n "$(ls -A "${{ steps.paths.outputs.depot }}/registries")" ]; then`. The `steps.*.outputs.*` context is workflow-controllable and must not be interpolated directly into shell.

Locations:

- `action.yml:160`

### github-env-injection (severity: high)

The `paths` step writes the `depot` variable to `$GITHUB_OUTPUT` via `echo "depot=$depot" | tee -a "$GITHUB_OUTPUT"` without sanitization. The `depot` variable is derived directly from `${{ inputs.depot }}` (an attacker-controlled input) interpolated earlier in the same script. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into the output file which could poison subsequent steps' environment.

Locations:

- `action.yml:79`

### github-env-injection (severity: high)

The `Generate Keys` step writes `restore_key` and `key` to `$GITHUB_OUTPUT` via `echo "restore-key=${restore_key}" >> $GITHUB_OUTPUT` and `echo "key=${key}" >> $GITHUB_OUTPUT` without sanitization. Both variables are derived from `${{ inputs.cache-name }}` (an attacker-controlled input) interpolated directly into the shell string. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the writes, allowing newline injection into the output file.

Locations:

- `action.yml:114`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in action.yml:

1. paths step: Moved all ${{ inputs.* }} expressions (depot, cache-artifacts, cache-packages, cache-registries, cache-compiled, cache-scratchspaces, cache-logs) to env: block as INPUT_DEPOT, INPUT_CACHE_ARTIFACTS, etc. Added newline sanitization (printf '%s' | tr -d '\n\r') before writing depot to $GITHUB_OUTPUT.

2. Generate Keys step: Moved ${{ inputs.include-matrix }}, ${{ inputs.cache-name }}, ${{ runner.os }}, ${{ github.run_id }}, ${{ github.run_attempt }} to env: block as INPUT_INCLUDE_MATRIX, INPUT_CACHE_NAME, INPUT_RUNNER_OS, INPUT_RUN_ID, INPUT_RUN_ATTEMPT. Added newline sanitization for restore_key and key before writing to $GITHUB_OUTPUT.

3. make depot step: Moved ${{ steps.paths.outputs.depot }} to env: block as DEPOT_PATH_VAR, referenced as "$DEPOT_PATH_VAR" in shell.

4. Update any cached registries step: Moved ${{ steps.paths.outputs.depot }} to env: block as DEPOT_PATH_VAR, referenced as "$DEPOT_PATH_VAR" in shell.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. Sub-rule (a) - Lines 196 and 210: Moved `${{ github.repository }}`, `${{ steps.keys.outputs.restore-key }}`, `${{ github.ref }}`, and `${{ inputs.delete-old-caches != 'required' }}` out of the `post:` field in both `pyTooling/Actions/with-post-step` steps (non-Windows and Windows) into the steps' `env:` blocks as `POST_REPOSITORY`, `POST_RESTORE_KEY`, `POST_REF`, and `POST_DELETE_NON_REQUIRED`. The `post:` commands now reference these as shell variables (`"$POST_REPOSITORY"` etc. on Linux/macOS, `"%POST_REPOSITORY%"` etc. on Windows cmd).

2. Sub-rule (b) - Line 68: Added double quotes around `$JULIA_DEPOT_PATH` and `$PATH_DELIMITER` in the unquoted shell expansion `depot=$(echo $JULIA_DEPOT_PATH | cut -d$PATH_DELIMITER -f1)`, changing it to `depot=$(echo "$JULIA_DEPOT_PATH" | cut -d"$PATH_DELIMITER" -f1)` to prevent word splitting and glob expansion on the untrusted environment variable.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the `hit` step in action.yml (line 232) to sanitize the CACHE_HIT value before writing to $GITHUB_OUTPUT. Changed from a direct `echo "cache-hit=$CACHE_HIT" >> $GITHUB_OUTPUT` to a two-step approach: first strip newlines/carriage-returns with `safe=$(printf '%s' "$CACHE_HIT" | tr -d '\n\r')`, then write `echo "cache-hit=$safe" >> "$GITHUB_OUTPUT"`. Also added proper quoting around `$GITHUB_OUTPUT`.

