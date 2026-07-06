# Form Submissions

A submission is one completed [form](forms.md) — the answers a single traveler filled in and
submitted. Every submission is tied to a [Contact](contacts.md) (found or created at submit time)
and, when the form was assigned to a file, to a [file / trip](files.md) (`TourRequest`). Each
submission carries a `responses` array — one entry per answered field.

This is a **read-only** API. Submissions can be listed (per form or per contact) and retrieved.
There are no create or update endpoints.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an account you are
an agent member of; an account you are not a member of returns `404`, and a token bound to a
different account returns `403`. A form, contact, or submission that does not exist in that
account returns `404`.

## The submission object

The same submission object is used by all three endpoints — the list endpoints wrap it in
`data: [...]` with pagination `meta`, and the retrieve endpoint wraps it in `data: {...}`. The
nested `responses` array is included in every case.

```json
{
  "id": "9001",
  "form_id": "42",
  "contact_id": "301",
  "file_id": "88",
  "submitted_at": "2026-06-15T09:12:00.000Z",
  "updated_at": "2026-06-15T09:12:00.000Z",
  "responses": [
    {
      "element_id": "wfe_1111aaaa2222",
      "label": "First name",
      "type": "string",
      "mapped_to": "first_name",
      "value": "Ada"
    },
    {
      "element_id": "wfe_3333bbbb4444",
      "label": "Interests",
      "type": "array",
      "mapped_to": null,
      "value": ["Safari", "Beach"]
    },
    {
      "element_id": "wfe_5555cccc6666",
      "label": "Passport",
      "type": "file",
      "mapped_to": "passport_file",
      "value": { "filename": "passport.pdf", "url": "https://…signed…" }
    }
  ]
}
```

### Submission object fields

| Field          | Type              | Description                                                                                          |
| -------------- | ----------------- | --------------------------------------------------------------------------------------------------- |
| `id`           | string            | Unique identifier for the submission                                                                |
| `form_id`      | string            | ID of the form that was submitted — resolve via the [Forms](forms.md) endpoint                       |
| `contact_id`   | string            | ID of the contact this submission belongs to — resolve via the [Contacts](contacts.md) endpoint      |
| `file_id`      | string \| null    | ID of the file (trip / tour request) this submission is tied to, or `null` when the submission is not assigned to a file — resolve via the [Files](files.md) endpoint |
| `submitted_at` | string (ISO 8601) | When the submission was submitted                                                                   |
| `updated_at`   | string (ISO 8601) | When the submission was last updated                                                                |
| `responses`    | array             | One entry per answered field — see [Response object](#response-object) below                         |

### Response object

Each entry in `responses` is a single answer, captured as a snapshot at submit time.

| Field        | Type           | Description                                                                                          |
| ------------ | -------------- | --------------------------------------------------------------------------------------------------- |
| `element_id` | string         | Identifier (`wfe_…`) of the form input this answer came from. The same input yields the same `element_id` across submissions, so you can align answers to the same question |
| `label`      | string         | The field's label, as captured at submit time                                                       |
| `type`       | string         | The answer's value type: `string`, `text`, `integer`, `boolean`, `date`, `array`, `file`            |
| `mapped_to`  | string \| null | The Contact or file field this answer synced to (e.g. `first_name`, `passport_file`), or `null`     |
| `value`      | varies         | The answer. A scalar for string/text/integer/boolean/date, a JSON array for array answers, a file object (below) for file answers, or `null` |

> **File responses.** When `type` is `file`, `value` is an object `{ "filename": …, "url": … }`,
> where `url` is a **signed, expiring** download link, or `null` when no file was attached. Fetch
> the file promptly — the URL is not stable.

## Behavior notes

- **Only completed submissions appear.** Submissions that require email verification and have not
  been verified are hidden; the API exposes only completed (email-verified) submissions.
- **Responses are self-contained snapshots.** Each response carries its own `label`, `type`,
  `mapped_to`, and `value`, captured at submit time — you do not need the form definition to
  interpret a submission. (Because they are a snapshot, a response reflects the form as it was
  when submitted, which may differ from the form's current state if it was later edited.)
- **A submission outlives its form.** A submission still appears even if its form was later
  soft-deleted.
- **Ordering.** Lists are ordered newest first (by `submitted_at` / `created_at` descending, then
  `id` descending) and are [paginated](README.md#pagination). Within a submission, `responses` are
  in a stable order that is **not** necessarily the form's display order.

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `forms:read` | List, Get |

There are no write scopes. This API is read-only. The same `forms:read` scope also governs the
[Forms](forms.md) endpoint.

## Endpoints

| Method | Path                                                          | Scope        | Description                     |
| ------ | ------------------------------------------------------------- | ------------ | ------------------------------- |
| `GET`  | `/acc/{account_id}/forms/:form_id/submissions`                | `forms:read` | List a form's submissions       |
| `GET`  | `/acc/{account_id}/contacts/:contact_id/form_submissions`     | `forms:read` | List a contact's submissions    |
| `GET`  | `/acc/{account_id}/form_submissions/:id`                      | `forms:read` | Retrieve a submission           |

## List a form's submissions

```
GET /acc/{account_id}/forms/:form_id/submissions
```

Requires `forms:read`. Returns the form's submissions ordered newest first, each with its nested
`responses`. Supports [pagination](README.md#pagination) via `page` and `limit`. Returns `404` if
the form does not exist or belongs to another account.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/forms/42/submissions?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "9001",
      "form_id": "42",
      "contact_id": "301",
      "file_id": "88",
      "submitted_at": "2026-06-15T09:12:00.000Z",
      "updated_at": "2026-06-15T09:12:00.000Z",
      "responses": [
        {
          "element_id": "wfe_1111aaaa2222",
          "label": "First name",
          "type": "string",
          "mapped_to": "first_name",
          "value": "Ada"
        },
        {
          "element_id": "wfe_5555cccc6666",
          "label": "Passport",
          "type": "file",
          "mapped_to": "passport_file",
          "value": { "filename": "passport.pdf", "url": "https://…signed…" }
        }
      ]
    }
  ],
  "meta": {
    "total_count": 12,
    "total_pages": 2,
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

## List a contact's submissions

```
GET /acc/{account_id}/contacts/:contact_id/form_submissions
```

Requires `forms:read`. Returns the submissions belonging to a single contact, ordered newest
first, each with its nested `responses`. Supports [pagination](README.md#pagination) via `page`
and `limit`. Returns `404` if the contact does not exist or belongs to another account. The
response shape is identical to the per-form listing above.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/contacts/301/form_submissions?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

## Retrieve a submission

```
GET /acc/{account_id}/form_submissions/:id
```

Requires `forms:read`. Returns one submission with its full `responses` array. Returns `404` if
the submission does not exist, belongs to another account, or is not visible.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/form_submissions/9001 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "9001",
    "form_id": "42",
    "contact_id": "301",
    "file_id": "88",
    "submitted_at": "2026-06-15T09:12:00.000Z",
    "updated_at": "2026-06-15T09:12:00.000Z",
    "responses": [
      {
        "element_id": "wfe_1111aaaa2222",
        "label": "First name",
        "type": "string",
        "mapped_to": "first_name",
        "value": "Ada"
      },
      {
        "element_id": "wfe_3333bbbb4444",
        "label": "Interests",
        "type": "array",
        "mapped_to": null,
        "value": ["Safari", "Beach"]
      },
      {
        "element_id": "wfe_5555cccc6666",
        "label": "Passport",
        "type": "file",
        "mapped_to": "passport_file",
        "value": { "filename": "passport.pdf", "url": "https://…signed…" }
      }
    ]
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for shared
conventions.
