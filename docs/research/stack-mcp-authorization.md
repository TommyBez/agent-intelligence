# Can Better Auth and Vercel serve a remote MCP server with the authorization MCP requires?

Research for issue #3. All sources were read on **2026-11-12**. Nothing was installed or run: every "verified" below means "read in a primary source", not "tested".

## Short answer

Yes on paper, with caveats.

- **MCP** (current revision **2026-07-28**) asks a remote server to be an OAuth 2.1 resource server: publish Protected Resource Metadata (RFC 9728) that names its authorization server, answer unauthenticated requests with `401` + `WWW-Authenticate`, and accept only bearer tokens issued for itself. The authorization server must do OAuth 2.1 with PKCE and publish its metadata; clients register through Client ID Metadata Documents (preferred), pre-registration, or Dynamic Client Registration (now deprecated).
- **Better Auth 1.7.22** documents exactly this profile: `@better-auth/mcp` (authorization server + resource-server helpers, built on `@better-auth/oauth-provider`) composed with `@better-auth/cimd` and the `jwt` plugin. Every authorization requirement of the specification that falls on the authorization server or on token checking has a documented counterpart. The caveats are a handful of points left to the application (consent screen content, well-known routes on Next.js, scope hierarchy), two places where Better Auth is more permissive than the specification (30-second refresh-token reuse window, private-use redirect schemes), and the age of the packages (stable since 2026-08-18).
- **Vercel** hosts it as a Next.js route handler on Vercel Functions. The 2026-07-28 revision removed protocol sessions, the standalone GET stream and stream resumability, so the transport no longer needs a connection or memory kept between requests. What remains limited is how long one response can stay open: 300 s by default, 800 s maximum on Pro and Enterprise (1,800 s in beta), 300 s on Hobby. No memory is shared between function instances.

Not verified: any end-to-end behaviour, and which protocol revision and registration mechanism the sample MCP clients (Claude, ChatGPT, Grok Bot) actually use.

## Sources

Each claim below cites one of these keys. Version or revision and date read apply to every claim that cites the key.

| Key | Source | Version / revision | Read |
| --- | --- | --- | --- |
| M1 | https://modelcontextprotocol.io/specification/versioning | states current revision is 2026-07-28 | 2026-11-12 |
| M2 | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization | 2026-07-28 | 2026-11-12 |
| M3 | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/authorization-server-discovery | 2026-07-28 | 2026-11-12 |
| M4 | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration | 2026-07-28 | 2026-11-12 |
| M5 | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations | 2026-07-28 | 2026-11-12 |
| M6 | https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http | 2026-07-28 | 2026-11-12 |
| M7 | https://modelcontextprotocol.io/specification/2026-07-28/changelog | 2026-07-28 (changes since 2025-11-25) | 2026-11-12 |
| M8 | https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning | 2026-07-28 | 2026-11-12 |
| M9 | https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/subscriptions | 2026-07-28 | 2026-11-12 |
| M10 | https://modelcontextprotocol.io/specification/2026-07-28/deprecated | 2026-07-28 | 2026-11-12 |
| B1 | https://better-auth.com/docs/plugins/mcp | docs for 1.7 (latest) | 2026-11-12 |
| B2 | https://better-auth.com/docs/plugins/oauth-provider | docs for 1.7 | 2026-11-12 |
| B3 | https://better-auth.com/docs/plugins/cimd | docs for 1.7 | 2026-11-12 |
| B4 | https://better-auth.com/docs/plugins/oauth-provider/tokens | docs for 1.7 | 2026-11-12 |
| B5 | https://better-auth.com/docs/plugins/oauth-provider/clients | docs for 1.7 | 2026-11-12 |
| B6 | https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow | docs for 1.7 | 2026-11-12 |
| B7 | https://better-auth.com/docs/plugins/oauth-provider/resource-server | docs for 1.7 | 2026-11-12 |
| B8 | https://better-auth.com/docs/guides/1-7-upgrade-guide | docs for 1.7 | 2026-11-12 |
| B9 | https://better-auth.com/docs/integrations/next | docs for 1.7 | 2026-11-12 |
| B10 | https://github.com/better-auth/better-auth/tree/v1.7.22 — `packages/mcp/src/{plugin,handler,require-mcp-auth}.ts`, `packages/oauth-provider/src/metadata.ts`, `packages/mcp/CHANGELOG.md`, `demo/nextjs/app/` | tag v1.7.22 (released 2026-11-12) | 2026-11-12 |
| B11 | npm registry: `better-auth`, `@better-auth/mcp`, `@better-auth/cimd`, `@better-auth/oauth-provider` | latest 1.7.22; beta 1.8.0-beta.9 | 2026-11-12 |
| B12 | https://github.com/better-auth/better-auth/issues (open issues #12224, #12357) | open on 2026-11-12 | 2026-11-12 |
| V1 | https://vercel.com/docs/functions/limitations | last updated 2026-08-24 | 2026-11-12 |
| V2 | https://vercel.com/docs/functions/configuring-functions/duration | last updated 2026-08-24 | 2026-11-12 |
| V3 | https://vercel.com/docs/fluid-compute | last updated 2026-08-24 | 2026-11-12 |
| V4 | https://vercel.com/docs/mcp/deploy-mcp-servers-to-vercel | last updated 2026-09-18 | 2026-11-12 |
| V5 | https://vercel.com/docs/functions/streaming-functions | last updated 2026-09-01 | 2026-11-12 |
| V6 | https://vercel.com/kb/guide/publish-and-subscribe-to-realtime-data-on-vercel | last updated 2026-09-10 | 2026-11-12 |
| V7 | https://vercel.com/docs/fundamentals/what-is-compute and https://vercel.com/docs/functions | current pages | 2026-11-12 |
| V8 | https://github.com/vercel/mcp-handler (README, `main`) | npm latest 2.3.1 | 2026-11-12 |
| S1 | https://github.com/modelcontextprotocol/typescript-sdk (`main`: `docs/serving/{http,legacy-clients,sessions-state-scaling}.md`) | `@modelcontextprotocol/server` latest 2.4.0 | 2026-11-12 |

