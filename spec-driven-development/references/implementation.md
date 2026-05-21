# Phase 2 — Implementation, review, and validation

Run this phase after the Confluence spec is approved or after the user confirms direct implementation mode with a complete brief.

## Step 1: Build the contract manifest

Create `/tmp/spec-driven-development-contract.md` with:

```markdown
# Implementation Contract

## Task
- Title:
- Tracking/slug:
- Spec page:

## Scope
- Repos/workstreams:
- In scope:
- Out of scope:

## Implementation contract
- Approach:
- Interfaces/API/schema changes:
- Feature flags/config:
- Security/privacy:
- Rollback:

## Validation loop
- Type:
- Environment:
- Setup:
- Command/flow:
- Success criteria:
- Cleanup:
- Blocked fallback:

## Worktrees
| Workstream | Repo | Worktree | Branch | Default branch |
|------------|------|----------|--------|----------------|

## Required checks
| Workstream | Unit/static | Integration | E2E/smoke |
|------------|-------------|-------------|-----------|
```

Do not launch implementation until this manifest is specific enough for sub-processes.

## Step 2: Prepare worktrees

Follow `git-workflow.md` for every repo/workstream. Initialize each agent log using `agent-log.md`.

Implementation prompts must say:

```markdown
You are already in a prepared git worktree. Do not edit the primary clone. Do not create a nested worktree.
```

## Step 3: Fan out implementation

Run one Claude Opus high subprocess per independent workstream.

Prompt template:

```markdown
You are implementing a development task in this repository/workstream.

## Worktree guardrails
Run and inspect before editing:
- pwd
- git branch --show-current
- git worktree list
- git status --short

## Contract manifest
<paste /tmp/spec-driven-development-contract.md>

## Spec / implementation brief
<paste relevant spec section or direct brief>

## Repo context
<paste discovered repo commands, relevant files, local patterns>

## Task
1. Implement only this workstream's scoped changes.
2. Follow existing codebase patterns.
3. Add/update unit/static/integration tests required by the contract.
4. Prepare the declared validation loop if this workstream owns it.
5. Run feasible tests and record skipped checks with reasons.
6. Commit focused changes.
7. Append a structured implementation report to the agent log.
```

Use `subprocesses.md` for commands.

## Step 4: Review

Run GPT 5.5 high for each workstream using `review-feedback.md`.

The review must compare against the default branch and validate implementation against the contract and declared E2E loop.

## Step 5: Feedback fixes

Run Claude Opus high for each workstream using `review-feedback.md`.

The fix pass must classify review comments, fix justified issues, run tests, and append a structured report.

## Step 6: Run validation loop(s)

Run the validation loop from the manifest. If multiple workstreams own different loops, run all required loops.

Examples:

- REST smoke command with response assertions
- Playwright/Cypress/agent-browser browser flow
- CLI smoke command with expected output
- integration/contract test suite
- deploy verification/health check

For each loop, record:

```markdown
## Validation Loop — <name>

| Field | Value |
|-------|-------|
| Type | ... |
| Environment | ... |
| Command/flow | ... |
| Result | pass/fail/skipped |
| Evidence | output path, screenshot, video, link, log snippet |
| Cleanup | completed/not needed/failed |
| Notes | ... |
```

If blocked, stop and ask the user whether to proceed with the documented fallback.

## Step 7: Final validation and handoff

Compile from agent logs:

- branches/commits,
- files changed,
- review findings and fixes,
- tests run,
- E2E validation evidence,
- skipped checks with reasons,
- open risks/NEEDS DISCUSSION,
- next steps/PR instructions.

Do not claim success without evidence from the declared validation loop or explicit user-approved fallback.

## Step 8: Offer Confluence implementation report

After the handoff is ready, ask the user:

> Would you like me to publish the final implementation report to Confluence? I can create a live doc with branches/commits, changed files, review resolution, validation evidence, rollout notes, and open risks.

If the user says yes:

1. Use `final-documentation.md`.
2. Fill `implementation-report-template.html` from the contract and agent logs.
3. Default to creating the report as a child of the Phase 1 spec page when a spec page exists; otherwise ask for a Confluence parent page.
4. Publish with TWG CLI using `--body-format html`.
5. Return the Confluence URL and summarize what was included.

Do not publish final documentation without explicit confirmation.
