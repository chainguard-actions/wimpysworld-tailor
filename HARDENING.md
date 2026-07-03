<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wimpysworld--tailor/v0.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Install tailor' step directly interpolates GitHub Actions expressions inside the run: shell command string, violating sub-rule (a). Specifically: (1) `case "${{ runner.os }}-${{ runner.arch }}" in` interpolates runner context values directly into the shell; (2) `url="https://github.com/${ACTION_REPO}/releases/download/${{ steps.version.outputs.tag }}/tailor-${os}-${arch}"` interpolates a step output directly into a URL string in the shell; (3) `echo "Installed tailor ${{ steps.version.outputs.version }} for ${os}/${arch}"` interpolates another step output directly into the shell. Any ${{ }} expression inside a run: block is a script-injection risk as the value is substituted before the shell parses the command. These should be moved to env: variables and referenced as $ENV_VAR.

Locations:

- `action.yml:64`
- `action.yml:73`
- `action.yml:76`

### github-env-injection (severity: high)

The 'Determine version' step writes the variable `v` to $GITHUB_OUTPUT without sanitization. When `inputs.version` is provided, `v` is set directly from the env var `INPUT_VERSION` (which holds `${{ inputs.version }}`), a caller-controlled value. The writes `echo "tag=${v}" >> "$GITHUB_OUTPUT"` and `echo "version=${v#v}" >> "$GITHUB_OUTPUT"` do not apply the required sanitization step (`printf '%s' "$v" | tr -d '\n\r'`) before writing, allowing a newline-injection attack to poison GITHUB_OUTPUT with attacker-controlled key-value pairs.

Locations:

- `action.yml:53`
- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings in action.yml:

1. script-injection (lines 64, 73, 76): Moved all ${{ }} expressions out of the 'Install tailor' run: block into the step's env: block. Added RUNNER_OS (${{ runner.os }}), RUNNER_ARCH (${{ runner.arch }}), TAILOR_TAG (${{ steps.version.outputs.tag }}), and TAILOR_VERSION (${{ steps.version.outputs.version }}) as env vars. The shell script now references these as plain environment variables.

2. github-env-injection (lines 53, 54): Added sanitization in the 'Determine version' step before writing to $GITHUB_OUTPUT. The version value `v` is now sanitized via `safe_v=$(printf '%s' "${v}" | tr -d '\n\r')` to strip newlines/carriage-returns, and `safe_v` is used in both GITHUB_OUTPUT writes to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Install tailor' step of action.yml (line 79). The `RUNNER_TEMP` environment variable was being written directly to `$GITHUB_PATH` without sanitization. Added a sanitization step: `safe_runner_temp=$(printf '%s' "${RUNNER_TEMP}" | tr -d '\n\r')` and then used `echo "${safe_runner_temp}" >> "$GITHUB_PATH"` to prevent newline injection attacks that could add arbitrary entries to `$GITHUB_PATH`.

