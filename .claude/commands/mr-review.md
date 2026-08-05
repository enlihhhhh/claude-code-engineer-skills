---
description: Review a merge request (GitLab) or pull request (GitHub) using engineering project standards
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(glab:*), Bash(gh:*)
---

# MR/PR Review

Review the merge request (GitLab) or pull request (GitHub): $ARGUMENTS

## Instructions

1. **Detect the provider** from the remote:
   - Run `git remote -v` and inspect the origin host.
   - host contains `gitlab` -> use the **GitLab** commands below (`glab`).
   - host contains `github` -> use the **GitHub** fallback (`gh`).
   - ambiguous or no remote -> default to **GitLab**.

2. **Get the change details**:
   - GitLab: `glab mr view $ARGUMENTS` and `glab mr diff $ARGUMENTS`
   - GitHub: `gh pr view $ARGUMENTS` and `gh pr diff $ARGUMENTS`

3. **Read review standards**:
   - Read `.claude/agents/engineering-reviewer.md` for the review checklist

4. **Apply the checklist** to all changed files:
   - Numerical correctness (shapes, dtypes, loss order)
   - Reproducibility (seeds, configs, determinism)
   - Data integrity (no leakage, consistent preprocessing)
   - Training loop correctness (eval mode, gradient handling)
   - Python quality (type hints, pathlib, logging)
   - Test coverage for new components

5. **Provide structured feedback**:
   - **Critical**: Must fix before merge (correctness bugs, data leakage, reproducibility failures)
   - **Warning**: Should fix (missing validation, hardcoded values, performance)
   - **Suggestion**: Nice to have (naming, docs, efficiency)

6. **Post review comments**:
   - GitLab: `glab mr note $ARGUMENTS -m "<comment>"`
   - GitHub: `gh pr comment $ARGUMENTS --body "<comment>"`
