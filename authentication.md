# Authentication

The Connect API uses **OAuth 2.1 authorization code grant with PKCE**. Tokens are
issued per user and bound to a single account, so an integration acts as that member
within that account, limited to the granted scopes.

## Discovery

Endpoints are published as OAuth Authorization Server Metadata:

```
GET https://connect.safariportal.dev/.well-known/oauth-authorization-server
```

```json
{
  "issuer": "https://connect.safariportal.dev",
  "authorization_endpoint": "https://account.safariportal.app/oauth/authorize",
  "token_endpoint": "https://connect.safariportal.dev/oauth/token",
  "scopes_supported": [
    "files:read", "files:write", "contacts:read", "contacts:write",
    "tasks:read", "tasks:write", "users:read", "financials:read", "forms:read"
  ],
  "code_challenge_methods_supported": ["S256"],
  "grant_types_supported": ["authorization_code", "refresh_token"]
}
```

> The **authorization** step runs on the main Safari Portal domain (it requires the
> user's logged-in browser session); **token exchange** and all API calls run on
> `connect.safariportal.dev`. Always read these URLs from the discovery document
> rather than hard-coding them.

## Registering a client

Register your application from the Safari Portal account that will use it:
**Settings → Integrations → OAuth Applications → Create**. Provide a name, a
redirect URI, and the scopes the app needs. On creation you'll receive a
`client_id` and a `client_secret`.

> **Copy the `client_secret` immediately — it is shown only once.** If you lose
> it, open the app and use **Regenerate secret** (the previous secret stops
> working). The `client_id` remains visible in the OAuth Applications list.

Notes:

- The **redirect URI is matched exactly** at authorization time. Register the
  exact callback URL your app uses.
- An app is managed by the member who created it and by the account owner. You
  can edit its name, redirect URI, and scopes, or delete it (deleting revokes
  any tokens it issued).

## Authorization code + PKCE

PKCE is **required** (`S256` only). Generate a `code_verifier` and its
`code_challenge`, then send the user to the authorization endpoint:

```
GET https://account.safariportal.app/oauth/authorize
  ?response_type=code
  &client_id=YOUR_CLIENT_ID
  &redirect_uri=https://myapp.example.com/oauth/callback
  &scope=contacts:read%20contacts:write
  &state=RANDOM_STATE
  &code_challenge=CODE_CHALLENGE
  &code_challenge_method=S256
```

The user signs in (if needed), selects which account to authorize, and approves the
requested scopes. They are redirected back to your `redirect_uri` with `code` and
`state`.

**Write scopes are declinable.** Any `:write` scope you request is offered as a
separate, unticked checkbox on the consent screen, and is dropped from the grant
unless the user ticks it — so a token may come back holding only the `:read` scopes
you asked for. Read the `scope` in the token response to find out: when it is narrower
than what you requested, the user declined, and the right move is to offer a
re-authorize rather than to retry the write and read the `403`. If you requested write
scopes *only* and the user declines, no grant is minted at all: the callback carries
`error=access_denied`.

Exchange the code for tokens on the token endpoint:

```bash
curl -X POST https://connect.safariportal.dev/oauth/token \
  -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" \
  -d grant_type=authorization_code \
  -d code=AUTHORIZATION_CODE \
  -d redirect_uri=https://myapp.example.com/oauth/callback \
  -d code_verifier=CODE_VERIFIER
```

```json
{
  "access_token": "…",
  "token_type": "bearer",
  "expires_in": 3600,
  "refresh_token": "…",
  "scope": "contacts:read"
}
```

## Tokens

| Token           | Lifetime    | Notes                                            |
| --------------- | ----------- | ------------------------------------------------ |
| Access token    | 60 minutes  | Sent as `Authorization: Bearer <token>`          |
| Refresh token   | 90 days     | Rotated on each use — always store the latest    |

Refresh an expired access token:

```bash
curl -X POST https://connect.safariportal.dev/oauth/token \
  -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" \
  -d grant_type=refresh_token \
  -d refresh_token=REFRESH_TOKEN
```

Each refresh returns a **new** refresh token; the previous one is invalidated.

## Scopes

| Scope             | Grants                                                        |
| ----------------- | ------------------------------------------------------------- |
| `files:read`      | List, search, and retrieve [files](files.md), and their stages, categories and events |
| `files:write`     | Create and update files                                       |
| `contacts:read`   | List, search, and retrieve [contacts](contacts.md)            |
| `contacts:write`  | Create and update contacts                                    |
| `tasks:read`      | List, filter, and retrieve [tasks](tasks.md)                  |
| `tasks:write`     | Create tasks                                                  |
| `users:read`      | List and retrieve account [users](users.md)                   |
| `financials:read` | List and retrieve [invoices](invoices.md) and [expenses](expenses.md) |
| `forms:read`      | List [forms](forms.md) and their [submissions](form_submissions.md) |

One `financials:read` covers invoices and expenses because no permission state
distinguishes them — the account permissions they derive from have no
invoice-versus-expense concept.

`GET /accounts` needs no scope — only a valid token. Every other endpoint must carry the
scope it requires (otherwise `403`), and the authorizing member must have the corresponding
access in the account. See [Resources](README.md#resources) for the scope each one takes.

More scopes are added as new resources ship; read `scopes_supported` from the discovery
document rather than hard-coding this list.
