# Forms

A Form is an account-built, drag-and-drop intake / lead-capture web form — the thing a
traveler fills in to inquire about a trip, provide guest details, give feedback, or sign a
waiver. This endpoint lets you list the account's forms and retrieve a single form's metadata.

This is a **read-only** API. Forms can be listed and retrieved. There are no create or update
endpoints, and this API does not create submissions. It does not expose a form's field/element
structure — this API is for reading data, not for rendering forms. To see what a form collects,
read its [submissions](form_submissions.md), whose responses are self-contained (each answer
carries its own label, type, and value).

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an account you
are an agent member of; an account you are not a member of returns `404`, and a token bound to a
different account returns `403`. Any account agent sees all of the account's forms. A form that
does not exist in that account returns `404`.

> **Identifiers: `id` vs `sqid`.** A form has two identifiers. `id` is the integer id used in
> all Connect API URLs (`/acc/{account_id}/forms/{id}`). `sqid` (`wfm_…`) is the form's
> **public / embed key** — the value from which share and embed URLs are built. Use `id` when
> calling this API; use `sqid` when you need the public identity of the form.

## The form object

The list and retrieve endpoints return the same form object:

```json
{
  "id": "42",
  "sqid": "wfm_ab12cd34ef56",
  "name": "Trip Inquiry",
  "category": "lead_form",
  "tags": ["safari", "2026"],
  "published": true,
  "published_at": "2026-06-01T08:00:00.000Z",
  "created_at": "2026-05-20T08:00:00.000Z",
  "updated_at": "2026-06-01T08:00:00.000Z"
}
```

| Field          | Type              | Description                                                                                          |
| -------------- | ----------------- | --------------------------------------------------------------------------------------------------- |
| `id`           | string            | Unique identifier for the form — used in all Connect API URLs                                        |
| `sqid`         | string            | The form's public / embed key (`wfm_…`); build share and embed URLs from it                          |
| `name`         | string            | Form name                                                                                           |
| `category`     | string            | One of `guest_form`, `lead_form`, `trip_planning`, `feedback_form`, `waivers`, `other`              |
| `tags`         | array of strings  | Free-form tags applied to the form                                                                  |
| `published`    | boolean           | `true` once the form has been published; `false` for a draft                                        |
| `published_at` | string (ISO 8601) \| null | When the form was published, or `null` if it is still a draft                              |
| `created_at`   | string (ISO 8601) | When the form was created                                                                           |
| `updated_at`   | string (ISO 8601) | When the form was last updated                                                                      |

## Behavior notes

- **Soft-deleted forms are hidden.** Deleted forms never appear in the list and return `404` on
  retrieve.
- **Drafts are included.** Unpublished forms are listed; use the `published` flag (or
  `published_at`) to distinguish drafts from published forms.
- **No field/element schema.** This API does not expose a form's inputs or layout. Read a form's
  [submissions](form_submissions.md) to see the collected data — each response carries its own
  `label`, `type`, `mapped_to`, and `value`.

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `forms:read` | List, Get |

There are no write scopes. This API is read-only. The same `forms:read` scope also governs
[Form Submissions](form_submissions.md).

## Endpoints

| Method | Path                            | Scope        | Description             |
| ------ | ------------------------------- | ------------ | ----------------------- |
| `GET`  | `/acc/{account_id}/forms`       | `forms:read` | List forms              |
| `GET`  | `/acc/{account_id}/forms/:id`   | `forms:read` | Retrieve a single form  |

## List forms

```
GET /acc/{account_id}/forms
```

Requires `forms:read`. Returns the account's forms ordered newest first (by `created_at`
descending, then `id` descending). Supports [pagination](README.md#pagination) via `page` and
`limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/forms?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "42",
      "sqid": "wfm_ab12cd34ef56",
      "name": "Trip Inquiry",
      "category": "lead_form",
      "tags": ["safari", "2026"],
      "published": true,
      "published_at": "2026-06-01T08:00:00.000Z",
      "created_at": "2026-05-20T08:00:00.000Z",
      "updated_at": "2026-06-01T08:00:00.000Z"
    }
  ],
  "meta": {
    "total_count": 7,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 7,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

## Retrieve a form

```
GET /acc/{account_id}/forms/:id
```

Requires `forms:read`. Returns the form object. Returns `404` if the form does not exist,
belongs to another account, or has been deleted.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/forms/42 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "42",
    "sqid": "wfm_ab12cd34ef56",
    "name": "Trip Inquiry",
    "category": "lead_form",
    "tags": ["safari", "2026"],
    "published": true,
    "published_at": "2026-06-01T08:00:00.000Z",
    "created_at": "2026-05-20T08:00:00.000Z",
    "updated_at": "2026-06-01T08:00:00.000Z"
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for shared
conventions.
