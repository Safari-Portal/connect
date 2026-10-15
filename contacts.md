# Contacts

Contacts are the people and organizations in an account: **travelers**, **partners**,
and **suppliers**. The `type` field distinguishes them. Read endpoints return all
types; create and update currently support **travelers** only; delete supports any type.

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
  "tags": ["vip", "honeymoon"],
  "city": "Austin",
  "state_code": "TX",
  "country_code": "US",
  "country_name": "United States",
  "dob": "1985-06-15",
  "anniversary": "2010-08-20",
  "medical_issues": "Penicillin allergy",
  "dietary_preferences": "Vegetarian, no nuts",
  "custom_fields": [
    { "handle": "wedding_date", "name": "Wedding Date", "type": "date", "category": "general", "value": "2026-12-01" },
    { "handle": "travel_style", "name": "Travel Style", "type": "select", "category": "preferences", "value": ["Luxury"] },
    { "handle": "blood_type", "name": "Blood Type", "type": "text", "category": "health_dietary", "value": null }
  ],
  "created_at": "2026-06-24T10:00:00.000Z",
  "updated_at": "2026-06-24T10:00:00.000Z"
}
```

`type` is one of `traveler`, `partner`, `supplier`.

`city`, `state_code`, `country_code` and `country_name` are the coarse location of the
contact's **primary** address; each is `null` when unset. The street line and postal code are
encrypted at rest and are neither exposed nor filterable.

### The traveler profile

`dob`, `anniversary`, `medical_issues` and `dietary_preferences` are the traveler profile.
All four can be set on write: `dob` and `anniversary` are `YYYY-MM-DD` or `null` (sent as
`date_of_birth` and `anniversary_date`), and `medical_issues` and `dietary_preferences` are
free text or `null`. All four are stored encrypted, and the same four appear on each entry of
a file's [`travelers`](files.md) array.

### Custom fields

`custom_fields` lists every custom field the account added to its Traveler contacts, in the
order the contact profile shows them, with this contact's value. A field with no value is still
listed: `value` is `null`, or `[]` for a multi-select field. Partners and suppliers have no
`custom_fields` key at all, because custom fields apply to travelers only.

| Field      | Type                  | Description                                                                       |
| ---------- | --------------------- | --------------------------------------------------------------------------------- |
| `handle`   | string                | The custom field handle, e.g. `wedding_date`. Stable when the field is renamed    |
| `name`     | string                | The field name shown on the profile                                               |
| `type`     | string                | `text`, `date` or `select`                                                        |
| `category` | string                | `personal_information`, `general`, `health_dietary` or `preferences`              |
| `value`    | string, array or null | Text or option as a string, a date as `YYYY-MM-DD`, a multi-select as a list     |

To list the fields with their options (for example, before writing a value), use
[Contact Custom Fields](contact_custom_fields.md). The same `custom_fields` array appears on
each entry of a file's [`travelers`](files.md) array.

### What the write endpoints return

Create and update answer to `contacts:write`, and echoing a record back is a **read**. A token
holding `contacts:write` without `contacts:read` therefore receives only the contact's id:

```json
{ "data": { "id": "123" } }
```

The id is always returned, so a write-only client can still reference what it created. Add
`contacts:read` to the grant to get the full contact back instead. The same rule applies to
[files](files.md).

## Scopes

| Scope             | Endpoints      |
| ----------------- | -------------- |
| `contacts:read`   | List, Get      |
| `contacts:write`  | Create, Update |
| `contacts:delete` | Delete         |

## Endpoints

| Method   | Path                                 | Scope             | Description            |
| -------- | ------------------------------------ | ----------------- | ---------------------- |
| `GET`    | `/acc/{account_id}/contacts`         | `contacts:read`   | List / filter contacts |
| `GET`    | `/acc/{account_id}/contacts/{id}`    | `contacts:read`   | Retrieve a contact     |
| `POST`   | `/acc/{account_id}/contacts`         | `contacts:write`  | Create a traveler      |
| `PATCH`  | `/acc/{account_id}/contacts/{id}`    | `contacts:write`  | Update a traveler      |
| `DELETE` | `/acc/{account_id}/contacts/{id}`    | `contacts:delete` | Delete a contact       |

A contact's form submissions are listed by the [Form Submissions](form_submissions.md)
endpoint, which answers to `forms:read` rather than `contacts:read`.

## List contacts

```
GET /acc/{account_id}/contacts
```

Requires `contacts:read`. Returns all contact types for the account, paginated.

| Query param          | Description                                                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `page`               | Page number (default `1`)                                                                                                     |
| `limit`              | Items per page (default `25`, maximum `100`)                                                                                  |
| `scope`              | Filter by type: `travelers`, `partners`, or `suppliers`                                                                       |
| `order`              | Sort order. One of `created-desc` (default), `created-asc`, `name-asc`, `name-desc`, `recent_trip-asc`, `recent_trip-desc`, `contact_person-asc`, `contact_person-desc` (the last two apply to `scope=suppliers` only) |
| `q`                  | Full-text search                                                                                                              |
| `tags[]`             | Filter by tag                                                                                                                 |
| `owner_ids[]`        | Filter by owning member id (any of a contact's owners). `none` matches contacts with no owner; combined with ids it returns the union |
| `country_codes[]`    | Filter by primary-address country (ISO 3166-1 alpha-2, e.g. `US`)                                                             |
| `cities[]`           | Filter by primary-address city (case-insensitive, exact match)                                                                |
| `birthday_window`    | Contacts whose birthday falls in `next_7_days`, `this_week`, or `next_week` (travelers only; a non-traveler `scope` is a 400) |
| `anniversary_window` | Contacts whose anniversary falls in the same windows (travelers only)                                                         |

When no filters are given, all types are returned, ordered by name. Providing
`scope`/`q`/filters narrows the results and switches the default order to `created-desc`;
`order` on its own is not a filter, so it does not make that switch and an otherwise
unfiltered list stays ordered by name.

An unrecognized `scope` is dropped rather than rejected, and what you get back then depends
on whether anything else narrowed the request: on its own it falls back to **all types**, but
alongside any other filter it falls back to **travelers** — so `?scope=bogus&q=smith` silently
excludes partners and suppliers. Spell `scope` correctly rather than relying on either.

`birthday_window`/`anniversary_window` are the opposite: an unrecognized window, or a window
combined with a non-traveler `scope`, is a `400` — silently dropping either would return an
unfiltered page that reads as an answer.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts?scope=partners&limit=50" \
  -H "Authorization: Bearer <access_token>"
```

