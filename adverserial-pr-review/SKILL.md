---
name: adverserial-pr-review
description: >
  Conduct thorough, diligent code reviews of pull requests; use when user asks for adverserial review.
---

# Thorough PR Code Review

Conduct a comprehensive, diligent code review. Your review should be as thorough as an experienced senior engineer who cares deeply about code quality, correctness, and maintainability. Leave no stone unturned.

## Core Principles

1. **Be thorough.** Read every line of every changed file. Do not skim.
2. **Understand context.** Expand surrounding code, trace call sites, check how changes integrate with the broader codebase.
3. **Find real problems.** Prioritise bugs, logic errors, and correctness issues over style nitpicks.
4. **Be constructive.** Every comment should be actionable with a clear explanation of why it matters.
5. **Never stop early.** Continue calling functions until you have fully completed your review. Do not stop to ask the user questions mid-review.

## Obtain the Diff

Determine how to get the changes:

**If a Bitbucket or GitHub PR URL is provided:**
Use Github (GH CLI) or Bitbucket (BB / TWG CLI) CLI tools to fetch the PR diff. 
For large changes, fetch the full branch into a separate worktree and use `git diff` to review locally.

**If reviewing local changes:**
1. Run `git diff` to check for uncommitted local changes`
2. If no uncommitted changes, identify the branch:
   - Run `git remote show origin` to identify the default branch
   - Run `git diff <default-branch>...HEAD` to see changes from the default branch to the current branch
3. Run `git log --oneline <default-branch>..HEAD` to understand the commit history

## Adverserial Review Process

* Enumerate all code paths that were touched by the PR, ensure you have a deep understanding of the domain that the changes are within.
* Assume that there is a better approach, expand your thinking to a higher level to consider the architectural implications of the changes, and whether there are better ways to achieve the same goal.
* Look for anti-patterns in the code - are there any code smells, or signs of technical debt being introduced?
* Consider edge cases and error handling - are there any scenarios where the code might fail or behave unexpectedly?
* Deeply consider the potential risk to this change, and whether it may introduce new regressions.
