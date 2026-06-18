<!-- markdownlint-disable -->

# Hardening Report: julia-actions--cache/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **julia-actions--cache/v2.1.0** was hardened automatically. 12 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell commands, violating rule (a). Before the shell executes the script, the Actions runner substitutes these expressions verbatim, so an attacker-controlled value (e.g. inputs.depot, inputs.cache-name, inputs.cache-artifacts, etc.) containing shell metacharacters (`;`, `$(...)`, `|`, backticks) would be executed by the shell.

Affected lines and expressions:
- `id: paths` step: `if [ -n "${{ inputs.depot }}" ]` and `depot="${{ inputs.depot }}"` — inputs.depot is user-controlled
- `id: paths` step: `[ "${{ inputs.cache-artifacts }}" = "true" ]`, `[ "${{ inputs.cache-packages }}" = "true" ]`, `[ "${{ inputs.cache-registries }}" = "true" ]`, `[ "${{ inputs.cache-compiled }}" = "true" ]`, `[ "${{ inputs.cache-scratchspaces }}" = "true" ]`, `[ "${{ inputs.cache-logs }}" = "true" ]` — all user-controlled inputs interpolated directly
- `id: keys` step: `if [ "${{ inputs.include-matrix }}" == "true" ]` and `restore_key="${{ inputs.cache-name }};os=${{ runner.os }};..."` and `key="...run_id=${{ github.run_id }};run_attempt=${{ github.run_attempt }}"` — inputs and github context interpolated directly
- `make depot` step: `mkdir -p ${{ steps.paths.outputs.depot }}` and `du -shc ${{ steps.paths.outputs.depot }}/*` — step output interpolated directly
- `Update any cached registries` step: `if [ -d "${{ steps.paths.outputs.depot }}/registries" ]` — step output interpolated directly

Fix: Move all ${{ ... }} values into env: variables and reference them as quoted shell variables (e.g. "$DEPOT") inside run: blocks.

Locations:

- `action.yml:65`
- `action.yml:66`
- `action.yml:76`
- `action.yml:79`
- `action.yml:83`
- `action.yml:88`
- `action.yml:91`
- `action.yml:94`
- `action.yml:104`
- `action.yml:108`
- `action.yml:113`
- `action.yml:118`
- `action.yml:122`
- `action.yml:148`
- `action.yml:149`
- `action.yml:158`
- `action.yml:159`

### github-env-injection (severity: high)

Two run: blocks write values derived from user-controlled inputs to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `id: paths` step: `echo "depot=$depot" | tee -a "$GITHUB_OUTPUT"` — `$depot` is derived from `${{ inputs.depot }}` (a user-controlled input). A newline in inputs.depot would allow injecting additional key=value pairs into GITHUB_OUTPUT.

2. `id: keys` step: `echo "restore-key=${restore_key}" >> $GITHUB_OUTPUT` and `echo "key=${key}" >> $GITHUB_OUTPUT` — `restore_key` and `key` are constructed from `${{ inputs.cache-name }}` (user-controlled) without sanitization. A newline embedded in inputs.cache-name would allow injecting arbitrary key=value pairs into GITHUB_OUTPUT.

Fix: Apply `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before each write to $GITHUB_OUTPUT.

Locations:

- `action.yml:78`
- `action.yml:120`
- `action.yml:121`

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

Fixed all script-injection, github-env-injection, and static-inline-injection findings in action.yml:

1. **id: paths step**: Moved `${{ inputs.depot }}`, `${{ inputs.cache-artifacts }}`, `${{ inputs.cache-packages }}`, `${{ inputs.cache-registries }}`, `${{ inputs.cache-compiled }}`, `${{ inputs.cache-scratchspaces }}`, `${{ inputs.cache-logs }}` into env: block as INPUT_DEPOT, INPUT_CACHE_ARTIFACTS, etc. Applied `printf '%s' "$depot" | tr -d '\n\r'` sanitization before writing depot to GITHUB_OUTPUT.

2. **id: keys step**: Moved `${{ inputs.include-matrix }}`, `${{ inputs.cache-name }}`, `${{ runner.os }}` into env: block. Used built-in `$GITHUB_RUN_ID` and `$GITHUB_RUN_ATTEMPT` env vars instead of `${{ github.run_id }}` and `${{ github.run_attempt }}`. Applied `printf '%s' ... | tr -d '\n\r'` sanitization to cache-name, restore-key, and key before writing to GITHUB_OUTPUT.

3. **make depot step**: Moved `${{ steps.paths.outputs.depot }}` into env: block as DEPOT_PATH_OUTPUT, referenced as `"$DEPOT_PATH_OUTPUT"` in shell.

4. **Update any cached registries step**: Moved `${{ steps.paths.outputs.depot }}` into env: block as DEPOT_PATH_OUTPUT, referenced as `"$DEPOT_PATH_OUTPUT"` in shell.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in both the non-Windows and Windows pyTooling/Actions/with-post-step steps (action.yml lines 178 and 192). Moved all ${{ github.repository }}, ${{ steps.keys.outputs.restore-key }}, ${{ github.ref }}, and ${{ inputs.delete-old-caches != 'required' }} expressions out of the post: command strings and into the env: block of each step. The post: commands now reference these values as shell environment variables ($CACHE_REPOSITORY, $CACHE_RESTORE_KEY, $CACHE_REF, $CACHE_DELETE_NON_REQUIRED for bash; %CACHE_REPOSITORY%, etc. for Windows cmd), preventing attacker-controlled values from being interpolated directly into shell command strings.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions on line 68 of action.yml. Changed `depot=$(echo $JULIA_DEPOT_PATH | cut -d$PATH_DELIMITER -f1)` to `depot=$(echo "$JULIA_DEPOT_PATH" | cut -d"$PATH_DELIMITER" -f1)`. Both `$JULIA_DEPOT_PATH` (an inherited process env var that can be set by calling workflows) and `$PATH_DELIMITER` (sourced from a `${{ }}` expression) are now double-quoted, preventing shell metacharacters from being interpreted as shell commands.

