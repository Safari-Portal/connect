# Itineraries

Itineraries are the trip proposals attached to a file (trip / tour request) — the dated,
day-by-day travel documents prepared for a client. Reading returns their **metadata**
(title, who they were prepared for, trip dates, and whether one is the file's confirmed
itinerary); the day-by-day content is not returned. Creating takes that content as events.

Itineraries are listed, retrieved individually and **created** — with their day-by-day
events in the same request. There is no update or delete yet.

Itineraries are a top-level resource: `/acc/{account_id}/itineraries`. Every itinerary
belongs to a file — its `file_id` — and you see an itinerary exactly when you can see its
file (the same visibility rules as [Files](files.md)). The `{account_id}` must be an account
you are an agent member of (otherwise `404`; a token bound to a different account returns
`403`).

> **Changed:** `GET /acc/{account_id}/itineraries` used to list only published itineraries,
> on every file in the account. It now lists drafts too, on the files you can see; send
> `published=true` for the published ones. The file-nested routes
> (`/files/{file_id}/itineraries` and `/files/{file_id}/itineraries/{id}`) still answer but
> are deprecated, and creating is now `POST /acc/{account_id}/itineraries` with `file_id`
> in the body.

Lookbooks are a separate document type with their own endpoint — see [Lookbooks](lookbooks.md).
The itineraries endpoint never returns lookbooks.

## Versioning (v2 / v3)

Safari Portal is migrating itineraries from a legacy engine (**v2**) to a new one (**v3**).
Each itinerary object carries a `version` field (`"v2"` or `"v3"`) telling you which engine
produced it. An account is served by exactly one engine at a time, so within a single
account every itinerary reports the same `version`; it flips from `v2` to `v3` when the
account is migrated. The object shape is identical across versions — only the `version` value
(and, for `confirmed`, its availability) differs. Treat `id` as an opaque string in all cases.

## The itinerary object

```json
{
  "id": "12345",
  "version": "v2",
  "file_id": "42",
  "title": "Kenya Safari — Draft 2",
  "prepared_for": "The Smith Family",
  "start_date": "2026-08-12",
  "end_date": "2026-08-22",
  "confirmed": true,
  "created_at": "2026-06-30T14:12:05.000Z",
  "updated_at": "2026-07-01T09:30:00.000Z"
}
```

### Itinerary object fields

