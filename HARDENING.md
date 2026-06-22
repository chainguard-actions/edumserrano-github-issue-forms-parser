<!-- markdownlint-disable -->

# Hardening Report: edumserrano--github-issue-forms-parser/v1.3.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **edumserrano--github-issue-forms-parser/v1.3.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references a Docker image using a mutable tag (':v1') instead of an immutable SHA digest. The image 'docker://ghcr.io/edumserrano/github-issue-forms-parser:v1' can be silently replaced at any time, enabling a supply-chain attack. It should be pinned to a specific SHA digest, e.g. 'docker://ghcr.io/edumserrano/github-issue-forms-parser@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag 'docker://ghcr.io/edumserrano/github-issue-forms-parser:v1' with the immutable SHA digest 'docker://ghcr.io/edumserrano/github-issue-forms-parser@sha256:6765a9c7aa830a55666b94af5f354728355167c65f9697e46a782e51eaf8da97' in action.yml line 16. The original tag ':v1' is preserved as a comment outside the YAML quotes for readability.

