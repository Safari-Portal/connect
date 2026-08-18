# File Channels

Channels are the built-in acquisition-channel values a file's `channel` field can
carry — e.g. "B2C - Direct Client", "B2B - Via Partner Channel". Unlike
[sources](file_sources.md), [trip types](file_trip_types.md), and
[travel regions](file_travel_regions.md), channels are a fixed list baked into Safari
Portal, not an account-configurable set — so the list has no ids, just the plain
string values a [file](files.md)'s `channel` field holds.

This is a **read-only** API: channels can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves the values behind a file's `channel` field, so it shares the
`files:read` scope.

## Endpoints

| Method | Path                              | Scope        | Description   |
| ------ | --------------------------------- | ------------ | ------------- |
| `GET`  | `/acc/{account_id}/file_channels` | `files:read` | List channels |

## List channels

```
GET /acc/{account_id}/file_channels
```

Requires `files:read`. Returns the built-in channel values, in a fixed order. No
pagination.

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/file_channels" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [ "B2C - Direct Client", "B2B - Via Partner Channel", "Other" ]
}
```

Each value is a plain string — there is no id. A file's `channel` field holds one of
these values (or `null`), and the same strings are what you send back as `channel`
when creating or updating a file — any other value is rejected with a `422`.

See [Errors](README.md#errors) in the README for the shared error envelope.
