---
name: pr-review
description: >
  Conduct thorough, diligent code reviews of pull requests. Use when the user asks
  to review a PR, check a PR for problems, review someone's code changes, or audit
  a pull request. Triggers include "review this PR", "review this pull request",
  "check this PR for problems", "find any issues in this PR", "code review",
  "review the diff", "look at this PR", "any problems with this PR".
---

# Thorough PR Code Review

Conduct a comprehensive, diligent code review. Your review should be as thorough as an experienced senior engineer who cares deeply about code quality, correctness, and maintainability. Leave no stone unturned.

## Core Principles

1. **Be thorough.** Read every line of every changed file. Do not skim.
2. **Understand context.** Expand surrounding code, trace call sites, check how changes integrate with the broader codebase.
3. **Find real problems.** Prioritise bugs, logic errors, and correctness issues over style nitpicks.
4. **Be constructive.** Every comment should be actionable with a clear explanation of why it matters.
5. **Never stop early.** Continue calling functions until you have fully completed your review. Do not stop to ask the user questions mid-review.

## Workflow

### Step 1: Obtain the Diff

Determine how to get the changes:

**If a Bitbucket PR URL is provided:**
1. Use `get_pr_diff` from the rovodev MCP tools to fetch the full diff
2. If the diff is large, also use the Bitbucket MCP tools to get the PR metadata (title, description) for context

**If reviewing local changes:**
1. Run `git diff` to check for uncommitted local changes
2. If no uncommitted changes, identify the branch:
   - Run `git remote show origin` to identify the default branch
   - Run `git diff <default-branch>...HEAD` to see changes from the default branch to the current branch
3. Run `git log --oneline <default-branch>..HEAD` to understand the commit history

### Step 2: Understand the Intent

Before reviewing line-by-line:
1. Read the PR title and description (if available) to understand what the change is trying to achieve
2. Scan the full diff to get a high-level picture — which files changed, what's the scope
3. Identify the type of change: bug fix, new feature, refactor, configuration change, test addition, etc.

### Step 3: Deep File-by-File Review

For **each modified file**, perform this analysis:

#### 3a. Examine Changes in Context
- Open each modified file using `open_files` or `expand_code_chunks`
- For each hunk in the diff, expand the **surrounding code** — not just the changed lines. You need enough context to understand:
  - What the function/method does overall
  - How the changed code fits into the larger flow
  - What callers expect from this code
- Trace the impact: if a function signature changed, find all call sites. If a data structure changed, find all consumers.

#### 3b. Check Each Change Against This Checklist

**Correctness & Logic:**
- [ ] Does the logic correctly implement the stated intent?
- [ ] Are all code paths handled, including edge cases?
- [ ] Are error conditions handled properly?
- [ ] Are there any off-by-one errors?
- [ ] Are boolean conditions correct? (watch for inverted logic, missing negations)
- [ ] For conditional changes: are both the `if` and `else` branches correct?

**Null Safety & Error Handling:**
- [ ] Could any value be null/undefined where it's not expected?
- [ ] Are exceptions/errors caught and handled appropriately?
- [ ] Are error messages clear and useful for debugging?
- [ ] Are resources properly closed/released in error paths?

**Concurrency & Thread Safety:**
- [ ] Are shared mutable state accesses properly synchronised?
- [ ] Could race conditions occur? (check-then-act patterns, non-atomic operations)
- [ ] Are concurrent data structures used where needed?
- [ ] For reactive/async code: are subscriptions properly managed?

**API & Contract Changes:**
- [ ] Do changes to public APIs maintain backward compatibility?
- [ ] Are new parameters validated?
- [ ] Do return types and values match what callers expect?
- [ ] Are serialization/deserialization contracts maintained?

**Performance:**
- [ ] Are there any obvious N+1 query patterns or unnecessary loops?
- [ ] Are large collections being fully materialised when they don't need to be?
- [ ] Are there blocking calls in async/reactive contexts?
- [ ] Are there unnecessary object allocations in hot paths?

**Security:**
- [ ] Is user input properly validated and sanitised?
- [ ] Are there any injection risks (SQL, XSS, command injection)?
- [ ] Are sensitive data (tokens, passwords) properly handled?
- [ ] Are authorisation checks in place for protected operations?

**Code Quality:**
- [ ] Are there any unused variables, imports, or dead code introduced?
- [ ] Is there duplicated code that could be extracted into a reusable function?
- [ ] Do new functions/methods follow naming conventions used elsewhere in the codebase?
- [ ] Are magic numbers/strings extracted into named constants?
- [ ] Is the code readable and self-documenting?

**Testing:**
- [ ] Are there tests covering the changed logic?
- [ ] Do tests cover both happy paths and error/edge cases?
- [ ] Are test assertions meaningful (not just checking that code runs without error)?
- [ ] For bug fixes: is there a regression test that would catch the original bug?
- [ ] Are mocks/stubs set up correctly and verifying the right behaviour?

