# File Sources

Sources are the account-defined lead source values used on files (trips / tour
requests) — e.g. "Referral", "Website", "Repeat Client". Use this endpoint to enumerate
the account's configured sources — for building pickers and filters — and to look up
the id to send as `source_category_id` when creating or updating a [file](files.md).
The file's own `source` field already embeds the resolved `{id, name}`, so no lookup is
needed just to read it back.

This is a **read-only** API: sources can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The source object

```json
{
  "id": "12",
  "name": "Referral"
}
```

### Source object fields

| Field  | Type   | Description                                                                                                |
| ------ | ------ | ---------------------------------------------------------------------------------------------------------- |
| `id`   | string | Unique identifier — matches the `id` in a file's `source` object, and the `source_category_id` write param |
| `name` | string | Display name (e.g. "Referral")                                                                             |

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves the values behind a file's `source` field, so it shares the
`files:read` scope.

## Endpoints

| Method | Path                             | Scope        | Description  |
| ------ | -------------------------------- | ------------ | ------------ |
| `GET`  | `/acc/{account_id}/file_sources` | `files:read` | List sources |

## List sources

```
GET /acc/{account_id}/file_sources
```

Requires `files:read`. Returns the account's sources, position-ordered. No pagination —
the full set is returned.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/file_sources" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "12", "name": "Referral" },
    { "id": "13", "name": "Website" }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
