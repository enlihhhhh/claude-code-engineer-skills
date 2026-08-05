---
description: Generate a summary (MR/PR body) for the current branch changes
allowed-tools: Bash(git:*)
---

# MR/PR Summary

Generate a merge request (GitLab) or pull request (GitHub) summary for the current branch.

## Instructions

1. **Analyze changes**:
   ```bash
   git log main..HEAD --oneline
   git diff main...HEAD --stat
   ```

2. **Generate summary** with:
   - Brief description of what changed and why
   - List of files modified
   - Whether this is a feature, fix, refactor, or chore
   - Motivation or context (if applicable)
   - Breaking changes or migration notes (if any)

3. **Format as an MR/PR body**:
   ```markdown
   ## Summary
   [1-3 bullet points describing the changes]

   ## Motivation
   [Why this change is needed — bug report, feature request, or improvement goal]

   ## Changes
   - [List of significant changes]

   ## Test Plan
   - [ ] `pytest` passes
   - [ ] `ruff check .` clean
   - [ ] Verified on sample data
   - [ ] [Additional testing items specific to the change]
   ```

The body above works as-is for both GitLab (`glab mr create --description`) and
GitHub (`gh pr create --body`). This command only drafts the text — creating the
MR/PR is left to you (see the `git-workflow` agent for the ready-to-run commands).
