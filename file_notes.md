# File Notes

Notes are the internal team notes on a file (trip / tour request) — the same notes shown on
the file's **Notes** tab in the app. They are never shown to travelers.

Notes can be listed, retrieved, and added. There is no update or delete.

Notes are nested under a file: `/acc/{account_id}/files/{file_id}/notes`. The
`{account_id}` must be an account you are an agent member of; an account you are not a
member of returns `404`, and a token bound to a different account returns `403`. The
`{file_id}` must be a file you can see in that account — a file that does not exist, or one
you are not permitted to view, returns `404` (the same visibility rules as
[Files](files.md)).

## The note object

```json
{
  "id": "981",
  "file_id": "42",
  "content": "<p>Client prefers a private vehicle in the Mara.</p>",
  "author_id": "11",
  "created_at": "2026-10-16T09:30:00.000Z",
  "updated_at": "2026-10-16T09:30:00.000Z"
}
```

### Note object fields

| Field        | Type              | Description                                                                                       |
| ------------ | ----------------- | ------------------------------------------------------------------------------------------------- |
| `id`         | string            | Unique identifier for the note                                                                    |
| `file_id`    | string            | ID of the file (trip / tour request) this note belongs to                                         |
| `content`    | string            | The note body. Notes written in the app are HTML from its rich-text editor; notes added through this API hold exactly what was sent |
| `author_id`  | string            | ID of the user who wrote the note — resolve via the [Users](users.md) endpoint                    |
| `created_at` | string (ISO 8601) | When the note was written                                                                         |
| `updated_at` | string (ISO 8601) | When the note was last edited                                                                     |

## Scopes

| Scope         | Endpoints        |
| ------------- | ---------------- |
| `files:read`  | List, Retrieve   |
| `files:write` | Create           |

Notes reuse the [Files](files.md) scopes: a token that can read a file can read its notes,
and a token that can write files can add notes to them.

## Endpoints

| Method | Path                                            | Scope         | Description           |
| ------ | ----------------------------------------------- | ------------- | --------------------- |
| `GET`  | `/acc/{account_id}/files/{file_id}/notes`       | `files:read`  | List a file's notes   |
| `GET`  | `/acc/{account_id}/files/{file_id}/notes/{id}`  | `files:read`  | Retrieve a note       |
| `POST` | `/acc/{account_id}/files/{file_id}/notes`       | `files:write` | Add a note to a file  |

## List a file's notes

```
GET /acc/{account_id}/files/{file_id}/notes
```

Requires `files:read`. Returns the file's notes ordered newest first (by `created_at`
descending). Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/notes" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "981",
      "file_id": "42",
      "content": "<p>Client prefers a private vehicle in the Mara.</p>",
      "author_id": "11",
      "created_at": "2026-10-16T09:30:00.000Z",
      "updated_at": "2026-10-16T09:30:00.000Z"
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

## Retrieve a note

```
GET /acc/{account_id}/files/{file_id}/notes/{id}
```

Requires `files:read`. Returns `{ "data": <note> }`. A note that belongs to a different
file returns `404`.

## Add a note to a file

```
POST /acc/{account_id}/files/{file_id}/notes
```

Requires `files:write`, and the authorizing member must be allowed to edit the file. The
note is authored by the member the token belongs to.

| Field     | Type   | Required | Description                                                       |
| --------- | ------ | -------- | ----------------------------------------------------------------- |
| `content` | string | yes      | The note body — HTML or plain text, stored as sent. Must not be blank |

```bash
curl -X POST "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/notes" \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{ "note": { "content": "Client prefers a private vehicle in the Mara." } }'
```

**Response `201`** — `{ "data": <note> }` for a token that also holds `files:read`;
otherwise `{ "data": { "id": "981" } }`, since echoing the record back is a read.

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                                                  |
| ------ | ---------------------------------------------------------------------------------------- |
| `200`  | Success (list, retrieve)                                                                  |
| `201`  | Note created                                                                              |
| `400`  | The body has no `note` object                                                             |
| `403`  | The token lacks the required scope, or the member may not edit the file                   |
| `404`  | The file or note does not exist, belongs to a different account, or you may not see it    |
| `422`  | `content` is blank                                                                        |
