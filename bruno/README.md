# Connect API — Bruno collection

A [Bruno](https://usebruno.com) collection for exercising the Safari Portal
Connect API locally and against production.

> **Generated file — do not hand-edit.** Every `.bru` here is generated from
> [`../openapi.yaml`](../openapi.yaml) by `Connect::BrunoGenerator`. Change the
> spec and regenerate; a test (`test/lib/connect/bruno_generator_test.rb`) fails
> CI if the committed collection drifts from the spec.
>
> ```
> make connect-bruno        # or: bundle exec rake connect:bruno
> ```

## Setup

1. Open this folder as a collection in Bruno (**Open Collection** → pick
   `docs/connect-api/bruno`).
2. Copy `.env.example` to `.env` (untracked) in this folder.
3. Mint a local access token and account id:

   ```
   bundle exec rake connect:token
   # optionally: rake connect:token acc=123 user=you@example.com scopes='contacts:read'
   ```

4. Paste the token into `CONNECT_TOKEN` and the account id into `CONNECT_ACC_ID`
   in `.env`. Select the **Local** environment in Bruno and send a request.

Secrets live only in `.env` (git-ignored). The tracked environment files
reference them via `process.env`, so the collection stays fully generated.

## Environments

| Environment | Base URL                             |
| ----------- | ------------------------------------ |
| Local       | `http://connect.localhost:3000`      |
| Production  | `https://connect.safariportal.dev`   |

## Auth

The collection sets a collection-level **bearer** token (`{{token}}`) that every
request inherits. Production tokens come from the OAuth 2.1 authorization-code +
PKCE flow (see [`../authentication.md`](../authentication.md)); for local work
the `connect:token` rake task mints one directly, bypassing the browser round-trip.