## What MCP requires

Revision read: **2026-07-28**, marked "current" [M1]. It replaced 2025-11-25 and changed both the transport and client registration [M7], so anything written against 2025 revisions is out of date.

Authorization is optional in MCP, but an HTTP server that supports it should follow this specification [M2]. Roles: the MCP server is an OAuth 2.1 resource server, the MCP client is an OAuth 2.1 client, the authorization server may be the same host or a separate one [M2]. All statements in this section are verified in the cited page.

### How the server advertises its authorization server

| # | Requirement | Level | Source |
| --- | --- | --- | --- |
| A1 | The MCP server implements OAuth 2.0 Protected Resource Metadata (RFC 9728); the document includes `authorization_servers` with at least one entry. | MUST | M3 |
| A2 | The server makes that document discoverable either through `resource_metadata` in the `WWW-Authenticate` header of a `401`, or at a well-known URI (`/.well-known/oauth-protected-resource/<mcp path>` or the root `/.well-known/oauth-protected-resource`). Clients must support both. | MUST (one of) | M3 |
| A3 | The `WWW-Authenticate` challenge carries a `scope` parameter naming the scopes needed. | SHOULD | M2 |
| A4 | Protected Resource Metadata and challenges do not list `offline_access`. | SHOULD NOT | M2 |
| A5 | The authorization server publishes OAuth 2.0 Authorization Server Metadata (RFC 8414) or OpenID Connect Discovery 1.0; clients try `/.well-known/oauth-authorization-server` (path-inserted for issuers with a path) and then the OpenID configuration URLs. The `issuer` in the document must equal the issuer used to build the URL. | MUST (one of) | M2, M3 |
| A6 | The metadata includes `code_challenge_methods_supported`; a client must refuse to proceed if it is absent. | MUST | M5 |

### How an MCP client registers

Three mechanisms, in the priority order a client should apply: pre-registered credentials, Client ID Metadata Documents, Dynamic Client Registration, then asking the user [M4].

