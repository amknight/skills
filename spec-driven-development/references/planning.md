# Phase 1 — Planning and Confluence spec

Use this phase when the task needs alignment before coding. Planning is optional, but it should be the default for ambiguous, risky, cross-cutting, or externally visible work.

## Decide whether to plan

Run the full planning phase when any of these are true:

- requirements are ambiguous,
- multiple repos/services/packages are involved,
- there are user-facing or production-risking behavior changes,
- the E2E validation loop needs design,
- rollout/rollback needs coordination,
- the user explicitly asks for a spec.

Skip planning only when the implementation brief already covers scope, files, acceptance criteria, validation, rollout, and branch/worktree slug — and the user agrees to skip.

## Step 1: Research

Research before asking questions:

```bash
pwd
git rev-parse --show-toplevel 2>/dev/null || true
rg -n "<keywords>" .
find . -maxdepth 3 -name 'README*' -o -name 'AGENTS.md' -o -name 'package.json' -o -name 'build.gradle*'
```

Capture:

- candidate repos/workstreams,
- relevant files/services/APIs,
- existing tests and E2E harnesses,
- existing feature flags/config patterns,
- owner/CODEOWNERS hints,
- docs or prior examples.

## Step 2: Clarify only missing information

Ask focused questions. Include the upfront E2E validation question from `validation-loops.md` if it has not already been answered.

Minimum information for a spec:

| Area | Required answer |
|------|-----------------|
| Goal | What changes for users/systems? |
| Scope | Repos, packages, services, APIs, UI surfaces |
| Non-goals | What should not change |
| Implementation contract | Approach, interfaces, data/schema changes, flags |
| Validation loop | Exact E2E/smoke/browser/CLI/integration proof or proposed surrogate |
| Rollout/rollback | How it ships safely and how to back out |
| Tracking/slug | Tracker key or branch/worktree slug |
| Confluence destination | Parent page URL/ID and space ID |

## Step 3: Draft spec with Claude Opus high

Write a prompt file and paste the full canonical template.

```bash
cat > /tmp/spec-dev-draft.prompt.md <<'EOF'
You are drafting a technical development spec.

## Requirements and research

<paste research, answers, desired validation loop, and repo context>

## Canonical template

<paste references/spec-template.html>

## Instructions

Output a Confluence HTML fragment only.
Use the exact section order, headings, and table shapes from the template.
Fill every {{PLACEHOLDER}}. Do not leave template tokens in the output.
If a section is not applicable, write "Not in scope" and explain why.
Make the E2E validation loop executable: include environment, command/flow, success criteria, and cleanup.
Do not invent facts. If something remains unknown, put it in Risks and Open Questions.
EOF
```

Run the Claude headless command from `subprocesses.md` and save stdout to:

```text
/tmp/spec-dev-draft.html
```

## Step 4: Review spec with GPT 5.5 medium

```bash
cat > /tmp/spec-dev-review.prompt.md <<'EOF'
You are reviewing a technical development spec before implementation.

## Spec

<paste /tmp/spec-dev-draft.html>

## Task

Review for:
1. Completeness: scope, non-goals, interfaces, workstreams, rollout, rollback.
2. Implementation readiness: could a subprocess implement without asking avoidable questions?
3. Validation readiness: is the E2E loop specific and executable? Are blocked fallbacks explicit?
4. Risk: security, privacy, data loss, backwards compatibility, operational risk.
5. Template compliance: section order preserved and no unfilled {{PLACEHOLDER}} tokens.
6. Confluence HTML quality: conservative HTML, no storage XML, no Markdown-only constructs.

Output:
- Must-fix issues before implementation
- Suggestions
- Approved sections
- Revised spec HTML if you found must-fix issues
EOF
```

Save stdout to:

```text
/tmp/spec-dev-review.md
```

Incorporate must-fix changes into:

```text
/tmp/spec-dev-final.html
```

## Step 5: Publish live doc

Use `confluence-spec-format.md`. Ask for destination if missing.

```bash
twg confluence page create \
  --space-id <SPACE_ID> \
  --parent-id <PARENT_PAGE_ID> \
  --title "Development Spec: <title>" \
  --body-file /tmp/spec-dev-final.html \
  --body-format html \
  --subtype live \
  --status draft \
  --mode agent \
  -o json \
  -y
```

## Step 6: Human checkpoint

Tell the user:

> The spec is published at <link>. Please review the scope, implementation contract, and E2E validation loop. Once you confirm, I will proceed to implementation using this spec as the contract.

Do not implement until the user confirms, unless they explicitly selected direct implementation mode earlier.
