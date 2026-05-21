# Confluence spec format

Use this when drafting or publishing the optional planning spec.

## Format rules

- Author the final page as a **Confluence HTML fragment**.
- Publish with TWG CLI using `--body-format html`.
- Do **not** generate Confluence storage XML.
- Do **not** rely on Markdown conversion for the final published body.
- Use the canonical template in `references/spec-template.html`.
- Preserve template section order and table shapes. If a section is not applicable, write `Not in scope` and explain why.
- Fill every `{{PLACEHOLDER}}` before publishing.

## Conservative HTML patterns

Use these patterns only unless the user specifically asks for richer formatting:

```html
<h1>Development Spec: Title</h1>
<h2>Section</h2>
<p>Paragraph</p>
<ul><li>Bullet</li></ul>
<ol><li>Step</li></ol>
<table>
  <thead><tr><th>Field</th><th>Value</th></tr></thead>
  <tbody><tr><td>Status</td><td>Draft</td></tr></tbody>
</table>
<code>inline-code</code>
<pre><code class="language-bash">command</code></pre>
<span data-type="status" data-color="yellow">Draft</span>
<div data-type="panel-info"><p>Info text</p></div>
<div data-type="panel-warning"><p>Warning text</p></div>
```

## Spec destination

Ask the user for one of:

- a Confluence parent page URL,
- a parent page ID plus space ID,
- or permission to create the spec in a team-default parent page they name.

Do not invent a destination. If the user only gives a space, ask for a parent page because live docs should be organized under a known parent.

## Publishing command

Save the final HTML to `/tmp/spec-driven-development-final.html`, then publish:

```bash
twg confluence page create \
  --space-id <SPACE_ID> \
  --parent-id <PARENT_PAGE_ID> \
  --title "Development Spec: <title>" \
  --body-file /tmp/spec-driven-development-final.html \
  --body-format html \
  --subtype live \
  --status draft \
  --mode agent \
  -o json \
  -y
```

Use the default authenticated site unless the user asks for another site.

## Final implementation report

At the end of implementation, optionally publish a separate report using `references/final-documentation.md` and `references/implementation-report-template.html`.

Default behavior:

- Ask before publishing.
- If a spec page exists, offer to create the implementation report as a child of that spec page.
- If no spec page exists, ask for a parent page URL or parent page ID + space ID.
- Include validation evidence and skipped-check reasons; do not publish a success narrative that the evidence does not support.

Command shape:

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

## Review-note insertion

After GPT review, update section `11. Review Notes` with:

- review model/date,
- must-fix issues found and addressed,
- remaining risks or questions,
- whether the E2E validation loop is specific enough to execute.
