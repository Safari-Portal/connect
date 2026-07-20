# Itineraries

Itineraries are the trip proposals attached to a file (trip / tour request) — the dated,
day-by-day travel documents prepared for a client. This endpoint exposes their **metadata**
(title, who they were prepared for, trip dates, and whether one is the file's confirmed
itinerary). The full day-by-day content is not part of this API.

This is a **read-only** API: itineraries can be listed for a whole account or for a single
file, and retrieved individually. There is no create or update.

Itineraries are nested under a file: `/acc/{account_id}/files/{file_id}/itineraries`. The
`{account_id}` must be an account you are an agent member of (otherwise `404`; a token bound
to a different account returns `403`). The `{file_id}` must be a file you can see in that
account — a file that does not exist, or one you are not permitted to view, returns `404`
(the same visibility rules as [Files](files.md)).

There is also an **account-level** index — `/acc/{account_id}/itineraries` — that lists the
account's published itineraries across all its files and can be filtered by country. See
[List the account's itineraries](#list-the-accounts-itineraries).

Lookbooks are a separate document type with their own endpoint — see [Lookbooks](lookbooks.md).
The itineraries endpoint never returns lookbooks.

## Versioning (v2 / v3)

Safari Portal is migrating itineraries from a legacy engine (**v2**) to a new one (**v3**).
Each itinerary object carries a `version` field (`"v2"` or `"v3"`) telling you which engine
produced it. An account is served by exactly one engine at a time, so within a single
account every itinerary reports the same `version`; it flips from `v2` to `v3` when the
account is migrated. The object shape is identical across versions — only the `version` value
(and, for `confirmed`, its availability) differs. Treat `id` as an opaque string in all cases.

## The itinerary object

```json
{
  "id": "12345",
  "version": "v2",
  "file_id": "42",
  "title": "Kenya Safari — Draft 2",
  "prepared_for": "The Smith Family",
  "start_date": "2026-08-12",
  "end_date": "2026-08-22",
  "confirmed": true,
  "created_at": "2026-06-30T14:12:05.000Z",
  "updated_at": "2026-07-01T09:30:00.000Z"
}
```

### Itinerary object fields

| Field          | Type              | Description                                                                                           |
| -------------- | ----------------- | ---------------------------------------------------------------------------------------------------- |
| `id`           | string            | Unique identifier for the itinerary                                                                  |
| `version`      | string            | Which engine produced it: `"v2"` or `"v3"`                                                            |
| `file_id`      | string            | ID of the file (trip / tour request) this itinerary belongs to                                       |
| `title`        | string \| null    | Internal title of the itinerary                                                                      |
| `prepared_for` | string \| null    | Who the itinerary was prepared for                                                                   |
| `start_date`   | string \| null    | Trip start date (`YYYY-MM-DD`), or `null` when the itinerary has no fixed dates                       |
| `end_date`     | string \| null    | Trip end date (`YYYY-MM-DD`), or `null` when the itinerary has no fixed dates                         |
| `confirmed`    | boolean \| null   | `true`/`false` whether this is the file's confirmed itinerary (`v2`). `null` for `v3`, which has no per-itinerary confirmed state yet |
| `created_at`   | string (ISO 8601) | When the itinerary was created                                                                       |
| `updated_at`   | string (ISO 8601) | When the itinerary was last updated                                                                  |
| `countries`    | array             | Countries the itinerary covers, each `{ id, code, name }`. Returned only by [Retrieve](#retrieve-a-single-itinerary) — populated for `v3`, an empty array (`[]`) for `v2` |

## Scopes

| Scope        | Endpoints        |
| ------------ | ---------------- |
| `files:read` | List, Retrieve   |

Itineraries reuse the [Files](files.md) scope: a token that can read a file can read its
itineraries. There is no dedicated itineraries scope and no write scope.

## Endpoints

| Method | Path                                                   | Scope        | Description                 |
| ------ | ------------------------------------------------------ | ------------ | --------------------------- |
| `GET`  | `/acc/{account_id}/itineraries`                        | `files:read` | List the account's published itineraries |
| `GET`  | `/acc/{account_id}/files/{file_id}/itineraries`        | `files:read` | List a file's itineraries   |
| `GET`  | `/acc/{account_id}/files/{file_id}/itineraries/{id}`   | `files:read` | Retrieve a single itinerary |

## List the account's itineraries

```
GET /acc/{account_id}/itineraries
```

Requires `files:read`. Lists the account's **published** itineraries (v3) across all of its
files, ordered newest first (by `created_at` descending). Only published proposals are
returned — unpublished drafts are omitted. Supports [pagination](README.md#pagination) via
`page` and `limit`, and filtering by country via `country_codes`.

| Query param     | Description                                                                                                            |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| `country_codes` | Comma-separated ISO 3166-1 alpha-2 codes (e.g. `KE,TZ`), case-insensitive. Keeps only itineraries whose published snapshot contains at least one of the given countries |
| `page`          | Page number (default `1`)                                                                                            |
| `limit`         | Items per page (default `25`, maximum `100`)                                                                          |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/itineraries?country_codes=KE,TZ" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "12345",
      "version": "v3",
      "file_id": "42",
      "title": "Kenya Safari — Draft 2",
      "prepared_for": "The Smith Family",
      "start_date": "2026-08-12",
      "end_date": "2026-08-22",
      "confirmed": null,
      "created_at": "2026-06-30T14:12:05.000Z",
      "updated_at": "2026-07-01T09:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 3,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 3,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

List items carry the same fields as the file-scoped list; fetch a single itinerary with
[Retrieve](#retrieve-a-single-itinerary) to get its `countries`.

## List a file's itineraries

```
GET /acc/{account_id}/files/{file_id}/itineraries
```

Requires `files:read`. Returns the file's itineraries ordered newest first (by `created_at`
descending). Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/itineraries" \
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
      "title": "Kenya Safari — Draft 2",
      "prepared_for": "The Smith Family",
      "start_date": "2026-08-12",
      "end_date": "2026-08-22",
      "confirmed": true,
      "created_at": "2026-06-30T14:12:05.000Z",
      "updated_at": "2026-07-01T09:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 3,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 3,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

## Retrieve a single itinerary

```
GET /acc/{account_id}/files/{file_id}/itineraries/{id}
```

Requires `files:read`. Returns one itinerary. The `{id}` must belong to the file **in the
account's active engine** — an id from the other engine (or an unknown id) returns `404`.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/itineraries/{id}" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "12345",
    "version": "v2",
    "file_id": "42",
    "title": "Kenya Safari — Draft 2",
    "prepared_for": "The Smith Family",
    "start_date": "2026-08-12",
    "end_date": "2026-08-22",
    "confirmed": true,
    "created_at": "2026-06-30T14:12:05.000Z",
    "updated_at": "2026-07-01T09:30:00.000Z",
    "countries": []
  }
}
```

The retrieve response additionally includes a `countries` array. For `v3` itineraries it lists
the countries the itinerary covers, each entry `{ "id": "…", "code": "KE", "name": "Kenya" }`; for
`v2` itineraries it is always empty (`[]`).

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                                        |
| ------ | ------------------------------------------------------------------------------ |
| `200`  | Success                                                                         |
| `403`  | The token lacks the `files:read` scope                                          |
| `404`  | The file or itinerary does not exist, belongs to a different account, or you may not see it |
