# Comments API

API reference for fetching and interacting with plan comments.

## Base URL

```
https://app.fp.dev/bs
```

## Get comments (JSON)

**No authentication required.** Supports CORS for cross-origin fetch.

```bash
curl -s https://app.fp.dev/bs/api/plans/a8k2m1x/comments
```

With version filter:

```bash
curl -s "https://app.fp.dev/bs/api/plans/a8k2m1x/comments?version=2"
```

**Response (200 OK):**

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "css_selector": "body > div > p:nth-of-type(1)",
    "content": "This paragraph needs more detail",
    "author_name": "Alice",
    "user_id": null,
    "version_number": 1,
    "created_at": 1710612600
  }
]
```

## Get comments (Markdown)

Returns all comments as structured markdown. **No authentication required.** Two equivalent routes:

```bash
# API route
curl -s https://app.fp.dev/bs/api/plans/a8k2m1x/comments.md

# Browser-accessible SSR route
curl -s https://app.fp.dev/bs/a8k2m1x/comments.md
```

With version filter (API route only):

```bash
curl -s "https://app.fp.dev/bs/api/plans/a8k2m1x/comments.md?version=2"
```

**Response (200 OK, Content-Type: text/markdown):**

```markdown
## Comments on "Auth System Migration"

**Plan URL:** https://app.fp.dev/bs/a8k2m1x

### Paragraph 1
_Selector:_ `body > div > p:nth-of-type(1)`

- Alice (2026-03-16T17:30:00.000Z): This paragraph needs more detail
- Bob (2026-03-16T18:00:00.000Z): Agreed, let's expand the rationale
```

### Markdown format

- Top-level heading with plan title and URL
- Comments grouped by CSS selector
- Each group has a human-readable element name derived from the selector tag (e.g., "Paragraph 1", "Heading 2")
- Each comment shows: author name, ISO timestamp, content text

## Sidebar Copy button

The plan page comment sidebar includes a **Copy** button that copies all comments in the same markdown format to the clipboard. This uses the client-side `commentsToMarkdown()` function which also includes element text previews when available.

## Post a comment

```bash
curl -s -X POST https://app.fp.dev/bs/api/plans/a8k2m1x/comments \
  -H "Content-Type: application/json" \
  -d '{
    "cssSelector": "body > div > p:nth-of-type(1)",
    "content": "This needs more detail",
    "authorName": "Alice",
    "versionNumber": 1
  }'
```

**Response (201 Created):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "css_selector": "body > div > p:nth-of-type(1)",
  "content": "This needs more detail",
  "author_name": "Alice",
  "user_id": null,
  "version_number": 1,
  "created_at": 1710612600,
  "delete_token": "a1b2c3d4-..."
}
```

Anonymous comments include a `delete_token` in the response. Store this token to allow the commenter to delete their own comment later.

## Delete a comment

Requires either authentication (plan owner or comment author) or a delete token for anonymous comments:

```bash
# Authenticated deletion
curl -s -X DELETE https://app.fp.dev/bs/api/plans/a8k2m1x/comments/550e8400-... \
  -H "Authorization: Bearer fp_pat_xxx..."

# Anonymous deletion with token
curl -s -X DELETE https://app.fp.dev/bs/api/plans/a8k2m1x/comments/550e8400-... \
  -H "X-Delete-Token: a1b2c3d4-..."
```

**Response (200 OK):**

```json
{
  "deleted": true
}
```

## Limits

| Field | Max size |
|-------|----------|
| `cssSelector` | 500 characters |
| `content` | 2000 characters |
| `authorName` | 100 characters |
