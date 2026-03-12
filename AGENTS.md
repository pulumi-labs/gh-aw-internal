# Agent Instructions

## What this repo is
This repository publishes shared GitHub Agentic Workflow (`gh-aw`) templates for Pulumi Labs PR review automation. The source of truth is a small set of workflow markdown files plus shared imported prompts; compiled `.lock.yml` outputs are generated artifacts.

## Start here
- `README.md` - repo overview, install/use/update flow for consumer repos
- `mise.toml` - pinned `gh-aw` version and canonical local commands
- `.github/workflows/gh-aw-pr-review.md` - top-level PR review workflow
- `.github/workflows/gh-aw-pr-rereview.md` - maintainer-triggered re-review workflow
- `.github/workflows/shared/review.md` - shared workflow contract imported by both top-level workflows
- `.github/workflows/shared/plugins/code-review/code-review.md` - repo-local review prompt fork
- `.github/workflows/validate-agentic-workflows.yml` - CI parity for validate/compile/drift checks
- `.gitattributes` - marks compiled `.lock.yml` files as generated

## Command canon
- Bootstrap tools: `mise trust && mise install`
- Show pinned `gh-aw` version: `mise run aw-version`
- Validate workflow sources: `mise run aw-validate`
- Recompile lockfiles: `mise run aw-compile`

## Repo map and boundaries
- `.github/workflows/*.md` - installable top-level workflow entrypoints
- `.github/workflows/shared/**` - shared workflow source imported by top-level workflows
- `.github/workflows/*.lock.yml` - compiled outputs; review them, but do not hand-edit them
- `.github/aw/actions-lock.json` - gh-aw action lock data used during compile
- `.github/agents/` and `.github/snippets/` - legacy reference material, not the current shared workflow source of truth

## Key invariants
- Top-level review workflows must continue importing both `shared/review.md` and `shared/plugins/code-review/code-review.md`.
- `.github/workflows/shared/**` is source-of-truth content. Changes there must be followed by validation and compile so generated lockfiles stay in sync.
- `.lock.yml` files are generated artifacts and should change only as a result of `mise run aw-compile`.
- The pinned `gh-aw` version in `mise.toml` is the local/CI source of truth. Keep CI aligned with it.
- `id-token: write`, ESC secret fetching, and `safe-outputs` settings are security-sensitive. Treat edits there as behavior changes, not refactors.

## Forbidden actions
- Do not edit `.github/workflows/*.lock.yml` by hand.
- Do not remove or rewrite `.github/aw/actions-lock.json` outside normal `gh-aw` compile/update flows.
- Do not switch workflow imports away from the shared files without updating the repo docs and validation flow in the same change.
- Do not change workflow permissions, ESC configuration, or safe-output limits casually.
- Do not claim validation passed unless you actually ran the command.

## Escalate immediately if
- A requested change alters workflow permissions, OIDC usage, or ESC secret sourcing.
- A change requires adding a new shared import layout or packaging strategy for installable workflows.
- `mise run aw-validate` or `mise run aw-compile` changes files you did not intend to touch.
- You need to change how consumer repos reference this repo (`@main` vs tag/SHA guidance).

## If you change...
- `.github/workflows/*.md` or `.github/workflows/shared/**/*.md`:
  Run `mise run aw-validate && mise run aw-compile`
- `.github/workflows/shared/plugins/code-review/code-review.md`:
  Also review `.github/workflows/shared/plugins/code-review/README.md` for fork-sync accuracy
- `mise.toml`:
  Run `mise run aw-version && mise run aw-validate`
- `.gitattributes` or `.github/aw/actions-lock.json`:
  Run `mise run aw-compile` and inspect the resulting diff carefully
