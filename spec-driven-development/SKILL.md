---
name: spec-driven-development
description: >-
  Generic agentic development workflow for software changes that benefit from an optional plan/spec,
  Confluence live-doc specification, parallel implementation sub-processes with different models,
  code review, feedback fixes, explicit end-to-end validation loops, and optional final
  implementation documentation pushed to Confluence. Use when the user asks to
  implement a feature, fix a non-trivial bug, make a cross-cutting change, create a technical spec,
  run agent sub-processes, coordinate multiple repos/workstreams, or prove a change with REST smoke
  tests, browser automation, CLI checks, integration tests, or other E2E validation.
---

# Spec-driven development workflow

Use this skill to run a **generic development workflow** that can plan, spec, implement, review, and validate software work. It is intentionally not tied to any specific product surface.

The orchestrator coordinates the work; model-heavy steps run in **headless sub-processes** using pi, Claude Code, or Codex with explicit model/thinking choices.

---

## When to use

Use this skill for development tasks where any of these are true:

- The task has unclear scope, multiple possible designs, or needs stakeholder alignment.
- The user wants a technical spec in Confluence before implementation.
- The task spans multiple repos, packages, services, or workstreams.
- The task should be implemented by sub-processes using different models for planning, implementation, review, and validation.
- Success requires a proof loop beyond unit tests, such as REST API smoke tests, browser automation, CLI smoke checks, integration tests, deploy verification, or synthetic checks.

## When not to use

- Tiny single-file edits where the user wants an immediate change and no orchestration.
- Pure investigation with no planned implementation.
- Tasks where the user explicitly says not to create plans/specs or sub-processes.

---

## Operating model

```text
User goal
  │
  ├─► Intake: classify task + ask for E2E validation loop upfront
  │
  ├─► Optional Phase 1 — Plan/spec
  │     ├─ Research code/docs
  │     ├─ Draft Confluence HTML spec — Claude Opus high
  │     ├─ Review spec — GPT 5.5 medium
  │     └─ Publish live doc to chosen Confluence parent
  │
  └─► Phase 2 — Implement/validate
        ├─ Freeze implementation contract
        ├─ Prepare/reuse git worktrees
        ├─ Fan out implementation — Claude Opus high per workstream
        ├─ Review changes — GPT 5.5 high per workstream
        ├─ Analyse/fix feedback — Claude Opus high
        ├─ Run declared validation loop(s)
        ├─ Handoff with evidence and open risks
        └─ Offer Confluence implementation report
```

---

## Phase 0 — Intake and validation loop discovery

Before planning or implementation, ask the user for the proof loop that will demonstrate the change works end-to-end.

Ask this explicitly:

> "What end-to-end validation loop can we use for this task? Examples: REST API smoke test, browser automation, CLI smoke command, integration test, product-specific eval, deploy verification, synthetic monitor, or a manual QA script. If there isn't one yet, should I design a lightweight one as part of the task?"

Capture:

- desired validation type(s),
- environment/site/tenant to use,
- credentials or auth assumptions,
- setup/seed data,
- exact command or browser flow if known,
- success criteria,
- whether the loop may mutate data,
- cleanup requirements,
- what to do if the loop is blocked.

If the user cannot provide a loop, propose the smallest credible surrogate and mark it as `proposed` in the spec/contract. Do not treat unit tests alone as E2E proof unless the user explicitly accepts that limitation.

**Full instructions:** `references/validation-loops.md`

---

## Phase 1 — Optional plan/spec

Default to a spec when the task is ambiguous, risky, multi-repo, externally visible, or has a non-trivial validation loop. Skip only when the user provides a complete implementation brief and agrees to proceed directly.

**Full instructions:** `references/planning.md`

Summary:

1. Research relevant code/docs and identify candidate files, services, APIs, owners, and existing tests.
2. Ask focused clarification questions. Do not ask what you can determine from the repo.
3. Draft a Confluence HTML spec using `references/spec-template.html`.
4. Review the spec with GPT 5.5.
5. Publish as a Confluence **live doc** using TWG CLI under the parent page chosen by the user.
6. Wait for human confirmation before implementing unless the user explicitly approved skip-plan/direct mode.

---

## Phase 2 — Implement, review, and validate

**Full instructions:** `references/implementation.md`

Summary:

1. Parse the spec or direct implementation brief.
2. Freeze a contract manifest: scope, branches, workstreams, acceptance criteria, validation loop(s), and rollback/cleanup notes.
3. Prepare one git worktree per repo/workstream. Do not edit primary clones.
4. Launch implementation sub-processes with Claude Opus high.
5. Launch code-review sub-processes with GPT 5.5 high.
6. Launch feedback/fix sub-processes with Claude Opus high.
7. Run the declared validation loop(s), including REST/browser/CLI/integration checks as appropriate.
8. Produce a validation evidence summary and flag unresolved risks.
9. Offer to publish the final implementation report to Confluence with branches, commits, review resolution, validation evidence, and follow-up notes.

---

## Agent runtime and model assignments

Use headless pi (`--print --no-session`) as the default runtime. Claude Code (`claude --print`) and Codex (`codex --quiet`) are supported alternatives. Read `references/subprocesses.md` for exact commands.

| Step | Default model | Thinking | Purpose |
|------|---------------|----------|---------|
| Spec drafting | Claude Opus 4.6 | high | Create the first spec from research and requirements |
| Spec review | GPT 5.5 | medium | Challenge completeness, ambiguity, and testability |
| Implementation | Claude Opus 4.6 | high | Make code changes in a prepared worktree |
| Code review | GPT 5.5 | high | Review implementation against the contract and repo norms |
| Feedback fixes | Claude Opus 4.6 | high | Evaluate review feedback and fix justified issues |
| Validation design/fixes | Claude Opus 4.6 | high | Build/run E2E checks and fix failures |

**Full commands:** `references/subprocesses.md`

---

## References

| File | When to read |
|------|-------------|
| `references/planning.md` | Running optional Phase 1 planning/spec creation |
| `references/spec-template.html` | Canonical Confluence HTML spec template |
| `references/confluence-spec-format.md` | HTML rules and TWG publishing command |
| `references/implementation.md` | Phase 2 implementation/review/fix workflow |
| `references/final-documentation.md` | Optional final implementation report publishing flow |
| `references/implementation-report-template.html` | Canonical Confluence HTML implementation report template |
| `references/subprocesses.md` | Headless pi, Claude Code, and Codex commands and prompt/output patterns |
| `references/validation-loops.md` | Upfront validation-loop question and loop templates |
| `references/git-workflow.md` | Required worktree setup and branch naming |
| `references/review-feedback.md` | GPT review + Claude feedback-analysis workflow |
| `references/agent-log.md` | Per-workstream log format and handoff evidence |

---

## Agent instructions

- Start by discovering the available E2E validation loop. Make it part of the contract, not an afterthought.
- Prefer a spec for ambiguous or risky tasks, but allow direct implementation when the brief is complete and the user agrees.
- Publish specs as Confluence live docs in the user-selected space/parent using HTML, not Markdown or storage XML.
- Use headless sub-processes for unattended model-heavy work. Use prompt files and output files; avoid huge inline prompts.
- Implement only in git worktrees. Reuse existing worktrees when they match the task.
- Keep validation factual: command/flow, environment, result, evidence link/file, cleanup, and blockers.
- At final handoff, offer to publish an implementation report to Confluence. Do not publish without confirmation.
- If validation is blocked, say exactly why and ask whether to proceed with a weaker surrogate or stop.
