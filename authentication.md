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
  "scopes_supported": ["contacts:read", "contacts:write"],
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

> Self-service **dynamic** client registration (RFC 7591, `POST /oauth/register`)
> is not available yet — it will be introduced alongside MCP support. For now,
> register apps through the OAuth Applications screen above.

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
  "refresh_token": "…"
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

| Scope            | Grants                                  |
| ---------------- | --------------------------------------- |
| `contacts:read`  | List, search, and retrieve contacts     |
| `contacts:write` | Create and update contacts              |

A request must carry the scope its endpoint requires (otherwise `403`), and the
authorizing member must have the corresponding access in the account. More scopes are
added as new resources ship.
