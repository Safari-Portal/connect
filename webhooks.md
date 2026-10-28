# Webhooks

Webhooks tell your integration when a record is created, so you don't have to poll. When a
file, contact, task, invoice, expense, or form submission is created in an account where a
member has authorized your app, Safari Portal sends a small signed `POST` to your endpoint.
Your integration then fetches the record from the Connect API with that member's token.

## Overview

Endpoints belong to your [OAuth application](authentication.md#registering-a-client), not to
a single account. Configure them once and they receive events from **every account** where
a member has authorized the app.

To add an endpoint, open **Settings → OAuth Applications**, choose the app's menu, and
select **Webhooks**. For each endpoint you set:

- **Endpoint URL** — a public `https://` URL. Plain HTTP, and addresses that point to a
  private or internal network, are rejected.
- **Description** — optional, for your own reference.
- **Events** — the event types to receive, or **All events**.

Notes:

- An app can have at most **5** endpoints.
- Each endpoint has a **signing secret** (starting with `whsec_`). Open the endpoint to copy
  it, and use the full value, prefix included, to [verify signatures](#verifying-signatures).
  Store it like a password.
- An endpoint only receives events for records that the authorizing member's token could
  read, so your app must hold the matching [scope](authentication.md#scopes).
- Apps registered by clients themselves, rather than from Settings, have no webhook
  endpoints.

## Events

Every event announces a record that was **created**.

| Event type | Sent when | Scope the grant needs |
| ---------- | --------- | --------------------- |
| `file.created` | A file (trip) is created | `files:read` |
| `contact.created` | A contact is created | `contacts:read` |
| `task.created` | A task is created | `tasks:read` |
| `invoice.created` | An invoice becomes shareable (see below) | `financials:read` |
| `expense.created` | An expense is created | `financials:read` |
| `form_submission.created` | A form submission is confirmed (see below) | `forms:read` |

An endpoint set to **All events** (`*`) receives every event type, including types added
later. Ignore any `type` your handler doesn't recognise and still respond with a `2xx`, so a new
event type never looks like a failing endpoint.

### When "created" fires

An event fires at the moment the record first becomes **available through the Connect API**,
so that fetching it never answers `404`. For most records that is the moment they are created.
Two are different:

- **Invoices** are announced once they can be shared, which means their payment methods are
  set. An invoice created without payment methods is announced later, when payment methods
  are added. Further edits to an invoice never send another event.
- **Form submissions** are announced once they are confirmed. When the form requires the
  traveler to verify their email, that is when the email is verified, not when the form is
  submitted.

### Who is told

Webhooks follow the same visibility rules as the API. A record is announced to a member's
grant only if the member could see it in the matching list endpoint at that moment, and only
if the grant holds the scope in the table above. For example, a member who can only see
their own contacts is never told about anyone else's.

- A record that **becomes** visible to a member later, for example a contact reassigned to
  them, is not announced to them.
- One event can produce several deliveries: if two members of an account authorized your app,
  each one's grant receives its own delivery of the same event, with their own `user_id`. If
  the same member authorized the app more than once, only their newest authorization
  receives it.
- When an account is exempted from the payment-methods requirement, its invoices that were
  hidden until then become available without being edited, so they are never announced.
  List the account's invoices to pick them up.
- Records created by bulk operations that bypass normal record creation are not announced.
- Several records created in one request send one event each.

## Payload

The body is a JSON object that identifies the record. It deliberately carries no record data,
so access is always checked against your token when you fetch it.

```json
{
  "id": "evt_9Xk2mQp4LrT7vB1z",
  "object": "event",
  "type": "contact.created",
  "created_at": "2026-10-16T12:00:00Z",
  "account_id": "42",
  "user_id": "7",
  "request": { "client_id": "Xk3…" },
  "data": { "object": { "type": "contact", "id": "991" } }
}
```

| Field | Description |
| ----- | ----------- |
| `id` | The event id, for example `evt_9Xk2mQp4LrT7vB1z`. Every delivery of the same event carries the same `id`. It is opaque: store and compare it, but don't parse it or rely on its length. |
| `object` | Always `"event"`. |
| `type` | The [event type](#events). |
| `created_at` | When the event was recorded, in ISO 8601 UTC. |
| `account_id` | The account the record belongs to. Use it as `{account_id}` in the API path. |
| `user_id` | The member whose authorization this delivery is for. Fetch the record with **that member's** access token. |
| `request` | Set when your app created this record through the API, `null` otherwise; records created as a side effect (e.g. tasks from a task template) carry `null` too. `request.client_id` is your app's `client_id`, so you can recognize records you created yourself. Events are still sent to you for them. |
| `data.object.type` | `file`, `contact`, `task`, `invoice`, `expense`, or `form_submission`. |
| `data.object.id` | The record's id. |

As everywhere in the Connect API, ids are strings — see
[Identifiers](README.md#identifiers).

### Fetching the record

Use the `account_id` and `data.object.id` from the payload, with the access token of the
member named by `user_id`:

| `data.object.type` | Fetch with |
| ------------------ | ---------- |
| `file` | `GET /acc/{account_id}/files/{id}` — see [Files](files.md) |
| `contact` | `GET /acc/{account_id}/contacts/{id}` — see [Contacts](contacts.md) |
| `task` | `GET /acc/{account_id}/tasks/{id}` — see [Tasks](tasks.md) |
| `invoice` | `GET /acc/{account_id}/invoices/{id}` — see [Invoices](invoices.md) |
| `expense` | `GET /acc/{account_id}/expenses/{id}` — see [Expenses](expenses.md) |
| `form_submission` | `GET /acc/{account_id}/form_submissions/{id}` — see [Form Submissions](form_submissions.md) |

The record can be changed or deleted between the event and your fetch. Treat the event as a
prompt to look, and expect an occasional `404`.

## Verifying signatures

Every request carries a `Safari-Signature` header, so you can confirm it came from Safari
Portal and wasn't altered:

```
Safari-Signature: t=1760616000,v1=fc177dff096ad3a3ec986e7b37eda970cebef5cda927fd25f9b8337118e9fb05
```

- `t` is the Unix time, in seconds, when the request was signed.
- `v1` is the hex-encoded HMAC-SHA256 of `"{t}.{body}"`, keyed with the endpoint's signing
  secret. `{body}` is the raw request body, exactly as received.

To verify a request:

1. Split the header into `t` and `v1`.
2. Compute the HMAC-SHA256 of `t`, a period, and the **raw** body, using your signing secret.
3. Compare it with `v1` in constant time.
4. Reject the request if `t` is more than 5 minutes from your clock. This narrows the window
   in which a captured request can be replayed but doesn't close it, so also skip any event
   `id` you have already processed (see [Delivery](#delivery)).

Sign the raw bytes you received. Re-serializing the parsed JSON can change the bytes and
break the signature.

```ruby
def valid_signature?(header, body, secret, tolerance: 300)
  parts = header.to_s.scan(/(\w+)=([^,]+)/).to_h
  return false unless parts["v1"] && parts["t"]&.match?(/\A\d+\z/)

  timestamp = parts["t"].to_i
  expected = OpenSSL::HMAC.hexdigest("SHA256", secret, "#{timestamp}.#{body}")
  (Time.now.to_i - timestamp).abs <= tolerance &&
    ActiveSupport::SecurityUtils.secure_compare(expected, parts["v1"])
end
```

```js
import crypto from "node:crypto";

export function validSignature(header, body, secret, tolerance = 300) {
  const { t, v1 } = Object.fromEntries(String(header).split(",").map((p) => p.split("=", 2)));
  if (!v1 || !/^\d+$/.test(t ?? "")) return false;
  const timestamp = Number(t);
  const expected = Buffer.from(crypto.createHmac("sha256", secret).update(`${timestamp}.${body}`).digest("hex"));
  const given = Buffer.from(v1);
  return Math.abs(Date.now() / 1000 - timestamp) <= tolerance &&
    expected.length === given.length &&
    crypto.timingSafeEqual(expected, given);
}
```

## Delivery

Each request is a `POST` with these headers:

| Header | Value |
| ------ | ----- |
| `Content-Type` | `application/json` |
| `User-Agent` | `SafariPortal-Webhooks/1.0` |
| `Safari-Signature` | `t=…,v1=…`, see [Verifying signatures](#verifying-signatures) |

- **Respond with a `2xx` within 10 seconds,** and finish sending the response within 30
  seconds overall; a response that trickles in slower is abandoned. Do slow work after you
  have responded, for example by queuing a job. Any other status, a timeout, or a connection
  error counts as a failure, and the delivery is [retried](#retries-and-endpoint-health).
- **Redirects are not followed.** A `3xx` response counts as a failure, so register the final
  URL.
- **Delivery is at least once.** A delivery can arrive more than once. Safari Portal leases
  each delivery and enforces the deadline above to keep duplicates rare, but cannot rule them
  out: after a timeout or network failure we record the attempt as failed even if you had
  already processed the request, and retry it. Deduplicate on the event `id`, and make your
  handler safe to run again. If you act per member, deduplicate on `id` together with
  `user_id`, since one event is delivered once per authorizing member.
- **Order is not guaranteed.** Events can arrive out of order, and a retried event can arrive
  after newer ones. Use `created_at` and the record itself, not arrival order.
- **Endpoints must be reachable from the public internet.** Safari Portal refuses to connect
  to private, loopback, and other internal addresses. To test locally, expose your server with
  a tunnel that gives you a public HTTPS URL.

## Retries and endpoint health

A failed delivery is retried up to 12 more times, after 1 minute, 5 minutes, 30 minutes,
2 hours and 5 hours, then every 10 hours. That is about three days in all. After the last
retry the delivery is marked failed and is not sent again.

Safari Portal also watches each endpoint as a whole. An endpoint is "failing" from its first
failed delivery until one succeeds.

- **After 24 hours of failing**, we email a warning to the member who created the OAuth
  application, or to the account owner if that member has left the account.
- **After 3 days of failing**, the endpoint is disabled and we email the same person. A
  disabled endpoint receives nothing, and no new deliveries are queued for it.

To re-enable an endpoint, open **Settings → OAuth Applications → Webhooks**, fix the URL if
needed, and choose **Enable**.

Events missed while an endpoint was failing or disabled stay readable for 30 days through
the [Events API](webhook_events.md), so you can catch up once it is back. Events are not
recorded while the account has no webhook endpoint at all, for example before you add one or
after you delete it. For that interval, list the resource endpoints instead.

## Catching up

After downtime, list the events you missed with the [Events API](webhook_events.md#catching-up-after-downtime)
instead of waiting on retries: start from a few minutes before the last event you processed,
page forward, and skip ids you have seen.
