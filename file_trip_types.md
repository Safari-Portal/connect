# File Trip Types

Trip types are the account-defined classification values used on files (trips / tour
requests) — e.g. "Guided", "Honeymooner", "Family". Use this endpoint to enumerate the
account's configured trip types — for building pickers and filters — and to look up
the id to send as `trip_type_category_id` when creating or updating a
[file](files.md). The file's own `trip_type` field already embeds the resolved
`{id, name}`, so no lookup is needed just to read it back.

This is a **read-only** API: trip types can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The trip type object

```json
{
  "id": "1",
  "name": "Honeymooner"
}
```

### Trip type object fields

| Field  | Type   | Description                                                                                                      |
| ------ | ------ | ---------------------------------------------------------------------------------------------------------------- |
| `id`   | string | Unique identifier — matches the `id` in a file's `trip_type` object, and the `trip_type_category_id` write param |
| `name` | string | Display name (e.g. "Honeymooner")                                                                                |

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves the values behind a file's `trip_type` field, so it shares the
`files:read` scope.

## Endpoints

| Method | Path                                | Scope        | Description     |
| ------ | ----------------------------------- | ------------ | --------------- |
| `GET`  | `/acc/{account_id}/file_trip_types` | `files:read` | List trip types |

## List trip types

```
GET /acc/{account_id}/file_trip_types
```

Requires `files:read`. Returns the account's trip types, position-ordered. No
pagination — the full set is returned.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/file_trip_types" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "1", "name": "Honeymooner" },
    { "id": "2", "name": "Family" }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
