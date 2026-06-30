# Invoices

Invoices are financial documents attached to a file (trip / tour request). Each invoice
tracks amounts, payment status, and one or more payments (receivables and refunds).

This is a **read-only** API. Invoices can be listed and retrieved. There are no create
or update endpoints.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`, and a token bound to a different account returns `403`. If an
invoice does not exist in that account, or is not visible to the member, the endpoint
returns `404`. Visibility is governed by the member's financials permissions: consultants
with no financials access see an empty list and get `404` on individual lookups; owners
and managers with full financials access see all invoices in the account.

> **Money values.** All monetary amounts in this API — `total_amount`, `amount_paid`,
> `balance_due`, and `amount` on individual payments — are integers in **minor units
> (cents)**, paired with an ISO 4217 `currency` string. For example, USD 1,050.00 is
> represented as `105000` with `"currency": "USD"`.

## The invoice object

### List shape

```json
{
  "id": "9",
  "number": "INV-1007",
  "status": "partially_paid",
  "file_id": "42",
  "currency": "USD",
  "total_amount": 525000,
  "created_at": "2026-03-10T09:00:00.000Z",
  "updated_at": "2026-06-15T14:22:00.000Z"
}
```

### Show shape

The retrieve endpoint returns everything above plus four additional fields:

```json
{
  "id": "9",
  "number": "INV-1007",
  "status": "partially_paid",
  "file_id": "42",
  "currency": "USD",
  "total_amount": 525000,
  "created_at": "2026-03-10T09:00:00.000Z",
  "updated_at": "2026-06-15T14:22:00.000Z",
  "primary_contact_id": "101",
  "amount_paid": 262500,
  "balance_due": 262500,
  "payments": [
    {
      "id": "55",
      "direction": "receivable",
      "amount": 262500,
      "currency": "USD",
      "status": "paid",
      "due_date": "2026-04-01",
      "paid_at": "2026-03-28T11:05:00.000Z"
    },
    {
      "id": "56",
      "direction": "receivable",
      "amount": 262500,
      "currency": "USD",
      "status": "planned",
      "due_date": "2026-07-01",
      "paid_at": null
    }
  ]
}
```

### Invoice object fields

| Field               | Type              | Description                                                                                  |
| ------------------- | ----------------- | -------------------------------------------------------------------------------------------- |
| `id`                | string            | Unique identifier for the invoice                                                            |
| `number`            | string            | Human-readable invoice number (e.g. `"INV-1007"`)                                           |
| `status`            | string            | One of `draft`, `not_paid`, `partially_paid`, `paid_in_full`, `voided`                      |
| `file_id`           | string            | ID of the file (trip / tour request) this invoice belongs to — resolve via the [Files](files.md) endpoint |
| `currency`          | string            | ISO 4217 currency code (e.g. `"USD"`)                                                       |
| `total_amount`      | integer \| null   | Invoice total in minor units (cents). May be `null` on a draft with no payments yet.        |
| `created_at`        | string (ISO 8601) | When the invoice was created                                                                 |
| `updated_at`        | string (ISO 8601) | When the invoice was last updated                                                            |

The following four fields are **only present on the retrieve (show) endpoint**:

| Field                | Type              | Description                                                                                                    |
| -------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------- |
| `primary_contact_id` | string \| null    | ID of the primary customer contact — resolve via the [Contacts](contacts.md) endpoint                         |
| `amount_paid`        | integer           | Sum of all paid receivables in minor units (cents)                                                             |
| `balance_due`        | integer           | Outstanding balance in minor units (cents)                                                                     |
| `payments`           | array             | The invoice's payments — see [Payment object](#payment-object) below                                          |

> **Arithmetic note.** Do not treat `amount_paid + balance_due = total_amount` as a
> closed identity. On discounted invoices, `amount_paid` is the raw paid total while
> `balance_due` is discount-aware, so the three values need not sum consistently.

### Payment object

| Field       | Type                     | Description                                                              |
| ----------- | ------------------------ | ------------------------------------------------------------------------ |
| `id`        | string                   | Unique identifier for the payment                                        |
| `direction` | string                   | `receivable` (incoming payment) or `refund` (outgoing refund)            |
| `amount`    | integer                  | Amount in minor units (cents)                                            |
| `currency`  | string                   | ISO 4217 currency code                                                   |
| `status`    | string                   | One of `planned`, `pending`, `paid`, `failed`, `voided`                 |
| `due_date`  | string (ISO date) \| null| Due date in `YYYY-MM-DD` format, or `null` if not set                   |
| `paid_at`   | string (ISO 8601) \| null| When the payment was marked paid, or `null` if not yet paid              |

## Scopes

| Scope            | Endpoints  |
| ---------------- | ---------- |
| `invoices:read`  | List, Get  |

There are no write scopes. This API is read-only.

## Endpoints

| Method | Path                               | Scope            | Description          |
| ------ | ---------------------------------- | ---------------- | -------------------- |
| `GET`  | `/acc/{account_id}/invoices`       | `invoices:read`  | List invoices        |
| `GET`  | `/acc/{account_id}/invoices/:id`   | `invoices:read`  | Retrieve an invoice  |

## List invoices

```
GET /acc/{account_id}/invoices
```

Requires `invoices:read`. Returns all invoices visible to the member, ordered newest
first (by `created_at` descending, then `id` descending). Returns invoice fields only —
payments are not included in the list response. Supports
[pagination](README.md#pagination) via `page` and `limit`.

| Query param | Description                                          |
| ----------- | ---------------------------------------------------- |
| `page`      | Page number (default `1`)                            |
| `limit`     | Items per page (default `25`, maximum `100`)         |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/invoices?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "9",
      "number": "INV-1007",
      "status": "partially_paid",
      "file_id": "42",
      "currency": "USD",
      "total_amount": 525000,
      "created_at": "2026-03-10T09:00:00.000Z",
      "updated_at": "2026-06-15T14:22:00.000Z"
    }
  ],
  "meta": {
    "total_count": 31,
    "total_pages": 4,
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

## Retrieve an invoice

```
GET /acc/{account_id}/invoices/:id
```

Requires `invoices:read`. Returns the invoice with its full payment detail, including
`amount_paid`, `balance_due`, and the `payments` array. Returns `404` if the invoice
does not exist, belongs to another account, or is not visible to the member.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/invoices/9 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "9",
    "number": "INV-1007",
    "status": "partially_paid",
    "file_id": "42",
    "currency": "USD",
    "total_amount": 525000,
    "created_at": "2026-03-10T09:00:00.000Z",
    "updated_at": "2026-06-15T14:22:00.000Z",
    "primary_contact_id": "101",
    "amount_paid": 262500,
    "balance_due": 262500,
    "payments": [
      {
        "id": "55",
        "direction": "receivable",
        "amount": 262500,
        "currency": "USD",
        "status": "paid",
        "due_date": "2026-04-01",
        "paid_at": "2026-03-28T11:05:00.000Z"
      },
      {
        "id": "56",
        "direction": "receivable",
        "amount": 262500,
        "currency": "USD",
        "status": "planned",
        "due_date": "2026-07-01",
        "paid_at": null
      }
    ]
  }
}
```

See [Errors](README.md#errors) and [Pagination](README.md#pagination) in the README for
shared conventions.