| # | Requirement | Level | Source |
| --- | --- | --- | --- |
| R1 | Authorization servers and clients support Client ID Metadata Documents (CIMD, `draft-ietf-oauth-client-id-metadata-document-00`): the `client_id` is an HTTPS URL of a JSON document with at least `client_id`, `client_name`, `redirect_uris`. | SHOULD | M2, M4 |
| R2 | An authorization server that supports CIMD advertises `client_id_metadata_document_supported: true`. | (how support is signalled) | M4 |
| R3 | With CIMD the authorization server validates that the document's `client_id` equals the URL exactly, that the document is valid JSON with the required fields, and that the requested redirect URI is in the document. | MUST | M4 |
| R4 | With CIMD the authorization server fetches the document for URL-shaped client IDs, caches it per HTTP cache headers, and considers SSRF risks. | SHOULD | M4, M5 |
| R5 | With CIMD the authorization server clearly displays the redirect URI hostname during authorization (MUST) and shows an additional warning for localhost-only redirect URIs (SHOULD). | MUST / SHOULD | M5 |
| R6 | Dynamic Client Registration (RFC 7591) may be supported. It is **deprecated** in 2026-07-28, kept for backwards compatibility, eligible for removal in the first revision on or after 2027-07-28. | MAY | M2, M4, M10 |
| R7 | Clients should support static (pre-registered) client credentials. | SHOULD (client) | M4 |

### How tokens are issued

| # | Requirement | Level | Source |
| --- | --- | --- | --- |
| T1 | The authorization server implements OAuth 2.1 (`draft-ietf-oauth-v2-1-13`) for confidential and public clients. | MUST | M2 |
| T2 | Authorization code flow with PKCE; clients use `S256`. | MUST (client) | M5 |
| T3 | Clients send the `resource` parameter (RFC 8707) with the MCP server's canonical URI in both the authorization and the token request; tokens are bound to that audience when the authorization server supports it. | MUST (client) | M2, M5 |
| T4 | The authorization server includes `iss` in authorization responses (RFC 9207) and, if it does, advertises `authorization_response_iss_parameter_supported: true`. The specification says a future revision is expected to make `iss` a MUST. | SHOULD / MUST | M2, M7 |
| T5 | Redirect URIs are matched exactly against registered values; all redirect URIs are `localhost` or HTTPS; all authorization server endpoints are HTTPS. | MUST | M5 |
| T6 | Access tokens are short-lived. | SHOULD | M5 |
| T7 | Refresh tokens issued to public clients are rotated (OAuth 2.1 §4.3.1). Issuing refresh tokens at all is at the authorization server's discretion. | MUST | M2, M5 |

### How tokens are checked

| # | Requirement | Level | Source |
| --- | --- | --- | --- |
| C1 | The client sends `Authorization: Bearer <token>` on **every** HTTP request; tokens never go in the query string. | MUST | M2 |
| C2 | The MCP server validates the access token per OAuth 2.1 §5.2 and validates that it was issued for this server as audience; it accepts no other tokens and does not pass the received token to upstream APIs. | MUST | M2, M5 |
| C3 | Invalid or expired token → `401`; insufficient scope → `403`; malformed request → `400`. | MUST | M2 |
| C4 | On insufficient scope, respond `403` with `WWW-Authenticate: Bearer error="insufficient_scope", scope="...", resource_metadata="..."`, listing all scopes the operation needs in one challenge. | SHOULD | M2 |
| C5 | The server accounts for scope hierarchies when deciding whether a token is sufficient. | MUST | M2 |

### Transport facts that bear on hosting

All verified in M6, M7, M8, M9.

- Two standard transports: stdio and Streamable HTTP. A remote server uses Streamable HTTP: one endpoint that accepts `POST`; every JSON-RPC message from the client is its own `POST`; the server answers with a single JSON object or with an SSE stream scoped to that request [M6].
- 2026-07-28 **removed** protocol-level sessions and the `Mcp-Session-Id` header, the `initialize` handshake, the HTTP `GET` stream, and SSE resumability (`Last-Event-ID`) [M6, M7]. Each request carries its protocol version and client capabilities in `_meta` and in the `MCP-Protocol-Version` header [M6, M8].
- Servers no longer send requests to clients. When a server needs input it returns an `InputRequiredResult` and the client retries the original request; any state travels in an opaque `requestState` the client echoes back [M6, M7].
- The only long-lived connection left is `subscriptions/listen`: a `POST` whose SSE response stays open and carries opted-in change notifications. If the transport closes without a final response, the client may treat it as an unexpected disconnect and reconnect [M6, M9].
- A broken response stream loses the in-flight request; the client must re-issue it as a new request [M7].
- Servers must validate the `Origin` header on all incoming connections and answer `403` when it is present and invalid [M6].
- A server may serve both eras. A 2025-era ("legacy") client talking to a modern-only server fails, and legacy clients have no way to move forward on their own [M8]. The deprecated HTTP+SSE transport (2024-11-05) is not part of this revision [M6, M10].

