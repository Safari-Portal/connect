# File Travel Regions

Travel regions are the account-defined destination values used on files (trips / tour
requests) — e.g. "East Africa", "Southern Africa". Use this endpoint to enumerate the
account's configured travel regions — for building pickers and filters — and to look
up the id to send as `travel_region_category_id` when creating or updating a
[file](files.md). The file's own `travel_region` field already embeds the resolved
`{id, name}`, so no lookup is needed just to read it back.

This is a **read-only** API: travel regions can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The travel region object

```json
{
  "id": "5",
  "name": "East Africa"
}
```

### Travel region object fields

| Field  | Type   | Description                                                                                                              |
| ------ | ------ | ------------------------------------------------------------------------------------------------------------------------ |
| `id`   | string | Unique identifier — matches the `id` in a file's `travel_region` object, and the `travel_region_category_id` write param |
| `name` | string | Display name (e.g. "East Africa")                                                                                        |

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves the values behind a file's `travel_region` field, so it shares
the `files:read` scope.

## Endpoints

| Method | Path                                    | Scope        | Description         |
| ------ | --------------------------------------- | ------------ | ------------------- |
| `GET`  | `/acc/{account_id}/file_travel_regions` | `files:read` | List travel regions |

## List travel regions

```
GET /acc/{account_id}/file_travel_regions
```

Requires `files:read`. Returns the account's travel regions, position-ordered. No
pagination — the full set is returned.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/file_travel_regions" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "5", "name": "East Africa" },
    { "id": "6", "name": "Southern Africa" }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
