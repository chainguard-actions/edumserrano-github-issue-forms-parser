<!-- markdownlint-disable -->

# Hardening Report: edumserrano--github-issue-forms-parser/v1.3.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **edumserrano--github-issue-forms-parser/v1.3.7** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands (sub-rule a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever parses the string.

build-test.yml:
- `run: dotnet restore ${{ env.SLN_FILENAME }}` — env context interpolated directly
- `run: dotnet build ${{ env.SLN_FILENAME }} ...` — env context interpolated directly
- `dotnet test ${{ env.SLN_FILENAME }} ...` with `${{github.workspace}}` and `${{ runner.os }}` in the same run block
- `${{ steps.dotnet-test.outputs.test-coverage-dir }}` and `${{ steps.dotnet-test.outputs.test-coverage-file }}` interpolated directly into a run: block
- `${{ github.repository }}` interpolated into a run: block

pr-dependabot-auto-merge.yml:
- `$prNumber = "${{ github.event.workflow_run.pull_requests[0].number }}"` — attacker-influenced github context interpolated directly into shell

test-action.yml:
- `$issue = '${{ steps.issue-parser.outputs.parsed-issue }}'` — step output interpolated directly into shell
- `$parseStepWithBadInputOutcome = '${{ steps.issue-parser-bad-input.outcome }}'` — step output interpolated directly into shell

test-action-gh-marketplace.yml:
- `$issue = '${{ steps.issue-parser.outputs.parsed-issue }}'` — step output interpolated directly into shell
- `$parseStepWithBadInputOutcome = '${{ steps.issue-parser-bad-input.outcome }}'` — step output interpolated directly into shell

markdown-link-check.yml:
- `[System.Convert]::ToBoolean("${{ steps.mlc-push.conclusion == 'success' }}")` — step conclusion interpolated directly into shell
- `[System.Convert]::ToBoolean("${{ steps.mlc.outputs.has-broken-links }}")` — step output interpolated directly into shell

All these should be moved to env: variables and referenced as $env:VAR_NAME in the shell.

Locations:

- `.github/workflows/build-test.yml:36`
- `.github/workflows/build-test.yml:40`
- `.github/workflows/build-test.yml:44`
- `.github/workflows/build-test.yml:57`
- `.github/workflows/build-test.yml:96`
- `.github/workflows/build-test.yml:107`
- `.github/workflows/build-test.yml:116`
- `.github/workflows/pr-dependabot-auto-merge.yml:28`
- `.github/workflows/test-action.yml:52`
- `.github/workflows/test-action.yml:80`
- `.github/workflows/test-action-gh-marketplace.yml:52`
- `.github/workflows/test-action-gh-marketplace.yml:80`
- `.github/workflows/markdown-link-check.yml:42`
- `.github/workflows/markdown-link-check.yml:51`

### unpinned-uses (severity: high)

All uses: references in workflow files are pinned to mutable version tags rather than immutable 40-character SHA commit hashes. Additionally, action.yml references a Docker image by mutable tag (v1) instead of a SHA digest, making the action vulnerable to supply-chain attacks if the upstream tag is moved.

Failing uses: references (all use version tags, not SHAs):
- build-test.yml: actions/checkout@v4, actions/setup-dotnet@v4, actions/cache@v4, actions/upload-artifact@v4 (×2), codecov/codecov-action@v4
- markdown-link-check.yml: actions/checkout@v4, gaurav-nelson/github-action-markdown-link-check@1.0.15 (×2)
- package-retention-policy.yml: snok/container-retention-policy@v2
- pr-dependabot-auto-merge.yml: actions/checkout@v4
- publish-docker-image.yml: actions/checkout@v4, docker/login-action@v3, docker/metadata-action@v5, docker/build-push-action@v6
- test-action-gh-marketplace.yml: actions/checkout@v4, edumserrano/github-issue-forms-parser@v1 (×2)
- test-action.yml: actions/checkout@v4 (×2)

Failing image: reference in action.yml:
- image: 'docker://ghcr.io/edumserrano/github-issue-forms-parser:v1' — uses a mutable tag, not a SHA digest

Locations:

- `action.yml:16`
- `.github/workflows/build-test.yml:30`
- `.github/workflows/build-test.yml:32`
- `.github/workflows/build-test.yml:34`
- `.github/workflows/build-test.yml:103`
- `.github/workflows/build-test.yml:109`
- `.github/workflows/build-test.yml:113`
- `.github/workflows/markdown-link-check.yml:20`
- `.github/workflows/markdown-link-check.yml:24`
- `.github/workflows/markdown-link-check.yml:30`
- `.github/workflows/package-retention-policy.yml:18`
- `.github/workflows/pr-dependabot-auto-merge.yml:22`
- `.github/workflows/publish-docker-image.yml:19`
- `.github/workflows/publish-docker-image.yml:21`
- `.github/workflows/publish-docker-image.yml:25`
- `.github/workflows/publish-docker-image.yml:30`
- `.github/workflows/test-action-gh-marketplace.yml:20`
- `.github/workflows/test-action-gh-marketplace.yml:38`
- `.github/workflows/test-action-gh-marketplace.yml:72`
- `.github/workflows/test-action.yml:20`
- `.github/workflows/test-action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned-uses findings by pinning all action references to full 40-character SHA commit hashes (with original tags preserved as comments) across all workflow files: build-test.yml, markdown-link-check.yml, package-retention-policy.yml, pr-dependabot-auto-merge.yml, publish-docker-image.yml, test-action.yml, test-action-gh-marketplace.yml. Also pinned the docker image in action.yml to its SHA digest while preserving the docker:// scheme and tag. Fixed all script-injection findings by moving ${{ }} expressions from run: blocks into env: blocks and referencing them as $env:VAR_NAME in PowerShell scripts across all affected workflow files.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings across two workflow files:

1. build-test.yml (script-injection + github-env-injection): Replaced the direct `${{ steps.dotnet-test.conclusion }}` interpolation in the run block with an env var `DOTNET_TEST_CONCLUSION: ${{ steps.dotnet-test.conclusion }}`. The PowerShell script now reads from `$env:DOTNET_TEST_CONCLUSION`, computes the boolean condition using safe `-eq` comparisons, strips newlines with `-replace '[\r\n]', ''` before writing to `$GITHUB_OUTPUT`, eliminating both the script-injection and github-env-injection risks.

2. pr-dependabot-auto-merge.yml (script-injection/unquoted-variable): Quoted `$prNumber` as `"$prNumber"` in the `gh pr merge` command to prevent shell metacharacter injection from the PR number value.

