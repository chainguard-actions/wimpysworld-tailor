<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wimpysworld--tailor/v0.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Install tailor' step directly interpolates GitHub Actions expressions inside a `run:` shell command string. Specifically: `case "${{ runner.os }}-${{ runner.arch }}" in` (line 62), `echo "::error::Unsupported platform: ${{ runner.os }}-${{ runner.arch }}"` (line 68), `url="https://github.com/${ACTION_REPO}/releases/download/${{ steps.version.outputs.tag }}/tailor-${os}-${arch}"` (line 72), and `echo "Installed tailor ${{ steps.version.outputs.version }} for ${os}/${arch}"` (line 76). Any `${{ ... }}` expression interpolated directly into a `run:` block — including `runner.*` and `steps.*.outputs.*` — is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yml:62`
- `action.yml:68`
- `action.yml:72`
- `action.yml:76`

### github-env-injection (severity: high)

The 'Determine version' step writes variable `v` to `$GITHUB_OUTPUT` without sanitization. `v` is derived from `INPUT_VERSION` (mapped from `inputs.version`, a user-controlled input) via `v="${INPUT_VERSION}"`. The writes `echo "tag=${v}" >> "$GITHUB_OUTPUT"` and `echo "version=${v#v}" >> "$GITHUB_OUTPUT"` do not apply the required sanitization pipeline (`printf '%s' "$v" | tr -d '\n\r'`) before writing to the special environment file, allowing a newline-injection attack that could set arbitrary output variables.

Locations:

- `action.yml:51`
- `action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed script-injection in the 'Install tailor' step by moving all four ${{ ... }} expressions (${{ runner.os }}, ${{ runner.arch }}, ${{ steps.version.outputs.tag }}, ${{ steps.version.outputs.version }}) into the step's env: block as RUNNER_OS, RUNNER_ARCH, TAILOR_TAG, and TAILOR_VERSION, then referencing them as plain shell variables in the run: script. Fixed github-env-injection in the 'Determine version' step by sanitizing the version value with `safe_v=$(printf '%s' "${v}" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, preventing newline-injection attacks.

