# Events

Events are the activity timeline of a file (trip / tour request). Each event is a
timestamped record of something that happened on the file or one of its related objects —
a proposal or lookbook being viewed, a file being shared with a client, a status change, a
payment being completed, a task being completed, a guest form being submitted, and so on.

This is a **read-only** API. Events for a file can be listed. There is no single-event
retrieval, no search, and no create or update.

Events are nested under a file: `/acc/{account_id}/files/{file_id}/events`. The
`{account_id}` must be an account you are an agent member of; an account you are not a
member of returns `404`, and a token bound to a different account returns `403`. The
`{file_id}` must be a file you can see in that account — a file that does not exist, or one
you are not permitted to view, returns `404` (the same visibility rules as
[Files](files.md)).

## The event object

```json
{
  "id": "12345",
  "type": "shared_with_client",
  "file_id": "42",
  "user_id": "11",
  "request_location": "Cape Town, South Africa",
  "request_ip": "196.25.1.10",
  "created_at": "2026-06-30T14:12:05.000Z"
}
```

### Event object fields

| Field              | Type              | Description                                                                                          |
| ------------------ | ----------------- | --------------------------------------------------------------------------------------------------- |
| `id`               | string            | Unique identifier for the event                                                                     |
| `type`             | string            | What happened — see [Event types](#event-types) below                                               |
| `file_id`          | string            | ID of the file (trip / tour request) this event belongs to                                          |
| `user_id`          | string \| null    | ID of the user who triggered the event, or `null` for automated / guest-driven events — resolve via the [Users](users.md) endpoint |
| `request_location` | string \| null    | Approximate (city-level) location the request originated from, or `null`. See the privacy note below |
| `request_ip`       | string \| null    | IP address the request originated from, or `null`. See the privacy note below                       |
| `created_at`       | string (ISO 8601) | When the event occurred                                                                             |

> **Privacy note.** For public/guest-facing events (for example, a client opening a shared
> proposal link) `request_location` and `request_ip` capture the **end visitor's**
> approximate location and IP address. Handle these fields in accordance with your privacy
> obligations. They are `null` for events that have no associated request (system or
> internal actions).

### Event types

`type` is one of the following values. The set is additive — new types may be introduced
over time, so treat unrecognised values gracefully.

`draft`, `link_generated`, `shared_with_client`, `shared_with_agent`, `confirmed`,
`unconfirmed`, `moved_to_leads`, `completed`, `archived`, `updated`, `received_from_agent`,
`public_link_viewed`, `public_link_shared`, `consultant_changed`, `private_link_viewed`,
`guest_form_submitted`, `private_link_shared`, `toggle_deactivate`,
`agent_notified_about_relocation_changes`, `guest_info_viewed`, `guest_info_sent`,
`deleted`, `restored`, `guest_info_can_be_shared`, `personal_info_can_be_saved`,
`guest_updated_shared_checkbox`, `guest_updated_saved_checkbox`,
`member_privacy_policy_accepted`, `payment_completed`, `task_completed`, `form_submitted`,
`guest_info_downloaded`.

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

Events reuse the [Files](files.md) scope: a token that can read a file can read its events.
There is no dedicated events scope and no write scope.

## Endpoints

| Method | Path                                        | Scope        | Description             |
| ------ | ------------------------------------------- | ------------ | ----------------------- |
| `GET`  | `/acc/{account_id}/files/{file_id}/events`  | `files:read` | List a file's events    |

## List a file's events

```
GET /acc/{account_id}/files/{file_id}/events
```

Requires `files:read`. Returns the file's events ordered newest first (by `created_at`
descending). Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/files/{file_id}/events?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "12345",
      "type": "shared_with_client",
      "file_id": "42",
      "user_id": "11",
      "request_location": "Cape Town, South Africa",
      "request_ip": "196.25.1.10",
      "created_at": "2026-06-30T14:12:05.000Z"
    }
  ],
  "meta": {
    "total_count": 42,
    "total_pages": 5,
    "current_page": 1,
    "current_count": 10,
    "next_page": 2,
    "prev_page": null,
    "first_page": true,
    "last_page": false,
    "out_of_range": false
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                                       |
| ------ | ----------------------------------------------------------------------------- |
| `200`  | Success                                                                        |
| `403`  | The token lacks the `files:read` scope                                        |
| `404`  | The file does not exist, belongs to a different account, or you may not see it |
