# gh-aw-internal

Shared internal GitHub Agentic Workflow templates for Pulumi Labs.

## What's In This Repo

- `.github/workflows/gh-aw-pr-review.md`
- `.github/workflows/gh-aw-pr-rereview.md`
- `.github/workflows/shared/review.md`
- `.github/workflows/shared/plugins/code-review/code-review.md`
- `workflows/gh-aw-pr-review.md`
- `workflows/gh-aw-pr-rereview.md`
- `workflows/shared/review.md`
- `workflows/shared/plugins/code-review/code-review.md`
- `mise.toml` (pinned `gh-aw` tooling for local use)

The review workflows import the shared review contract from:

`shared/review.md`

That shared workflow, in turn, imports the detailed reviewer prompt from:

`shared/plugins/code-review/code-review.md`

The top-level `workflows/` and `workflows/shared/` trees are packaging/export entrypoints for `gh aw add owner/repo/workflow-name@ref`. The `.github/workflows/` tree remains the in-repo authoring path and is kept for backward compatibility with existing consumers that already track the old source location.

## Local Tooling (mise)

This repo uses mise to pin the local `gh-aw` version and provide repeatable commands.

```bash
mise trust
mise install
mise run aw-version
```

Useful tasks:

```bash
mise run aw-validate
mise run aw-compile
```

Version source of truth is `GH_AW_VERSION` in `mise.toml`.

## Use In A New Repo

1. Initialize gh-aw in the target repo:

```bash
gh-aw init
```

2. Add workflows from this repo:

```bash
gh-aw add pulumi-labs/gh-aw-internal/gh-aw-pr-review@main
gh-aw add pulumi-labs/gh-aw-internal/gh-aw-pr-rereview@main
```

3. Compile and commit:

```bash
gh-aw compile
git add .github/workflows .gitattributes
git commit -m "Add shared PR review workflows"
```

## Update In Consumer Repos

If workflows were added via `gh-aw add`, use:

```bash
gh-aw update
```

Common variants:

```bash
gh-aw update gh-aw-pr-review
gh-aw update --create-pull-request
gh-aw update --no-merge
```

## Maintain This Repo

Typical workflow when changing review behavior:

1. Update `.github/workflows/shared/review.md` for shared workflow contract changes.
2. Update `.github/workflows/shared/plugins/code-review/code-review.md` for reviewer behavior changes.
3. Recompile the workflow lock files.
4. Update consumers via `gh-aw update`.

Legacy helper files may still exist in `.github/snippets/` or `.github/agents/`, but the review workflows on this branch are driven by the shared workflow files above.

Note: consumers compile against remote refs. If an import points at `@main`, the referenced file must already exist on GitHub before consumer compile succeeds.

## Versioning Strategy

- `@main`: fastest rollout, less stability.
- `@vX.Y.Z` or `@<sha>`: stable and reproducible.

Recommended path:

1. Start with `@main` while iterating.
2. Cut tags for stable rollouts.
3. Move consumers to tags/SHA.

## CLI Version Drift (Important)

`.lock.yml` output can change across `gh-aw` versions. If CI recompiles workflows, pin the same gh-aw version in CI and local workflows to avoid false diffs.

Recommended CI install pattern (single source of truth from `mise.toml`):

```yaml
- name: Checkout
  uses: actions/checkout@v4

- name: Install tools via mise
  uses: jdx/mise-action@v3
  with:
    install: true
```

`jdx/mise-action` reads `mise.toml`, so CI and local use the same pinned `github:github/gh-aw` version.

Then validate:

```bash
gh-aw validate
```

For repo-wide upgrades:

```bash
gh extension upgrade gh-aw
gh-aw upgrade
```

## References

- gh-aw CLI: https://github.github.com/gh-aw/setup/cli/
- Creating workflows: https://github.github.com/gh-aw/setup/creating-workflows/
- Imports reference: https://github.github.com/gh-aw/reference/imports/
- Reusing workflows/imports: https://github.github.com/gh-aw/guides/packaging-imports/
- Upgrading workflows: https://github.github.com/gh-aw/guides/upgrading/
- How lock files work: https://github.github.com/gh-aw/introduction/how-they-work/
- mise action: https://mise.jdx.dev/ci/github-action.html