| Field          | Type              | Description                                                                                           |
| -------------- | ----------------- | ---------------------------------------------------------------------------------------------------- |
| `id`           | string            | Unique identifier for the itinerary                                                                  |
| `version`      | string            | Which engine produced it: `"v2"` or `"v3"`                                                            |
| `file_id`      | string            | ID of the file (trip / tour request) this itinerary belongs to                                       |
| `title`        | string \| null    | Internal title of the itinerary                                                                      |
| `prepared_for` | string \| null    | Who the itinerary was prepared for                                                                   |
| `start_date`   | string \| null    | Trip start date (`YYYY-MM-DD`), or `null` when the itinerary has no fixed dates                       |
| `end_date`     | string \| null    | Trip end date (`YYYY-MM-DD`), or `null` when the itinerary has no fixed dates                         |
| `confirmed`    | boolean \| null   | `true`/`false` whether this is the file's confirmed itinerary (`v2`). `null` for `v3`, which has no per-itinerary confirmed state yet |
| `created_at`   | string (ISO 8601) | When the itinerary was created                                                                       |
| `updated_at`   | string (ISO 8601) | When the itinerary was last updated                                                                  |
| `countries`    | array             | Countries the itinerary covers, each `{ id, code, name }`. Returned only by [Retrieve](#retrieve-a-single-itinerary) — populated for `v3`, an empty array (`[]`) for `v2` |

## Scopes

| Scope         | Endpoints        |
| ------------- | ---------------- |
| `files:read`  | List, Retrieve   |
| `files:write` | Create           |

Itineraries reuse the [Files](files.md) scopes: a token that can read a file can read its
itineraries, and one that can write a file can create them. There is no dedicated
itineraries scope.

A token holding `files:write` but not `files:read` may create an itinerary; the `201` then
carries only the new `id`, since the right to write is not the right to read back.

## Endpoints

| Method | Path                                    | Scope         | Description                 |
| ------ | --------------------------------------- | ------------- | --------------------------- |
| `GET`  | `/acc/{account_id}/itineraries`         | `files:read`  | List itineraries            |
| `GET`  | `/acc/{account_id}/itineraries/{id}`    | `files:read`  | Retrieve a single itinerary |
| `POST` | `/acc/{account_id}/itineraries`         | `files:write` | Create an itinerary         |

Deprecated, still answering:

| Method | Path                                                 | Use instead                       |
| ------ | ---------------------------------------------------- | --------------------------------- |
| `GET`  | `/acc/{account_id}/files/{file_id}/itineraries`      | `GET /itineraries?file_id=…`      |
| `GET`  | `/acc/{account_id}/files/{file_id}/itineraries/{id}` | `GET /itineraries/{id}`           |

## List itineraries

```
GET /acc/{account_id}/itineraries
```

Requires `files:read`. Lists the itineraries, drafts included, on the files you can see,
ordered newest first (by `created_at` descending). Supports
[pagination](README.md#pagination) via `page` and `limit`.

| Query param     | Description                                                                                                            |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| `file_id`       | Keep only the itineraries on this file. A file that does not exist, or one you may not see, returns `404`             |
| `published`     | `true` keeps only published itineraries; omitted or `false` lists drafts too                                          |
| `country_codes` | Comma-separated ISO 3166-1 alpha-2 codes (e.g. `KE,TZ`), case-insensitive. Keeps only published itineraries whose published snapshot contains at least one of the given countries |
| `page`          | Page number (default `1`)                                                                                            |
| `limit`         | Items per page (default `25`, maximum `100`)                                                                          |

`published=true` and `country_codes` need a published snapshot, which only `v3` itineraries
have: on an account still on the legacy (`v2`) builder they return `422`. `published=false`
filters nothing, so it lists as usual there.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/itineraries?file_id=42" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "12345",
      "version": "v3",
      "file_id": "42",
      "title": "Kenya Safari — Draft 2",
      "prepared_for": "The Smith Family",
      "start_date": "2026-08-12",
      "end_date": "2026-08-22",
      "confirmed": null,
      "created_at": "2026-06-30T14:12:05.000Z",
      "updated_at": "2026-07-01T09:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 3,
    "total_pages": 1,
    "current_page": 1,
    "current_count": 3,
    "next_page": null,
    "prev_page": null,
    "first_page": true,
    "last_page": true,
    "out_of_range": false
  }
}
```

Fetch a single itinerary with [Retrieve](#retrieve-a-single-itinerary) to get its `countries`.

## Retrieve a single itinerary

```
GET /acc/{account_id}/itineraries/{id}
```

Requires `files:read`. Returns one itinerary. The `{id}` must be an itinerary **in the
account's active engine** on a file you can see — an id from the other engine, one on a
file you may not see, or an unknown id returns `404`.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/itineraries/{id}" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "12345",
    "version": "v2",
    "file_id": "42",
    "title": "Kenya Safari — Draft 2",
    "prepared_for": "The Smith Family",
    "start_date": "2026-08-12",
    "end_date": "2026-08-22",
    "confirmed": true,
    "created_at": "2026-06-30T14:12:05.000Z",
    "updated_at": "2026-07-01T09:30:00.000Z",
    "countries": []
  }
}
```

The retrieve response additionally includes a `countries` array. For `v3` itineraries it lists
the countries the itinerary covers, each entry `{ "id": "…", "code": "KE", "name": "Kenya" }`; for
`v2` itineraries it is always empty (`[]`).

## Create an itinerary

```
POST /acc/{account_id}/itineraries
```

Requires `files:write`, and the account's itineraries must be on the v3 builder. New itineraries are
created unpublished. Send the fields in an `itinerary` object; fields sent at the top level
instead, `events` included, are folded into it, as on every Connect write.

| Field             | Type             | Description                                                                 |
| ----------------- | ---------------- | --------------------------------------------------------------------------- |
| `file_id`         | string           | **Required.** The file the itinerary belongs to; it must be a file you can see |
| `title`           | string           | **Required.** Internal title                                                |
| `prepared_for`    | string \| null   | Who the itinerary is for                                                    |
| `subheadline`     | string \| null   | Subheadline shown under the title                                           |
| `events`          | array            | Day-by-day events: at most 50; those laid out day by day cover at most 180 days in total (`multi_day_page` does not count). See below |

Each entry in `events`:

