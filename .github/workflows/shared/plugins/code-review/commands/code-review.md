---
description: High-signal PR review for gh-aw workflows using safe outputs
---

Provide a code review for the given pull request.

**Agent assumptions (applies to all agents and subagents):**
- All tools are functional and will work without error. Do not test tools or make exploratory calls. Make sure this is clear to every subagent that is launched.
- Only call a tool if it is required to complete the task. Every tool call should have a clear purpose.
- Use GitHub MCP tools for repository reads. Do not use `gh` CLI commands for repository inspection or for posting review output.
- Use the workflow PR number as the authoritative target.
- Use only gh-aw safe outputs for review side effects:
  - `create-pull-request-review-comment` for actionable inline findings on changed lines
  - `submit-pull-request-review` for the final review decision
  - `noop` when no action should be taken
- Never post free-form issue comments or use any side channel for review output.
- Respect the workflow safe-output limits. Prioritize the highest-signal unique findings and keep the inline review set within the configured maximum.

To do this, follow these steps precisely:

1. Create a short todo list for yourself before starting.

2. Launch a fast subagent to check if any of the following are true:
   - The pull request is closed
   - The pull request is a draft
   - The pull request does not need code review (e.g. automated PR, trivial change that is obviously correct)
   - Required PR context cannot be read from the workflow tools

   If any condition is true, call `noop` with a brief reason and stop.

Note: Do not skip solely because prior automated review comments exist. Use prior comments for deduplication and stale-thread cleanup instead.

3. Launch a fast subagent to return a list of file paths (not their contents) for all relevant `CLAUDE.md` files including:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing files modified by the pull request

4. Launch a subagent to summarize:
   - The PR title and description
   - The changed files
   - The main behavioral changes in the diff
   - Any obvious risk areas worth checking carefully

5. Fetch existing review comments on the PR before preparing any new findings. Use them to identify:
   - Similar issues already flagged
   - Threads where a human already acknowledged the feedback
   - Comments on code that has changed since the earlier review and may now be stale

6. Launch 4 review subagents in parallel. Each agent should return a list of candidate issues, where each issue includes:
   - A concise description
   - The reason it was flagged (for example `CLAUDE.md adherence` or `bug`)
   - The changed file path
   - The changed line or closest changed hunk
   - Why the issue is likely real

   Agents 1 + 2: CLAUDE.md compliance sonnet agents
   Audit changes for `CLAUDE.md` compliance in parallel. When evaluating a file, only consider `CLAUDE.md` files that share a path scope with that file or its parents.

   Agent 3: bug-focused review agent
   Scan for obvious bugs. Focus only on the diff itself without reading extra context. Flag only significant bugs; ignore nitpicks and likely false positives. Do not flag issues that you cannot validate without looking at context outside of the git diff.

   Agent 4: behavior-focused review agent
   Look for problems introduced by the new code, including security issues, incorrect logic, regressions, and missing error handling. Only look for issues that fall within changed code.

   **CRITICAL: We only want HIGH SIGNAL issues.** Flag issues where:
   - The code will fail to compile or parse (syntax errors, type errors, missing imports, unresolved references)
   - The code will definitely produce wrong results regardless of inputs (clear logic errors)
   - The code introduces a clear security or regression bug in changed lines
   - Clear, unambiguous `CLAUDE.md` violations where you can quote the exact rule being broken

   Do NOT flag:
   - Code style or quality concerns
   - Potential issues that depend on specific inputs or state
   - Subjective suggestions or improvements

   If you are not certain an issue is real, do not flag it. False positives erode trust and waste reviewer time.

   Each review subagent should receive the PR title and description so it can evaluate intent.

7. For each candidate issue, launch validation subagents in parallel. Each validator should receive the PR title, description, issue description, affected file, and affected hunk. The validator must confirm that the issue is real with high confidence.

   For bug and logic issues, verify the changed code actually causes the stated problem.

   For `CLAUDE.md` issues, verify both:
   - The cited rule exists
   - The rule applies to that file path and is actually violated

8. Filter out any issue that fails validation.

9. Deduplicate and prune the validated issue list. Remove:
   - Issues already covered by an existing review comment
   - Issues in threads where a human has already acknowledged the feedback
   - Issues that were present in an earlier revision but are fixed in the latest code
   - Duplicate findings reported by multiple subagents
   - Findings that are not on changed lines or cannot be tied to a changed hunk

10. Classify the remaining issues:
   - `Blocking`: correctness, security, regression, data loss, or clear required-rule violations
   - `Non-blocking`: actionable but not merge-blocking concerns

11. Produce a short internal summary of findings for yourself:
   - If issues remain, list the highest-signal ones first
   - If no issues remain, summarize that no actionable high-signal issues were found

12. If no actionable issues remain, submit exactly one final review with `submit-pull-request-review`:
   - Use `APPROVE`
   - Briefly state that no issues were found after checking for bugs and `CLAUDE.md` compliance
   - Do not create inline comments
   - Stop after the final review is submitted

13. If actionable issues remain, choose the highest-signal unique issues up to the safe-output comment limit. Create a list of planned inline comments for yourself before posting anything.

14. Post one inline comment per chosen issue using `create-pull-request-review-comment`. For each comment:
   - Provide a brief description of the issue
   - Explain why it matters
   - Reference the exact changed line
   - Cite the relevant `CLAUDE.md` rule when applicable
   - Keep the comment concise and actionable
   - Do not post duplicate comments for the same issue

15. Submit exactly one final review using `submit-pull-request-review`:
   - Use `REQUEST_CHANGES` when at least one blocking issue remains
   - Use `APPROVE` otherwise, including when only non-blocking inline comments were left
   - Do not use `COMMENT` as the final review state
   - Keep the summary short and aligned with the issues you posted

Use this list when evaluating issues in Steps 4 and 5 (these are false positives, do NOT flag):

- Pre-existing issues
- Something that appears to be a bug but is actually correct
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter will catch (do not run the linter to verify)
- General code quality concerns (e.g., lack of test coverage, general security issues) unless explicitly required in CLAUDE.md
- Issues mentioned in CLAUDE.md but explicitly silenced in the code (e.g., via a lint ignore comment)

Notes:

- Use GitHub tools for all repository reads. Do not use web fetch.
- Always operate on the workflow PR target rather than guessing from local git state.
- Inline comments should only be created for actionable issues on changed lines.
- When linking to code in an inline comment, use a full GitHub blob URL with a full SHA and a line range, for example: https://github.com/anthropics/claude-code/blob/c21d3c10bc8e898b7ac1a2d745bdc9bc4e423afe/package.json#L10-L15
  - Requires full git sha
  - Do not use shell substitution in the URL
  - Repo name must match the repo you're code reviewing
  - Use `#L[start]-L[end]`
  - Provide at least one line of context before and after when possible
- If context checks fail or the PR is not reviewable, call `noop` with a brief explanation instead of exiting silently.
