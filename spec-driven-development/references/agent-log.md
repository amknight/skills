# Agent log

Use per-workstream logs to preserve implementation, review, feedback, and validation evidence.

## Location

```text
/tmp/spec-driven-development-logs/<workstream>.log.md
```

Examples:

```text
/tmp/spec-driven-development-logs/api.log.md
/tmp/spec-driven-development-logs/frontend.log.md
/tmp/spec-driven-development-logs/worker.log.md
```

## Initialize

```bash
mkdir -p /tmp/spec-driven-development-logs
cat > /tmp/spec-driven-development-logs/<workstream>.log.md <<'EOF'
# Agent Log — <workstream>

**Task:** <title>
**Worktree:** <path>
**Branch:** <branch>
**Started:** <ISO timestamp>
**Validation loop:** <type + command/flow>

---
EOF
```

## Append after every sub-process

Append sections, never overwrite:

```bash
cat >> /tmp/spec-driven-development-logs/<workstream>.log.md <<'LOGEOF'

---
**Completed:** <ISO timestamp>

## <Section>

<structured output>
LOGEOF
```

Sections:

| Step | Header |
|------|--------|
| Implementation | `## Implementation` |
| Code review | `## Code Review` |
| Feedback fixes | `## Feedback Analysis` |
| Validation loop | `## Validation Loop` |
| Final validation | `## Final Validation` |

## Final handoff extraction

At the end, read every log and produce:

```markdown
## Development Workflow Handoff

### Workstreams
| Workstream | Branch | Status | Key changes |
|------------|--------|--------|-------------|

### Validation evidence
| Loop | Environment | Command/flow | Result | Evidence |
|------|-------------|--------------|--------|----------|

### Tests
| Workstream | Command | Result |
|------------|---------|--------|

### Open risks / NEEDS DISCUSSION
- ...

### Next steps
- ...

### Confluence documentation offer
- Offer made: yes/no
- Published report: <url or n/a>
```

After compiling this handoff, follow `final-documentation.md` if the user wants the implementation report published to Confluence.
