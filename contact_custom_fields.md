# Contact Custom Fields

Custom fields are the fields an account adds to its Traveler contacts on top of the built-in
ones, e.g. "Wedding Date" or "Travel Style". Use this endpoint to list them with their type and
options, and to look up the `handle` to send in `custom_fields` when creating or updating a
[contact](contacts.md). A traveler's own `custom_fields` array already carries each field's
name and value, so no lookup is needed just to read them.

This is a **read-only** API: fields are created and edited in the web app only.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The custom field object

```json
{
  "handle": "travel_style",
  "name": "Travel Style",
  "type": "select",
  "category": "preferences",
  "multi_select": true,
  "options": ["Luxury", "Adventure"]
}
```

### Custom field object fields

| Field          | Type             | Description                                                                         |
| -------------- | ---------------- | ----------------------------------------------------------------------------------- |
| `handle`       | string           | Stable identifier made from the name on create (`wedding_date`), unique in the account, unchanged by a rename. A deleted field keeps its handle, and adding a field with the same name brings it back with that handle. The `handle` in a traveler's `custom_fields` and in the write param |
| `name`         | string           | The field name shown on the profile                                                 |
| `type`         | string           | `text`, `date` or `select`                                                          |
| `category`     | string           | `personal_information`, `general`, `health_dietary` or `preferences`                |
| `multi_select` | boolean          | `true` when a select field takes several options                                    |
| `options`      | array of strings | The allowed values of a select field. Empty for text and date fields               |

## Scopes

| Scope           | Endpoints |
| --------------- | --------- |
| `contacts:read` | List      |

This endpoint describes the fields behind a contact's `custom_fields`, so it shares the
`contacts:read` scope.

## Endpoints

| Method | Path                                      | Scope           | Description        |
| ------ | ----------------------------------------- | --------------- | ------------------ |
| `GET`  | `/acc/{account_id}/contact_custom_fields` | `contacts:read` | List custom fields |

## List custom fields

```
GET /acc/{account_id}/contact_custom_fields
```

Requires `contacts:read`. Returns the account's custom fields in the order the contact profile
shows them. No pagination, the full set is returned.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contact_custom_fields" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "handle": "wedding_date", "name": "Wedding Date", "type": "date", "category": "general", "multi_select": false, "options": [] },
    { "handle": "travel_style", "name": "Travel Style", "type": "select", "category": "preferences", "multi_select": true, "options": ["Luxury", "Adventure"] }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
