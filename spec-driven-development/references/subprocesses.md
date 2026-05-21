# Agent sub-processes and runtimes

Use headless sub-processes for unattended spec drafting, spec review, implementation, code review, feedback fixes, and validation-loop creation/execution. Three runtimes are supported: **pi** (default), **Claude Code**, and **Codex**.

## Runtime preference

Default to **pi headless mode** for non-interactive orchestration:

- `--print` so output is captured
- `--no-session` so the subprocess exits
- explicit provider/model/thinking
- prompt files rather than huge inline prompts

Use **Claude Code** (`claude --print`) when the user prefers it or when pi is unavailable. Use **Codex** (`codex --quiet`) for OpenAI-model steps or when the user prefers it.

## Runtime detection

```bash
HAS_PI=false
command -v pi >/dev/null 2>&1 && HAS_PI=true

HAS_CLAUDE_CODE=false
command -v claude >/dev/null 2>&1 && HAS_CLAUDE_CODE=true

HAS_CODEX=false
command -v codex >/dev/null 2>&1 && HAS_CODEX=true

echo "pi: $HAS_PI | claude-code: $HAS_CLAUDE_CODE | codex: $HAS_CODEX"
```

If no runtime is available, ask the user before proceeding.

## pi commands

### Claude Opus 4.6 high

```bash
pi --provider anthropic \
  --model claude-opus-4-6 \
  --thinking high \
  --print --no-session \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

### GPT 5.5 medium

```bash
pi --provider openai \
  --model gpt-5.5 \
  --thinking medium \
  --print --no-session \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

### GPT 5.5 high

```bash
pi --provider openai \
  --model gpt-5.5 \
  --thinking high \
  --print --no-session \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

## Claude Code commands

Claude Code uses `claude --print` for headless batch execution.

### Claude Opus 4.6

```bash
claude --print \
  --model claude-opus-4-6 \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

### Claude Sonnet 4

```bash
claude --print \
  --model claude-sonnet-4-20250514 \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

## Codex commands

Codex uses `codex --quiet` for headless batch execution.

### GPT 5.5

```bash
codex --quiet \
  --model gpt-5.5 \
  "$(cat /tmp/<prompt>.md)" \
  > /tmp/<output>.log 2>&1
```

## Prompt-file pattern

Always write prompts to files:

```bash
cat > /tmp/spec-dev-<step>.prompt.md <<'EOF'
<prompt content>
EOF
```

Use output files:

```text
/tmp/spec-dev-plan-draft.html
/tmp/spec-dev-plan-review.md
/tmp/spec-dev-<workstream>-impl.log
/tmp/spec-dev-<workstream>-review.md
/tmp/spec-dev-<workstream>-feedback.log
/tmp/spec-dev-<workstream>-validation.log
```

## Parallel workstream pattern

Run each implementation subprocess from its prepared worktree:

```bash
(
  cd /path/to/worktree && \
  pi --provider anthropic --model claude-opus-4-6 --thinking high \
    --print --no-session "$(cat /tmp/spec-dev-api-impl.prompt.md)" \
    > /tmp/spec-dev-api-impl.log 2>&1
) &
API_PID=$!
```

Wait and collect statuses:

```bash
wait $API_PID; API_STATUS=$?
```

## Sub-process guardrails

Every implementation/review/fix prompt must include:

```markdown
You are in a prepared git worktree. Do not edit the primary clone. Do not create nested worktrees.
Before editing or reviewing, run:
- pwd
- git branch --show-current
- git worktree list
- git status --short

Append your structured result to the agent log path provided in the prompt.
```

## Model assignment table

| Step | Default runtime + model | Alternative |
|------|------------------------|-------------|
| Spec draft | pi: Claude Opus high | Claude Code: claude-opus-4-6 |
| Spec review | pi: GPT 5.5 medium | Codex: gpt-5.5 |
| Implementation | pi: Claude Opus high | Claude Code: claude-opus-4-6 |
| Code review | pi: GPT 5.5 high | Codex: gpt-5.5 |
| Feedback fixes | pi: Claude Opus high | Claude Code: claude-opus-4-6 |
| Validation-loop design/fix | pi: Claude Opus high | Claude Code: claude-opus-4-6 |

Model selection follows the same pattern across runtimes. If the user prefers a specific runtime, use it consistently for all steps of that model family.
