# Contents

Content pages are the Content Library's write-ups — a property, an area, an activity, an
"About us" page and so on. An itinerary event points at one through its `content_id`. Use this
endpoint to find that id and to keep a mapping between the services in your system and the
pages in Safari Portal.

This is a **read-only** API: pages can only be listed, and only their identity is returned —
page bodies (sections, media) are not part of it.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an account you
are an agent member of; an account you are not a member of returns `404`, and a token bound to
a different account returns `403`.

## Which pages are listed

The pages the account's itinerary builder offers when an event is attached to a page:

- the account's own pages;
- Safari Portal pages that have been approved;
- pages another agency shared with the account.

Each of them published, active, not deleted and listed in the Content Library. Every listed
page is accepted as an event's `content_id` (an accommodation event takes a `property` page
only).

## The content object

```json
{
  "id": "40161",
  "kind": "property",
  "name": "Angama Mara",
  "handle": "angama_mara"
}
```

### Content object fields

| Field    | Type   | Description                                                              |
| -------- | ------ | ------------------------------------------------------------------------ |
| `id`     | string | Unique identifier — what an itinerary event takes as `content_id`        |
| `kind`   | string | `property`, `area`, `activity`, `miscellaneous`, `about_us`, `about_safari` or `pre_travel_info` |
| `name`   | string | Page name as shown in the Content Library                                |
| `handle` | string | The page's handle                                                        |

## Scopes

| Scope           | Endpoints |
| --------------- | --------- |
| `contents:read` | List      |

The Content Library is not part of a file, so `files:read` does not open it: an integration
that writes itineraries asks for `contents:read` too. An app registered before this scope
shipped has to add it first — see [Scopes](authentication.md#scopes).

## Endpoints

| Method | Path                         | Scope           | Description        |
| ------ | ---------------------------- | --------------- | ------------------ |
| `GET`  | `/acc/{account_id}/contents` | `contents:read` | List content pages |

## List content pages

```
GET /acc/{account_id}/contents
```

Requires `contents:read`. Paginated (see [Pagination](README.md#pagination)).

| Query param | Type    | Description |
| ----------- | ------- | ----------- |
| `q`         | string  | Optional. Every word must start a word of the page's name, handle, tags, area or country: `masai mar` finds "Masai Mara", `asai` does not. |
| `kind`      | string  | Optional. One page kind from the list above. Any other value returns `400`; an empty value is the same as leaving it out. |
| `source`    | string  | Optional. `own`, `safari_portal` or `shared` — where the pages come from (the legacy `only_my` / `only_sp`). Any other value returns `400`; an empty value is the same as leaving it out. |
| `page`      | integer | Optional. 1-based page number. |
| `limit`     | integer | Optional. Page size, up to 100. |

Results are ordered by match: an exact name first, then names starting with `q`, then names
containing it, then by name. Without `q` they are ordered by name.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contents?q=angama&kind=property" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "40161", "kind": "property", "name": "Angama Mara", "handle": "angama_mara" }
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

See [Errors](README.md#errors) in the README for the shared error envelope.

| Status | Meaning                                         |
| ------ | ----------------------------------------------- |
| `200`  | Success                                         |
| `400`  | `kind` or `source` is not one of its values     |
| `403`  | The token lacks the `contents:read` scope, or the account has no Content Library |
| `404`  | The account does not exist or you are not an agent member of it |
