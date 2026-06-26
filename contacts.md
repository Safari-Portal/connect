# Contacts

Contacts are the people and organizations in an account: **travelers**, **partners**,
and **suppliers**. The `type` field distinguishes them. Read endpoints return all
types; write endpoints currently support **travelers** only.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; using a different account id returns `403`. If a
contact does not exist in that account, the endpoint returns `404`.

## The contact object

```json
{
  "id": 123,
  "first_name": "Jane",
  "last_name": "Doe",
  "middle_name": null,
  "preferred_name": "Janie",
  "email": "jane@example.com",
  "alternative_email": null,
  "phone": "+1 555 0100",
  "alternative_phone": null,
  "company_name": null,
  "type": "traveler",
  "tag_list": ["vip", "honeymoon"],
  "created_at": "2026-06-24T10:00:00.000Z",
  "updated_at": "2026-06-24T10:00:00.000Z"
}
```

`type` is one of `traveler`, `partner`, `supplier`.

## List contacts

```
GET /acc/{account_id}/contacts
```

Requires `contacts:read`. Returns all contact types for the account, paginated.

| Query param   | Description                                                       |
| ------------- | ----------------------------------------------------------------- |
| `page`        | Page number (default `1`)                                         |
| `limit`       | Items per page (default `25`)                                     |
| `scope`       | Filter by type: `travelers`, `partners`, or `suppliers`           |
| `q`           | Full-text search                                                  |
| `tags[]`      | Filter by tag                                                     |
| `owner_ids[]` | Filter by owning member                                           |

When no filters are given, all types are returned. Providing `scope`/`q`/filters
narrows the results.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts?scope=partners&limit=50" \
  -H "Authorization: Bearer <access_token>"
```

## Search contacts

```
GET /acc/{account_id}/contacts/search
```

Requires `contacts:read`. Same filtering as the list endpoint; `scope` defaults to
`travelers`. An unrecognized `scope` falls back to the default rather than erroring.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts/search?scope=travelers&q=jane" \
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
| `referred_by_id`               | integer (contact id) |
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