## Better Auth

### Plugin and configuration

Verified in B1, B2, B3, B8, B10, B11.

- The plugin is **`@better-auth/mcp`** (`mcp()`), a separate package since 1.7.0. It is the OAuth 2.1 provider (`@better-auth/oauth-provider`) preconfigured for MCP, so `oauthProvider()` is not registered alongside it [B1]. The older in-core `mcp` plugin and the `oidcProvider` plugin were removed in 1.7 [B8].
- Documented composition for the MCP 2026-07-28 profile: `jwt()` (required, signs tokens and exposes `/jwks`), `mcp({ loginPage, consentPage, resource })`, and `cimd({ fetchClientMetadataResource, metadataProfile: "mcp-2026-07-28" })` from **`@better-auth/cimd`** [B1, B3]. The JWT plugin's `/token` route and `set-auth-jwt` header are to be disabled in this mode [B1].
- Better Auth covers authorization only. Its docs assign the MCP protocol and transport to version 2 of the official TypeScript SDK (`@modelcontextprotocol/server`), wrapped by `requireMcpAuth` (same app) or `createMcpProtectedRequestHandler` (separate resource server) [B1, B10].
- OAuth endpoints are served under `/oauth2/*` of the auth base path. The login and consent pages are pages the application builds [B1, B2, B6].
- Storage: tables `oauthClient`, `oauthAccessToken`, `oauthRefreshToken`, `oauthConsent`, `oauthClientAssertion`, `oauthResource`, `oauthClientResource`, generated by the Better Auth CLI [B1]. A Drizzle adapter with a `pg` provider is documented (https://better-auth.com/docs/adapters/drizzle, read 2026-11-12).
- Next.js: the docs state full compatibility with Next.js 16 [B9]; the repository's own Next.js demo depends on `next ^16.4.2` and contains an MCP route and the well-known routes [B10].

### Maturity

Verified in B10, B11 unless marked.

- `@better-auth/oauth-provider` first published 2025-12-22; `@better-auth/mcp` first published 2026-01-14; `@better-auth/cimd` first published 2026-04-15 (as 1.7.0-beta.0) [B11].
- The current shape (separate MCP package, stateless 2026-07-28 transport, CIMD instead of implicit Dynamic Client Registration, `/oauth2/*` endpoints) became stable in **1.7.0, published 2026-08-18** [B10, B11]. Latest stable is **1.7.22, published 2026-11-12**: 22 patch releases in about 12 weeks. 1.8.0-beta.9 exists; its changes were not read [B11].
- The 1.6 → 1.7 move was breaking: package, import names, endpoint paths, database models and defaults all changed [B8].
- The 1.7 docs pages for MCP, OAuth Provider and CIMD carry no "experimental" or "beta" label (verified by reading B1, B2, B3).
- CIMD: the plugin implements IETF draft-02 and offers an explicit `mcp-2026-07-28` profile because the MCP specification references draft-00 [B3].
- Open, user-reported issues on the 1.7 line that touch this stack (not confirmed by maintainers, not reproduced here) [B12]:
  - #12224 (2026-11-04): the Node CIMD transport pins only the first DNS answer, so metadata fetches fail on hosts without outbound IPv6 when the client's host lists an AAAA record first; the report names a Claude connector host as an example.
  - #12357 (2026-11-09): CIMD registration collision recovery does not recognise a unique-violation error wrapped by Drizzle (reported on 1.7.17 and 1.7.18).

### Coverage, requirement by requirement

"Covered" means the 1.7 docs or the v1.7.22 source state that Better Auth does it. None of it was run.

| # | Requirement | Status | What Better Auth provides |
| --- | --- | --- | --- |
| A1 | Protected Resource Metadata with `authorization_servers` | Covered with caveats | `mcp()` builds the RFC 9728 document (`resource`, `authorization_servers` = the provider issuer, `bearer_methods_supported: ["header"]`, scopes) and serves it at `/.well-known/oauth-protected-resource` and the resource-path-inserted alias [B1, B10]. Caveat: it is served by the auth handler; on Next.js the auth handler is mounted under the auth base path, and the official demo adds explicit route files under `app/.well-known/` for it [B2, B10]. The MCP docs page itself does not show those Next.js routes. |
| A2 | `401` + `WWW-Authenticate` with `resource_metadata` | Covered | `requireMcpAuth` and `createMcpProtectedRequestHandler` return a JSON-RPC `401` with the RFC 9728 header [B1, B10]. |
| A3 | `scope` in the challenge | Covered | `challengeScopes` / `requiredScopes` options [B1]. |
| A4 | No `offline_access` in resource metadata | Covered | The source filters OIDC identity scopes and `offline_access` out of the resource's `scopes_supported` [B10]. |
| A5 | Authorization server metadata (RFC 8414 or OIDC Discovery) | Covered with caveats | Both are provided [B2]. Same Next.js caveat as A1: the well-known paths outside the auth base path need their own routes; helpers `oauthProviderAuthServerMetadata` and `oauthProviderOpenIdConfigMetadata` are documented for that [B2]. |
| A6 | `code_challenge_methods_supported` in metadata | Covered | Source emits `code_challenge_methods_supported: ["S256"]` [B10]. |
| R1 | Client ID Metadata Documents | Covered with caveats | `@better-auth/cimd` with `metadataProfile: "mcp-2026-07-28"` [B1, B3]. Caveats: a separate package; requires an application-supplied network transport, bundled only for Node.js (`@better-auth/cimd/node`) [B3]; open issues above [B12]. |
| R2 | Advertise `client_id_metadata_document_supported` | Covered | Advertised only when `cimd()` is installed [B1]. |
| R3 | CIMD validation (exact `client_id`, required fields, redirect URI in document) | Covered | Simple string comparison of `client_id` with the URL; the MCP profile requires `client_id`, `client_name`, non-empty `redirect_uris`; redirect policy stays with the provider, exact match by default [B3, B5]. |
| R4 | CIMD fetch, caching, SSRF protection | Covered | HTTP cache semantics, fetch budget, 5 s timeout, 5 KB limit, no redirects; the Node transport resolves once, rejects non-public addresses and pins the connection [B3]. |
| R5 | Show redirect URI hostname; warn on localhost-only | Not covered by the library | The consent page is built by the application. The docs tell the application to show the host of the signed `redirect_uri` for clients an administrator did not create [B6]. The docs are silent on a localhost-only warning. |
| R6 | Dynamic Client Registration (optional, deprecated) | Covered (opt-in) | Off by default. Enabled with `allowDynamicClientRegistration` and `allowUnauthenticatedClientRegistration`; `clientRegistrationDefaultApplicationType: "infer"` handles clients that omit `application_type`. `registration_endpoint` is absent from discovery unless enabled [B1, B5]. |
| R7 | Pre-registered clients | Covered | Admin and user client APIs (`adminCreateOAuthClient`, public clients with `token_endpoint_auth_method: "none"`) [B2, B5]. |
| T1 | OAuth 2.1 authorization server | Covered | Authorization code, client credentials, rotating refresh tokens; implicit and password grants not offered [B2]. Stated by Better Auth; no independent conformance evidence was looked for. |
| T2 | PKCE, `S256` | Covered | Required by default for every authorization code flow, `S256` only; public clients cannot opt out [B4]. |
| T3 | `resource` parameter, audience binding | Covered | `mcp({ resource })` registers the resource; a token requested with it is a JWT whose `aud` is that resource; a client only gets tokens for resources it is linked to, and `mcp()` links new clients to the MCP resource by default [B1, B4]. |
| T4 | `iss` in authorization responses, advertised | Covered | The docs state that every authorization response carries `iss` [B2]; source emits `authorization_response_iss_parameter_supported: true` [B10]. |
| T5 | Exact redirect matching; redirect URIs localhost or HTTPS | Covered with caveats | Exact match, except the port of a native client's loopback URI [B5]. Caveat: Better Auth also accepts private-use (custom scheme) redirect URIs for native clients [B5, B10], which is wider than the specification, which allows only localhost or HTTPS. |
| T6 | Short-lived access tokens | Covered with caveats | Default `accessTokenExpiresIn` is 3600 s, configurable globally, per resource and per scope [B4]. JWT access tokens cannot be revoked before they expire; only introspection reflects revocation, session end or a disabled client [B4]. |
| T7 | Refresh token rotation for public clients | Covered with caveats | Each refresh issues a new refresh token and invalidates the old one; reuse revokes the family [B4]. Caveat: `mcp()` overrides the default and allows a rotated refresh token to be reused for **30 seconds** without being treated as replay; the docs flag the risk and the setting `refreshTokenReuseInterval: 0` [B1, B4]. |
| C1 | Bearer token in the `Authorization` header on every request | Covered | `requireMcpAuth` reads the `Authorization` header on each request; metadata advertises header-only bearer [B1, B10]. |
| C2 | Validate token and audience; accept nothing else | Covered | Checks signature (against the JWKS), issuer, audience, expiry, and requires the `azp` claim so that session JWTs signed with the same key are rejected (fix shipped in 1.7.12) [B1, B10]. Verification is local, without a database round trip [B1]. Not passing tokens upstream is the application's concern. |
| C3 | `401` / `403` status codes | Covered | `401` for missing or invalid token, `403` for missing scope [B1]. `400` for malformed authorization requests: not looked for. |
| C4 | `insufficient_scope` challenge with all missing scopes | Covered | `requiredScopes`, or `createInsufficientScopeError` thrown from the handler; the challenge names every missing scope and carries `resource_metadata` [B1, B7]. |
| C5 | Scope hierarchies | Covered with caveats | Default matching is exact membership; a hierarchy has to be supplied through `isScopeSatisfied` [B10]. |

Also documented, beyond what the specification requires: DPoP sender-constrained tokens (RFC 9449), token introspection and revocation, device authorization grant for the product's own CLI [B1, B2].

## Vercel

All verified in the cited Vercel pages unless marked.

### Transport

- Vercel's own guide deploys an MCP server as a Next.js App Router route (`app/api/mcp/route.ts`) on Vercel Functions, reached by clients over Streamable HTTP. It uses Vercel's `mcp-handler` v2 on top of `@modelcontextprotocol/server` v2 and requires Node.js 20 or later [V4].
- `mcp-handler` 2.x serves 2026-07-28 natively and falls back to stateless Streamable HTTP for 2025-era clients; it removed the legacy HTTP+SSE transport and the Redis configuration [V4, V8]. The SDK's own `createMcpHandler` (the one Better Auth's docs use) defaults to the same stateless fallback and has a `legacy: "reject"` option that makes the endpoint modern-only [S1]. In both, 2025-era `GET` and `DELETE` session operations answer `405` [V8, S1].
- `mcp-handler` does not issue tokens or run an authorization server; it only verifies bearer tokens and publishes protected resource metadata [V4].
- WebSockets on Vercel Functions are in public beta [V1]; MCP's standard transports do not use them [M6].
- stdio is not applicable to a hosted server.

