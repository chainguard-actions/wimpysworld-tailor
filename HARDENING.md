<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wimpysworld--tailor/v0.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Install tailor' step in action.yml directly interpolates ${{ }} expressions inside the run: shell script, violating sub-rule (a). The following expressions are embedded directly in shell code before the shell ever sees them: `${{ runner.os }}` and `${{ runner.arch }}` in the case statement (line 64) and error echo (line 70); `${{ steps.version.outputs.tag }}` in the URL construction (line 75); and `${{ steps.version.outputs.version }}` in the final echo (line 79). Any ${{ ... }} expression directly inside a run: block is a script-injection risk regardless of context (runner.*, steps.*.outputs.*, etc.). These should be moved to env: variables and referenced as quoted shell variables (e.g. "$RUNNER_OS", "$RUNNER_ARCH", "$STEP_TAG").

Locations:

- `action.yml:64`
- `action.yml:70`
- `action.yml:75`
- `action.yml:79`

### github-env-injection (severity: high)

The 'Determine version' step writes the variable `v` to $GITHUB_OUTPUT (lines 53–54) without the required sanitization step (`printf '%s' "$v" | tr -d '\n\r'`). The value of `v` is derived from the inherited env var `${INPUT_VERSION}` (mapped from `inputs.version`, a caller-controlled input) or `${ACTION_REF}` (mapped from `github.action_ref`, a github context value). Both are untrusted sources. A newline embedded in either value could inject arbitrary key=value pairs into $GITHUB_OUTPUT, allowing an attacker to poison subsequent steps' outputs. The fix is to sanitize before writing: `safe=$(printf '%s' "$v" | tr -d '\n\r')` and then `echo "tag=${safe}" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:53`
- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings in hardened/action/action.yml:

1. github-env-injection: Added sanitization before writing to $GITHUB_OUTPUT in the 'Determine version' step. Values derived from caller-controlled inputs and github context are now sanitized with `printf '%s' "${v}" | tr -d '\n\r'` before being written as `tag` and `version` outputs.

2. script-injection: In the 'Install tailor' step, moved all four ${{ }} expressions out of the run: shell script into the env: block — runner.os → RUNNER_OS, runner.arch → RUNNER_ARCH, steps.version.outputs.tag → STEP_TAG, steps.version.outputs.version → STEP_VERSION. The shell script now references these as plain environment variables, preventing any expression value from being interpreted as shell code.

