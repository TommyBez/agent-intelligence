# What identity do Claude, ChatGPT and Grok Bot present to a remote MCP server?

Research for issue #2. All sources were read on 2026-11-12. Every claim is marked **[verified]** (read in a primary source, URL given) or listed under [Not verified](#not-verified). Facts only; no design.

## Short answer

| | Claude | ChatGPT | Grok Bot |
| - | - | - | - |
| 1. The MCP client itself | OAuth client: Anthropic-hosted Client ID Metadata Document (CIMD) `https://claude.ai/oauth/mcp-oauth-client-metadata`, or a DCR registration, or a client the customer or Anthropic pre-registered. Fixed redirect URI, published egress IP range. `clientInfo` is sent but is unauthenticated. | OAuth client: CIMD `https://chatgpt.com/oauth/client.json` (optionally `private_key_jwt` against a published JWKS), or DCR, or a predefined client. Fixed redirect URIs, OpenAI-managed mTLS client certificate, published egress IPs. | Remote MCP servers are reached from Cursor's servers; OAuth tokens sit on "Cursor's connector backend". A custom connector takes a URL, optional HTTP headers and an optional OAuth client ID/secret. Nothing else about the client identity is documented. |
| 2. The human | Whoever signs in at the server's own authorization server. Claude adds nothing, except in Enterprise Managed Auth, where a signed assertion carries `sub` and usually `email`. | Whoever signs in at the server's own authorization server. In addition, every tool call carries an anonymized user id (`_meta["openai/subject"]`) and, when available, an anonymized organization id. | Whoever signs in at the provider's login. "Bots act as the signed-in member." Nothing documented as sent by Grok Bot about the human. |
| 3. The individual agent | Nothing documented, at authorization or per request. | Nothing documented. An anonymized conversation id (`_meta["openai/session"]`) is sent per tool call; no agent or dot identifier is documented. | Nothing documented, at authorization or per request. |
| 4. Whose connection | The human's Claude account (each member connects with their own account). Exceptions: an organization-wide static header credential; Claude Tag channel connectors, which use an admin-provisioned service account. | The human's ChatGPT account; a dot uses its owner's existing connections. Exception: admin-authorized workspace connections that use a company-managed account. | The human's account, account-wide across all their Bots. Exception: Team Bot shared connectors (owner's account, a Manager's shared account, or "the Bot's own account"). |

In all three, a connection made through OAuth is a grant by a human account to the MCP client application. None of the three documents an identifier for an individual agent, persona or Bot reaching the MCP server.

## Baseline: what the MCP specification makes available

The current specification revision is **2026-07-28** (the `/specification/latest/` URLs redirect to it). **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization

- **Roles.** The MCP server is an OAuth 2.1 resource server; the MCP client is an OAuth 2.1 client "making protected resource requests on behalf of a resource owner"; the authorization server "is responsible for interacting with the user (if necessary) and issuing access tokens". The human is therefore authenticated by the server side's own authorization server, not by anything the client asserts. **[verified]** same URL.
- **Client registration.** A client obtains a client ID through one of three mechanisms: Client ID Metadata Documents (SHOULD be supported), pre-registration, or Dynamic Client Registration (MAY; "deprecated and retained for backwards compatibility"). **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration
- **CIMD.** The `client_id` is an HTTPS URL pointing to a JSON document that must include `client_id`, `client_name`, `redirect_uris`. The authorization server must check that the document's `client_id` equals the URL and must validate redirect URIs against the document. A client may use `private_key_jwt` with a JWKS for client authentication. **[verified]** same URL.
- **Trust in CIMD.** "Client ID Metadata Documents cannot prevent `localhost` URL impersonation by themselves"; authorization servers may apply domain-based trust policies and must display the redirect URI hostname during authorization. **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations
- **Client info, current revision.** There is no `initialize` handshake in 2026-07-28: the protocol is stateless and "Every request supplies this metadata in its `_meta` field". `io.modelcontextprotocol/clientInfo` (type `Implementation`: client name and version) is optional per request; clients SHOULD include it. The spec states that `clientInfo` is "self-reported by the sender and are not verified by the protocol", is "intended for display, logging, and debugging", and implementations "SHOULD NOT rely on them for security decisions". **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/index
- **Client info, legacy revisions.** Revisions 2025-11-25 and earlier establish a session with an `initialize` handshake that carries `clientInfo` (name, title, version, description, icons). **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning and https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle
- **No agent or user field.** The reserved `_meta` keys of 2026-07-28 are `progressToken`, `protocolVersion`, `clientInfo`, `clientCapabilities`, `logLevel`, `subscriptionId` and the OpenTelemetry trace keys. None identifies a human or an individual agent. Vendors may add keys under their own prefix. **[verified]** https://modelcontextprotocol.io/specification/2026-07-28/basic/index

