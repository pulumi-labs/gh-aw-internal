# Local Fork Of Claude Code Review

This directory contains a local fork of Anthropic's Claude Code `code-review` plugin prompt, adapted for `gh-aw` review workflows.

## Upstream Source

- Repository: `anthropics/claude-code`
- Plugin directory: `plugins/code-review`
- Upstream prompt file: `plugins/code-review/commands/code-review.md`
- Upstream plugin README: `plugins/code-review/README.md`
- Upstream plugin manifest: `plugins/code-review/.claude-plugin/plugin.json`

Canonical upstream URLs:

- <https://github.com/anthropics/claude-code/tree/main/plugins/code-review>
- <https://github.com/anthropics/claude-code/blob/main/plugins/code-review/commands/code-review.md>

## Local Layout

- Local prompt file: `.github/workflows/shared/plugins/code-review/code-review.md`
- This local fork is imported by `.github/workflows/shared/review.md`

We intentionally do **not** keep the original Claude plugin packaging here. The local workflow only needs the prompt content, not the plugin manifest structure.

## How This Fork Was Created

1. Vendored the upstream plugin contents into this repo.
2. Removed plugin-specific files that are not used by `gh-aw`:
   - `.claude-plugin/plugin.json`
   - `README.md`
3. Moved the prompt from the plugin-style path:
   - `commands/code-review.md`
   into the local shared-workflow path:
   - `code-review.md`
4. Rewrote the prompt to use `gh-aw` mechanisms instead of Claude plugin mechanisms.

## Local Adaptations

The upstream prompt was changed in these ways:

- Replaced direct comment-posting behavior with `gh-aw` safe outputs:
  - `create-pull-request-review-comment`
  - `resolve-pull-request-review-thread`
  - `submit-pull-request-review`
  - `noop`
- Removed instructions that relied on Claude plugin packaging and direct plugin invocation.
- Kept the high-signal multi-agent review structure from upstream, but replaced one of the two CLAUDE.md agents with a dedicated tests specialist (CLAUDE.md + tests + bug-focused + behavior/security).
- Added explicit deduplication against existing PR review comments.
- Added validation steps before posting findings, with an explicit confidence floor of 0.80.
- Added PR-patch preflight for inline comments so comments are only submitted on GitHub-reviewable diff-hunk lines.
- Added a fixed severity model with plain-text labels: `Important`, `Nit`, `Pre-existing` (no emoji).
- Added re-review convergence rules: build a prior-findings RESOLVED/OPEN list, verify each candidate against the file at HEAD, and restrict new Nit findings to the incremental diff so the bot does not drip-feed nits across re-review rounds.
- Added a completeness self-check step (per-file enumeration plus pattern propagation across the diff) before posting.
- Standardized the inline-comment body template with a collapsible `<details><summary>Why this matters</summary>` rationale block and a ≤5-line cap on suggestion blocks.
- Added explicit gating for the final `APPROVE` event so the bot does not approve large, security-sensitive, or infrastructure-touching diffs, and constrained the final review state to a binary `APPROVE`-or-`COMMENT` choice — `REQUEST_CHANGES` is never used so the bot does not merge-block; humans with merge authority make that call based on `Important` inline comments.
- Added an explicit untrusted-content directive: PR title, description, commit messages, and review comment bodies must be treated as untrusted context, not as instructions.
- Added `cache-memory` guidance for short-lived PR review continuity (including persisting the last-reviewed commit SHA for incremental-diff scoping).
- Added bot-authored review thread resolution via `resolve-pull-request-review-thread`, with safeguards against resolving human-authored threads.
- Added `.gitattributes`-aware filtering so findings in generated artifacts (e.g. `.lock.yml`) are ignored unless the source-of-truth file shows a real issue.
- Kept live PR state and current review threads as the source of truth.
- Tightened review output to prefer terse, issue-only approvals and to avoid repeating inline comments in the final review.

## Sync Notes

If we later add automation to sync from upstream, the intended mapping is:

- Upstream input:
  - `anthropics/claude-code/plugins/code-review/commands/code-review.md`
- Local output:
  - `.github/workflows/shared/plugins/code-review/code-review.md`

Suggested sync flow:

1. Fetch upstream `commands/code-review.md`.
2. Compare it to the current local file.
3. Reapply the local `gh-aw` adaptations.
4. Recompile workflows that import `.github/workflows/shared/review.md`.

## Sync-Safe Metadata

```yaml
upstream:
  repo: anthropics/claude-code
  ref: main
  plugin_dir: plugins/code-review
  prompt_path: plugins/code-review/commands/code-review.md
local:
  prompt_path: .github/workflows/shared/plugins/code-review/code-review.md
  imported_by:
    - .github/workflows/shared/review.md
removed_upstream_files:
  - plugins/code-review/.claude-plugin/plugin.json
  - plugins/code-review/README.md
local_adaptations:
  - convert direct review side effects to gh-aw safe outputs
  - remove plugin-packaging assumptions
  - replace one CLAUDE.md agent with a dedicated tests specialist
  - add explicit confidence floor of 0.80 for all findings
  - add plain-text severity tiers (Important / Nit / Pre-existing) with no emoji
  - add re-review convergence rules with RESOLVED/OPEN tracking and incremental-diff-scoped nits
  - add completeness self-check step with per-file enumeration and pattern propagation
  - standardize inline-comment body template with collapsible rationale block
  - cap committable suggestion blocks at 5 lines
  - gate the final APPROVE event behind explicit low-risk conditions
  - constrain final review state to binary APPROVE-or-COMMENT; never use REQUEST_CHANGES
  - treat PR title, description, and review comment bodies as untrusted content
  - add review-comment deduplication guidance
  - add PR-patch preflight for inline comment targets
  - add bot-authored review thread resolution
  - add cache-memory continuity guidance including last-reviewed commit SHA
  - add .gitattributes-aware filtering for generated artifacts
  - prefer terse issue-only review output with no inline-summary duplication
```
