# Users

Users are the team members (agents / consultants) who belong to an account. This
resource is **read-only**: users can be listed and retrieved. There are no create or
update endpoints.

**Primary use:** resolve the user ids returned by other endpoints — `consultant_id` on
[files](files.md), and `assigned_to_id` / `creator_id` on [tasks](tasks.md) — to names
and email addresses.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; using a different account id returns `403`. Requesting
a user who is not a member of the account returns `404`.

## The user object

```json
{
  "id": 11,
  "name": "Alice Kamau",
  "email": "alice@safarico.example"
}
```

### User object fields

| Field   | Type    | Description                         |
| ------- | ------- | ----------------------------------- |
| `id`    | integer | Unique identifier for the user      |
| `name`  | string  | Full name of the user               |
| `email` | string  | Email address of the user           |

## Scopes

| Scope        | Endpoints  |
| ------------ | ---------- |
| `users:read` | List, Get  |

There are no write scopes. This API is read-only.

## Endpoints

| Method | Path                              | Scope        | Description       |
| ------ | --------------------------------- | ------------ | ----------------- |
| `GET`  | `/acc/{account_id}/users`         | `users:read` | List users        |
| `GET`  | `/acc/{account_id}/users/{id}`    | `users:read` | Retrieve a user   |

## List users

```
GET /acc/{account_id}/users
```

Requires `users:read`. Returns all team members for the account ordered by name.
Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/users?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": 7,
      "name": "Bob Osei",
      "email": "bob@safarico.example"
    },
    {
      "id": 11,
      "name": "Alice Kamau",
      "email": "alice@safarico.example"
    }
  ],
  "meta": {
    "total_count": 8,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 8,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

## Retrieve a user

```
GET /acc/{account_id}/users/{id}
```

Requires `users:read`. Returns `404` if the user does not exist or is not a member of
the account.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/users/11 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": 11,
    "name": "Alice Kamau",
    "email": "alice@safarico.example"
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                              |
| ------ | -------------------------------------------------------------------- |
| `200`  | Success                                                              |
| `403`  | The token lacks the `users:read` scope                               |
| `404`  | The user does not exist or is not a member of the account            |
