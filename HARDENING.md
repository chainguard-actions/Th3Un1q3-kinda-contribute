<!-- markdownlint-disable -->

# Hardening Report: Th3Un1q3--kinda-contribute/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Th3Un1q3--kinda-contribute/v1.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The step `uses: actions/github-script@v6` references a mutable tag (`@v6`) instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `action.yml:23`

### script-injection (severity: high)

Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings (rule a), allowing script injection. (1) Line 39: `echo "${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}"` — a `steps.*.outputs.*` value is injected directly into the shell. (2) Line 42: `git config --global user.email "${{ github.actor }}@users.noreply.github.com"` — `github.actor` is attacker-controllable and injected directly into a shell command. (3) Line 43: `git config --global user.name "${{ github.actor }}"` — same issue. (4) Line 45: `commits=${{ fromJson(steps.define-commits-to-make.outputs.result).commitsNumber }}` — unquoted `steps.*.outputs.*` expression injected directly into the shell. Additionally, `inputs.exact-commits` and `inputs.max-commits` are interpolated directly into the `github-script` JS `script:` block (lines 28, 29, 35), which is also a script-injection vector since the values are substituted before the script is evaluated.

Locations:

- `action.yml:28`
- `action.yml:29`
- `action.yml:35`
- `action.yml:39`
- `action.yml:42`
- `action.yml:43`
- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/github-script@v6 to full SHA d7906e4ad0b1822421a7e6a35d5ca353c962f410 with # v6 comment. 2. Fixed all script-injection issues: moved inputs.exact-commits and inputs.max-commits into env: block on the github-script step and updated JS to use process.env.*; moved github.actor into GITHUB_ACTOR env var; moved fromJson(steps.define-commits-to-make.outputs.result).commitsNumber into COMMITS_NUMBER env var for both run: steps that used it. All ${{ }} expressions are now in env: blocks, not inline in shell or JS script strings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in the `for` loop's `seq` command. Changed `commits=$COMMITS_NUMBER` to `commits="$COMMITS_NUMBER"` and `$(seq 1 $commits)` to `$(seq 1 "$commits")` in action.yml. This prevents shell metacharacter injection from the workflow-controllable `COMMITS_NUMBER` environment variable (sourced from `steps.define-commits-to-make.outputs.result`).

