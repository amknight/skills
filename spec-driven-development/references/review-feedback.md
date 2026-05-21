# Review and feedback workflow

Use this after an implementation sub-process finishes in a worktree.

## GPT code review

Run GPT 5.5 high in the same worktree.

Prompt shape:

```markdown
You are reviewing code changes for a development task.

## Worktree guardrails
Run and inspect:
- pwd
- git branch --show-current
- git worktree list
- git status --short

## Contract
<paste spec or direct implementation contract>

## Validation loop
<paste declared validation loop>

## Task
Review the diff against the default branch:
- git diff <default-branch>...HEAD
- git log --oneline <default-branch>..HEAD

Check:
1. Contract compliance: no unplanned scope or missing required behavior.
2. Correctness: edge cases, error handling, race conditions, data integrity.
3. Safety: auth, permissions, privacy, destructive behavior, rollback.
4. Tests: changed logic covered by unit/integration tests.
5. E2E proof: declared validation loop is implemented/runnable and assertions are meaningful.
6. Maintainability: follows local patterns and avoids unnecessary complexity.

Output:
- Critical issues
- Important issues
- Minor issues
- Missing tests/validation
- Questions or NEEDS DISCUSSION items
- Positive observations
```

Append the review to the workstream agent log.

## Feedback analysis and fixes

Run Claude Opus high after review. The implementing model must evaluate feedback rather than blindly accept it.

Verdicts:

| Verdict | Meaning |
|---------|---------|
| VALID — WILL FIX | Correct and worth fixing now |
| VALID — WON'T FIX | Correct but not worth changing; explain why |
| PARTIALLY VALID | Concern is real but suggested fix is wrong/incomplete |
| INVALID | Based on misunderstanding; explain with evidence |
| NEEDS DISCUSSION | Trade-off requires human decision |

Prompt shape:

```markdown
You implemented this work and received a GPT code review.

## Review feedback
<paste review>

## Contract and validation loop
<paste contract + validation loop>

## Task
1. Classify each review item with a verdict.
2. Fix VALID — WILL FIX and PARTIALLY VALID items.
3. Add/update tests for logic changes.
4. Run relevant tests.
5. Re-run or prepare the declared E2E validation loop if affected.
6. Commit focused fixes.
7. Append a structured feedback analysis to the agent log.
```

## Required feedback-analysis output

```markdown
## Feedback Analysis — <workstream>

### Summary
- Total comments:
- VALID — WILL FIX:
- VALID — WON'T FIX:
- PARTIALLY VALID:
- INVALID:
- NEEDS DISCUSSION:

### Detail
#### [REVIEW-1] <summary>
**Concern:** ...
**Verdict:** ...
**Reasoning:** ...
**Action taken:** ...

### Changes made
- `path` — change and reason

### Tests run
| Command | Result | Notes |
|---------|--------|-------|

### Items needing human attention
- ...
```