### How long a connection can stay open

- A function invocation, **including a streamed response**, is terminated at its maximum duration; the result is a `504 FUNCTION_INVOCATION_TIMEOUT` [V1].
- With Fluid compute (default for projects created on or after 2025-04-23 [V3]), Node.js runtime: default **300 s** on every plan; maximum **300 s** on Hobby, **800 s** on Pro and Enterprise; **1,800 s** extended maximum in beta on Pro and Enterprise, configured per function, for `nodejs22.x`, `nodejs24.x`, `nodejs26.x` [V1, V2]. In the Next.js App Router the value is the route's `maxDuration` export [V2].
- Edge runtime: must start responding within 25 s and can stream for up to 300 s [V1].
- Vercel states the same ceiling for long-lived connections: a connection is closed when its function reaches the maximum duration and nothing can keep it open beyond that [V6]. For MCP this bounds each tool call's response stream [V4] and the `subscriptions/listen` stream; the specification has no resumption, so a cut stream loses the in-flight request or ends the subscription until the client reconnects [M7, M9].
- For work longer than the limit Vercel points to Vercel Workflows [V1, V4].

### State kept between requests

- Functions scale to zero when there are no requests; Vercel routes requests to instances by load and availability [V7].
- Under Fluid compute several invocations can share one instance and its global state (in-memory caches, database connections) [V3, V7], but memory is never shared between instances, and module-level state breaks once traffic spans two instances; state that must outlive an instance belongs in an external store [V6].
- The 2026-07-28 transport keeps no protocol session, so it needs none of that state [M7]; the SDK handler builds a fresh server per request and keeps nothing from one request to the next [S1].
- What still needs shared or durable state: OAuth clients, consent, authorization codes, refresh tokens, CIMD metadata and DPoP replay records (Better Auth keeps them in the database) [B1]; and `subscriptions/listen` when notifications must reach streams held by other instances, for which the SDK expects a shared event bus [B1, S1].

