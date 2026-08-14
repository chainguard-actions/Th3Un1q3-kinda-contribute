<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Th3Un1q3--kinda-contribute/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml.

1. The 'Get result' step runs: `echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — the expression is substituted into the shell command before execution, enabling injection via a crafted step output.

2. The following run: block interpolates `${{ github.actor }}` directly into git config commands (e.g. `git config --global user.email "${{ github.actor }}@users.noreply.github.com"`) and `${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` into a bare shell variable assignment (`commits=${{ ... }}`). A malicious actor controlling `github.actor` or the step output could inject arbitrary shell commands.

Locations:

- `action.yml:34`
- `action.yml:37`
- `action.yml:38`
- `action.yml:40`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised.

action.yml:
  - uses: actions/github-script@v6  (tag)

.github/workflows/scheduled-contribute.yml:
  - uses: actions/checkout@v3  (tag)
  - uses: 'Th3Un1q3/kinda-contribute@main'  (branch)

.github/workflows/test-contribute.yml:
  - uses: actions/checkout@v3  (tag)
  - uses: 'Th3Un1q3/kinda-contribute@main'  (branch)

Locations:

- `action.yml:21`
- `.github/workflows/scheduled-contribute.yml:12`
- `.github/workflows/scheduled-contribute.yml:22`
- `.github/workflows/test-contribute.yml:20`
- `.github/workflows/test-contribute.yml:26`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level permissions: key, and neither job within them defines a job-level permissions: key. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

Affected files:
  - .github/workflows/scheduled-contribute.yml
  - .github/workflows/test-contribute.yml

Locations:

- `.github/workflows/scheduled-contribute.yml:1`
- `.github/workflows/test-contribute.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings:

1. script-injection (action.yml): Moved all ${{ }} expressions from run: shell strings into env: blocks. The github-script step now reads inputs via process.env instead of inline interpolation. The 'Get result' step and the git config/commit step both use env: variables (COMMITS_NUMBER, ACTOR) referenced as plain shell variables.

2. unpinned-uses: Pinned all action references to full 40-char SHAs: actions/github-script@v6 → d7906e4ad0b1822421a7e6a35d5ca353c962f410, actions/checkout@v3 → a37ce9120846195fa4ece8f58b268e6043cb2f26, Th3Un1q3/kinda-contribute@main → 9984bd619f7b4528b4dd7b365ac1c9a56fff0a2b.

3. missing-permissions: Added `permissions: {}` at the top level of both .github/workflows/scheduled-contribute.yml and .github/workflows/test-contribute.yml. The workflows rely on a PERSONAL_ACCESS_TOKEN secret for git operations, so GITHUB_TOKEN requires no permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in action.yml by quoting the `commits` variable in both its assignment (`commits="$COMMITS_NUMBER"`) and its use in the seq command substitution (`$(seq 1 "$commits")`). This prevents attacker-controlled values from the step output from being interpreted as shell metacharacters.

