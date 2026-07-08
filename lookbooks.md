# Lookbooks

Lookbooks are curated collections of inspiration and destination content attached to a file
(trip / tour request) — presentation documents shared with a client, distinct from the dated
day-by-day [Itineraries](itineraries.md). This endpoint exposes their **metadata** (title and
who they were prepared for). The full content of a lookbook is not part of this API.

This is a **read-only** API: a file's lookbooks can be listed and retrieved individually.
There is no search, create, or update.

Lookbooks are nested under a file: `/acc/{account_id}/files/{file_id}/lookbooks`. The
`{account_id}` must be an account you are an agent member of (otherwise `404`; a token bound
to a different account returns `403`). The `{file_id}` must be a file you can see in that
account — a file that does not exist, or one you are not permitted to view, returns `404`
(the same visibility rules as [Files](files.md)).

Itineraries are a separate document type with their own endpoint — see
[Itineraries](itineraries.md). The lookbooks endpoint never returns itineraries.

## Versioning (v2 / v3)

Safari Portal is migrating lookbooks from a legacy engine (**v2**) to a new one (**v3**).
Each lookbook object carries a `version` field (`"v2"` or `"v3"`) telling you which engine
produced it. An account is served by exactly one engine at a time, so within a single account
every lookbook reports the same `version`; it flips from `v2` to `v3` when the account is
migrated. The object shape is identical across versions. Treat `id` as an opaque string.

## The lookbook object

```json
{
  "id": "12345",
  "version": "v2",
  "file_id": "42",
  "title": "Kenya Inspiration",
  "prepared_for": "The Smith Family",
  "created_at": "2026-06-30T14:12:05.000Z",
  "updated_at": "2026-07-01T09:30:00.000Z"
}
```

### Lookbook object fields

| Field          | Type              | Description                                                    |
| -------------- | ----------------- | ------------------------------------------------------------- |
| `id`           | string            | Unique identifier for the lookbook                            |
| `version`      | string            | Which engine produced it: `"v2"` or `"v3"`                     |
| `file_id`      | string            | ID of the file (trip / tour request) this lookbook belongs to |
| `title`        | string \| null    | Internal title of the lookbook                                |
| `prepared_for` | string \| null    | Who the lookbook was prepared for                             |
| `created_at`   | string (ISO 8601) | When the lookbook was created                                 |
| `updated_at`   | string (ISO 8601) | When the lookbook was last updated                            |

## Scopes

| Scope        | Endpoints        |
| ------------ | ---------------- |
| `files:read` | List, Retrieve   |

Lookbooks reuse the [Files](files.md) scope: a token that can read a file can read its
lookbooks. There is no dedicated lookbooks scope and no write scope.

## Endpoints

| Method | Path                                                 | Scope        | Description               |
| ------ | ---------------------------------------------------- | ------------ | ------------------------- |
| `GET`  | `/acc/{account_id}/files/{file_id}/lookbooks`        | `files:read` | List a file's lookbooks   |
| `GET`  | `/acc/{account_id}/files/{file_id}/lookbooks/{id}`   | `files:read` | Retrieve a single lookbook |

## List a file's lookbooks

```
GET /acc/{account_id}/files/{file_id}/lookbooks
```

Requires `files:read`. Returns the file's lookbooks ordered newest first (by `created_at`
descending). Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/lookbooks" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "12345",
      "version": "v2",
      "file_id": "42",
      "title": "Kenya Inspiration",
      "prepared_for": "The Smith Family",
      "created_at": "2026-06-30T14:12:05.000Z",
      "updated_at": "2026-07-01T09:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 1,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 1,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

## Retrieve a single lookbook

```
GET /acc/{account_id}/files/{file_id}/lookbooks/{id}
```

Requires `files:read`. Returns one lookbook. The `{id}` must belong to the file **in the
account's active engine** — an id from the other engine (or an unknown id) returns `404`.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/lookbooks/{id}" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "12345",
    "version": "v2",
    "file_id": "42",
    "title": "Kenya Inspiration",
    "prepared_for": "The Smith Family",
    "created_at": "2026-06-30T14:12:05.000Z",
    "updated_at": "2026-07-01T09:30:00.000Z"
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                                       |
| ------ | ----------------------------------------------------------------------------- |
| `200`  | Success                                                                        |
| `403`  | The token lacks the `files:read` scope                                         |
| `404`  | The file or lookbook does not exist, belongs to a different account, or you may not see it |
