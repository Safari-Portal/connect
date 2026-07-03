# File Categories

Categories are the account-defined classification values used on files (trips / tour
requests): **lead sources**, **trip types**, and **travel regions**. The `type` field
distinguishes them. Use this endpoint to resolve the `source_category_id`,
`trip_type_category_id`, and `travel_region_category_id` ids returned on a
[file](files.md) into names.

This is a **read-only** API: categories can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The category object

```json
{
  "id": "12",
  "name": "Honeymooner",
  "type": "trip_type"
}
```

### Category object fields

| Field  | Type   | Description                                                              |
| ------ | ------ | ------------------------------------------------------------------------ |
| `id`   | string | Unique identifier — matches the `*_category_id` ids on a file            |
| `name` | string | Display name (e.g. "Honeymooner")                                       |
| `type` | string | One of `source`, `trip_type`, `travel_region`                           |

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves ids that appear on files, so it shares the `files:read` scope.

## Endpoints

| Method | Path                            | Scope        | Description       |
| ------ | ------------------------------- | ------------ | ----------------- |
| `GET`  | `/acc/{account_id}/file_categories`  | `files:read` | List categories   |

## List categories

```
GET /acc/{account_id}/file_categories
```

Requires `files:read`. Returns the account's categories, ordered by `type` then
position. No pagination — the full set is returned.

| Query param | Type   | Description                                                                   |
| ----------- | ------ | ----------------------------------------------------------------------------- |
| `type`      | string | Optional. One of `source`, `trip_type`, `travel_region`. An unrecognized value returns an empty list. |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/file_categories?type=trip_type" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "12", "name": "Honeymooner", "type": "trip_type" },
    { "id": "13", "name": "Family",      "type": "trip_type" }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
