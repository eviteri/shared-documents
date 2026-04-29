---
name: jira-adf
description: Atlassian Document Format (ADF) reference for building issue descriptions and comment bodies. Use this whenever writing structured content into Jira via the REST API v3.
---

# Atlassian Document Format (ADF)

Jira Cloud REST API v3 stores rich text as ADF — a JSON tree, not Markdown or HTML. Every comment body and issue description sent via `POST` or `PUT` MUST be a valid ADF document.

## Top-level shape

```json
{
  "type": "doc",
  "version": 1,
  "content": [ /* block nodes */ ]
}
```

Required: `type: "doc"`, `version: 1`, and a `content` array of block nodes.

## Block nodes

### Paragraph

```json
{
  "type": "paragraph",
  "content": [
    { "type": "text", "text": "Hello world." }
  ]
}
```

### Heading

```json
{
  "type": "heading",
  "attrs": { "level": 2 },
  "content": [{ "type": "text", "text": "Acceptance Criteria" }]
}
```

`level`: 1–6.

### Bullet list

```json
{
  "type": "bulletList",
  "content": [
    {
      "type": "listItem",
      "content": [
        { "type": "paragraph", "content": [{ "type": "text", "text": "First item" }] }
      ]
    },
    {
      "type": "listItem",
      "content": [
        { "type": "paragraph", "content": [{ "type": "text", "text": "Second item" }] }
      ]
    }
  ]
}
```

### Ordered list

Same shape as `bulletList`, but `"type": "orderedList"`.

### Code block

```json
{
  "type": "codeBlock",
  "attrs": { "language": "bash" },
  "content": [{ "type": "text", "text": "npm install" }]
}
```

### Block quote

```json
{
  "type": "blockquote",
  "content": [
    { "type": "paragraph", "content": [{ "type": "text", "text": "Quoted line." }] }
  ]
}
```

### Rule (horizontal divider)

```json
{ "type": "rule" }
```

### Panel (callout box)

```json
{
  "type": "panel",
  "attrs": { "panelType": "info" },
  "content": [
    { "type": "paragraph", "content": [{ "type": "text", "text": "FYI message." }] }
  ]
}
```

`panelType`: `info`, `note`, `warning`, `success`, `error`.

## Inline marks (apply to text nodes)

```json
{
  "type": "text",
  "text": "important",
  "marks": [{ "type": "strong" }]
}
```

| Mark | Effect |
|------|--------|
| `strong` | Bold |
| `em` | Italic |
| `code` | Inline code |
| `strike` | Strikethrough |
| `underline` | Underline |
| `link` | `{ "type": "link", "attrs": { "href": "https://..." } }` |

Multiple marks can be combined in the same array.

## Mentions

```json
{
  "type": "mention",
  "attrs": {
    "id": "5b10ac8d82e05b22cc7d4ef5",
    "text": "@Alice"
  }
}
```

`id` is the user's `accountId` (from `GET /rest/api/3/user/search`). Place the mention node inline inside a paragraph alongside text nodes.

## Hard break (line break inside a paragraph)

```json
{ "type": "hardBreak" }
```

## Complete example — comment with heading, bullets, mention, and code

```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 3 },
      "content": [{ "type": "text", "text": "Review feedback" }]
    },
    {
      "type": "paragraph",
      "content": [
        {
          "type": "mention",
          "attrs": { "id": "5b10ac8d82e05b22cc7d4ef5", "text": "@Alice" }
        },
        { "type": "text", "text": " — please look at the items below." }
      ]
    },
    {
      "type": "bulletList",
      "content": [
        {
          "type": "listItem",
          "content": [
            {
              "type": "paragraph",
              "content": [{ "type": "text", "text": "Add error handling for empty state" }]
            }
          ]
        },
        {
          "type": "listItem",
          "content": [
            {
              "type": "paragraph",
              "content": [{ "type": "text", "text": "Confirm AC for unauthenticated users" }]
            }
          ]
        }
      ]
    },
    {
      "type": "codeBlock",
      "attrs": { "language": "javascript" },
      "content": [{ "type": "text", "text": "if (!user) return redirect('/login');" }]
    }
  ]
}
```

## Reading ADF (descriptions returned from GET)

When summarizing an issue, the description comes back as ADF. To convert to readable text:

1. Walk `content` recursively
2. Concatenate `text` from `text` nodes
3. Insert `\n` between block nodes
4. Render headings with `#` prefix, list items with `- `, code blocks with backtick fences

Tip: `GET /rest/api/3/issue/{key}?expand=renderedFields` returns HTML-rendered fields alongside ADF — easier for summaries when ADF traversal is overkill.

## Validation rules

- The root MUST be `{ "type": "doc", "version": 1, "content": [...] }`
- `content` arrays must contain at least one node (no empty paragraphs at top level)
- Marks apply only to `text` nodes
- `panel`, `bulletList`, `orderedList`, `blockquote` always wrap block nodes (paragraphs, not raw text)

If a `POST` returns 400 with a body referencing "ADF" or "doc", you have a malformed tree — most often a raw string where a node was expected.