`country_codes[]` and `cities[]` read the same primary address the contact displays, so a
contact is never returned for a country it does not show.

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
GET /acc/{account_id}/contacts/{id}
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
| `nationality_country_code`     | string |
| `medical_issues`               | string |
| `dietary_preferences`          | string |
| `referred_by_id`               | string (contact id) |
| `tags`                         | array of strings         |
| `emergency_contacts_attributes`| array of `{ name, phone }` |
| `custom_fields`                | array of `{ handle, value }` |

`nationality_country_code` is an ISO 3166-1 alpha-2 code (e.g. `BR`) and takes precedence
over the free-text `nationality` when both are sent.

`custom_fields` sets custom field values. Each entry names a field by its
[handle](contact_custom_fields.md) and gives its value:

- **text**: a string.
- **date**: a `YYYY-MM-DD` string.
- **select**: one of the field's `options`, as a string. A multi-select field takes a list of
  options instead.
- `null`, `""` or `[]` clears the value.

Fields you leave out keep their value, and an empty `custom_fields` list changes nothing. A handle
listed twice in the same request is a `422`. Changing only custom fields does not change the
contact's `updated_at`. If any
entry is invalid (unknown handle, no `value` key, a value that is not a string, a date in another
format, an option the field does not have, a list on a single-value field), the request returns `422` naming the field, and nothing
is saved. Writing custom fields needs an account plan that
includes custom profile fields; without it the request returns `403`. Reading them does not.

```bash
curl -X POST https://connect.safariportal.dev/acc/{account_id}/contacts \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "first_name": "Jane",
      "last_name": "Doe",
      "email": "jane@example.com",
      "tags": ["vip", "honeymoon"]
    }
  }'
```

## Update a contact (traveler)

```
PATCH /acc/{account_id}/contacts/{id}
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

Set one custom field and clear another:

```bash
curl -X PATCH https://connect.safariportal.dev/acc/{account_id}/contacts/123 \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "contact": {
      "custom_fields": [
        { "handle": "wedding_date", "value": "2026-12-01" },
        { "handle": "blood_type", "value": null }
      ]
    }
  }'
```

## Delete a contact

```
DELETE /acc/{account_id}/contacts/{id}
```

Requires `contacts:delete`. Deletes any contact type: partners and suppliers need the member's
permission to manage them, and a member restricted to their own contacts can delete only those
(otherwise `403`). The delete is **permanent**. Returns `204` with no
body, `404` if the contact does not exist in the account, or `409` if the contact is the sole
recipient of an invoice with payments.

```bash
curl -X DELETE https://connect.safariportal.dev/acc/{account_id}/contacts/123 \
  -H "Authorization: Bearer <access_token>"
```
