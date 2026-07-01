# Stages

Stages are the pipeline steps a file (trip / tour request) moves through. Each stage
belongs to a pipeline — typically `leads` or `planning`. Use this endpoint to resolve the
`stage_id` returned on a [file](files.md) into a name.

This is a **read-only** API: stages can only be listed.

All endpoints are nested under `/acc/{account_id}/`. The `{account_id}` must be an
account you are an agent member of; an account you are not a member of returns `404`,
and a token bound to a different account returns `403`.

## The stage object

```json
{
  "id": "3",
  "name": "Proposal Sent",
  "pipeline": "planning"
}
```

### Stage object fields

| Field      | Type   | Description                                              |
| ---------- | ------ | -------------------------------------------------------- |
| `id`       | string | Unique identifier — matches the `stage_id` on a file     |
| `name`     | string | Display name (e.g. "Proposal Sent")                     |
| `pipeline` | string | The pipeline the stage belongs to — typically `leads` or `planning` |

## Scopes

| Scope        | Endpoints |
| ------------ | --------- |
| `files:read` | List      |

This endpoint resolves ids that appear on files, so it shares the `files:read` scope.

## Endpoints

| Method | Path                        | Scope        | Description   |
| ------ | --------------------------- | ------------ | ------------- |
| `GET`  | `/acc/{account_id}/stages`  | `files:read` | List stages   |

## List stages

```
GET /acc/{account_id}/stages
```

Requires `files:read`. Returns the account's stages ordered by pipeline then position
(so the array order reflects each stage's position within its pipeline). No pagination.

| Query param | Type   | Description                                                              |
| ----------- | ------ | ------------------------------------------------------------------------ |
| `pipeline`  | string | Optional. Filters to a pipeline (e.g. `leads`, `planning`). An unrecognized value returns an empty list. |

```bash
curl "https://connect.safariportal.dev/acc/{account_id}/stages?pipeline=planning" \
  -H "Authorization: Bearer <access_token>"
```

**Response `200`**

```json
{
  "data": [
    { "id": "3", "name": "Planning",      "pipeline": "planning" },
    { "id": "4", "name": "Proposal Sent", "pipeline": "planning" }
  ]
}
```

See [Errors](README.md#errors) in the README for the shared error envelope.
