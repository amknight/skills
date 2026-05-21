# Git workflow

Implementation must happen in git worktrees unless the user explicitly asks to edit the current checkout.

## Rules

1. Check for existing worktrees before creating a new one.
2. Branch from latest `origin/<default-branch>`, not a stale local branch.
3. Use a tracker key in the branch if one exists; otherwise use `feature/<slug>`.
4. Create one worktree per repo or independent workstream.
5. Launch sub-processes inside the worktree path only.

## Setup flow

From the primary clone:

```bash
git status --short
git worktree list
git fetch origin
DEFAULT_BRANCH=$(git remote show origin | sed -n 's/.*HEAD branch: //p')
```

Choose names:

```text
With tracker: issue/<KEY>-<slug>
No tracker:   feature/<slug>
Worktree:     ../<repo>-<slug>
```

Create:

```bash
git worktree add ../<repo>-<slug> \
  -b <branch-name> \
  origin/<default-branch>
```

Verify:

```bash
cd ../<repo>-<slug>
pwd
git branch --show-current
git status --short
git worktree list
```

## Reuse flow

If a matching worktree already exists:

```bash
cd /path/to/existing-worktree
git status --short
git fetch origin
# Rebase only if appropriate for this repo and branch state:
git rebase origin/<default-branch>
```

Resolve conflicts before launching sub-processes.

## Contract manifest fields

Record for each worktree:

- repo path,
- worktree path,
- default branch,
- feature branch,
- implementation owner/subprocess,
- expected validation commands,
- cleanup/removal instructions.
