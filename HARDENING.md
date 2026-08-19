<!-- markdownlint-disable -->

# Hardening Report: edumserrano--github-issue-forms-parser/v1.3.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **edumserrano--github-issue-forms-parser/v1.3.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if a tag is moved. Affected references include: actions/checkout@v4, actions/setup-dotnet@v3, actions/cache@v3, actions/upload-artifact@v3, codecov/codecov-action@v3, snok/container-retention-policy@v2, docker/login-action@v3, docker/metadata-action@v5, docker/build-push-action@v5, edumserrano/github-issue-forms-parser@v1. Additionally, action.yml uses `image: 'docker://ghcr.io/edumserrano/github-issue-forms-parser:v1'` — a mutable image tag rather than a SHA digest.

Locations:

- `action.yml:16`
- `.github/workflows/build-test.yml:28`
- `.github/workflows/build-test.yml:30`
- `.github/workflows/build-test.yml:33`
- `.github/workflows/build-test.yml:96`
- `.github/workflows/build-test.yml:101`
- `.github/workflows/build-test.yml:107`
- `.github/workflows/package-retention-policy.yml:19`
- `.github/workflows/pr-dependabot-auto-merge.yml:20`
- `.github/workflows/publish-docker-image.yml:17`
- `.github/workflows/publish-docker-image.yml:20`
- `.github/workflows/publish-docker-image.yml:25`
- `.github/workflows/publish-docker-image.yml:30`
- `.github/workflows/test-action-gh-marketplace.yml:19`
- `.github/workflows/test-action-gh-marketplace.yml:44`
- `.github/workflows/test-action-gh-marketplace.yml:76`
- `.github/workflows/test-action.yml:19`
- `.github/workflows/test-action.yml:76`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, violating sub-rule (a). Before the shell executes the script, GitHub Actions performs template substitution, allowing an attacker to inject shell metacharacters.

**build-test.yml**: (1) `dotnet test ${{ env.SLN_FILENAME }}` — env context interpolated directly into a shell command. (2) `$(Join-Path -Path ${{ steps.dotnet-test.outputs.test-coverage-dir }} ...)` — step output interpolated into a command substitution. (3) `"-reports:${{ steps.dotnet-test.outputs.test-coverage-file }}"` — step output interpolated into a quoted argument. (4) `$condition = '${{ (steps.dotnet-test.conclusion == 'success' || steps.dotnet-test.conclusion == 'failure') }}'` — steps context interpolated into a shell variable. (5) `"https://app.codecov.io/gh/${{ github.repository }}/"` — github context interpolated into a URL string.

**test-action.yml**: (1) `$issue = '${{ steps.issue-parser.outputs.parsed-issue }}'` — step output (which contains user-controlled issue body content) interpolated directly into a shell string. (2) `$parseStepWithBadInputOutcome = '${{ steps.issue-parser-bad-input.outcome }}'` — step output interpolated into a shell variable.

**test-action-gh-marketplace.yml**: Same patterns as test-action.yml — `${{ steps.issue-parser.outputs.parsed-issue }}` and `${{ steps.issue-parser-bad-input.outcome }}` interpolated directly in `run:` blocks.

**pr-dependabot-auto-merge.yml**: `$prNumber = "${{ github.event.workflow_run.pull_requests[0].number }}"` — attacker-influenced github context interpolated directly into a shell variable used in a `gh pr merge` command.

Locations:

- `.github/workflows/build-test.yml:57`
- `.github/workflows/build-test.yml:88`
- `.github/workflows/build-test.yml:96`
- `.github/workflows/build-test.yml:97`
- `.github/workflows/build-test.yml:113`
- `.github/workflows/test-action.yml:48`
- `.github/workflows/test-action.yml:76`
- `.github/workflows/test-action-gh-marketplace.yml:48`
- `.github/workflows/test-action-gh-marketplace.yml:76`
- `.github/workflows/pr-dependabot-auto-merge.yml:30`

### github-env-injection (severity: high)

In build-test.yml, the `Set run even if tests fail condition` step interpolates `${{ (steps.dotnet-test.conclusion == 'success' || steps.dotnet-test.conclusion == 'failure') }}` into the shell variable `$condition`, then writes it to `$GITHUB_OUTPUT` via `Write-Output "condition=$condition" >> $env:GITHUB_OUTPUT` without any sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Although `steps.*.conclusion` is GitHub-controlled and unlikely to contain newlines in practice, the pattern is unsafe: routing through a shell variable does not sanitize the value, and the write to `$GITHUB_OUTPUT` must be preceded by the sanitization pipeline whenever the source is not a literal computed in the same run block.

Locations:

- `.github/workflows/build-test.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across 6 workflow files and action.yml:

1. **unpinned-uses**: Pinned all 10 action references to full SHA hashes and pinned the docker image in action.yml to its sha256 digest (keeping the docker:// scheme and :v1 tag inline).

2. **script-injection**: Moved all ${{ }} expressions out of run: shell blocks into step env: blocks, then referenced them as $env:VAR_NAME (PowerShell syntax). This affects build-test.yml (SLN_FILENAME, github.workspace, runner.os, step outputs, github.repository), test-action.yml (parsed-issue output, outcome), test-action-gh-marketplace.yml (same patterns), and pr-dependabot-auto-merge.yml (PR number).

3. **github-env-injection**: Rewrote the 'Set run even if tests fail condition' step in build-test.yml to compute the boolean condition in PowerShell from the env var value (avoiding direct expression interpolation), then sanitizes with -replace '[\r\n]', '' before writing to GITHUB_OUTPUT.

