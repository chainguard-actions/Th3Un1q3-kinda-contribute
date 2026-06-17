<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Th3Un1q3--kinda-contribute/v1.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings, violating rule (a). This allows an attacker to inject arbitrary shell commands via controlled inputs or context values.

- Line 36: `run: echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — step output interpolated directly in shell.
- Line 40: `git config --global user.email "${{ github.actor }}@users.noreply.github.com"` — `github.actor` interpolated directly; an attacker-controlled actor name could inject shell metacharacters.
- Line 41: `git config --global user.name "${{ github.actor }}"` — same issue.
- Line 43: `commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` — unquoted expression assigned to a shell variable, allowing word-splitting and shell metacharacter injection.

All of these should be moved to `env:` variables and then referenced as properly double-quoted shell variables (e.g., `"$ACTOR"`).

Locations:

- `action.yml:36`
- `action.yml:40`
- `action.yml:41`
- `action.yml:43`

### unpinned-uses (severity: high)

The action uses `actions/github-script@v6`, which is pinned to a mutable version tag (`v6`) rather than an immutable 40-character commit SHA. If the upstream repository is compromised or the tag is moved, this action will silently execute different code. It should be pinned to a full SHA, e.g. `actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v6`.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/github-script@v6 to full SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with # v6 comment. 2. Fixed all four script-injection locations: moved ${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }} into COMMITS_NUMBER env var for both the 'Get result' step and the git commit loop step; moved ${{ github.actor }} into ACTOR env var for the git config commands. All shell references now use properly double-quoted $VAR syntax.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection sub-rules in action.yml: (a) Moved `${{ inputs.exact-commits }}` and `${{ inputs.max-commits }}` out of the inline JavaScript source in the `actions/github-script` step into an `env:` block (EXACT_COMMITS, MAX_COMMITS), and updated the script to reference them via `process.env.EXACT_COMMITS` / `process.env.MAX_COMMITS` — including the catch block fallback. (b) Quoted the `$commits` variable in `$(seq 1 "$commits")` to prevent shell metacharacter injection.

