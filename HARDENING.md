<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Th3Un1q3--kinda-contribute/v1.0.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings, violating rule (a). This allows attacker-controlled values to be injected as shell code before the shell ever sees them.

1. Line 38: `run: echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — steps output interpolated directly into shell.
2. Line 42: `git config --global user.email "${{ github.actor }}@users.noreply.github.com"` — github.actor (attacker-controlled) interpolated directly into shell.
3. Line 43: `git config --global user.name "${{ github.actor }}"` — same issue.
4. Line 45: `commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` — steps output interpolated directly and unquoted (also violates rule b: unquoted shell variable assignment from untrusted data).

Locations:

- `action.yml:38`
- `action.yml:42`
- `action.yml:43`
- `action.yml:45`

### unpinned-uses (severity: high)

The step `uses: actions/github-script@v6` references a mutable version tag (`@v6`) instead of a full 40-character commit SHA. This means the action could be silently updated to a malicious version without any change to this file, creating a supply-chain risk. It should be pinned to a specific commit SHA, e.g. `actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v6`.

Locations:

- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/github-script@v6 to full SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with the tag preserved as a comment.
2. Fixed all four script-injection locations by moving ${{ }} expressions into env: blocks: COMMITS_NUMBER for the echo step (line 38), ACTOR for git config commands (lines 42-43), and COMMITS_NUMBER for the commits variable assignment (line 45). Shell scripts now reference these as plain environment variables ($COMMITS_NUMBER, $ACTOR) instead of interpolating GitHub expressions directly.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection sub-findings in hardened/action/action.yml:

(a) Moved `${{ inputs.exact-commits }}` and `${{ inputs.max-commits }}` out of the inline JavaScript in the `actions/github-script` step into an `env:` block as `EXACT_COMMITS` and `MAX_COMMITS`. The script now reads them via `process.env.EXACT_COMMITS` and `process.env.MAX_COMMITS` (including in the catch block), eliminating the risk of attacker-controlled input injecting arbitrary JavaScript.

(b) Quoted the shell variable assignment (`commits="$COMMITS_NUMBER"`) and the expansion in the seq call (`$(seq 1 "$commits")`), preventing shell metacharacter injection from the untrusted step output value.

