# Tasks

Tasks are to-dos attached to a file (trip / tour request). Each task has an optional
due date, one or more types, tags, and an assignee. Every task belongs to exactly one
file.

Tasks can be listed (with optional filtering), retrieved, and created. There is no
update or delete endpoint — a task's state changes in the app.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`, and a token bound to a different account returns `403`. A token
sees all tasks for that account (subject to the member being an agent). Requesting a
task that does not exist in the account returns `404`.

## The task object

```json
{
  "id": "301",
  "name": "Send pre-trip welcome email",
  "note": "Include packing list and visa reminders.",
  "types": ["email", "follow-up"],
  "tags": ["vip", "africa"],
  "due_date": "2027-03-15",
  "completed": false,
  "completed_at": null,
  "file_id": "42",
  "assigned_to_id": "11",
  "creator_id": "7",
  "template_id": null,
  "created_at": "2026-06-01T08:00:00.000Z",
  "updated_at": "2026-06-15T12:30:00.000Z"
}
```

### Task object fields

| Field            | Type              | Description                                                                 |
| ---------------- | ----------------- | --------------------------------------------------------------------------- |
| `id`             | string            | Unique identifier for the task                                              |
| `name`           | string            | Display name of the task                                                    |
| `note`           | string \| null    | Optional free-text note                                                     |
| `types`          | array of strings  | Task type labels. Valid values: `email`, `payments`, `check-in`, `follow-up`, `phone-call`, `admin`, `feedback` |
| `tags`           | array of strings  | Tags applied to the task                                                    |
| `due_date`       | string \| null    | Due date in `YYYY-MM-DD` format, or `null` if not set (TBD)                |
| `completed`      | boolean           | `true` if the task has been completed                                       |
| `completed_at`   | string \| null    | ISO 8601 datetime when the task was completed, or `null`                   |
| `file_id`        | string            | ID of the file (trip / tour request) this task belongs to                  |
| `assigned_to_id` | string            | ID of the user the task is assigned to — resolve via the [Users](users.md) endpoint |
| `creator_id`     | string            | ID of the user who created the task — resolve via the [Users](users.md) endpoint    |
| `template_id`    | string \| null    | ID of the task template this task was created from, or `null`              |
| `created_at`     | string (ISO 8601) | When the task was created                                                   |
| `updated_at`     | string (ISO 8601) | When the task was last updated                                              |

## Scopes

| Scope         | Endpoints  |
| ------------- | ---------- |
| `tasks:read`  | List, Get  |
| `tasks:write` | Create     |

