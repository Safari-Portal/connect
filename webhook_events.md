# Webhook Events

Webhook events are the events behind [webhooks](webhooks.md): one record for every file,
contact, task, invoice, expense, or form submission that was created. Use them to catch up
after your endpoint was down, or to check what a webhook announced.

> **Not File Events.** This is unrelated to [File Events](file_events.md), which is the
> activity timeline of a single file (a proposal being viewed, a status change, and so on).
> Webhook events only announce that a record was created.

This is a **read-only** API. Events can be listed and retrieved. There is no create, update,
or delete, and no search beyond the filters below.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an account
you are an agent member of; an account you are not a member of returns `404`, and a token
bound to a different account returns `403`.

## The event object

The event is the same object a webhook delivers, minus `user_id` (a list is not tied to one
delivery, and you are already calling with a member's token).

```json
{
  "id": "evt_9Xk2mQp4LrT7vB1z",
  "object": "event",
  "type": "contact.created",
  "created_at": "2026-10-16T12:00:00Z",
  "account_id": "42",
  "request": { "client_id": "Xk3…" },
  "data": { "object": { "type": "contact", "id": "991" } }
}
```

### Event object fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | string | The event id, for example `evt_9Xk2mQp4LrT7vB1z`. It is the same id a webhook delivery of this event carries. It is opaque: store and compare it, but don't parse it or rely on its length. |
| `object` | string | Always `"event"`. |
| `type` | string | One of `file.created`, `contact.created`, `task.created`, `invoice.created`, `expense.created`, `form_submission.created`. See [Events](webhooks.md#events). |
| `created_at` | string (ISO 8601) | When the event was recorded, in UTC, to the second. |
| `account_id` | string | The account the record belongs to. |
| `request` | object \| null | `{ "client_id": "…" }` only when **your own app** created the record through the API; `null` otherwise, including for records created in the app or by another integration. |
| `data.object.type` | string | `file`, `contact`, `task`, `invoice`, `expense`, or `form_submission`. |
| `data.object.id` | string | The record's id. |

An event is a thin pointer. It carries no record data, so fetch the record itself for its
current state: see [Fetching the record](webhooks.md#fetching-the-record). Like everywhere in
the Connect API, ids are strings — see [Identifiers](README.md#identifiers).

## Scopes and visibility

There is no scope of its own. Any token can call these endpoints, and what it sees depends on
the read scopes it holds:

| Event type | Scope the token needs |
| ---------- | --------------------- |
| `file.created` | `files:read` |
| `contact.created` | `contacts:read` |
| `task.created` | `tasks:read` |
| `invoice.created` | `financials:read` |
| `expense.created` | `financials:read` |
| `form_submission.created` | `forms:read` |

An event is listed only if both of these hold:

- the token holds the scope for its type, and
- the member could list the record in its own endpoint right now. For example, a member who
  can only see their own contacts never sees events for anyone else's.

A token with none of these scopes gets an empty list. An event that is not visible to you
returns `404` when retrieved, exactly like one that does not exist. A record that later
becomes visible, for example a contact reassigned to you, shows up in the list from then on.

## Retention and recording

- Events are kept for **30 days**, then removed.
- Events are recorded while the account has at least one connected app with a
  [webhook endpoint](webhooks.md#overview), including a disabled one, so you can recover
  what a failing endpoint missed. Recording stops only once no such app remains (every app's
  endpoints deleted or every such app revoked): records created in the meantime have no event,
  now or later.

## Endpoints

| Method | Path | Scope | Description |
| ------ | ---- | ----- | ----------- |
| `GET` | `/acc/{account_id}/webhook_events` | any read scope | List events |
| `GET` | `/acc/{account_id}/webhook_events/{id}` | any read scope | Retrieve an event |

## List events

```
GET /acc/{account_id}/webhook_events
```

Returns the events visible to your token, **oldest first**, so you can page forward through
history. Supports [pagination](README.md#pagination) via `page` and `limit`.

| Query param | Type | Description |
| ----------- | ---- | ----------- |
| `page` | integer | Page number (default `1`) |
| `limit` | integer | Items per page (default `25`, maximum `100`) |
| `type` | string | Comma-separated event types, for example `contact.created,task.created`. Only those types are returned. An unknown type is a `400`. |
| `created_after` | string (ISO 8601) | Only events created **at or after** this timestamp, for example `2026-10-16T12:00:00Z`. Anything that is not an ISO 8601 timestamp is a `400`. If the timestamp contains a `+`, URL-encode it as `%2B`. |
| `starting_after` | string (event id) | An event `id`, for example `evt_9Xk2mQp4LrT7vB1z`. Only events recorded **after** that one are returned. Events are listed in the order they were recorded, so this pages forward within one walk. It is not a lossless resume point on its own; start recovery from an overlap, see [catching up after downtime](#catching-up-after-downtime). It still works for an event you can no longer see; an id that is not an event of this account is a `400`. |

`created_after` includes events created at exactly that time, and event timestamps are
precise to the second, so the boundary event can appear again. To page forward without
repeats within a walk, use `starting_after`; for recovery after downtime, see
[catching up after downtime](#catching-up-after-downtime).

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/webhook_events?type=contact.created,task.created&created_after=2026-10-16T12:00:00Z&limit=100" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "evt_9Xk2mQp4LrT7vB1z",
      "object": "event",
      "type": "contact.created",
      "created_at": "2026-10-16T12:00:00Z",
      "account_id": "42",
      "request": null,
      "data": { "object": { "type": "contact", "id": "991" } }
    },
    {
      "id": "evt_3hTq8WzR1nYc6KdP",
      "object": "event",
      "type": "task.created",
      "created_at": "2026-10-16T12:01:30Z",
      "account_id": "42",
      "request": { "client_id": "Xk3…" },
      "data": { "object": { "type": "task", "id": "301" } }
    }
  ],
  "meta": {
    "total_count": 2,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 2,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

An unknown `type`, a `created_after` that is not a timestamp, or a `starting_after` that is not an
event id of this account is a `400` with the usual
[error envelope](README.md#errors):

```json
{
  "errors": {
    "status": 400,
    "title": "Unknown event type",
    "source": "type",
    "description": "Unknown: contact.updated. Expected: file.created, contact.created, task.created, invoice.created, expense.created, form_submission.created"
  }
}
```

```json
{
  "errors": {
    "status": 400,
    "title": "Bad request",
    "source": "created_after",
    "description": "created_after must be an ISO 8601 timestamp"
  }
}
```

## Retrieve an event

```
GET /acc/{account_id}/webhook_events/{id}
```

Returns one event, under the same visibility rules as the list. An event that does not exist,
is older than 30 days, or is not visible to your token returns `404`.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/webhook_events/evt_9Xk2mQp4LrT7vB1z" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "evt_9Xk2mQp4LrT7vB1z",
    "object": "event",
    "type": "contact.created",
    "created_at": "2026-10-16T12:00:00Z",
    "account_id": "42",
    "request": null,
    "data": { "object": { "type": "contact", "id": "991" } }
  }
}
```

## Catching up after downtime

Webhooks that fail are [retried for about three days](webhooks.md#retries-and-endpoint-health),
and events stay readable here for 30 days, so you can recover anything you missed:

An event's place in the order is fixed before the event is committed, so a slightly older event
can become visible after a newer one. Paging with `starting_after` alone from the last processed
event can therefore miss it; start each catch-up walk with an overlap and deduplicate:

1. Keep the `id` and `created_at` of the last event you processed, plus the ids you processed
   in the last few minutes.
2. After your endpoint is back, list events with `created_after` set to that `created_at`
   minus 5 minutes.
3. Process the page in order, then set `starting_after` to the last `id` seen in **this**
   walk and query page 1 again. Stop when a page comes back empty. Don't follow `next_page`: offset pages can skip an event when
   visibility changes or the daily purge runs mid-walk.
4. Skip any `id` you have already processed. The overlap, and events that also reached you by
   webhook while you were catching up, will repeat some.
5. Fetch each record with the member's token to read its current state — see
   [Fetching the record](webhooks.md#fetching-the-record).

If you stored no event, make the first request with `created_after` set to the time you went
offline (minus the same 5 minutes) and continue the same way. `created_after` is inclusive.

```bash
# Page forward within a walk: everything after event evt_9Xk2mQp4LrT7vB1z, 100 at a time
curl "https://connect.safariportal.dev/acc/{account_id}/webhook_events?starting_after=evt_9Xk2mQp4LrT7vB1z&limit=100&page=1" \
  -H "Authorization: Bearer <access_token>"
```

If you were offline for longer than 30 days, the oldest events are already gone. List the
resource endpoints (for example [Contacts](contacts.md) or [Tasks](tasks.md)) to find records
created in that gap.

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning |
| ------ | ------- |
| `200` | Success |
| `400` | Unknown `type`, `created_after` is not an ISO 8601 timestamp, or `starting_after` is not an event id of this account (list only) |
| `403` | The token is bound to a different account |
| `404` | The account does not exist or you are not a member, or the event does not exist or is not visible to your token |
