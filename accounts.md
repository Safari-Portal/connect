# Accounts

The Accounts endpoint is the **discovery entry point** for the Connect API. It returns
the agent accounts the authenticated token may act on. Call it first to learn the
`{account_id}` values used in every other resource path.

A token bound to a single account lists only that account. Tokens issued with a broader
scope list all accounts the user is an agent member of.

## Endpoint

| Method | Path        | Scope | Description               |
| ------ | ----------- | ----- | ------------------------- |
| `GET`  | `/accounts` | —     | List accessible accounts  |

`GET /accounts` requires a valid bearer token but **no specific scope**. It is not
nested under `/acc/{account_id}/`.

## List accounts

```
GET /accounts
```

Returns all agent accounts the token may act on. No pagination — the list is bounded by
the number of accounts associated with the token.

```bash
curl https://connect.safariportal.dev/accounts \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": 7,
      "name": "Acacia Travel",
      "created_at": "2024-03-10T09:00:00.000Z"
    },
    {
      "id": 19,
      "name": "Savanna Expeditions",
      "created_at": "2025-01-22T14:30:00.000Z"
    }
  ]
}
```

### Account object fields

| Field        | Type              | Description                                                          |
| ------------ | ----------------- | -------------------------------------------------------------------- |
| `id`         | integer           | The account identifier — use this as `{account_id}` in all resource paths |
| `name`       | string            | Display name of the account                                          |
| `created_at` | string (ISO 8601) | When the account was created                                         |

## Errors

| Status | Meaning                                                                            |
| ------ | ---------------------------------------------------------------------------------- |
| `401`  | Missing, expired, or invalid access token (`WWW-Authenticate: Bearer` is returned) |

See [Errors](README.md#errors) in the README for the shared error envelope format.