`tasks:write` is offered as its own tick on the consent screen and is granted only if
the user takes it — see [Authentication](authentication.md#authorization-code--pkce).
It does not imply `tasks:read`; the two are independent, and a token can hold either
alone.

## Endpoints

| Method | Path                             | Scope         | Description         |
| ------ | -------------------------------- | ------------- | ------------------- |
| `GET`  | `/acc/{account_id}/tasks`        | `tasks:read`  | List / filter tasks |
| `GET`  | `/acc/{account_id}/tasks/:id`    | `tasks:read`  | Retrieve a task     |
| `POST` | `/acc/{account_id}/tasks`        | `tasks:write` | Create a task on a file |

## List tasks

```
GET /acc/{account_id}/tasks
```

Requires `tasks:read`. With no filter parameters, returns all tasks for the account
ordered by due date ascending. Supplying any of the filters below narrows the results;
`scope` alone is enough to filter by due-date window. Supports
[pagination](README.md#pagination) via `page` and `limit`.

| Query param          | Type             | Description                                                                                                                                           |
| -------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `page`               | integer          | Page number (default `1`)                                                                                                                             |
| `limit`              | integer          | Items per page (default `25`, maximum `100`)                                                                                                          |
| `scope`              | string           | Due-date / completion window. One of `overdue`, `due-today`, `due-this-week`, `upcoming`, `completed`, `tbd`, `active-tasks`. When any filter other than `scope` is supplied without a `scope`, it defaults to `overdue`. |
| `completion`         | string           | Completion filter. One of `done`, `not_done`.                                                                                                         |
| `order`              | string           | Sort order. One of `due-date-asc` (default), `due-date-desc`, `assigned-to-asc`, `assigned-to-desc`, `file-asc`, `file-desc`.                       |
| `assigned_to_ids[]`  | array of strings | Filter by assigned user ID.                                                                                                                           |
| `file_ids[]`         | array of strings | Filter by file (tour request) ID.                                                                                                                     |
| `types[]`            | array of strings | Filter by task type. Valid values: `email`, `payments`, `check-in`, `follow-up`, `phone-call`, `admin`, `feedback`. Tasks matching any supplied type are returned. |
| `tags[]`             | array of strings | Filter by tag.                                                                                                                                        |
| `include_tags`       | boolean          | When `tags[]` is supplied: `true` (default) returns tasks that have **all** of the given tags; `false` returns tasks that have **none** of them.      |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/tasks?scope=due-this-week&types[]=email&assigned_to_ids[]=11" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "301",
      "name": "Send pre-trip welcome email",
      "note": "Include packing list and visa reminders.",
      "types": ["email", "follow-up"],
      "tags": ["vip", "africa"],
      "due_date": "2027-03-15",
      "completed": false,
      "completed_at": null,
      "file_id": "42",
      "assigned_to_id": "11",
      "creator_id": "7",
      "template_id": null,
      "created_at": "2026-06-01T08:00:00.000Z",
      "updated_at": "2026-06-15T12:30:00.000Z"
    }
  ],
  "meta": {
    "total_count": 58,
    "total_pages": 6,
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

## Retrieve a task

```
GET /acc/{account_id}/tasks/:id
```

Requires `tasks:read`. Returns `404` if the task does not exist or does not belong to
the account.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/tasks/301 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "301",
    "name": "Send pre-trip welcome email",
    "note": "Include packing list and visa reminders.",
    "types": ["email", "follow-up"],
    "tags": ["vip", "africa"],
    "due_date": "2027-03-15",
    "completed": false,
    "completed_at": null,
    "file_id": "42",
    "assigned_to_id": "11",
    "creator_id": "7",
    "template_id": null,
    "created_at": "2026-06-01T08:00:00.000Z",
    "updated_at": "2026-06-15T12:30:00.000Z"
  }
}
```

## Create a task

```
POST /acc/{account_id}/tasks
```

Requires `tasks:write`. Files one task on one file. `file_id`, `name` and `due_date`
are required — a task with no due date (`tbd`) can exist in the app but cannot be
created through this endpoint, because the alternative is a dynamic due date computed
from trip events, which is not writable here.

| Field                     | Type             | Notes                                                                       |
| ------------------------- | ---------------- | --------------------------------------------------------------------------- |
| `file_id` *(required)*    | string           | The file (trip) the task belongs to. Must be a file **you** can see — see below. |
| `name` *(required)*       | string           | Up to 255 characters                                                        |
| `due_date` *(required)*   | string           | `YYYY-MM-DD`. Other parseable date formats are accepted but not guaranteed. |
| `assigned_to_id`          | string (user id) | Defaults to the member the token belongs to. Must be an **active** member.  |
| `note`                    | string           | Free text                                                                   |
| `types`                   | array of strings | `email`, `payments`, `check-in`, `follow-up`, `phone-call`, `admin`, `feedback` |
| `tags`                    | array of strings | Matched exactly, and case-sensitive                                         |

The body is wrapped in a `task` envelope. A `task` whose values are wrong is a `422`.
A body that carries neither a `task` key nor any recognised task field is a `400` — send
the envelope, but note that top-level task fields are folded into it rather than rejected,
so `{"name": "..."}` reaches the same `422`s as `{"task": {"name": "..."}}`.

**`tags` is an array** — `["vip", "africa"]`, the same shape reads return, on every
resource in this API.

Creating a task with type `email` creates it **without** email content; it cannot be
sent until content is added in the app.

```bash
curl -X POST https://connect.safariportal.dev/acc/{account_id}/tasks \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "task": {
      "file_id": "42",
      "name": "Chase the Kenya deposit",
      "due_date": "2026-08-14",
      "assigned_to_id": "11",
      "types": ["payments", "follow-up"],
      "tags": ["vip"]
    }
  }'
```

**Response `201`** — the full task object, exactly as `GET /tasks/:id` returns it.

### What create returns without `tasks:read`

Echoing the record back is a **read**. A token holding `tasks:write` without
`tasks:read` therefore receives only the new task's id:

```json
{ "data": { "id": "301" } }
```

The id is always returned, so a write-only client can still reference what it created.
Add `tasks:read` to the grant to get the full task back instead. The same rule applies
to [contacts](contacts.md) and [files](files.md).

### Refusals

Every refusal below is a `422` carrying the reason in `errors.description`, phrased in
the field names you sent rather than the columns they land in.

| `errors.description`                                    | Cause                                                                 |
| ------------------------------------------------------- | --------------------------------------------------------------------- |
| `File This field is required`                           | No `file_id`                                                          |
| `Name This field is required`                           | No `name`, or an empty one                                            |
| `Due date This field is required and Due date conditions This field is required` | No `due_date`, or one that does not parse as a date |
| `File must be accessible to you in this account`        | The file is in another account, is deleted, or is one your membership does not reach — a consultant sees only the files they consult on, and can write to exactly the files they can read |
| `Assigned to must be an active member of this account`  | Unknown user id, or a member deactivated in this account or globally. The caller is **never** substituted in: an id naming nobody is refused, not quietly replaced |
| `Task Type is invalid`                                  | A value in `types` outside the list above                             |

Values must have the shapes the table above documents. A value of the wrong shape (an object
where a scalar belongs, a bare string where an array belongs) is **dropped** before processing,
as if the field had not been sent — so a wrongly-shaped optional field is silently ignored, and
a wrongly-shaped required field reports as missing.

A token without `tasks:write` gets `403`, and a member who is not an agent of the
account gets `404` — the same gates the read endpoints apply.

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                            |
| ------ | ------------------------------------------------------------------ |
| `200`  | Success                                                            |
| `201`  | The task was created                                               |
| `400`  | The body carries neither a `task` key nor any recognised task field |
| `403`  | The token lacks the `tasks:read` / `tasks:write` scope             |
| `404`  | The task does not exist or belongs to a different account          |
| `422`  | The task could not be created — see the refusals above             |
