<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Th3Un1q3--kinda-contribute/v1.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses `actions/github-script@v6`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks.

Locations:

- `action.yml:23`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, violating sub-rule (a). This allows an attacker to inject arbitrary shell commands:

- Line 38: `echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — `steps.*.outputs.*` interpolated directly into shell.
- Line 41: `git config --global user.email "${{ github.actor }}@users.noreply.github.com"` — `github.actor` is attacker-controllable (e.g. via a crafted username) and is interpolated directly into a shell command.
- Line 42: `git config --global user.name "${{ github.actor }}"` — same attacker-controllable `github.actor` value.
- Line 44: `commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` — unquoted expression assigned directly in shell, allowing shell metacharacter injection.

All of these should be moved to `env:` variables and then referenced as double-quoted shell variables (e.g. `"$COMMITS"`).

Locations:

- `action.yml:38`
- `action.yml:41`
- `action.yml:42`
- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/github-script@v6 to full SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with # v6 comment. 2. Fixed all four script-injection locations: moved github.actor into ACTOR env var (used for both git config email and name), and moved fromJson(steps.define-commits-to-make.outputs.result).commitsNumber into COMMITS_NUMBER env var (used in both the echo step and the main run step). All shell references now use properly double-quoted $ACTOR and $COMMITS_NUMBER variables.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the github-script step by moving all three ${{ }} expression interpolations out of the JavaScript code and into the step's env: block. Added EXACT_COMMITS and MAX_COMMITS environment variables, then replaced the direct string interpolations with process.env.EXACT_COMMITS and process.env.MAX_COMMITS references in the JavaScript code. This prevents attackers from breaking out of string literals by supplying crafted input values containing JavaScript metacharacters.