### Step 4: Understand the Broader Codebase Context

This step is **mandatory**, not optional. You must go beyond the diff and deeply understand how the changes fit into the wider system.

#### 4a. Architectural Understanding
- **Trace the full call chain.** For every modified function/method, search the codebase for all callers and callees. Understand where this code sits in the overall architecture.
- **Identify the module/layer boundaries.** Does this change respect the existing architectural layering? (e.g., does a service-layer change leak persistence concerns, does a controller do business logic it shouldn't?)
- **Check dependency direction.** Are new imports/dependencies pointing in the right direction, or do they introduce circular dependencies or violate module boundaries?
- **Understand the data flow.** Trace how data enters the system, flows through the modified code, and exits. Look for places where assumptions about data shape or state could break.

#### 4b. Codebase Pattern Consistency
- **Search for existing patterns.** Use `grep` and `search_code` extensively to find how similar problems are solved elsewhere in the codebase. If the PR introduces a new pattern where an established one exists, flag it.
- **Check for existing utilities.** Search for helper methods, utility classes, or shared abstractions that already do what new code is doing. Duplicated logic is a common source of future bugs when one copy gets updated but the other doesn't.
- **Verify naming conventions.** Search for how similar classes, methods, constants, and packages are named in the codebase. Inconsistent naming makes code harder to discover and maintain.
- **Look at how similar features were implemented.** If the PR adds a new endpoint, check how other endpoints in the same service are structured. If it adds a new event handler, check existing handlers for the expected pattern.

#### 4c. Impact Analysis
- **Search for similar patterns that may need the same change.** If the PR fixes a bug in one place, search for the same pattern elsewhere — the same bug may exist in other locations.
- **Check configuration and wiring.** For changes to dependency injection, configuration, or service wiring, verify that the full chain is correctly connected.
- **Verify backward compatibility.** For API changes, search for all consumers. For database schema changes, check migration ordering and rollback safety. For event/message format changes, check all producers and consumers.
- **Assess blast radius.** If this change breaks, what else breaks? How many users/features are affected? The riskier the blast radius, the higher the bar for correctness and test coverage.

#### 4d. Test Coverage Verification
- **Check that modified functions have corresponding test files** with adequate coverage.
- **Search for integration/functional tests** that exercise the modified code paths end-to-end.
- **Verify test fixtures and mocks** accurately represent real-world data and behaviour.

### Step 5: Compile and Present Findings

Present your review in a structured format:

```
## PR Review: [PR Title or brief summary]

### Summary
[1-2 sentence overall assessment — is this PR ready to merge, needs minor fixes, or has significant issues?]

### Critical Issues (Must Fix)
These issues must be addressed before merging.

#### [CRITICAL-1] [Short description]
**File:** `path/to/file.kt` (line X)
**Issue:** [Clear explanation of the problem]
**Impact:** [What could go wrong if this ships]
**Suggestion:** [Concrete fix or approach]
```diff
- problematic code
+ suggested fix
```

### Important Issues (Should Fix)
These should be addressed but are not blocking.

#### [IMPORTANT-1] ...

### Minor Issues (Consider)
Nice-to-have improvements.

#### [MINOR-1] ...

### Questions
Things that aren't clearly wrong but warrant explanation from the author.

### Positive Observations
[Call out things done well — good test coverage, clean abstractions, etc.]
```

## Priority Order

Always focus your energy in this order:
1. **Bugs and logic errors** — things that would cause incorrect behaviour
2. **Security vulnerabilities** — things that could be exploited
3. **Data integrity issues** — things that could corrupt or lose data
4. **Performance problems** — things that could cause outages or degradation at scale
5. **Missing error handling** — things that could cause unhelpful failures
6. **Missing tests** — things that reduce confidence in correctness
7. **Code quality** — readability, maintainability, conventions
8. **Style and naming** — lowest priority, only mention if genuinely confusing

## Important Rules

- **Be specific.** Reference exact file names and line numbers. Include code snippets.
- **Provide fixes, not just complaints.** If you identify a problem, suggest how to fix it with a diff or code snippet suitable for find-and-replace.
- **Check your assumptions.** Before flagging something as wrong, verify by examining the codebase. What looks like a bug may be intentional.
- **Don't flag TODOs or pre-existing issues** unless the PR makes them worse.
- **Context is king.** A function that looks wrong in isolation may be correct given how it's called. Always check.
- **Distinguish between "this is wrong" and "I would do this differently".** Only flag the former as issues; frame the latter as suggestions.
- **For modified functions, always check if tests exist and cover the changes.** If tests are missing for changed logic, flag it.
- **Continue calling functions until you have fully completed your review.** NEVER stop to ask the user questions during the review. If you encounter errors, attempt to resolve them.