## Claude

Sources are Anthropic's connector documentation at `claude.com/docs`. The page states that "The same authentication infrastructure backs claude.ai, Claude Desktop, Claude mobile, Claude Code, and Cowork". **[verified]** https://claude.com/docs/connectors/building/authentication

### 1. The MCP client itself

- **Three ways Claude identifies itself for a custom connector**: "with the Client ID Metadata Document that Anthropic hosts for it, through Dynamic Client Registration, or with an OAuth client ID the customer registered with you and enters in the dialog". The person adding the connector chooses among "Use Claude's published identity (Recommended)", "Register automatically" and "Use your own OAuth client". **[verified]** https://claude.com/docs/connectors/building/authentication and https://claude.com/docs/connectors/custom/add-unlisted
- **The hosted CIMD** is served at `https://claude.ai/oauth/mcp-oauth-client-metadata`: `client_name` "Claude", `client_uri` `https://claude.ai`, one redirect URI `https://claude.ai/api/mcp/auth_callback`, `token_endpoint_auth_method` "none" (public client). A second document, `https://claude.ai/oauth/mcp-oauth-client-metadata/confidential_client`, declares `private_key_jwt` with `jwks_uri` `https://claude.ai/v1/mcp/jwks`. **[verified]** by fetching both documents; the second URL is also named in https://claude.com/docs/connectors/building/enterprise-managed-auth
- **Claude Code is a separate client**: its own CIMD at `https://claude.ai/oauth/claude-code-client-metadata` (`client_name` "Claude Code", loopback redirect URIs, public client). **[verified]** by fetching the document; linked from https://claude.com/docs/connectors/building/authentication
- **When CIMD is used**: "Claude normally uses a Client ID Metadata Document only when your authorization server metadata advertises both" `client_id_metadata_document_supported: true` and `"none"` in `token_endpoint_auth_methods_supported`, "and otherwise falls back to DCR". **[verified]** same page.
- **DCR gives no stable identity**: "DCR causes Claude to register a new client on every fresh connection". **[verified]** same page.
- **Redirect URI**: hosted apps (web, Desktop, mobile, Cowork) always use `https://claude.ai/api/mcp/auth_callback`; Claude Code uses a loopback redirect on an ephemeral port. **[verified]** same page.
- **Network origin**: "Anthropic's outbound traffic to your server originates from `160.79.104.0/21`." Custom connectors are reached "from Anthropic's infrastructure". **[verified]** https://claude.com/docs/connectors/building/authentication and https://claude.com/docs/connectors/building/lazy-authentication
- **`clientInfo`**: "Claude identifies itself in the MCP `initialize` handshake through `clientInfo`, but the exact value depends on the surface and the request path. You may see `"name": "claude-ai"`, `"name": "Anthropic"`, sometimes with a service suffix, or `"name": "claude-code"`." **[verified]** https://claude.com/docs/connectors/building/testing
- **Trust**:
  - `clientInfo` is "unauthenticated: any client can claim any name, so it must never feed an authorization decision". **[verified]** https://claude.com/docs/connectors/building/testing
  - The CIMD's `client_name` is self-asserted: "On the consent screen, name the host of the `client_id` URL as the app asking for access, not the document's `client_name`, because the document is self-asserted". **[verified]** https://claude.com/docs/connectors/building/lazy-authentication
  - For loopback redirects (Claude Code), "any local process can bind a port and claim to be the client". **[verified]** https://claude.com/docs/connectors/building/authentication
  - Stronger options that are documented: the `confidential_client` document with a signed client assertion; a pre-registered confidential client (Anthropic-held credentials or customer-entered credentials); the published egress range. **[verified]** same page.
- **Authorization spec versions followed**: 2025-03-26, 2025-06-18 and 2025-11-25. **[verified]** https://claude.com/docs/connectors/building/index

### 2. The human who authorized the connection

