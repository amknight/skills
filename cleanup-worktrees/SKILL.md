---
name: cleanup-worktrees
description: Use when the user asks to clean up, prune, delete, or remove stale git worktrees, especially worktrees whose pull request is merged, closed, declined, or superseded. This skill performs a safe audit first, verifies PR state from the hosting provider, then removes only eligible stale worktrees.
---

# Cleanup Worktrees

Use this skill to remove stale git worktrees whose associated PR is no longer open.

## Safety rules

- Start with an audit; do not delete anything until you have a deletion plan.
- Never remove the current worktree, the default branch worktree, dirty worktrees, locked worktrees, or worktrees with unclear PR status.
- Treat PR states as stale only when verified by the provider:
  - Bitbucket: `MERGED`, `DECLINED`, or `SUPERSEDED`
  - GitHub: `CLOSED` with either merged or unmerged closure
- If no matching PR is found, skip the worktree.
- Remove worktrees by default; delete local branches only if the user explicitly asks.

## Workflow

1. Confirm you are in a git repo:
   ```bash
   git rev-parse --show-toplevel
   ```

2. List worktrees:
   ```bash
   git worktree list --porcelain
   ```

3. For each non-current worktree, collect:
   - path
   - branch name
   - cleanliness: `git -C <path> status --porcelain`
   - lock/prunable info from `git worktree list --porcelain`

4. Skip immediately if:
   - it is the current worktree
   - it has no branch or is detached
   - branch is the default branch (`main`, `master`, or repo default)
   - `status --porcelain` is non-empty
   - it is locked

5. Verify PR state for each remaining branch.

   **Bitbucket / Atlassian repos:** use TWG, discovering exact syntax from live help if needed:
   ```bash
   twg bitbucket pull-requests query --source <branch> --state MERGED --output json --agent-fields @compact
   twg bitbucket pull-requests query --source <branch> --state DECLINED --output json --agent-fields @compact
   twg bitbucket pull-requests query --source <branch> --state SUPERSEDED --output json --agent-fields @compact
   ```

   **GitHub repos:** use `gh`:
   ```bash
   gh pr list --head <branch> --state all --json number,title,state,mergedAt,url,headRefName
   ```

6. Present a concise plan:
   ```text
   Will remove:
   - <path>  branch=<branch>  pr=<url>  state=<MERGED|DECLINED|SUPERSEDED|CLOSED>

   Skipped:
   - <path>  branch=<branch>  reason=<dirty|current|no PR|open PR|locked|default branch>
   ```

7. Ask for confirmation before deletion unless the user explicitly said to proceed without prompting.

8. Remove eligible worktrees:
   ```bash
   git worktree remove <path>
   git worktree prune
   ```

9. If the user explicitly requested local branch cleanup too, delete only branches whose worktree was removed and whose stale PR was verified:
   ```bash
   git branch -d <branch>
   ```
   Use `git branch -D` only after explicitly explaining why `-d` failed and getting confirmation.

10. Finish with a summary of removed and skipped worktrees.
