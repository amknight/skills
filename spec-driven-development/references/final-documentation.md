# Final implementation documentation

At the end of Phase 2, offer to publish an implementation report to Confluence. This is optional but recommended for multi-repo, high-risk, or stakeholder-visible work.

## When to offer

Always ask after final validation/handoff is ready:

> Would you like me to publish the final implementation report to Confluence? I can create a live doc with branches/commits, changed files, review resolution, validation evidence, rollout notes, and open risks.

Default destination:

1. If a Phase 1 spec page exists, offer to create the report as a **child of the spec page**.
2. Otherwise, ask for a Confluence parent page URL or parent page ID + space ID.
3. If the user wants it somewhere else, use their requested parent.

Do not publish without confirmation.

## What to include

Use `references/implementation-report-template.html` and fill every placeholder. The report should be factual and evidence-based.

Include:

- spec link, if any,
- tracking key or branch slug,
- workstream/repo branches,
- commit hashes,
- changed file summaries,
- code-review outcome,
- feedback fixes and NEEDS DISCUSSION items,
- test commands and results,
- E2E validation loop evidence,
- screenshots/videos/output paths/links where available,
- skipped checks with reasons,
- cleanup status,
- rollout/rollback notes,
- PR/build/pipeline links,
- follow-up tasks.

Do not claim implementation success unless the declared validation loop passed or the user approved a documented fallback.

## Drafting prompt

Use Claude Opus high or the orchestrator itself to fill the template from agent logs.

```bash
cat > /tmp/spec-dev-implementation-report.prompt.md <<'EOF'
You are drafting a final implementation report for Confluence.

## Contract / spec
<paste implementation contract and spec link if any>

## Agent logs
<paste /tmp/spec-driven-development-logs/*.log.md summaries or full logs>

## Template
<paste references/implementation-report-template.html>

## Instructions
Output a Confluence HTML fragment only.
Use the exact template structure and fill every {{PLACEHOLDER}}.
If a workstream slot is not needed, write "Not in scope".
Keep results factual: command/flow, environment, result, evidence, blockers.
Do not invent PR/build links. Use "Not created" or "Not available" when absent.
EOF
```

Save final body to:

```text
/tmp/spec-dev-implementation-report.html
```

## Publishing command

```bash
twg confluence page create \
  --space-id <SPACE_ID> \
  --parent-id <PARENT_PAGE_ID_OR_SPEC_PAGE_ID> \
  --title "Implementation Report: <title>" \
  --body-file /tmp/spec-dev-implementation-report.html \
  --body-format html \
  --subtype live \
  --status current \
  --mode agent \
  -o json \
  -y
```

Use `--status draft` instead of `current` if the user wants to review before sharing.

## Final response after publishing

Tell the user:

```markdown
Published implementation report: <Confluence URL>

Included:
- branches/commits
- changed files
- review resolution
- validation evidence
- rollout/follow-up notes

Open risks:
- ...
```