- With OAuth, "each user signs in to your service with their own account when they connect". The server learns who the human is from its own sign-in; the documentation describes nothing Claude adds about the human in the interactive flow. **[verified]** https://claude.com/docs/connectors/building/authentication
- With a static credential in request headers, the server cannot tell users apart: "every member's requests carry the same key, so don't use it to identify which user is calling". **[verified]** same page.
- **Enterprise Managed Auth (Team and Enterprise plans)** replaces the consent screen with a signed identity assertion (ID-JAG JWT) sent to the server's token endpoint. It is issued either by the customer's identity provider or by Claude (issuer `https://identity.anthropic.com/issuers/<issuer-id>`, beta). Claims include `iss`, `sub`, `email`, `aud`, `client_id`, `scope`, `aud_tenant`, `jti`. For a Claude-issued assertion, "`sub` is an opaque value that differs under each issuer" and the assertion "always carries `email`", "the address of their Claude account"; it says only that "the user is a current member of the customer's Claude organization". **[verified]** https://claude.com/docs/connectors/building/enterprise-managed-auth

### 3. The individual agent inside the client

- The connector documentation read (authentication, lazy authentication, Enterprise Managed Auth, testing, troubleshooting, build overview, add-unlisted) describes no agent, persona, project, conversation or session identifier sent to the MCP server, either in the OAuth flow or per request. The only per-surface signal described is the unauthenticated `clientInfo.name` (`claude-ai`, `Anthropic`, `claude-code`). **[verified as absent from those pages]** URLs above. Absence in the docs is not proof of absence on the wire; see [Not verified](#not-verified).

### 4. Whose connection it is

- **The human's account.** On Free, Pro, Max and Ultra plans "You add custom connectors to your own account". On Team and Enterprise plans "an Owner adds the connector for the organization, and members then connect with their own account". **[verified]** https://claude.com/docs/connectors/custom/add-unlisted
- **Organization-level exception.** Request-header credentials (beta) are one credential "everyone in your organization shares". **[verified]** same page.
- **Agent-owned exception: Claude Tag (Claude in Slack, public beta).** "In Slack channels ... Claude acts with its own service accounts, rather than as a specific user. An organization Owner provisions this identity". For the MCP connector credential type: "Sign in once as an admin; the agent acts as that account." Personal connectors remain the individual's: "Claude works with your permissions" and "Claude never uses your connector for their request". In one-to-one DMs "the session runs on the individual's own claude.ai account". **[verified]** https://claude.com/docs/claude-tag/concepts/agent-identity , https://claude.com/docs/claude-tag/concepts/personal-connectors , https://claude.com/docs/claude-tag/admins/connections/custom

## ChatGPT

Sources are OpenAI's developer documentation (the former Apps SDK pages now live under `developers.openai.com/plugins/`; `/apps-sdk/...` URLs redirect there) and the ChatGPT product documentation at `learn.chatgpt.com`. The developer documentation says the contract "applies across both products", ChatGPT and Codex.

### 1. The MCP client itself

- **Registration mechanisms**: "Supported clients use Client ID Metadata Documents (CIMD), dynamic client registration (DCR), predefined OAuth clients, and PKCE." CIMD is "the preferred client registration method". **[verified]** https://developers.openai.com/plugins/build/auth
- **The CIMD** is `https://chatgpt.com/oauth/client.json` when the authorization server supports RFC 9207 issuer identification; otherwise a callback-specific `https://chatgpt.com/oauth/{callback_id}/client.json`. The fetched document declares `client_name` "ChatGPT", `client_uri` `https://chatgpt.com/`, redirect URI `https://chatgpt.com/connector_platform_oauth_redirect`, `token_endpoint_auth_methods_supported` `["none","private_key_jwt"]`, `token_endpoint_auth_method` "private_key_jwt", `jwks_uri` `https://chatgpt.com/oauth/jwks.json`. **[verified]** https://developers.openai.com/plugins/build/auth and by fetching https://chatgpt.com/oauth/client.json
- **Redirect URIs**: `https://chatgpt.com/connector_platform_oauth_redirect` (stable) or `https://chatgpt.com/connector/oauth/{callback_id}`. **[verified]** https://developers.openai.com/plugins/build/auth
- **DCR**: "ChatGPT runs DCR once per MCP server connection, then keeps and reuses the registered OAuth client for that connection." **[verified]** same page.
- **Trust**:
  - mTLS: "ChatGPT presents an OpenAI-managed client certificate when connecting to MCP servers, so you can verify the client at the transport layer with mTLS." The leaf certificate's SAN `dnsName` is `mtls.prod.connectors.openai.com` and chains to a published OpenAI CA. **[verified]** same page.
  - "You can also allowlist ChatGPT's published egress IP ranges." **[verified]** same page.
  - With `private_key_jwt`, the authorization server can "verify ChatGPT's token endpoint client assertion against the public JWKS published in the CIMD metadata". **[verified]** same page.
  - ChatGPT "does **not** support machine-to-machine OAuth grants such as client credentials, service accounts, or JWT bearer assertions, nor can it present custom API keys or customer-provided mTLS certificates". **[verified]** same page.
  - `_meta["openai/userAgent"]` is a hint only: "servers should never rely on them for authorization decisions" and it is not "a stable way to detect which host surface is calling your server". **[verified]** https://developers.openai.com/plugins/reference

### 2. The human who authorized the connection

- The human signs in at the server's own authorization server; ChatGPT "attaches the access token it received to subsequent MCP requests". **[verified]** https://developers.openai.com/plugins/build/auth
- **Per tool call, ChatGPT sends**: `_meta["openai/subject"]`, an "Anonymized user id sent to MCP servers for the purposes of rate limiting and identification", and `_meta["openai/organization"]`, an "Anonymized organization id associated with the current ChatGPT organization, when available". Also `openai/locale`, and the hints `openai/userAgent` and `openai/userLocation`. **[verified]** https://developers.openai.com/plugins/reference
- If the authorization server advertises OIDC scopes (`openid`, `email`, `profile`), ChatGPT requests them by default; ChatGPT uses the verified email for its own workspace domain restriction. This is information flowing from the server side to ChatGPT, not the reverse. **[verified]** https://developers.openai.com/plugins/build/auth
- A user can connect several accounts to the same plugin; "Each tool call uses the selected account's credentials and permissions." **[verified]** same page.

### 3. The individual agent inside the client

- Per tool call, ChatGPT sends `_meta["openai/session"]`, an "Anonymized conversation id for correlating tool calls within the same ChatGPT session". This identifies a conversation, not an agent. **[verified]** https://developers.openai.com/plugins/reference
- ChatGPT has a persistent agent, the **dot** ("an always-on agent"), with a name and appearance. A dot "can use supported plugins installed and enabled for your account"; "A personal dot uses its owner's existing plugin connections." **[verified]** https://learn.chatgpt.com/docs/dots , https://learn.chatgpt.com/docs/dots/computers-and-apps , https://learn.chatgpt.com/docs/enterprise/dots-security
- The dot identifier exists in OpenAI's own Compliance API records ("Dots-attributed records use `client_id = DOTS`; the agent object includes the dot ID, name, and type when available"). That is an OpenAI-side audit record for workspace admins, not something documented as sent to the MCP server. **[verified]** https://learn.chatgpt.com/docs/enterprise/dots-admin-guide
- The list of `_meta` fields the client provides contains no agent or dot field. **[verified as absent]** https://developers.openai.com/plugins/reference

### 4. Whose connection it is

- **The human's account.** "A personal connection uses your own account." For a shared plugin: "Each person installs and connects with their own account." A dot's access goes through "its owner's authorized app connection". **[verified]** https://learn.chatgpt.com/docs/plugins , https://learn.chatgpt.com/docs/extend/mcp , https://learn.chatgpt.com/docs/enterprise/dots-security
- **Workspace exception.** "A workspace connection lets an eligible Team Task or @ChatGPT surface use a company-managed app account without each participant connecting a personal account." Admins authorize it; the recommended connection account is "an approved account or service identity". **[verified]** https://learn.chatgpt.com/docs/enterprise/shared-connections
- No connection type that belongs to one individual dot is described in the pages read. A dot can have "its own Slack identity", which is a messaging channel, not an MCP connection. **[verified]** https://learn.chatgpt.com/docs/enterprise/dots-admin-guide

## Grok Bot

This extends the "Known so far" section of issue #2 and does not repeat its verified points. Sources are xAI's documentation at `docs.x.ai/grok-bot/`.

### Corrections and additions to "Known so far"

- **The connector backend is Cursor's.** The exact sentence is "OAuth tokens are held on Cursor's connector backend, and the Bot invokes tools without receiving them." The same docs state "Grok Bot signs in with your Cursor account" and "Bots reach remote MCP servers from Cursor's servers". **[verified]** https://docs.x.ai/grok-bot/computer-and-apps , https://docs.x.ai/grok-bot/faq , https://docs.x.ai/grok-bot/security
- **The email address belongs to the account, not to a Bot.** "You can claim one email address at `mail.grokbot.com`", "Yours, not one Bot's. All your Bots can use the address in your own chats and routines", and "a shared Bot doesn't have a team address". The secondary-source claim that a Bot has its own email address is therefore not what the official docs say. **[verified]** https://docs.x.ai/grok-bot/computer-and-apps
- **The security page does make an identity statement**, though not about identifiers on the wire: "Bots act as the signed-in member. A Bot can never hold more access than the person it belongs to, every action stays attributable to a named member, and there is no separate machine identity outside your identity provider to provision, rotate, or audit. Team-managed connectors are the one exception: they may use team or service-account credentials." **[verified]** https://docs.x.ai/grok-bot/security

### 1. The MCP client itself

- A custom connector is added as **Remote HTTPS** with "a **Server URL**, with optional **HTTP headers** and an OAuth **Client ID** (a **Client Secret** field appears once you enter one)", or as a **Command** that runs on the Bot's computer. A Bot can also add an MCP server itself "by URL or by a command". **[verified]** https://docs.x.ai/grok-bot/computer-and-apps
- Remote MCP servers are contacted from Cursor's servers, not from the Bot's cloud computer, which is why the computer network allowlist does not cover them by default. **[verified]** https://docs.x.ai/grok-bot/security
- For Google's own OAuth (Gmail, Calendar, Drive), "Grok Bot signs in to Google as **Grok**, not Cursor". This is the client name on Google's consent screen for built-in Google apps, not a statement about custom MCP servers. **[verified]** https://docs.x.ai/grok-bot/computer-and-apps
- Nothing in the Grok Bot docs states the OAuth client name, redirect URI, registration mechanism (CIMD or DCR), `clientInfo` value, client certificate or egress range used toward a custom remote MCP server. See [Not verified](#not-verified).

### 2. The human who authorized the connection

- The human signs in at the provider: "finish the provider login in your browser". "An app uses the account you authorize. It can do only what that account can already do in that service." **[verified]** https://docs.x.ai/grok-bot/computer-and-apps
- Nothing is documented as sent by Grok Bot to the server about the human.

### 3. The individual agent inside the client

- Each Bot has "its own name, picture, and label" and its own memory. **[verified]** https://docs.x.ai/grok-bot/bots
- No page read states that any identifier of the individual Bot is sent to an MCP server, at authorization or per request. Pages checked: overview, how-grok-bot-works, bots, computer-and-apps, team-bots, teams-and-enterprises, security, security-faq, identity-and-access, approvals-security-and-privacy, faq. See [Not verified](#not-verified).

### 4. Whose connection it is

- **The human's account, across all Bots.** "App connections belong to the same account you used to sign in to Grok Bot." "Apps are account-wide. A connected app is available to every Bot you run." A command-type MCP server "is available to your other Bots too". **[verified]** https://docs.x.ai/grok-bot/computer-and-apps
- **Team Bots.** "A Team Bot uses the apps and secrets its owner added." In a member's own chat it can also use that member's apps after asking. "A connected account acts as the person the Bot is answering. One teammate's connection is never used for someone else, unless a Manager shares it as a shared connector." **[verified]** https://docs.x.ai/grok-bot/team-bots
- **Shared connectors** have three modes: "Use individual accounts", "Share your account", and "The Bot's own account: a separate account the Bot uses for the team. Some apps don't support this yet." Only the third mode is a connection that belongs to one Bot, and it is an account in the external service chosen for that Bot, set up under the Bot's **Service accounts**. **[verified]** https://docs.x.ai/grok-bot/team-bots

## Not verified

- **Grok Bot: any Bot identifier sent to an MCP server.** Not stated in any `docs.x.ai/grok-bot/` page listed above. The docs neither affirm nor deny it. Settling it needs a connection from Grok Bot to a server that logs requests.
- **Grok Bot: client identity toward a custom remote MCP server** (OAuth client name, `client_id` form, redirect URI, CIMD or DCR, `clientInfo`, source IP range of Cursor's connector backend). Not in the Grok Bot docs. Cursor's own MCP documentation (https://cursor.com/docs/mcp) says Cursor uses fixed redirect URLs (`https://www.cursor.com/agents/mcp/oauth/callback` for "Web and Cursor Agents", `http://localhost:8787/callback` for the desktop app) and "With dynamic client registration, Cursor registers as a public client with no client secret". That page describes Cursor, and no source read says it applies to Grok Bot's connector backend, so it does not settle the question. The "shared static egress IP addresses" in the Grok Bot security page are for the hosted computers; whether they also cover the connector backend is not stated.
- **Grok Bot: whether "the Bot's own account" mode is available for a custom MCP server.** The docs say only "Some apps don't support this yet."
- **Claude: whether any conversation, project, session or agent identifier reaches the MCP server.** The connector docs describe none. They also do not state that none is sent. Not checked against live traffic.
- **Claude: which MCP protocol revision the hosted client speaks**, and so whether `clientInfo` arrives in `initialize` (legacy) or per request (2026-07-28). The testing page describes the `initialize` handshake; the build overview lists authorization spec versions up to 2025-11-25 only.
- **ChatGPT: the `clientInfo` value.** Not documented in the pages read (auth, reference, MCP server, security and privacy, connect and test, and the full plugins export searched for `clientInfo`).
- **ChatGPT: stability and scope of `openai/subject`.** The reference says only "Anonymized user id". Whether it is stable over time, per plugin, or shared across plugins is not stated.
- **ChatGPT: whether a dot's tool calls are distinguishable from the owner's own chat** on the MCP server side (for example a different `openai/session` or `openai/userAgent` value). Not stated.
- **ChatGPT: whether one account can hold several dots.** The docs speak of "your dot" in the singular; not established either way.
- **How any of the three behaves in practice.** Everything above is documentation. No connection was made from any client to a test server.

## Sources

MCP specification (revision 2026-07-28 unless noted):

- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration
- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations
- https://modelcontextprotocol.io/specification/2026-07-28/basic/index
- https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning
- https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle

Anthropic:

- https://claude.com/docs/connectors/building/authentication
- https://claude.com/docs/connectors/building/lazy-authentication
- https://claude.com/docs/connectors/building/enterprise-managed-auth
- https://claude.com/docs/connectors/building/testing
- https://claude.com/docs/connectors/building/index
- https://claude.com/docs/connectors/building/troubleshooting
- https://claude.com/docs/connectors/custom/add-unlisted
- https://claude.com/docs/claude-tag/concepts/agent-identity
- https://claude.com/docs/claude-tag/concepts/personal-connectors
- https://claude.com/docs/claude-tag/admins/connections/custom
- https://claude.ai/oauth/mcp-oauth-client-metadata
- https://claude.ai/oauth/mcp-oauth-client-metadata/confidential_client
- https://claude.ai/oauth/claude-code-client-metadata

OpenAI:

- https://developers.openai.com/plugins/build/auth
- https://developers.openai.com/plugins/reference
- https://developers.openai.com/plugins/build/mcp-server
- https://developers.openai.com/plugins/guides/security-privacy
- https://developers.openai.com/plugins/deploy/connect-chatgpt
- https://chatgpt.com/oauth/client.json
- https://learn.chatgpt.com/docs/extend/mcp
- https://learn.chatgpt.com/docs/plugins
- https://learn.chatgpt.com/docs/dots
- https://learn.chatgpt.com/docs/dots/computers-and-apps
- https://learn.chatgpt.com/docs/enterprise/dots-security
- https://learn.chatgpt.com/docs/enterprise/dots-admin-guide
- https://learn.chatgpt.com/docs/enterprise/shared-connections

xAI (and Cursor, which the xAI docs name as the operator of the connector backend):

- https://docs.x.ai/grok-bot/overview
- https://docs.x.ai/grok-bot/how-grok-bot-works
- https://docs.x.ai/grok-bot/bots
- https://docs.x.ai/grok-bot/computer-and-apps
- https://docs.x.ai/grok-bot/team-bots
- https://docs.x.ai/grok-bot/teams-and-enterprises
- https://docs.x.ai/grok-bot/security
- https://docs.x.ai/grok-bot/security-faq
- https://docs.x.ai/grok-bot/identity-and-access
- https://docs.x.ai/grok-bot/approvals-security-and-privacy
- https://docs.x.ai/grok-bot/faq
- https://cursor.com/docs/mcp (not a Grok Bot source; see Not verified)
- https://cursor.com/help/grok-bot/connect-apps
