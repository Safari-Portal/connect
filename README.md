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

> **OpenAPI 3.0 spec:** the full API is described in [`openapi.yaml`](openapi.yaml) — use it
> for code generation, client SDKs, and tooling.

> **Bruno collection:** a ready-to-run [Bruno](https://usebruno.com) collection lives in
> [`bruno/`](bruno/README.md), generated from `openapi.yaml`, for exercising every endpoint.

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

Every resource is readable over `GET`. The **Access** badge says whether the API can also
change it:

- ![read-only](https://img.shields.io/badge/read--only-blue) — `GET` only; nothing in this API creates or updates it.
- ![read + write](https://img.shields.io/badge/read%20%2B%20write-brightgreen) — also creates or updates, via `POST` / `PATCH`.

| Resource | Access | Scope | Covers |
| -------- | ------ | ----- | ------ |
| [Accounts](accounts.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | — | The accounts your token can act on — start here to get `{account_id}` |
| [Contacts](contacts.md) | ![read + write](https://img.shields.io/badge/read%20%2B%20write-brightgreen) | `contacts:read`, `contacts:write` | Travelers, partners, and suppliers; filtering on list, writes for travelers only |
| [Files](files.md) | ![read + write](https://img.shields.io/badge/read%20%2B%20write-brightgreen) | `files:read`, `files:write` | Trips / tour requests, with filtering on list |
| [File Events](file_events.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `files:read` | A file's activity timeline |
| [File Categories](file_categories.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `files:read` | Category values behind a file's `*_category_id` fields |
| [File Stages](file_stages.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `files:read` | Pipeline stages behind a file's `stage_id` |
| [Itineraries](itineraries.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `files:read` | A file's or the whole account's itineraries, metadata only (v2 / v3) |
| [Lookbooks](lookbooks.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `files:read` | A file's or the whole account's lookbooks, metadata only (v2 / v3) |
| [Forms](forms.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `forms:read` | The account's forms |
| [Form Submissions](form_submissions.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `forms:read` | Submissions, per form or per contact |
| [Invoices](invoices.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `financials:read` | Invoices with their payments |
| [Expenses](expenses.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `financials:read` | Expenses with their outgoing payments |
| [Tasks](tasks.md) | ![read + write](https://img.shields.io/badge/read%20%2B%20write-brightgreen) | `tasks:read`, `tasks:write` | Tasks, with filtering on list; create a task on a file |
| [Users](users.md) | ![read-only](https://img.shields.io/badge/read--only-blue) | `users:read` | The account's team members — resolve user ids to names |

A token must carry the scope an endpoint requires, and the authorizing member must hold the
matching permission in the account — see [Authentication](authentication.md#scopes). Write
scopes are declinable at consent, so a grant may come back read-only.

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

| Param   | Default | Description                                  |
| ------- | ------- | -------------------------------------------- |
| `page`  | `1`     | Page number (1-based)                        |
| `limit` | `25`    | Items per page — clamped to a maximum of `100` |

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
