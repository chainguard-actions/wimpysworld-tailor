<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wimpysworld--tailor/v0.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Determine version' writes a value taken from inputs.version (through env INPUT_VERSION into shell variable v) to $GITHUB_OUTPUT without sanitizing it first: `echo "tag=${v}" >> "$GITHUB_OUTPUT"` and `echo "version=${v#v}" >> "$GITHUB_OUTPUT"`. A newline in the input could inject extra outputs. Fix: run `printf '%s' "$v" | tr -d '\n\r'` on the value before writing it.

Locations:

- `action.yml:53`
- `action.yml:54`

### github-env-injection (severity: high)

Step 'Install tailor' writes the inherited process env var RUNNER_TEMP to $GITHUB_PATH without sanitizing it first: `echo "${RUNNER_TEMP}" >> "$GITHUB_PATH"`. In a composite action, inherited env vars count as workflow-controlled, so the value must go through `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:78`

### script-injection (severity: high)

Rule (a) violation in step 'Install tailor': expressions are placed directly inside the run: script. Offending lines: `case "${{ runner.os }}-${{ runner.arch }}" in`, `echo "::error::Unsupported platform: ${{ runner.os }}-${{ runner.arch }}"`, `url="https://github.com/${ACTION_REPO}/releases/download/${{ steps.version.outputs.tag }}/tailor-${os}-${arch}"`, `echo "Installed tailor ${{ steps.version.outputs.version }} for ${os}/${arch}"`. steps.version.outputs.* comes from inputs.version, which the caller controls. Fix: use $RUNNER_OS/$RUNNER_ARCH and pass the step outputs in through env: variables, quoted when used.

Locations:

- `action.yml:64`
- `action.yml:70`
- `action.yml:75`
- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Strip newlines from the version before writing tag/version to GITHUB_OUTPUT and from RUNNER_TEMP before writing GITHUB_PATH. Replaced inline ${{ runner.os/arch }} with $RUNNER_OS/$RUNNER_ARCH and passed steps.version outputs via env (VERSION_TAG, TAILOR_VERSION). Since the action downloads releases from its own repository, ACTION_REPO now points at upstream 'wimpysworld/tailor' and ACTION_REF at upstream_version 'v0.1.1' (the mirror has no releases).

