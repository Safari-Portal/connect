# Expenses

Expenses are outgoing supplier costs attached to a file (trip / tour request) — money the
agency **pays out**, as opposed to [Invoices](invoices.md) (money it receives). Each expense
tracks a payment status and one or more **outgoing payments**.

This is a **read-only** API. Expenses can be listed and retrieved. There are no create or
update endpoints.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an account
you are an agent member of; an account you are not a member of returns `404`, and a token
bound to a different account returns `403`. If an expense does not exist in that account, or
is not visible to the member, the endpoint returns `404`. Visibility is governed by the
member's financials permissions: consultants with no financials access see an empty list and
get `404` on individual lookups; owners and managers with full financials access see all
expenses in the account.

> **Money values.** All monetary amounts — the `amount` on each payment — are integers in
> **minor units (cents)**, paired with an ISO 4217 `currency` string. For example, USD
> 1,000.00 is `100000` with `"currency": "USD"`.

> **⚠️ Multi-currency: an expense has no single total.** Unlike an invoice, an expense's
> payments may each be in a **different currency**, so there is no meaningful single expense
> currency or total. This API therefore exposes **no** `currency`, `total_amount`,
> `amount_paid`, or `balance_due` at the expense level. All monetary detail lives in the
> `payments` array, where **each payment carries its own `amount` + `currency`**. If you need
> a total, aggregate the payments **per currency** — never sum amounts across different
> currencies.

## The expense object

### List shape

```json
{
  "id": "51",
  "number": "000010",
  "status": "partially_paid",
  "file_id": "88",
  "created_at": "2026-06-01T08:00:00.000Z",
  "updated_at": "2026-06-15T12:30:00.000Z"
}
```

### Show shape

The retrieve endpoint returns everything above plus `pay_to_id` and the `payments` array:

```json
{
  "id": "51",
  "number": "000010",
  "status": "partially_paid",
  "file_id": "88",
  "created_at": "2026-06-01T08:00:00.000Z",
  "updated_at": "2026-06-15T12:30:00.000Z",
  "pay_to_id": "204",
  "payments": [
    {
      "id": "9",
      "direction": "payable",
      "amount": 100000,
      "currency": "USD",
      "status": "paid",
      "due_date": "2026-06-10",
      "paid_at": "2026-06-10T14:00:00.000Z"
    },
    {
      "id": "10",
      "direction": "payable",
      "amount": 50000,
      "currency": "EUR",
      "status": "planned",
      "due_date": "2026-07-01",
      "paid_at": null
    }
  ]
}
```

### Expense object fields

| Field        | Type              | Description                                                                                     |
| ------------ | ----------------- | ----------------------------------------------------------------------------------------------- |
| `id`         | string            | Unique identifier for the expense                                                               |
| `number`     | string            | Account-scoped expense number (e.g. `"000010"`)                                                |
| `status`     | string            | One of `draft`, `not_paid`, `partially_paid`, `paid_in_full`                                    |
| `file_id`    | string            | ID of the file (trip / tour request) this expense belongs to — resolve via the [Files](files.md) endpoint |
| `created_at` | string (ISO 8601) | When the expense was created                                                                    |
| `updated_at` | string (ISO 8601) | When the expense was last updated                                                               |

The following fields are **only present on the retrieve (show) endpoint**:

| Field        | Type           | Description                                                                                          |
| ------------ | -------------- | ---------------------------------------------------------------------------------------------------- |
| `pay_to_id`  | string \| null | ID of the supplier/beneficiary contact being paid — resolve via the [Contacts](contacts.md) endpoint |
| `payments`   | array          | The expense's outgoing payments — see [Payment object](#payment-object) below                        |

### Payment object

Expense payments are always **outgoing** (`direction: "payable"`).

| Field       | Type                      | Description                                             |
| ----------- | ------------------------- | ------------------------------------------------------- |
| `id`        | string                    | Unique identifier for the payment                       |
| `direction` | string                    | Always `payable` (an outgoing payment to the supplier)  |
| `amount`    | integer                   | Amount in minor units (cents)                           |
| `currency`  | string                    | ISO 4217 currency code — **may differ between payments** |
| `status`    | string                    | One of `planned`, `pending`, `paid`, `failed`, `voided` |
| `due_date`  | string (ISO date) \| null | Due date in `YYYY-MM-DD` format, or `null` if not set   |
| `paid_at`   | string (ISO 8601) \| null | When the payment was marked paid, or `null`             |

## Scopes

| Scope              | Endpoints  |
| ------------------ | ---------- |
| `financials:read`  | List, Get  |

There are no write scopes. This API is read-only. `financials:read` is the SAME scope the
Invoices API requires: expenses and invoices were once separate scopes, but no permission
state distinguishes them — both are governed by the same `financials_*` member permissions —
so a token can no longer hold one without the other.

### Financial access

`financials:read` is the token's half of the gate; the authorizing member's financial
permission is the other half, and the lower of the two wins. Access level decides which
records exist for you: all files, only your own, or the files you are sales consultant on.
A member with no financial access sees an empty list, and `404` on any individual lookup.

That `404` is the same answer a record outside your access level gives, and the same one an
id that never existed gives — so it reveals nothing about which records are there.

Note that the financials **capabilities** — CSV export, statistics, Flowbrite — are not
financial access. A member holding only those sees an empty list like any other member
without access.

## Endpoints

| Method | Path                               | Scope             | Description          |
| ------ | ---------------------------------- | ----------------- | -------------------- |
| `GET`  | `/acc/{account_id}/expenses`       | `financials:read` | List expenses        |
| `GET`  | `/acc/{account_id}/expenses/{id}`   | `financials:read` | Retrieve an expense  |

## List expenses

```
GET /acc/{account_id}/expenses
```

Requires `financials:read`. Returns all expenses visible to the member, ordered newest first
(by `created_at` descending, then `id` descending). Returns expense fields only — payments
are not included in the list response. Supports [pagination](README.md#pagination) via `page`
and `limit`.

| Query param | Description                                  |
| ----------- | -------------------------------------------- |
| `page`      | Page number (default `1`)                    |
| `limit`     | Items per page (default `25`, maximum `100`) |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/expenses?page=1&limit=10" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    {
      "id": "51",
      "number": "000010",
      "status": "partially_paid",
      "file_id": "88",
      "created_at": "2026-06-01T08:00:00.000Z",
      "updated_at": "2026-06-15T12:30:00.000Z"
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

## Retrieve an expense

```
GET /acc/{account_id}/expenses/{id}
```

Requires `financials:read`. Returns the expense with its `pay_to_id` and the full `payments`
array (each payment carrying its own currency). Returns `404` if the expense does not exist,
belongs to another account, or is not visible to the member.

```bash
curl https://connect.safariportal.dev/acc/{account_id}/expenses/51 \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": {
    "id": "51",
    "number": "000010",
    "status": "partially_paid",
    "file_id": "88",
    "created_at": "2026-06-01T08:00:00.000Z",
    "updated_at": "2026-06-15T12:30:00.000Z",
    "pay_to_id": "204",
    "payments": [
      {
        "id": "9",
        "direction": "payable",
        "amount": 100000,
        "currency": "USD",
        "status": "paid",
        "due_date": "2026-06-10",
        "paid_at": "2026-06-10T14:00:00.000Z"
      },
      {
        "id": "10",
        "direction": "payable",
        "amount": 50000,
        "currency": "EUR",
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
