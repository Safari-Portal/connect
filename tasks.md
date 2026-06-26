# Tasks

Tasks are to-dos attached to a file (trip / tour request). Each task has an optional
due date, one or more types, tags, and an assignee. Every task belongs to exactly one
file.

This is a **read-only** API. Tasks can be listed, searched, and retrieved. There are no
create or update endpoints.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; using a different account id returns `403`. A token
sees all tasks for that account (subject to the member being an agent). Requesting a
task that does not exist in the account returns `404`.

## The task object

```json
{
  "id": 301,
  "name": "Send pre-trip welcome email",
  "note": "Include packing list and visa reminders.",
  "types": ["email", "follow-up"],
  "tags": ["vip", "africa"],
  "due_date": "2027-03-15",
  "completed": false,
  "completed_at": null,
  "file_id": 42,
  "assigned_to_id": 11,
  "creator_id": 7,
  "template_id": null,
  "created_at": "2026-06-01T08:00:00.000Z",
  "updated_at": "2026-06-15T12:30:00.000Z"
}
```

### Task object fields

| Field            | Type              | Description                                                                 |
| ---------------- | ----------------- | --------------------------------------------------------------------------- |
| `id`             | integer           | Unique identifier for the task                                              |
| `name`           | string            | Display name of the task                                                    |
| `note`           | string \| null    | Optional free-text note                                                     |
| `types`          | array of strings  | Task type labels. Valid values: `email`, `payments`, `check-in`, `follow-up`, `phone-call`, `admin`, `feedback` |
| `tags`           | array of strings  | Tags applied to the task                                                    |
| `due_date`       | string \| null    | Due date in `YYYY-MM-DD` format, or `null` if not set (TBD)                |
| `completed`      | boolean           | `true` if the task has been completed                                       |
| `completed_at`   | string \| null    | ISO 8601 datetime when the task was completed, or `null`                   |
| `file_id`        | integer           | ID of the file (trip / tour request) this task belongs to                  |
| `assigned_to_id` | integer           | ID of the user the task is assigned to — resolve via the [Users](users.md) endpoint |
| `creator_id`     | integer           | ID of the user who created the task — resolve via the [Users](users.md) endpoint    |
| `template_id`    | integer \| null   | ID of the task template this task was created from, or `null`              |
| `created_at`     | string (ISO 8601) | When the task was created                                                   |
| `updated_at`     | string (ISO 8601) | When the task was last updated                                              |

## Scopes

| Scope        | Endpoints           |
| ------------ | ------------------- |
| `tasks:read` | List, Search, Get   |

There are no write scopes. This API is read-only.

## Endpoints

| Method | Path                             | Scope        | Description      |
| ------ | -------------------------------- | ------------ | ---------------- |
| `GET`  | `/acc/{account_id}/tasks`        | `tasks:read` | List tasks       |
| `GET`  | `/acc/{account_id}/tasks/search` | `tasks:read` | Search tasks     |
| `GET`  | `/acc/{account_id}/tasks/:id`    | `tasks:read` | Retrieve a task  |

## List tasks

```
GET /acc/{account_id}/tasks
```

Requires `tasks:read`. Returns all tasks for the account ordered by due date ascending.
Supports [pagination](README.md#pagination) via `page` and `limit`.

If any search parameter other than `order` or `scope` is present, the request behaves
identically to [Search tasks](#search-tasks).

| Query param | Description                                          |
| ----------- | ---------------------------------------------------- |
| `page`      | Page number (default `1`)                            |
| `limit`     | Items per page (default `25`, maximum `100`)         |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/tasks?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": 301,
      "name": "Send pre-trip welcome email",
      "note": "Include packing list and visa reminders.",
      "types": ["email", "follow-up"],
      "tags": ["vip", "africa"],
      "due_date": "2027-03-15",
      "completed": false,
      "completed_at": null,
      "file_id": 42,
      "assigned_to_id": 11,
      "creator_id": 7,
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

## Search tasks

```
GET /acc/{account_id}/tasks/search
```

Requires `tasks:read`. Filters tasks by the supplied parameters. Supports
[pagination](README.md#pagination) via `page` and `limit`. The `scope` param narrows
results to a specific due-date or completion window; it defaults to `"overdue"`.

| Query param          | Type             | Description                                                                                                                                           |
| -------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope`              | string           | Due-date / completion window. One of `overdue` (default), `due-today`, `due-this-week`, `upcoming`, `completed`, `tbd`, `active-tasks`.               |
| `completion`         | string           | Completion filter. One of `done`, `not_done`.                                                                                                         |
| `order`              | string           | Sort order. One of `due-date-asc` (default), `due-date-desc`, `assigned-to-asc`, `assigned-to-desc`, `file-asc`, `file-desc`.                       |
| `assigned_to_ids[]`  | array of integers| Filter by assigned user ID.                                                                                                                           |
| `file_ids[]`         | array of integers| Filter by file (tour request) ID.                                                                                                                     |
| `types[]`            | array of strings | Filter by task type. Valid values: `email`, `payments`, `check-in`, `follow-up`, `phone-call`, `admin`, `feedback`. Tasks matching any supplied type are returned. |
| `tags[]`             | array of strings | Filter by tag.                                                                                                                                        |
| `include_tags`       | boolean          | When `tags[]` is supplied: `true` (default) returns tasks that have **all** of the given tags; `false` returns tasks that have **none** of them.      |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/tasks/search?scope=due-this-week&types[]=email&assigned_to_ids[]=11" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`** — same envelope shape as [List tasks](#list-tasks).

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
    "id": 301,
    "name": "Send pre-trip welcome email",
    "note": "Include packing list and visa reminders.",
    "types": ["email", "follow-up"],
    "tags": ["vip", "africa"],
    "due_date": "2027-03-15",
    "completed": false,
    "completed_at": null,
    "file_id": 42,
    "assigned_to_id": 11,
    "creator_id": 7,
    "template_id": null,
    "created_at": "2026-06-01T08:00:00.000Z",
    "updated_at": "2026-06-15T12:30:00.000Z"
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.

| Status | Meaning                                                            |
| ------ | ------------------------------------------------------------------ |
| `200`  | Success                                                            |
| `403`  | The token lacks the `tasks:read` scope                             |
| `404`  | The task does not exist or belongs to a different account          |
