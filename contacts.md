# Contacts

Contacts are the people and organizations in an account: **travelers**, **partners**,
and **suppliers**. The `type` field distinguishes them. Read endpoints return all
types; write endpoints currently support **travelers** only.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`, and a token bound to a different account returns `403`. If a
contact does not exist in that account, the endpoint returns `404`.

The token scope is not the only gate. Every endpoint here also requires the member the
token belongs to to hold the **Contacts** permission in that account — the same permission
that grants the Contacts area in the web UI. A member without it gets `403` on read and
write alike, whatever scopes the token carries, because a member who cannot browse the
contact list in the product must not be able to page it out through the API.

## The contact object

```json
{
  "id": "123",
  "first_name": "Jane",
  "last_name": "Doe",
  "middle_name": null,
  "preferred_name": "Janie",
  "email": "jane@example.com",
  "alternative_email": null,
  "phone": "+1 555 0100",
  "alternative_phone": null,
  "company_name": null,
  "referred_by": { "id": "45", "display_name": "Jane Doe" },
  "type": "traveler",
  "tag_list": ["vip", "honeymoon"],
  "dob": "1985-06-15",
  "anniversary": "2010-08-20",
  "medical_issues": "Penicillin allergy",
  "dietary_preferences": "Vegetarian, no nuts",
  "created_at": "2026-06-24T10:00:00.000Z",
  "updated_at": "2026-06-24T10:00:00.000Z"
}
```

`type` is one of `traveler`, `partner`, `supplier`.

### The traveler profile

`dob`, `anniversary`, `medical_issues` and `dietary_preferences` are the traveler profile.
`dob` and `anniversary` are `YYYY-MM-DD` or `null` and can be set on write (as `date_of_birth`
and `anniversary_date`); `medical_issues` and `dietary_preferences` are free text or `null` and
are **read-only** — nothing in this API can set them. All four are stored encrypted, and the
same four appear on each entry of a file's [`travelers`](files.md) array.

### What the write endpoints return

Create and update answer to `contacts:write`, and echoing a record back is a **read**. A token
holding `contacts:write` without `contacts:read` therefore receives only the contact's id:

```json
{ "data": { "id": "123" } }
```

The id is always returned, so a write-only client can still reference what it created. Add
`contacts:read` to the grant to get the full contact back instead. The same rule applies to
[files](files.md).

## List contacts

```
GET /acc/{account_id}/contacts
```

Requires `contacts:read`. Returns all contact types for the account, paginated.

| Query param          | Description                                                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `page`               | Page number (default `1`)                                                                                                     |
| `limit`              | Items per page (default `25`)                                                                                                 |
| `scope`              | Filter by type: `travelers`, `partners`, or `suppliers`                                                                       |
| `q`                  | Full-text search                                                                                                              |
| `tags[]`             | Filter by tag                                                                                                                 |
| `owner_ids[]`        | Filter by owning member                                                                                                       |
| `country_codes[]`    | Filter by primary-address country (ISO 3166-1 alpha-2, e.g. `US`)                                                             |
| `cities[]`           | Filter by primary-address city (case-insensitive, exact match)                                                                |
| `birthday_window`    | Contacts whose birthday falls in `next_7_days`, `this_week`, or `next_week` (travelers only; a non-traveler `scope` is a 400) |
| `anniversary_window` | Contacts whose anniversary falls in the same windows (travelers only)                                                         |

When no filters are given, all types are returned. Providing `scope`/`q`/filters
narrows the results. An unrecognized `scope` falls back to all types rather than
erroring. `birthday_window`/`anniversary_window` are the opposite: an unrecognized
window, or a window combined with a non-traveler `scope`, is a `400` — silently
dropping either would return an unfiltered page that reads as an answer.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts?scope=partners&limit=50" \
  -H "Authorization: Bearer <access_token>"
```

Contacts carry the coarse location of their **primary** address — `city`, `state_code`,
`country_code`, and `country_name`. Both filters read that same primary address, so a contact
is never returned for a country it does not display. The street line and postal code are
encrypted at rest and are neither exposed nor filterable.

Repeat both filters rather than comma-separating them — `country_codes[]=US&country_codes[]=KE`,
not `country_codes=US,KE`. A comma-separated value is not a shape this API reads, so it is
ignored like any other unrecognized parameter and you get an unfiltered list back. An
unrecognized country *code* is different: `country_codes[]=Kenya` returns a `400`, because the
parameter was understood and its value was not.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts?country_codes[]=US&country_codes[]=KE" \
  -H "Authorization: Bearer <access_token>"
```

## Retrieve a contact

```
GET /acc/{account_id}/contacts/:id
```

Requires `contacts:read`. Returns `404` if the contact does not exist in the account.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/contacts/123 \
  -H "Authorization: Bearer <access_token>"
```

## Create a contact (traveler)

```
POST /acc/{account_id}/contacts
```

Requires `contacts:write`. Creates a traveler. `first_name` and `last_name` are
required. Returns `201` with the created contact, or `422` with validation details.

| Field                          | Type   |
| ------------------------------ | ------ |
| `first_name` *(required)*      | string |
| `last_name` *(required)*       | string |
| `middle_name`                  | string |
| `preferred_name`               | string |
| `title`                        | string |
| `email`                        | string |
| `alternative_email`            | string |
| `phone`                        | string |
| `alternative_phone`            | string |
| `date_of_birth`                | date   |
| `anniversary_date`             | date   |
| `known_traveler_number`        | string |
| `gender`                       | string |
| `nationality`                  | string |
| `referred_by_id`               | string (contact id) |
| `tag_list`                     | string (comma-separated) |
| `emergency_contacts_attributes`| array of `{ name, phone }` |

```bash
curl -X POST https://connect.safariportal.dev/acc/{account_id}/contacts \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "first_name": "Jane",
      "last_name": "Doe",
      "email": "jane@example.com",
      "tag_list": "vip,honeymoon"
    }
  }'
```

## Update a contact (traveler)

```
PATCH /acc/{account_id}/contacts/:id
```

Requires `contacts:write`. Accepts the same fields as create. Returns `200` with the
updated contact, `404` if it does not exist in the account, or `422` on validation
failure.

```bash
curl -X PATCH https://connect.safariportal.dev/acc/{account_id}/contacts/123 \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{ "contact": { "preferred_name": "Janie" } }'
```