| Field          | Type             | Description                                                                          |
| -------------- | ---------------- | ------------------------------------------------------------------------------------ |
| `kind`         | string           | **Required.** One of `flight`, `transportation`, `accommodation`, `activity`, `area`, `information` |
| `start_date`   | string           | **Required.** `YYYY-MM-DD`                                                           |
| `end_date`     | string           | `YYYY-MM-DD`. Omit for a single-day event: an absent `end_date` takes the `start_date` |
| `title`        | string \| null   | Event title, shown on its first day instead of the automatic one ("Check in at …", "Flight from … to …"), as when typed in the builder. Omit it to keep the automatic title |
| `display_settings` | object        | `{ "mode": … }` — how the event is drawn across its days. The modes depend on `kind`, see [Events by kind](#events-by-kind); omit for the default |
| `content_id`   | string \| null   | Content Library page to attach. Must be published and visible to the account; an `accommodation` event accepts a property page only. Find the id with [List content pages](contents.md#list-content-pages) |
| `data`         | object           | Kind-specific fields, see [Events by kind](#events-by-kind)                         |

> **Send dates, not day numbers.** There is no `positions`, no `days_range` and no day
> index anywhere in this payload. You give real dates; the server derives the per-day
> slots and the ordering within each day.

> **The trip's dates come from its events.** There is no `trip_start_date` to send and no
> builder to pick: the itinerary is created on the Super Builder, its `start_date` is the
> first event's day and its `end_date` the last event's, as in the builder. An itinerary
> created with no events has no dates until one is added.

### Events by kind

| `kind`           | `display_settings.mode` (default first)                              | `data` fields |
| ---------------- | -------------------------------------------------------------------- | ------------- |
| `flight`         | `content_on_start_day_with_daily_summary`                             | `flight_number`, `seat_numbers`, `cabin_class`, `departure_terminal`, `arrival_terminal`, `from_fs_service`; manual flight: `manual_airline`, `manual_departure_from`, `manual_arrival_to`, `manual_departure_time`, `manual_arrival_time`, `manual_duration`, `booked_by`, `confirmation_number`; FlightStats flight: `flight_date`, `departure_time`, `arrival_time`, `new_departure_time`, `new_arrival_time`, `cancelled`, `record_locator`, `airline`, `departure_airport`, `arrival_airport` |
| `transportation` | `content_on_start_day_with_daily_summary`                             | `transportation_type` (`transfer` — the default, `train`, `rental_car`, `other`), `departure_time`, `arrival_time`, `departure_from`, `arrival_to`, `duration`, `confirmation_number`, `provider`, `booked_by`; `transfer`/`other`/`rental_car`: `vehicle_type`; `train`: `train_type`, `train_number`, `coach_number`, `seats`, `service_class`; `rental_car`: `vehicle_number`, `rental_company_name`, `rental_company_address`; contacts: `contacts_mode`, `contacts`, `manual_contact` |
| `accommodation`  | `multi_day_page`, `check_in_out_only`, `check_in_overnights_check_out` | `room_basis`, `bed_configuration`, `confirmation_number`, `check_in`, `check_out`, `standard_check_in_time`, `standard_check_out_time`, `policies_note`, `amenities_note`, `inclusions`, `exclusions`, `is_alternative`, `rooms`, `different_rooms_per_travelers`, `roomings`; contacts: `contacts_mode`, `contacts`, `manual_contact` |
| `activity`       | `multi_day_page`, `content_on_start_day_with_daily_summary`           | `start_time`, `end_time`, `activity_type`, `confirmation_number`, `supplier_name`, `booked_via_name`, `inclusions`, `exclusions`; contacts: `contacts_mode`, `contacts`, `manual_contact` |
| `area`           | `multi_day_page`, `arrival_day_only`                                  | none |
| `information`    | `multi_day_page`, `content_on_start_day_with_daily_summary`           | none |

- **`multi_day_page` needs more than one day.** A single-day event gets the kind's other mode,
  whether you asked for `multi_day_page` or sent no mode at all.
- **Times** are strings as `HH:MM` on a 24-hour clock, as the builder's time fields send them;
  `""` leaves one empty, and anything else (`9:30`, `2pm`, `24:00`) is a `422`. A FlightStats
  `departure_time`/`arrival_time`/`new_departure_time`/`new_arrival_time` is a full
  `YYYY-MM-DDTHH:MM`, seconds allowed.
- **A flight is dated by the event.** Its `start_date`/`end_date` are written into the flight,
  including the date part of a FlightStats departure and arrival time, so `data` needs no date
  but a FlightStats flight's `flight_date` (below). A FlightStats flight ignores `end_date` and,
  as in the builder, ends as many days after `start_date` as it lands after it departs: an
  overnight `21:40` → `08:00` on `2026-08-12` ends on `2026-08-13`. Fields of the other source are cleared: a manual flight (`from_fs_service`
  false or absent) keeps no FlightStats fields, and a FlightStats one keeps no manual ones.
  A FlightStats flight needs `flight_number`, `departure_time` and `arrival_time`, as the
  builder's flight search fills them; without them it is a `422`. Its `flight_date` is the date
  the flight was searched for and is kept as sent: when it differs from `start_date`, the builder
  warns that the times may be stale.
- **`airline`** is `{ "id", "fs_code", "name", "active" }`; **`departure_airport`** and
  **`arrival_airport`** are `{ "id", "fs_code", "name", "city", "city_code", "country_name",
  "utc_offset_hours", "active" }`.
- **`rooms`** is a list of `{ "name", "count" }` — `name` is required, `count` runs 1–100 and
  defaults to 1. The rooms of a property page carry a `uuid` as well, which the API cannot list
  yet; a room without a `uuid` is your own.
- **`roomings`** splits the stay per traveler and applies only with
  `"different_rooms_per_travelers": true`, which clears the shared room fields above. Each entry
  takes `travelers` (`[{ "id", "name" }]`), `rooms`, `room_basis`, `bed_configuration`,
  `confirmation_number`, `check_in` and `check_out`. Traveler ids come from
  [Retrieve a file](files.md), which lists them for a token that also holds `contacts:read`.
- **Contacts** are the event's Contacts section, a copy rather than a link to the account's
  contacts. `contacts` is a list of `{ "name", "email", "address", "website", "phone", "role" }`
  (`name` required), `manual_contact` one such object without `role`, and `contacts_mode` picks
  which the event shows: `list` (the default) or `manual`. Send the fields of that tab only, as
  the builder does: `contacts` filled with `contacts_mode` `manual`, or `manual_contact` with
  `list` (or no mode), is a `422`. An empty other tab — `[]` or `null`, as a read returns it — is fine.
- **`activity_type`** is stored with the activity but not drawn anywhere yet.

```json
{ "kind": "accommodation", "title": "Chobe Game Lodge",
  "start_date": "2026-11-06", "end_date": "2026-11-08",
  "display_settings": { "mode": "check_in_overnights_check_out" },
  "data": { "room_basis": "Full Board", "rooms": [ { "name": "River Suite", "count": 2 } ] } }
```

```bash
curl -X POST https://connect.safariportal.dev/acc/{account_id}/itineraries \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "itinerary": {
      "file_id": "42",
      "title": "Victoria Falls — Draft 1",
      "prepared_for": "The Smith Family",
      "events": [
        { "kind": "flight", "title": "LHR → LVI",
          "start_date": "2026-11-04", "end_date": "2026-11-04" },
        { "kind": "accommodation", "title": "Matetsi River Lodge",
          "start_date": "2026-11-04", "end_date": "2026-11-07",
          "display_settings": { "mode": "check_in_overnights_check_out" },
          "data": { "room_basis": "Full Board" } }
      ]
    }
  }'
```

**Response `201`**

```json
{
  "data": {
    "id": "178",
    "version": "v3",
    "file_id": "42",
    "title": "Victoria Falls — Draft 1",
    "prepared_for": "The Smith Family",
    "start_date": "2026-11-04",
    "end_date": "2026-11-07",
    "confirmed": null,
    "created_at": "2026-09-14T11:28:20.784Z",
    "updated_at": "2026-09-14T11:28:21.113Z",
    "countries": []
  }
}
```

### Refusals

The whole request is one transaction: if any event is refused, nothing is created — not the
itinerary, not the events before it. A refusal on an event **names its index**, so you know
which entry to fix:

```json
{
  "errors": {
    "status": 422,
    "title": "Validation failed",
    "description": "events[1]: End date must be on or after the start date"
  }
}
```

| Status | When                                                                                          |
| ------ | --------------------------------------------------------------------------------------------- |
| `400`  | Neither an `itinerary` key nor any itinerary field, or `itinerary` is not an object; `events` is not a list of objects; the body is not valid JSON |
| `403`  | The token lacks `files:write`, is bound to another account, or the account's itineraries are still on the legacy builder |
| `422`  | A field was refused — missing `file_id` or a file that does not exist, belongs to another account or you may not see; missing `title`, a date that is not `YYYY-MM-DD`, more than 50 events or 180 days of day-by-day events, a `data` field or `display_settings` of the wrong shape, a number inside `data` sent as anything but a whole number or a yes/no field as anything but `true`/`false`, a time not as `HH:MM`, a FlightStats flight without its number and times, or any event rule (`description` names the index and, inside `data`, the field) |
| `429`  | More than ten create requests in a minute on one token, refused ones included; wait the seconds `Retry-After` gives (60) and resend |

A request at both limits (50 events, 180 days of day-by-day events) takes the server up to
about 25 seconds to answer. Give the request a client timeout of at least 30 seconds, and do not
resend on a slow answer: the first request may still create the itinerary.

Dates are checked before anything is written, because `12/08/2026` would otherwise be read
as 8 December and quietly build a different trip.

A field the endpoint does not know is **dropped** before processing, as if it had not been
sent — on the itinerary, on an event and inside `data`. On the itinerary a value of the wrong
type (an object where text belongs) is dropped too, the same as the other Connect writes; on an
event and inside `data` it is a `422` naming the field. Check names against the tables
above: `trip_events` instead of `events` creates an itinerary with no events, and
`display_mode` instead of `display_settings` leaves the event on its default mode.

> **Create is not idempotent yet.** Retrying after a timeout creates a second itinerary. Before
> retrying, [list the file's itineraries](#list-itineraries) (`file_id=…`) and look for the one you sent.

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.
