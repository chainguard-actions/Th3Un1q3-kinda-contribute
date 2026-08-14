<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Th3Un1q3--kinda-contribute/v1.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside run: shell blocks. In the 'Get result' step, `${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` is interpolated directly into the shell command string (line 38). In the following run: block, `${{ github.actor }}` is interpolated directly into git config commands (lines 42–43) and `${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` is assigned to a shell variable without quoting (line 45). Any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value, allowing an attacker-controlled value to inject shell metacharacters.

Locations:

- `action.yml:38`
- `action.yml:42`
- `action.yml:43`
- `action.yml:45`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag or branch is moved or overwritten. Failing references: action.yml — `actions/github-script@v6` (tag); .github/workflows/scheduled-contribute.yml — `actions/checkout@v3` (tag), `Th3Un1q3/kinda-contribute@main` (branch); .github/workflows/test-contribute.yml — `actions/checkout@v3` (tag), `Th3Un1q3/kinda-contribute@main` (branch).

Locations:

- `action.yml:23`
- `.github/workflows/scheduled-contribute.yml:12`
- `.github/workflows/scheduled-contribute.yml:21`
- `.github/workflows/test-contribute.yml:20`
- `.github/workflows/test-contribute.yml:26`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default token permissions (which may be write-all depending on repository settings), granting broader access than necessary. Both scheduled-contribute.yml and test-contribute.yml are affected.

Locations:

- `.github/workflows/scheduled-contribute.yml:1`
- `.github/workflows/test-contribute.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in action.yml by moving all ${{ }} expressions from run: blocks into env: blocks and referencing them as plain shell variables; (2) unpinned-uses by pinning actions/github-script@v6 to SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410, actions/checkout@v3 to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26, and Th3Un1q3/kinda-contribute@main to SHA dcca072696848613f4d1bd590402dd31d0fa09d6; (3) missing-permissions by adding top-level 'permissions: {}' and job-level 'permissions: contents: write' to both workflow files (contents: write is needed for git push operations).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in hardened/action/action.yml at line 50. Changed `$(seq 1 $commits)` to `$(seq 1 "$commits")` to properly double-quote the `$commits` variable (derived from the untrusted `steps.define-commits-to-make.outputs.result` step output). This prevents attacker-controlled shell metacharacters from being interpreted as shell commands.

