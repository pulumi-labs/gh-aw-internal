# Contributing

## Scope of this repo
This repo contains reusable `gh-aw` review workflows and shared imported prompts. Most changes should be small and targeted: update the workflow source markdown, recompile generated lockfiles, and verify the resulting diff.

## Local setup
From the repo root:

```bash
mise trust
mise install
mise run aw-version
```

Expected result: `gh aw version v0.57.2`

## Source of truth
- Edit `.github/workflows/*.md` for installable top-level workflows.
- Edit `.github/workflows/shared/**` for shared workflow contract or prompt behavior.
- Treat `.github/workflows/*.lock.yml` as generated output from `mise run aw-compile`.
- Treat `.github/aw/actions-lock.json` as tool-managed data that should move with normal compile/update flows.

## Standard change flow
1. Make the smallest source-of-truth edit that solves the problem.
2. Run:

```bash
mise run aw-validate
mise run aw-compile
```

3. Review the full diff, including generated lockfiles.
4. Update docs in the same change if you altered workflow behavior, imports, or maintenance expectations.

## AI-assisted contributions
AI changes work best here when they stay grounded in the real workflow graph and compile flow.

- Read `AGENTS.md`, `README.md`, and the touched workflow source files before editing.
- Prefer editing `.github/workflows/shared/**` when behavior is shared; prefer editing top-level workflow files only for trigger/runtime differences.
- Do not hand-edit `.lock.yml` files to force the desired output.
- Include exact validation commands in the change description. Do not paraphrase test results.
- Keep vendored prompt adaptations narrow and explain any intentional divergence from upstream behavior.

## Validation expectations
Run these commands from the repo root unless the change is documentation-only:

```bash
mise run aw-validate
mise run aw-compile
git diff --exit-code
```

For workflow-source changes, the final `git diff --exit-code` should be run only after you have reviewed and staged the expected generated changes. If it exits non-zero, either commit the generated files or fix the unexpected drift before submitting.

## Review checklist
- Does the change touch the correct source-of-truth file?
- If a shared file changed, did the compiled lockfiles update as expected?
- Do README or contributor docs need an update for changed workflow behavior?
- If permissions, OIDC, ESC usage, or safe outputs changed, is the risk called out explicitly?