### Other limits that apply

- Request and response body: 4.5 MB; the response limit does not apply to streamed responses [V1].
- 1,024 file descriptors per instance, shared by all concurrent executions (open streams, sockets, database connections) [V1].
- Concurrency auto-scales to 30,000 on Hobby and Pro, 100,000+ on Enterprise [V1].
- Billing: active CPU pauses while waiting on I/O; provisioned memory is billed for the instance lifetime, so an open stream costs memory time [V4, V6].
- Deployment Protection on preview deployments blocks MCP clients unless they send the automation bypass header [V4].

## Gaps and not verified

Gaps (verified as gaps in the sources):

1. **Consent screen content (R5).** The specification requires the authorization server to show the redirect URI hostname for CIMD clients. Better Auth leaves the consent page to the application and only advises it [B6]. Localhost-only warning: Better Auth's docs are silent.
2. **Well-known routes on Next.js (A1, A5).** The metadata documents live outside the auth base path; the OAuth Provider page documents route helpers for the authorization server metadata [B2] and the demo adds routes for the protected resource metadata [B10], but the MCP page describes the latter as served automatically [B1].
3. **Refresh-token reuse window (T7).** `mcp()` defaults to 30 s during which a rotated refresh token still works [B1].
4. **Redirect URI schemes (T5).** Better Auth accepts private-use schemes for native clients; the specification allows only localhost or HTTPS [B5, M5].
5. **Scope hierarchy (C5).** Exact matching by default [B10].
6. **`Origin` validation (transport).** The specification requires it [M6]. Better Auth's helpers do not mention it, and the SDK states that its HTTP handler checks neither the `Host` header, nor the `Origin` header, nor the token, and expects those checks in front of it [S1]. Vercel's MCP guide does not mention it [V4].
7. **JWT access tokens cannot be revoked before expiry** unless the resource server introspects [B4]. The specification does not require revocation; listed because the default check is local.

