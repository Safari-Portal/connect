# Files

"Files" are trips / tour requests in Safari Portal — the central planning record for
a traveler's trip. Every file moves through a `status` lifecycle: **leads → pipeline →
confirmed → completed → archived**. (Note: the `search` endpoint's `scope` filter uses a
broader vocabulary — `planning`, `traveling`, `deleted` — described under [Search](#search).)

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`, and a token bound to a different account returns `403`. If a
file does not exist in that account, or is not visible to the member, the endpoint
returns `404`. Visibility is governed by the member's dashboard permissions: consultants
see only their own files; owners and managers see all files in the account.

## The file object

```json
{
  "id": "42",
  "name": "Smith Safari 2027",
  "status": "pipeline",
  "priority": "high",
  "stage_id": "7",
  "source_category_id": "3",
  "trip_type_category_id": "1",
  "travel_region_category_id": "5",
  "license_type": "standard",
  "channel": "direct",
  "consultant_id": "11",
  "tag_list": ["honeymoon", "africa"],
  "traveler_ids": ["101", "102"],
  "created_at": "2026-06-01T08:00:00.000Z",
  "updated_at": "2026-06-15T12:30:00.000Z"
}
```

### File object fields

| Field                       | Type             | Description                                                    |
| --------------------------- | ---------------- | -------------------------------------------------------------- |
| `id`                        | string           | Unique identifier for the file                                 |
| `name`                      | string           | Display name of the file (`file_name` in the app)             |
| `status`                    | string           | Lifecycle status: one of `leads`, `pipeline`, `confirmed`, `completed`, `archived` |
| `priority`                  | string           | Priority level: one of `na`, `low`, `medium`, `high`, `top`   |
| `stage_id`                  | string \| null   | Opaque ID of the pipeline stage the file is in                |
| `source_category_id`        | string \| null   | Opaque ID of the lead source category                          |
| `trip_type_category_id`     | string \| null   | Opaque ID of the trip type category                            |
| `travel_region_category_id` | string \| null   | Opaque ID of the travel region category                        |
| `license_type`              | string \| null   | License type associated with the file                          |
| `channel`                   | string \| null   | Acquisition channel for the file                               |
| `consultant_id`             | string \| null   | ID of the primary consultant (a **user** id) assigned to the file — resolve via the [Users](users.md) endpoint |
| `tag_list`                  | array of strings | Tags applied to the file                                       |
| `traveler_ids`              | array of strings | IDs of the traveler contacts linked to the file                |
| `created_at`                | string (ISO 8601)| When the file was created                                      |
| `updated_at`                | string (ISO 8601)| When the file was last updated                                 |

## Scopes

| Scope          | Endpoints                     |
| -------------- | ----------------------------- |
| `files:read`   | List, Search, Get             |
| `files:write`  | Create, Update                |

## Endpoints

| Method  | Path                              | Scope          | Description         |
| ------- | --------------------------------- | -------------- | ------------------- |
| `GET`   | `/acc/{account_id}/files`         | `files:read`   | List files          |
| `GET`   | `/acc/{account_id}/files/search`  | `files:read`   | Search files        |
| `GET`   | `/acc/{account_id}/files/:id`     | `files:read`   | Retrieve a file     |
| `POST`  | `/acc/{account_id}/files`         | `files:write`  | Create a file       |
| `PATCH` | `/acc/{account_id}/files/:id`     | `files:write`  | Update a file       |

## List files

```
GET /acc/{account_id}/files
```

Requires `files:read`. Returns all files visible to the member, ordered by name.
Supports [pagination](README.md#pagination) via `page` and `limit`.

If any search parameter other than `order` or `scope` is present, the request behaves
identically to [Search files](#search-files).

| Query param                   | Description                                         |
| ----------------------------- | --------------------------------------------------- |
| `page`                        | Page number (default `1`)                           |
| `limit`                       | Items per page (default `25`, maximum `100`)        |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "42",
      "name": "Smith Safari 2027",
      "status": "pipeline",
      "priority": "high",
      "stage_id": "7",
      "source_category_id": "3",
      "trip_type_category_id": "1",
      "travel_region_category_id": "5",
      "license_type": "standard",
      "channel": "direct",
      "consultant_id": "11",
      "tag_list": ["honeymoon", "africa"],
      "traveler_ids": ["101", "102"],
      "created_at": "2026-06-01T08:00:00.000Z",
      "updated_at": "2026-06-15T12:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 84,
    "total_pages": 9,
    "current_page": 1,
    "current_count": 10,
    "next_page": 2,
    "prev_page": null,
    "first_page": true,
    "last_page": false,
    "out_of_range": false
  }
}
```

## Search files

```
GET /acc/{account_id}/files/search
```

Requires `files:read`. Filters files by the supplied parameters. Supports
[pagination](README.md#pagination) via `page` and `limit`. The `scope` param narrows
results to a specific lifecycle stage; it defaults to `"planning"`.

| Query param                    | Type             | Description                                                                                       |
| ------------------------------ | ---------------- | ------------------------------------------------------------------------------------------------- |
| `scope`                        | string           | Lifecycle stage to search within. One of `leads`, `planning`, `confirmed`, `traveling`, `completed`, `archived`, `deleted`. Defaults to `planning`. |
| `q`                            | string           | Full-text search across file name, linked itineraries, lookbooks, portals, and dashboards         |
| `order`                        | string           | Sort order. One of `created-desc` (default), `created-asc`, `file_name-asc`, `file_name-desc`, `priority-asc`, `priority-desc`, `start_date-asc`, `start_date-desc` |
| `stage_id`                     | string           | Filter by pipeline stage ID                                                                       |
| `consultant_ids[]`             | array of strings | Filter by primary consultant                                                                      |
| `sales_consultant_ids[]`       | array of strings | Filter by sales consultant (member)                                                               |
| `partner_agent_ids[]`          | array of strings | Filter by partner agent (contact)                                                                 |
| `supplier_ids[]`               | array of strings | Filter by supplier (contact)                                                                      |
| `trip_leader_ids[]`            | array of strings | Filter by trip leader (contact)                                                                   |
| `priorities[]`                 | array of strings | Filter by priority token (`na`, `low`, `medium`, `high`, `top`)                                   |
| `tags[]`                       | array of strings | Filter by tag (files must have all supplied tags)                                                 |
| `source_category_ids[]`        | array of strings | Filter by lead source category                                                                    |
| `trip_type_category_ids[]`     | array of strings | Filter by trip type category                                                                      |
| `travel_region_category_ids[]` | array of strings | Filter by travel region category                                                                  |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/search?scope=leads&q=safari&tags[]=honeymoon" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`** — same envelope shape as [List files](#list-files).

## Retrieve a file

```
GET /acc/{account_id}/files/:id
```

Requires `files:read`. Returns `404` if the file does not exist, belongs to another
account, or is not visible to the member.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/files/42 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "42",
    "name": "Smith Safari 2027",
    "status": "pipeline",
    "priority": "high",
    "stage_id": "7",
    "source_category_id": "3",
    "trip_type_category_id": "1",
    "travel_region_category_id": "5",
    "license_type": "standard",
    "channel": "direct",
    "consultant_id": "11",
    "tag_list": ["honeymoon", "africa"],
    "traveler_ids": ["101", "102"],
    "created_at": "2026-06-01T08:00:00.000Z",
    "updated_at": "2026-06-15T12:30:00.000Z"
  }
}
```

## Create a file

```
POST /acc/{account_id}/files
```

Requires `files:write`. New files are always created in **leads** status. Returns `201`
with the created file on success. Returns `422` with validation details on failure.
Returns `400` if the request body does not contain a `file` key.

The request body must wrap fields in a `file` object:

| Field                        | Type                     | Description                                              |
| ---------------------------- | ------------------------ | -------------------------------------------------------- |
| `file_name`                  | string                   | Name / title of the file                                 |
| `priority`                   | string                   | Priority token: one of `na`, `low`, `medium`, `high`, `top` |
| `stage_id`                   | string                   | Pipeline stage ID                                        |
| `source_category_id`         | string                   | Lead source category ID                                  |
| `trip_type_category_id`      | string                   | Trip type category ID                                    |
| `travel_region_category_id`  | string                   | Travel region category ID                                |
| `license_type`               | string                   | License type                                             |
| `channel`                    | string                   | Acquisition channel                                      |
| `tag_list`                   | string (comma-separated) | Tags to apply to the file                                |
| `sales_consultant_ids[]`     | array of strings         | IDs of sales consultant members                          |
| `partner_agent_ids[]`        | array of strings         | IDs of partner agent contacts                            |
| `trip_leader_ids[]`          | array of strings         | IDs of trip leader contacts                              |
| `supplier_ids[]`             | array of strings         | IDs of supplier contacts                                 |
| `traveler_ids[]`             | array of strings         | Traveler contacts to link. Must belong to your account (otherwise `422`). |

> **`traveler_ids` uses replace semantics.** The supplied array becomes the file's
> complete traveler set: contacts not listed are unlinked, new ones are added. Send `[]`
> to remove all travelers. Omit the key entirely to leave the existing travelers unchanged.

```bash
curl -X POST https://connect.safariportal.dev/acc/{account_id}/files \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "file_name": "Smith Safari 2027",
      "priority": "high",
      "tag_list": "honeymoon,africa",
      "traveler_ids": ["101", "102"]
    }
  }'
```

**Response `201`**

```json
{
  "data": {
    "id": "42",
    "name": "Smith Safari 2027",
    "status": "leads",
    "priority": "high",
    "stage_id": null,
    "source_category_id": null,
    "trip_type_category_id": null,
    "travel_region_category_id": null,
    "license_type": null,
    "channel": null,
    "consultant_id": null,
    "tag_list": ["honeymoon", "africa"],
    "traveler_ids": ["101", "102"],
    "created_at": "2026-06-25T09:00:00.000Z",
    "updated_at": "2026-06-25T09:00:00.000Z"
  }
}
```

## Update a file

```
PATCH /acc/{account_id}/files/:id
```

Requires `files:write`. Accepts the same fields as [Create a file](#create-a-file).
Returns `200` with the updated file, `404` if the file does not exist in the account or
is not visible to the member, or `422` on validation failure.

```bash
curl -X PATCH https://connect.safariportal.dev/acc/{account_id}/files/42 \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{ "file": { "priority": "medium", "tag_list": "honeymoon,africa,luxury" } }'
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.
