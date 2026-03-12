## Summary
<!-- What changed and why? Link to the issue or context if available. -->

## Source Of Truth Touched
<!-- List the workflow source files you intentionally edited. -->
- [ ] `.github/workflows/*.md`
- [ ] `.github/workflows/shared/**`
- [ ] `mise.toml`
- [ ] Documentation only

## Validation
<!-- Paste the exact commands you ran and note whether they changed generated files. -->
- [ ] `mise run aw-version`
- [ ] `mise run aw-validate`
- [ ] `mise run aw-compile`
- [ ] Reviewed generated `.lock.yml` and `.github/aw/actions-lock.json` changes

## Risk
<!-- What could go wrong in consumer repos or in this repo's own workflow runs? -->

## Rollback
<!-- How would you safely revert this if it causes bad reviews or compile drift? -->

## Notes For Reviewers
<!-- Mention import-path changes, prompt-fork updates, or any intentional divergence from upstream behavior. -->
