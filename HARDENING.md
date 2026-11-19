<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wimpysworld--tailor/v0.1.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Determine version' writes values derived from untrusted inputs (INPUT_VERSION from inputs.version, ACTION_REF from github.action_ref, and gh API output) to $GITHUB_OUTPUT without the sanitization step: `echo "tag=${v}" >> "$GITHUB_OUTPUT"` and `echo "version=${v#v}" >> "$GITHUB_OUTPUT"`. If the value contains a newline, extra outputs can be injected. Fix: `safe=$(printf '%s' "$v" | tr -d '\n\r')` before writing.

Locations:

- `action.yml:53`
- `action.yml:54`

### github-env-injection (severity: high)

Step 'Install tailor' writes the inherited process env var RUNNER_TEMP to $GITHUB_PATH without sanitization: `echo "${RUNNER_TEMP}" >> "$GITHUB_PATH"`. Under this check, env vars that the same script did not set count as workflow-controlled. Fix: `safe=$(printf '%s' "$RUNNER_TEMP" | tr -d '\n\r')` before writing.

Locations:

- `action.yml:78`

### script-injection (severity: high)

Rule (a) is violated in step 'Install tailor': GitHub expressions are put straight into the run: script. Offending lines: `case "${{ runner.os }}-${{ runner.arch }}" in`, `echo "::error::Unsupported platform: ${{ runner.os }}-${{ runner.arch }}"`, `url="https://github.com/${ACTION_REPO}/releases/download/${{ steps.version.outputs.tag }}/tailor-${os}-${arch}"`, `echo "Installed tailor ${{ steps.version.outputs.version }} for ${os}/${arch}"`. The steps.version outputs come from inputs.version, so an attacker can control them. Fix: use "$RUNNER_OS"/"$RUNNER_ARCH" and pass the step outputs in through env: variables, quoted where they are used.

Locations:

- `action.yml:64`
- `action.yml:70`
- `action.yml:75`
- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Determine version: the version value now has newlines stripped (tr -d '\n\r') before it is written to GITHUB_OUTPUT. Install tailor: RUNNER_TEMP has newlines stripped before it is written to GITHUB_PATH. The runner.os and runner.arch expressions are replaced with $RUNNER_OS and $RUNNER_ARCH. The steps.version outputs are now passed in through env (VERSION_TAG, VERSION_NUM) and quoted where they are used. Because the mirror has no releases of its own, the release lookup and download now point at upstream: ACTION_REPO is set to 'wimpysworld/tailor' (from action_name) instead of github.action_repository, and ACTION_REF is set to 'v0.1.2' (upstream_version) instead of github.action_ref.

