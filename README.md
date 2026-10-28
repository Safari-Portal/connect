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

## Endpoints

Every path below is relative to `https://connect.safariportal.dev/acc/{account_id}`. Each
description links to the full reference for that endpoint.

> **Start here:** ![GET](https://img.shields.io/badge/GET-0969da) `/accounts` lists the
> accounts your token can act on (no scope needed). Use the returned `id` as `{account_id}` —
> see [Accounts](accounts.md#list-accounts).

### Contacts

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/contacts` | `contacts:read` | [List / filter contacts](contacts.md#list-contacts) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/contacts/{id}` | `contacts:read` | [Retrieve a contact](contacts.md#retrieve-a-contact) |
| ![POST](https://img.shields.io/badge/POST-1a7f37) | `/contacts` | `contacts:write` | [Create a traveler](contacts.md#create-a-contact-traveler) |
| ![PATCH](https://img.shields.io/badge/PATCH-9a6700) | `/contacts/{id}` | `contacts:write` | [Update a traveler](contacts.md#update-a-contact-traveler) |
| ![DELETE](https://img.shields.io/badge/DELETE-cf222e) | `/contacts/{id}` | `contacts:delete` | [Delete a contact](contacts.md#delete-a-contact) |

### Files

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files` | `files:read` | [List / filter files](files.md#list-files) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{id}` | `files:read` | [Retrieve a file](files.md#retrieve-a-file) |
| ![POST](https://img.shields.io/badge/POST-1a7f37) | `/files` | `files:write` | [Create a file (as a lead)](files.md#create-a-file) |
| ![PATCH](https://img.shields.io/badge/PATCH-9a6700) | `/files/{id}` | `files:write` | [Update a file](files.md#update-a-file) |
| ![DELETE](https://img.shields.io/badge/DELETE-cf222e) | `/files/{id}` | `files:delete` | [Delete a file](files.md#delete-a-file) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{file_id}/events` | `files:read` | [List a file's events](file_events.md#list-a-files-events) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{file_id}/notes` | `files:read` | [List a file's notes](file_notes.md#list-a-files-notes) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{file_id}/notes/{id}` | `files:read` | [Retrieve a file note](file_notes.md#retrieve-a-note) |
| ![POST](https://img.shields.io/badge/POST-1a7f37) | `/files/{file_id}/notes` | `files:write` | [Add a note to a file](file_notes.md#add-a-note-to-a-file) |

### Itineraries

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/itineraries` | `files:read` | [List itineraries, by file, published state or country](itineraries.md#list-itineraries) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/itineraries/{id}` | `files:read` | [Retrieve an itinerary](itineraries.md#retrieve-a-single-itinerary) |
| ![POST](https://img.shields.io/badge/POST-1a7f37) | `/itineraries` | `files:write` | [Create a v3 itinerary with its events](itineraries.md#create-an-itinerary) |

### Lookbooks

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/lookbooks` | `files:read` | [List published lookbooks, by country](lookbooks.md#list-the-accounts-lookbooks) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{file_id}/lookbooks` | `files:read` | [List a file's lookbooks](lookbooks.md#list-a-files-lookbooks) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/files/{file_id}/lookbooks/{id}` | `files:read` | [Retrieve a lookbook](lookbooks.md#retrieve-a-single-lookbook) |

### Contents

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/contents` | `contents:read` | [Search Content Library pages for a `content_id`](contents.md#list-content-pages) |

### Forms

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/forms` | `forms:read` | [List forms](forms.md#list-forms) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/forms/{id}` | `forms:read` | [Retrieve a form](forms.md#retrieve-a-form) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/forms/{form_id}/submissions` | `forms:read` | [List a form's submissions](form_submissions.md#list-a-forms-submissions) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/form_submissions/{id}` | `forms:read` | [Retrieve a submission](form_submissions.md#retrieve-a-submission) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/contacts/{contact_id}/form_submissions` | `forms:read` | [List a contact's submissions](form_submissions.md#list-a-contacts-submissions) |

### Financials

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/invoices` | `financials:read` | [List invoices](invoices.md#list-invoices) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/invoices/{id}` | `financials:read` | [Retrieve an invoice with payments](invoices.md#retrieve-an-invoice) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/expenses` | `financials:read` | [List expenses](expenses.md#list-expenses) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/expenses/{id}` | `financials:read` | [Retrieve an expense with payments](expenses.md#retrieve-an-expense) |

### Tasks

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/tasks` | `tasks:read` | [List / filter tasks](tasks.md#list-tasks) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/tasks/{id}` | `tasks:read` | [Retrieve a task](tasks.md#retrieve-a-task) |
| ![POST](https://img.shields.io/badge/POST-1a7f37) | `/tasks` | `tasks:write` | [Create a task on a file](tasks.md#create-a-task) |

### Webhook Events

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/webhook_events` | any read scope | [List events, for catching up after downtime](webhook_events.md#list-events) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/webhook_events/{id}` | any read scope | [Retrieve an event](webhook_events.md#retrieve-an-event) |

### Users

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/users` | `users:read` | [List team members](users.md#list-users) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/users/{id}` | `users:read` | [Retrieve a team member](users.md#retrieve-a-user) |

### Lookups

| Method | Endpoint | Scope | Description |
| ------ | -------- | ----- | ----------- |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/contact_custom_fields` | `contacts:read` | [Traveler custom fields](contact_custom_fields.md#list-custom-fields) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/file_sources` | `files:read` | [Lead sources](file_sources.md#list-sources) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/file_trip_types` | `files:read` | [Trip types](file_trip_types.md#list-trip-types) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/file_travel_regions` | `files:read` | [Travel regions](file_travel_regions.md#list-travel-regions) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/file_channels` | `files:read` | [Built-in channels](file_channels.md#list-channels) |
| ![GET](https://img.shields.io/badge/GET-0969da) | `/file_stages` | `files:read` | [Pipeline stages](file_stages.md#list-stages) |

A token must carry the scope an endpoint requires, and the authorizing member must hold the
matching permission in the account — see [Authentication](authentication.md#scopes). Write
scopes are declinable at consent, so a grant may come back read-only.

## Webhooks

Get notified when records are created instead of polling — see [Webhooks](webhooks.md). To
catch up after downtime, see [Webhook Events](webhook_events.md).

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
| `400`  | The request body is not valid JSON, or not the shape the endpoint expects           |
| `401`  | Missing, expired, or invalid access token (`WWW-Authenticate: Bearer` is returned)  |
| `403`  | The token lacks the required scope, or is bound to a different account               |
| `404`  | The resource or account does not exist, or is not accessible to your token           |
| `409`  | The request is valid but the record's current state forbids it (`description` says why) |
| `422`  | The request body failed validation (`description` lists the reasons)                |
| `429`  | Too many requests on one token (`Retry-After` gives the seconds to wait); only [itinerary create](itineraries.md#create-an-itinerary) is limited |
