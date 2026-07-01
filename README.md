# Safari Portal Connect API

> ℹ️ This documentation is published automatically from Safari Portal's source.
> Don't edit it here — changes are overwritten on the next sync.

The Connect API is a JSON HTTP API for building integrations on top of Safari Portal.
Access is authorized per user and per account using OAuth 2.1, so an integration acts
on behalf of a specific member within a specific account, limited to the scopes the
user granted.

- **Base URL:** `https://connect.safariportal.dev`
- **Format:** JSON only (`Content-Type: application/json` for request bodies)
- **Auth:** OAuth 2.1 bearer tokens — see [Authentication](authentication.md)

> The Connect API is hosted on a dedicated, cookie-free domain. It never relies on
> browser session cookies; every request authenticates with a bearer token.

## Quickstart

1. Register an app under **Settings → Integrations → OAuth Applications** to get a
   `client_id` / `client_secret` — see [Authentication](authentication.md#registering-a-client).
2. Send the user through the authorization flow to obtain an access token.
3. Call the API with `Authorization: Bearer <access_token>`:

```bash
curl https://connect.safariportal.dev/accounts \
  -H "Authorization: Bearer <access_token>"
```

   The response lists the accounts your token may act on. Use the `id` from that
   response as `{account_id}` in all resource paths:

```bash
curl https://connect.safariportal.dev/acc/{account_id}/contacts \
  -H "Authorization: Bearer <access_token>"
```

## Resources

- [Accounts](accounts.md) — list the accounts your token can act on
- [Authentication](authentication.md) — OAuth flow, discovery, scopes, tokens
- [Categories](categories.md) — list category values to resolve file `*_category_id` ids (read-only)
- [Contacts](contacts.md) — list, search, retrieve, create, and update contacts
- [Files](files.md) — list, search, retrieve, create, and update files (trips / tour requests)
- [File Events](file_events.md) — list a file's activity timeline (read-only)
- [Invoices](invoices.md) — list and retrieve invoices with payments (read-only)
- [Stages](stages.md) — list pipeline stages to resolve file `stage_id` (read-only)
- [Tasks](tasks.md) — list, search, and retrieve tasks (read-only)
- [Users](users.md) — list and retrieve the account's team members (resolve user ids to names)

## Conventions

### Account in the URL

All resource paths are nested under `/acc/{account_id}/…`. The `{account_id}` must be
an account you are an agent member of. Requesting an account you are not a member of
returns `404`. A token bound to a single account may only address that account;
addressing any other account returns `403`.

Call `GET /accounts` first to discover the account ids available to your token. Tokens
bound to a single account list only that account.

### Identifiers

All resource identifiers — `id` and every `*_id` / `*_ids` field — are returned as
**strings**, not numbers. Safari Portal IDs are larger than the 2⁵³ range a JavaScript
`number` can hold exactly, so parsing them as numbers silently corrupts them (a later
request built from a corrupted id then fails as a `404`). Treat IDs as opaque strings:
store them as strings and pass them back verbatim.

### Response envelope

Successful responses wrap the payload in a `data` key. List endpoints also include a
`meta` object with pagination details.

```json
{
  "data": { "id": "123", "first_name": "Jane" }
}
```

```json
{
  "data": [ { "id": "123" }, { "id": "124" } ],
  "meta": {
    "total_count": 240,
    "total_pages": 10,
    "current_page": 1,
    "current_count": 25,
    "next_page": 2,
    "prev_page": null,
    "first_page": true,
    "last_page": false,
    "out_of_range": false
  }
}
```

### Pagination

List endpoints accept:

| Param   | Default | Description                     |
| ------- | ------- | ------------------------------- |
| `page`  | `1`     | Page number (1-based)           |
| `limit` | `25`    | Items per page                  |

### Errors

Errors use a consistent envelope and the matching HTTP status code:

```json
{
  "errors": {
    "status": 422,
    "title": "Validation failed",
    "description": "First name can't be blank, Last name can't be blank"
  }
}
```

| Status | Meaning                                                                             |
| ------ | ----------------------------------------------------------------------------------- |
| `401`  | Missing, expired, or invalid access token (`WWW-Authenticate: Bearer` is returned)  |
| `403`  | The token lacks the required scope, or is bound to a different account               |
| `404`  | The resource or account does not exist, or is not accessible to your token           |
| `422`  | The request body failed validation (`description` lists the reasons)                |
