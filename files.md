# Files

"Files" are trips / tour requests in Safari Portal — the central planning record for
a traveler's trip. Every file moves through a `status` lifecycle: **leads → pipeline →
confirmed → completed → archived**. (Note: the list endpoint's `scope` filter uses a
broader vocabulary — `planning`, `traveling`, `deleted` — described under
[List files](#list-files).)

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
  "source": { "id": "3", "name": "Referral" },
  "trip_type": { "id": "1", "name": "Honeymooner" },
  "travel_region": { "id": "5", "name": "East Africa" },
  "license_type": "standard",
  "channel": "B2C - Direct Client",
  "consultant_id": "11",
  "sales_consultants": [{ "id": "11", "name": "Rachel West" }],
  "partner_agents": [{ "id": "77", "name": "Sarah Johansson (Moody Inc)" }],
  "trip_leaders": [],
  "suppliers": [{ "id": "79", "name": "Old Buddy (Bulls and Co)" }],
  "tags": ["honeymoon", "africa"],
  "travelers": [
    {
      "id": "101",
      "first_name": "Ada",
      "last_name": "Smith",
      "middle_name": null,
      "preferred_name": null,
      "email": "ada.smith@example.com",
      "alternative_email": null,
      "phone": "+1-555-0100",
      "alternative_phone": null,
      "company_name": null,
      "type": "traveler",
      "tags": [],
      "city": "Austin",
      "state_code": "TX",
      "country_code": "US",
      "country_name": "United States",
      "dob": "1985-06-15",
      "anniversary": "2010-08-20",
      "medical_issues": "Penicillin allergy",
      "dietary_preferences": "Vegetarian, no nuts",
      "created_at": "2026-01-10T09:00:00.000Z",
      "updated_at": "2026-05-02T11:15:00.000Z"
    }
  ],
  "start_date": "2027-03-10",
  "end_date": "2027-03-24",
  "estimated_start_date": "2027-03-08",
  "estimated_end_date": "2027-03-22",
  "created_at": "2026-06-01T08:00:00.000Z",
  "confirmed_at": "2026-08-14T16:45:00.000Z",
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
| `stage_id`                  | string \| null   | ID of the pipeline stage the file is in — resolve via the [File Stages](file_stages.md) endpoint |
| `source`                    | object \| null   | `{id, name}` reference to the lead source, or `null` — see [File Sources](file_sources.md) |
| `trip_type`                 | object \| null   | `{id, name}` reference to the trip type, or `null` — see [File Trip Types](file_trip_types.md) |
| `travel_region`             | object \| null   | `{id, name}` reference to the travel region, or `null` — see [File Travel Regions](file_travel_regions.md) |
| `license_type`              | string \| null   | License type associated with the file                          |
| `channel`                   | string \| null   | Acquisition channel for the file — one of the values listed by [File Channels](file_channels.md) |
| `consultant_id`             | string \| null   | ID of the primary consultant (a **user** id) assigned to the file — resolve via the [Users](users.md) endpoint |
| `sales_consultants`         | array of objects | `{id, name}` references to the team members (**user** ids) assigned as sales consultants — resolve via the [Users](users.md) endpoint. `[]` when none are assigned. Returned by show, create and update; the list endpoint never includes it. |
| `partner_agents`            | array of objects | `{id, name}` references to the partner agent **contacts** on the file — resolve via the [Contacts](contacts.md) endpoint. Names carry the company in parentheses when the contact has one. Returned by show, create and update; the list endpoint never includes it. |
| `trip_leaders`              | array of objects | `{id, name}` references to the trip leader **contacts** on the file — resolve via the [Contacts](contacts.md) endpoint. Names carry the company in parentheses when the contact has one. Returned by show, create and update; the list endpoint never includes it. |
| `suppliers`                 | array of objects | `{id, name}` references to the supplier **contacts** on the file — resolve via the [Contacts](contacts.md) endpoint. Names carry the company in parentheses when the contact has one. Returned by show, create and update; the list endpoint never includes it. |
| `tags`                  | array of strings | Tags applied to the file                                       |
| `travelers`                 | array of [Contact](contacts.md) objects | The traveler contacts linked to the file, expanded in full. **Only present when your token holds `contacts:read`** — it carries contact data, not file data, so a `files:read`-only token receives no `travelers` key at all (distinct from `[]`, which means the file genuinely has no travelers). Returned by show, create and update; the list endpoint never includes it. |
| `start_date`                | string (date) \| null | Trip start date (`YYYY-MM-DD`), taken from the confirmed itinerary; `null` until the file is confirmed with dates. This is the value the `start_date-asc` / `start_date-desc` search order sorts on. |
| `end_date`                  | string (date) \| null | Trip end date (`YYYY-MM-DD`), taken from the confirmed itinerary; `null` until the file is confirmed with dates |
| `estimated_start_date`      | string (date) \| null | Planned trip start (`YYYY-MM-DD`) the team entered while the file was a lead or in planning; `null` when not set. Kept, and still editable, after confirmation, so read `start_date` for the confirmed trip. This is the value the `estimated_start_date-asc` / `estimated_start_date-desc` search order sorts on. |
| `estimated_end_date`        | string (date) \| null | Planned trip end (`YYYY-MM-DD`); `null` when not set. Always set together with `estimated_start_date`, on or after it. |
| `created_at`                | string (ISO 8601)| When the file was created                                      |
| `confirmed_at`              | string (ISO 8601) \| null | When the file was confirmed; `null` while it is still a lead or in the pipeline. Together with `created_at` this brackets the file's time to confirm. |
| `updated_at`                | string (ISO 8601)| When the file was last updated                                 |

## Scopes

| Scope          | Endpoints                     |
| -------------- | ----------------------------- |
| `files:read`   | List, Get                     |
| `files:write`  | Create, Update                |
| `files:delete` | Delete                        |

## Endpoints

| Method   | Path                              | Scope          | Description         |
| -------- | --------------------------------- | -------------- | ------------------- |
| `GET`    | `/acc/{account_id}/files`         | `files:read`   | List / filter files |
| `GET`    | `/acc/{account_id}/files/{id}`     | `files:read`   | Retrieve a file     |
| `POST`   | `/acc/{account_id}/files`         | `files:write`  | Create a file       |
| `PATCH`  | `/acc/{account_id}/files/{id}`     | `files:write`  | Update a file       |
| `DELETE` | `/acc/{account_id}/files/{id}`    | `files:delete` | Delete a file       |

## List files

```
GET /acc/{account_id}/files
```

Requires `files:read`. With no filter parameters, returns all files visible to the
member, ordered by name. The list response carries no traveler information at all —
`travelers` is returned only by [Retrieve a file](#retrieve-a-file), create and update. Supplying any of the filters below narrows the results;
`scope` alone is enough to filter by lifecycle stage. Supports
[pagination](README.md#pagination) via `page` and `limit`.

| Query param                    | Type             | Description                                                                                       |
| ------------------------------ | ---------------- | ------------------------------------------------------------------------------------------------- |
| `page`                         | integer          | Page number (default `1`)                                                                         |
| `limit`                        | integer          | Items per page (default `25`, maximum `100`)                                                       |
| `scope`                        | string           | Lifecycle stage to filter within. One of `leads`, `planning`, `confirmed`, `traveling`, `completed`, `archived`, `deleted`. When any filter other than `scope` is supplied without a `scope`, it defaults to `planning`. |
| `q`                            | string           | Full-text search across file name, linked itineraries, lookbooks, portals, and dashboards         |
| `order`                        | string           | Sort order. One of `created-desc` (default), `created-asc`, `file_name-asc`, `file_name-desc`, `priority-asc`, `priority-desc`, `start_date-asc`, `start_date-desc`, `estimated_start_date-asc`, `estimated_start_date-desc` |
| `stage_id`                     | string           | Filter by pipeline stage ID                                                                       |
| `consultant_ids[]`             | array of strings | Filter by primary consultant                                                                      |
| `sales_consultant_ids[]`       | array of strings | Filter by sales consultant (member)                                                               |
| `partner_agent_ids[]`          | array of strings | Filter by partner agent (contact)                                                                 |
| `supplier_ids[]`               | array of strings | Filter by supplier (contact)                                                                      |
| `trip_leader_ids[]`            | array of strings | Filter by trip leader (contact)                                                                   |
| `priorities[]`                 | array of strings | Filter by priority token (`na`, `low`, `medium`, `high`, `top`)                                   |
| `tags[]`                       | array of strings | Filter by tag (files must have all supplied tags)                                                 |
| `source_category_ids[]`        | array of strings | Filter by lead source ID — see [File Sources](file_sources.md)                                    |
| `trip_type_category_ids[]`     | array of strings | Filter by trip type ID — see [File Trip Types](file_trip_types.md)                                |
| `travel_region_category_ids[]` | array of strings | Filter by travel region ID — see [File Travel Regions](file_travel_regions.md)                    |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files?scope=leads&q=safari&tags[]=honeymoon" \
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
      "source": { "id": "3", "name": "Referral" },
      "trip_type": { "id": "1", "name": "Honeymooner" },
      "travel_region": { "id": "5", "name": "East Africa" },
      "license_type": "standard",
      "channel": "B2C - Direct Client",
      "consultant_id": "11",
      "tags": ["honeymoon", "africa"],
      "start_date": "2027-03-10",
      "end_date": "2027-03-24",
      "estimated_start_date": "2027-03-08",
      "estimated_end_date": "2027-03-22",
      "created_at": "2026-06-01T08:00:00.000Z",
      "confirmed_at": "2026-08-14T16:45:00.000Z",
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

## Retrieve a file

```
GET /acc/{account_id}/files/{id}
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
    "source": { "id": "3", "name": "Referral" },
    "trip_type": { "id": "1", "name": "Honeymooner" },
    "travel_region": { "id": "5", "name": "East Africa" },
    "license_type": "standard",
    "channel": "B2C - Direct Client",
    "consultant_id": "11",
    "tags": ["honeymoon", "africa"],
    "travelers": [
      {
        "id": "101",
        "first_name": "Ada",
        "last_name": "Smith",
        "middle_name": null,
        "preferred_name": null,
        "email": "ada.smith@example.com",
        "alternative_email": null,
        "phone": "+1-555-0100",
        "alternative_phone": null,
        "company_name": null,
        "type": "traveler",
        "tags": [],
        "city": "Austin",
        "state_code": "TX",
        "country_code": "US",
        "country_name": "United States",
        "dob": "1985-06-15",
        "anniversary": "2010-08-20",
        "medical_issues": "Penicillin allergy",
        "dietary_preferences": "Vegetarian, no nuts",
        "created_at": "2026-01-10T09:00:00.000Z",
        "updated_at": "2026-05-02T11:15:00.000Z"
      }
    ],
    "start_date": "2027-03-10",
    "end_date": "2027-03-24",
    "estimated_start_date": "2027-03-08",
    "estimated_end_date": "2027-03-22",
    "created_at": "2026-06-01T08:00:00.000Z",
    "confirmed_at": "2026-08-14T16:45:00.000Z",
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
| `source_category_id`         | string                   | Lead source ID — look up via [File Sources](file_sources.md) |
| `trip_type_category_id`      | string                   | Trip type ID — look up via [File Trip Types](file_trip_types.md) |
| `travel_region_category_id`  | string                   | Travel region ID — look up via [File Travel Regions](file_travel_regions.md) |
| `license_type`               | string                   | License type                                             |
| `channel`                    | string                   | Acquisition channel — one of the values listed by [File Channels](file_channels.md) (otherwise `422`); `null` clears it |
| `tags`                       | array of strings         | Tags to apply to the file                                |
| `sales_consultant_ids[]`     | array of strings         | IDs of sales consultant members. Echoed back in the `sales_consultants` response field. |
| `partner_agent_ids[]`        | array of strings         | IDs of partner agent contacts. Echoed back in the `partner_agents` response field. |
| `trip_leader_ids[]`          | array of strings         | IDs of trip leader contacts. Echoed back in the `trip_leaders` response field. |
| `supplier_ids[]`             | array of strings         | IDs of supplier contacts. Echoed back in the `suppliers` response field. |
| `traveler_ids[]`             | array of strings         | Traveler contacts to link. Must belong to your account (otherwise `422`). |
| `estimated_start_date`       | string (date) \| null   | Planned trip start (`YYYY-MM-DD`, otherwise `422`). The file must end up with both estimated dates or neither (otherwise `422`). On update an omitted date keeps its stored value, so clear them by sending both as `null`. |
| `estimated_end_date`         | string (date) \| null   | Planned trip end (`YYYY-MM-DD`, otherwise `422`), on or after `estimated_start_date` (otherwise `422`). Same both-or-neither rule. |

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
      "tags": ["honeymoon", "africa"],
      "estimated_start_date": "2027-03-08",
      "estimated_end_date": "2027-03-22",
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
    "source": null,
    "trip_type": null,
    "travel_region": null,
    "license_type": null,
    "channel": null,
    "consultant_id": null,
    "tags": ["honeymoon", "africa"],
    "travelers": [
      {
        "id": "101",
        "first_name": "Ada",
        "last_name": "Smith",
        "middle_name": null,
        "preferred_name": null,
        "email": "ada.smith@example.com",
        "alternative_email": null,
        "phone": "+1-555-0100",
        "alternative_phone": null,
        "company_name": null,
        "type": "traveler",
        "tags": [],
        "city": "Austin",
        "state_code": "TX",
        "country_code": "US",
        "country_name": "United States",
        "dob": "1985-06-15",
        "anniversary": "2010-08-20",
        "medical_issues": "Penicillin allergy",
        "dietary_preferences": "Vegetarian, no nuts",
        "created_at": "2026-01-10T09:00:00.000Z",
        "updated_at": "2026-05-02T11:15:00.000Z"
      }
    ],
    "start_date": null,
    "end_date": null,
    "estimated_start_date": "2027-03-08",
    "estimated_end_date": "2027-03-22",
    "created_at": "2026-06-25T09:00:00.000Z",
    "confirmed_at": null,
    "updated_at": "2026-06-25T09:00:00.000Z"
  }
}
```

### What the write endpoints return

Create and update answer to `files:write`, and echoing a record back is a **read**. A token
holding `files:write` without `files:read` receives only the file's id:

```json
{ "data": { "id": "42" } }
```

The id is always returned, so a write-only client can still reference what it created. Add
`files:read` for the full file, and `contacts:read` on top for the `travelers` array.

## Update a file

```
PATCH /acc/{account_id}/files/{id}
```

Requires `files:write`. Accepts the same fields as [Create a file](#create-a-file).
Returns `200` with the updated file, `404` if the file does not exist in the account or
is not visible to the member, or `422` on validation failure.

```bash
curl -X PATCH https://connect.safariportal.dev/acc/{account_id}/files/42 \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{ "file": { "priority": "medium", "tags": ["honeymoon", "africa", "luxury"] } }'
```

## Delete a file

```
DELETE /acc/{account_id}/files/{id}
```

Requires `files:delete`. Deletes the file with its itineraries, lookbooks, guest portals and
proposals. The delete is soft: the file can be restored in Safari Portal from **Recently
Deleted** during the retention period. Returns `204` with no body, `404` if the file does not
exist in the account, is not visible to the member, or is already deleted, or `409` if the file
has pending or paid payments.

```bash
curl -X DELETE https://connect.safariportal.dev/acc/{account_id}/files/42 \
  -H "Authorization: Bearer <access_token>"
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.
