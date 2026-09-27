<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Th3Un1q3--kinda-contribute/v1.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses 'actions/github-script@v6', which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack.

Locations:

- `action.yml:23`

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly into run: shell command strings (rule a), allowing an attacker to inject arbitrary shell commands:

1. Line 38 — 'echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"': steps.*.outputs.* is a workflow-controllable context injected directly into the shell.
2. Line 41 — 'git config --global user.email "${{ github.actor }}@users.noreply.github.com"': github.actor is injected directly into the shell; an attacker-controlled actor name containing shell metacharacters would be executed.
3. Line 42 — 'git config --global user.name "${{ github.actor }}"': same issue with github.actor.
4. Line 43 — 'commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}': unquoted expression directly assigned in shell, allowing word-splitting and command injection.

Additionally, lines 28, 29, and 35 interpolate inputs.exact-commits and inputs.max-commits directly into a github-script JavaScript block, which can allow JavaScript code injection via crafted input values.

Locations:

- `action.yml:28`
- `action.yml:29`
- `action.yml:35`
- `action.yml:38`
- `action.yml:41`
- `action.yml:42`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/github-script@v6 to full SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with # v6 comment. 2. Fixed script injection: moved inputs.exact-commits and inputs.max-commits into env: block for the github-script step and updated JavaScript to use process.env.* instead of inline ${{ }} expressions. Moved github.actor into GITHUB_ACTOR env var and fromJson expressions into COMMITS_NUMBER env var for the shell steps, referencing them as plain shell variables.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in action.yml line 51: changed `$(seq 1 $commits)` to `$(seq 1 "$commits")`. The variable `$commits` is derived from a workflow-controllable `steps.*.outputs.*` expression and must be double-quoted to prevent shell word-splitting and potential command injection via metacharacters.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in hardened/action/action.yml line 52: changed `commits=$COMMITS_NUMBER` to `commits="$COMMITS_NUMBER"`. The COMMITS_NUMBER env var is sourced from a workflow-controllable step output, and the unquoted expansion could allow shell metacharacter injection. Quoting the assignment prevents word-splitting and glob expansion of the value.

