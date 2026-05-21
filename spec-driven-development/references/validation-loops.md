# Validation loops

The workflow must identify the best available end-to-end proof loop **upfront**. This loop becomes part of the spec and implementation contract.

## Upfront question

Ask:

> What end-to-end validation loop can we use for this task? Examples: REST API smoke test, browser automation, CLI smoke command, integration test, product-specific eval, deploy verification, synthetic monitor, or a manual QA script. If there isn't one yet, should I design a lightweight one as part of the task?

## What to capture

| Field | Why it matters |
|-------|----------------|
| Validation type | Determines tools, model prompt, and risk level |
| Environment/site/tenant | Avoids tests accidentally hitting prod or the wrong tenant |
| Auth/credentials | Tells agents how to run or why they cannot run the loop |
| Setup/seed data | Makes the test repeatable |
| Exact command or flow | Prevents vague "test it" handoffs |
| Success criteria | Defines pass/fail objectively |
| Mutation/cleanup | Prevents leaving test data behind |
| Blocked fallback | Lets reviewers understand residual risk |

## Loop types

### REST API smoke test

Use when the change exposes or depends on an HTTP API.

Capture:

```markdown
- Method/path:
- Environment/base URL:
- Auth mechanism:
- Request body/query:
- Expected status:
- Expected response assertions:
- Cleanup command if mutating:
```

Example command shape:

```bash
curl -sS -X POST "$BASE_URL/path" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  --data @/tmp/request.json | jq .
```

### Browser automation

Use when success is visible in UI or requires frontend/user interaction.

Preferred tools, in order:

1. Existing repo E2E framework (Playwright, Cypress, Webdriver, product-specific harness)
2. `agent-browser` skill/CLI for exploratory browser automation
3. Manual QA script only when automation is not feasible

Capture:

```markdown
- URL:
- User/role:
- Starting state:
- Steps:
- Assertions:
- Screenshots/video expected:
- Cleanup:
```

### CLI smoke check

Use when the change exposes a CLI command or developer workflow.

Capture:

```markdown
- Command:
- Required config/auth:
- Expected stdout/stderr:
- Expected exit code:
- JSON/schema assertions if machine-readable:
```

### Integration or contract test

Use when the repo already has deterministic integration tests and live E2E is too expensive.

Capture:

```markdown
- Test command:
- Test class/file/scenario:
- External dependencies:
- Assertions covered:
- Gaps vs real E2E:
```

### Deploy or runtime verification

Use when the change needs a deployed environment.

Capture:

```markdown
- Deployment target:
- Build/deploy command or pipeline:
- Feature flag/config state:
- Health check:
- Log/metric query:
- Rollback trigger:
```

### No existing loop

If no loop exists, propose the smallest credible loop:

1. Prefer a smoke test that exercises the changed boundary.
2. If that is not possible, add a deterministic integration/contract test.
3. If automation is not possible, produce a manual QA script with exact steps and evidence requirements.

Record this as:

```markdown
Validation loop: Proposed — <type>
Reason no existing loop is available: <reason>
Residual risk: <what this does not prove>
Reviewer approval needed: yes
```

## Execution rules

- Run the declared loop after implementation and review fixes.
- Save command output, screenshots, videos, or links as evidence where feasible.
- If credentials or environment access block execution, document the blocker and ask whether to proceed with a surrogate.
- Do not silently downgrade E2E validation to unit tests.
