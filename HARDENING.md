<!-- markdownlint-disable -->

# Hardening Report: wimpysworld--tailor/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wimpysworld--tailor/v0.1.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in the 'Install tailor' step of action.yml. This includes: `case "${{ runner.os }}-${{ runner.arch }}"` (line 64), `echo "::error::Unsupported platform: ${{ runner.os }}-${{ runner.arch }}"` (line 70), `url="https://github.com/${ACTION_REPO}/releases/download/${{ steps.version.outputs.tag }}/tailor-${os}-${arch}"` (line 75), and `echo "Installed tailor ${{ steps.version.outputs.version }} for ${os}/${arch}"` (line 79). Any ${{ ... }} expression directly inside a run: block is a script-injection risk regardless of context (runner.*, steps.*.outputs.*).

Locations:

- `action.yml:64`
- `action.yml:70`
- `action.yml:75`
- `action.yml:79`

### script-injection (severity: high)

Sub-rule (a): In builder.yml, the 'sentinel' job's 'Check results' step directly interpolates `${{ toJSON(needs.*.result) }}` inside a run: shell command: `results='${{ toJSON(needs.*.result) }}'`. This allows workflow-controlled data to be injected into the shell before quoting.

Locations:

- `.github/workflows/builder.yml:130`

### script-injection (severity: high)

Sub-rule (a): In builder.yml, the 'release' job's 'Log in to GHCR' step (inside nick-fields/retry command:) directly interpolates `${{ secrets.GITHUB_TOKEN }}` and `${{ github.actor }}` inside a run: shell command: `echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin`. Both expressions are interpolated directly into the shell string.

Locations:

- `.github/workflows/builder.yml:192`

### script-injection (severity: high)

Sub-rule (a): In builder.yml, the 'update-tag' job's 'Update floating major version tag' step directly interpolates `${{ github.event.release.tag_name }}` inside a run: shell command: `tag="${{ github.event.release.tag_name }}"`. This is attacker-controlled data (release tag name) injected directly into the shell.

Locations:

- `.github/workflows/builder.yml:228`

### github-env-injection (severity: high)

In action.yml's 'Determine version' step, the shell variable `v` is derived from `${INPUT_VERSION}` (mapped from `inputs.version`) or `${ACTION_REF}` (mapped from `github.action_ref`) — both untrusted, caller-controlled inputs. The value is written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "tag=${v}" >> "$GITHUB_OUTPUT"` and `echo "version=${v#v}" >> "$GITHUB_OUTPUT"`. A newline-injection attack could allow an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:53`
- `action.yml:54`

### unpinned-uses (severity: high)

All uses: references in workflow files use mutable tags, version strings, or branch names instead of immutable 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references in builder.yml: actions/checkout@v6, actions/setup-go@v6, golangci/golangci-lint-action@v9, freerangebytes/setup-actionlint@v0.1.1, robherley/go-test-action@v0, actions/dependency-review-action@v4, golang/govulncheck-action@master, github/codeql-action/upload-sarif@v4.32.6, docker/setup-buildx-action@v4, nick-fields/retry@v3, DeterminateSystems/nix-installer-action@v21, sigstore/cosign-installer@v4.1.0, goreleaser/goreleaser-action@v7, DeterminateSystems/determinate-nix-action@v3, DeterminateSystems/update-flake-lock@v28. Failing references in tailor-automerge.yml: dependabot/fetch-metadata@v2. Failing references in tailor.yml: actions/checkout@v6, wimpysworld/tailor@v0, peter-evans/create-pull-request@v8, DeterminateSystems/determinate-nix-action@v3, DeterminateSystems/update-flake-lock@v28.

Locations:

- `.github/workflows/builder.yml:27`
- `.github/workflows/builder.yml:28`
- `.github/workflows/builder.yml:40`
- `.github/workflows/builder.yml:47`
- `.github/workflows/builder.yml:48`
- `.github/workflows/builder.yml:62`
- `.github/workflows/builder.yml:63`
- `.github/workflows/builder.yml:67`
- `.github/workflows/builder.yml:100`
- `.github/workflows/builder.yml:101`
- `.github/workflows/builder.yml:109`
- `.github/workflows/builder.yml:113`
- `.github/workflows/builder.yml:175`
- `.github/workflows/builder.yml:180`
- `.github/workflows/builder.yml:183`
- `.github/workflows/builder.yml:186`
- `.github/workflows/builder.yml:196`
- `.github/workflows/builder.yml:200`
- `.github/workflows/builder.yml:204`
- `.github/workflows/tailor-automerge.yml:14`
- `.github/workflows/tailor.yml:14`
- `.github/workflows/tailor.yml:17`
- `.github/workflows/tailor.yml:20`
- `.github/workflows/tailor.yml:32`
- `.github/workflows/tailor.yml:37`
- `.github/workflows/tailor.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings across action.yml, .github/workflows/builder.yml, .github/workflows/tailor-automerge.yml, and .github/workflows/tailor.yml:

1. script-injection (action.yml): Moved runner.os, runner.arch, steps.version.outputs.tag, and steps.version.outputs.version out of run: shell strings into env: blocks as RUNNER_OS, RUNNER_ARCH, TAILOR_TAG, TAILOR_VERSION.

2. github-env-injection (action.yml): Added printf '%s' | tr -d '\n\r' sanitization before writing tag and version values to $GITHUB_OUTPUT.

3. script-injection (builder.yml sentinel): Moved toJSON(needs.*.result) into env: block as NEEDS_RESULTS.

4. script-injection (builder.yml release/Log in to GHCR): Moved secrets.GITHUB_TOKEN and github.actor into env: block as GHCR_TOKEN and GHCR_ACTOR.

5. script-injection (builder.yml update-tag): Moved github.event.release.tag_name into env: block as RELEASE_TAG.

6. unpinned-uses: Pinned all 18 action references across builder.yml, tailor-automerge.yml, and tailor.yml to their full 40-character SHA commit hashes with original tags preserved as comments.