Not verified (what was looked at and why it does not settle it):

1. **End-to-end behaviour.** No package was installed and no flow was run. Every "covered" rests on documentation and on reading the v1.7.22 source for the points marked B10.
2. **Sample clients.** Which protocol revision (2026-07-28 or a 2025 one) and which registration mechanism (CIMD, Dynamic Client Registration, pre-registration) Claude, ChatGPT and Grok Bot use was not researched; it is outside this ticket's three questions. It decides whether the deprecated Dynamic Client Registration and the 2025-era fallback are needed: the specification says a legacy client against a modern-only server fails [M8], and Better Auth's documented setup is modern-only (`legacy: "reject"`) with Dynamic Client Registration off [B1].
3. **The CIMD Node transport on Vercel Functions.** Vercel documents full Node.js API coverage [V1] and Better Auth documents the transport for Node.js [B3], but neither documents the combination. Whether Vercel Functions have outbound IPv6 (relevant to open issue #12224 [B12]) was not looked up.
4. **Open issues #12224 and #12357.** Read as filed; not reproduced, not confirmed by maintainers, fix status unknown [B12].
5. **Neon.** No Neon-specific statement was found in the Better Auth pages read; Neon's own limits were not researched. Only the Drizzle adapter with the `pg` provider is documented.
6. **OAuth 2.1 conformance of Better Auth.** Stated by its docs [B2]; no certification or third-party test result was looked for.
7. **Differences between CIMD draft-00 (referenced by MCP) and draft-02 (implemented by Better Auth)** beyond what B3 states for the `mcp-2026-07-28` profile.
8. **Better Auth 1.8 beta.** Exists [B11]; not read.
9. **Vercel guide vs current packages.** The guide pins `mcp-handler@2.1.1` and links to the specification's `draft` URL [V4]; npm latest is 2.3.1 [V8]. The differences were not read.
10. **`400` for malformed authorization requests (C3)** and the remaining OAuth 2.1 security considerations the specification incorporates by reference [M5] were not checked line by line against Better Auth.
