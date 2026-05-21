---
name: clean-up-code-comments
description: >
  Remove stale, redundant, and misleading code comments while preserving valuable
  documentation. Use when the user asks to clean up comments, remove noisy comments,
  audit code comments, or reduce comment clutter. Triggers include "clean up comments",
  "remove stale comments", "audit comments", "comment cleanup", "too many comments",
  "comments are outdated", or any request to review and tidy code comments in a file,
  directory, or repo.
---

# Clean Up Code Comments

Remove noise and keep signal. Comments should explain **why**, not **what**.

## Scope

If the user provides a path or scope, use it. Otherwise infer from context (current
directory, recent files, or the full repo).

## What to remove

- Comments that restate what the code obviously does
- Comments describing behaviour that no longer exists
- References to deleted workarounds, removed features, or resolved TODOs that are clearly done
- Commented-out code with no explanation of why it's kept
- Filler comments (`// end of function`, `// loop through items`, `// constructor`)

## What to preserve

- Non-obvious intent, invariants, and business rules
- Protocol constraints and caveats
- Security notes and warnings
- Public API documentation (Javadoc, JSDoc, docstrings, etc.)
- Links or references future maintainers genuinely need
- License headers and generated-file notices
- Formatter, linter, and suppression directives (`// eslint-disable`, `// noinspection`, `@SuppressWarnings`, etc.)
- Comments that tools or build systems rely on

## What to fix rather than delete

When a comment is useful but inaccurate, update it to match the current code instead
of removing it.

## Workflow

1. **Identify files in scope.** List the files to review based on the user's input or
   context. Prioritise files with the most comments or the most recent churn.

2. **Review each file.** Check all comment styles relevant to the language: line
   comments, block comments, doc comments, and docstrings. Include test files — stale
   comments linger there often.

3. **Classify each comment** as one of:
   - **Remove** — adds no value, restates the obvious, or is stale/misleading
   - **Update** — useful intent but factually wrong or outdated
   - **Keep** — genuinely valuable

4. **Prefer making code clearer** over adding explanatory comments. Rename a variable
   or extract a function if that removes the need for a comment. Do not perform
   unrelated refactors.

5. **Apply changes.** Use precise edits — do not rewrite entire files. Group edits
   per file for efficiency.

6. **Summarise.** After completing the cleanup, provide a short summary:
   - Number of comments removed, updated, and kept
   - Files changed
   - Any comments you were unsure about (flag for the user to decide)

## Rules

- Never remove licence headers, generated-file notices, or tool directives.
- Never introduce new comments unless fixing an inaccurate one.
- Do not perform unrelated refactors — stay focused on comments.
- When in doubt, keep the comment and flag it for the user.
