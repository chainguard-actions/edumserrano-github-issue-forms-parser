<!-- markdownlint-disable -->

# Hardening Report: edumserrano--github-issue-forms-parser/v1.3.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **edumserrano--github-issue-forms-parser/v1.3.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag (`v1`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The failing reference is: `image: 'docker://ghcr.io/edumserrano/github-issue-forms-parser:v1'`. It should be replaced with a SHA-pinned reference such as `image: 'docker://ghcr.io/edumserrano/github-issue-forms-parser@sha256:<64-hex-char-digest>'`.

Locations:

- `action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag reference `docker://ghcr.io/edumserrano/github-issue-forms-parser:v1` with the immutable SHA256 digest `docker://ghcr.io/edumserrano/github-issue-forms-parser@sha256:6765a9c7aa830a55666b94af5f354728355167c65f9697e46a782e51eaf8da97` in action.yml line 14. The original tag `v1` is preserved as a comment for readability.

