# Publish API

API reference for creating, updating, and claiming bs plans.

## Base URL

```
https://app.fp.dev/bs/api
```

## Create a plan

### Anonymous (no auth)

```bash
curl -s -X POST https://app.fp.dev/bs/api/plans \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Auth System Migration",
    "html_content": "<!DOCTYPE html><html>...</html>"
  }'
```

**Response (201 Created):**

```json
{
  "slug": "a8k2m1x",
  "url": "https://app.fp.dev/bs/a8k2m1x",
  "claimToken": "ct_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "expiresAt": "2026-03-19T17:30:00.000Z",
  "action": "created"
}
```

Anonymous plans expire after 3 days. Save the `claimToken` to claim ownership later.

### Authenticated

```bash
curl -s -X POST https://app.fp.dev/bs/api/plans \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer fp_pat_xxx..." \
  -d '{
    "title": "Auth System Migration",
    "html_content": "<!DOCTYPE html><html>...</html>"
  }'
```

**Response (201 Created):**

```json
{
  "slug": "a8k2m1x",
  "url": "https://app.fp.dev/bs/a8k2m1x",
  "action": "created"
}
```

Authenticated plans don't expire and can be updated.

### Reading fp credentials

The fp CLI stores credentials at `~/.fiberplane/credentials.toml`:

```bash
FP_TOKEN=$(grep -m1 'api_token' ~/.fiberplane/credentials.toml 2>/dev/null | sed 's/.*= *"\(.*\)"/\1/')
```

If the file doesn't exist or the token is empty, fall back to anonymous publishing.

## Get plan metadata

```bash
curl -s https://app.fp.dev/bs/api/plans/a8k2m1x
```

**Response (200 OK):**

```json
{
  "slug": "a8k2m1x",
  "title": "Auth System Migration",
  "userId": "user_abc123",
  "versions": [
    { "versionNumber": 1, "createdAt": "2026-03-16T17:30:00.000Z" },
    { "versionNumber": 2, "createdAt": "2026-03-16T19:45:00.000Z" }
  ],
  "createdAt": "2026-03-16T17:30:00.000Z",
  "updatedAt": "2026-03-16T19:45:00.000Z"
}
```

This endpoint does **not** return `html_content` — the SSR page route queries the database directly for HTML rendering.

## Update a plan (new version)

Requires authentication. Must be the plan owner.

```bash
curl -s -X PUT https://app.fp.dev/bs/api/plans/a8k2m1x \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer fp_pat_xxx..." \
  -d '{
    "html_content": "<!DOCTYPE html><html>...updated...</html>"
  }'
```

**Response (200 OK):**

```json
{
  "slug": "a8k2m1x",
  "version": 2,
  "url": "https://app.fp.dev/bs/a8k2m1x",
  "action": "updated"
}
```

Each update creates a new version. Version numbers are sequential (1, 2, 3, ...).

## Delete a plan

Requires authentication. Must be the plan owner.

```bash
curl -s -X DELETE https://app.fp.dev/bs/api/plans/a8k2m1x \
  -H "Authorization: Bearer fp_pat_xxx..."
```

**Response (200 OK):**

```json
{
  "slug": "a8k2m1x",
  "action": "deleted"
}
```

Deleting a plan cascades to all versions and associated comments.

## Claim an anonymous plan

Convert an anonymous plan to an owned plan using the claim token received at creation.

```bash
curl -s -X POST https://app.fp.dev/bs/api/plans/a8k2m1x/claim \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer fp_pat_xxx..." \
  -d '{
    "claimToken": "ct_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
  }'
```

**Response (200 OK):**

```json
{
  "slug": "a8k2m1x",
  "action": "claimed"
}
```

After claiming, the plan:
- Is owned by the authenticated user
- No longer expires
- Can be updated and deleted

## Limits

| Field | Max size |
|-------|----------|
| `html_content` | 2 MB |
| `title` | 200 characters |

## Error responses

All errors return JSON with a `message` field:

```json
{
  "message": "Plan not found"
}
```

| Status | Meaning |
|--------|---------|
| 400 | Invalid request (missing fields, validation failure) |
| 401 | Authentication required |
| 403 | Not the plan owner |
| 404 | Plan not found |
| 413 | HTML content exceeds 2 MB |
