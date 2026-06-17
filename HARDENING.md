<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Th3Un1q3--kinda-contribute/v1.0.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/github-script@v6`, which is pinned to a mutable version tag (`@v6`) rather than an immutable 40-character commit SHA. This means the action could silently pull in changed or malicious code if the tag is moved. It should be pinned to a full SHA, e.g. `actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v6`.

Locations:

- `action.yml:23`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` GitHub Actions expressions into shell command strings (rule a), allowing script injection:

- Line 38: `echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — `steps.*.outputs.*` is a workflow-controllable value injected directly into a shell echo command.
- Line 41: `git config --global user.email "${{ github.actor }}@users.noreply.github.com"` — `github.actor` is attacker-controllable (e.g. via a fork PR) and injected directly into a shell command.
- Line 42: `git config --global user.name "${{ github.actor }}"` — same issue with `github.actor`.
- Line 44: `commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` — `steps.*.outputs.*` is injected unquoted directly into a shell variable assignment, enabling command injection via shell metacharacters.

All these values must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g. `"$GITHUB_ACTOR"`) instead of being interpolated via `${{ }}` inside the `run:` block.

Locations:

- `action.yml:38`
- `action.yml:41`
- `action.yml:42`
- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two findings in action.yml: (1) Pinned actions/github-script@v6 to its full commit SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with the tag preserved as a comment. (2) Moved all four ${{ }} expressions from run: shell blocks into env: variables — COMMITS_NUMBER for the step output value and GITHUB_ACTOR_NAME for github.actor — and referenced them as double-quoted shell variables in the run: scripts to prevent script injection.

