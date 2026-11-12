# MCP features supported by the sample MCP clients

Research for issue #4. All sources were read on **2026-11-12**. Only primary sources were used: vendor documentation (Anthropic, OpenAI, xAI and, where the xAI pages delegate to it, Cursor) and the MCP specification. Every claim carries a source tag such as `[A1]`; the tags resolve in [Sources](#sources).

Status marks: **V** = verified, read in a primary source on the date above. **NV** = not verified; the reason is given in [Not verified](#not-verified).

Terms follow the MCP specification. Revision 2025-11-25 defines three server features (resources, prompts, tools) and three client features (sampling, roots, elicitation) `[M1]`. The latest revision, 2026-07-28, keeps the three server features, lists only elicitation as a client feature, and moves tasks and MCP Apps to opt-in extensions `[M2]`.

## Short answer

1. **Features.** Tools are the only feature that all three clients are documented to support. Claude documents tools, prompts and resources for custom connectors and names what it does not support (resource subscriptions, sampling, draft capabilities) `[A1]`. OpenAI says plugins "primarily use tools", and documents server instructions, elicitation, MCP Apps UI and a draft events extension; it does not say what ChatGPT does with prompts or plain resources `[O3][O4][O5]`. Grok Bot's own pages only speak of tools; its feature support is not stated `[X2][X5]`.
2. **Limits.** Only Anthropic publishes result-size limits: about 150,000 characters and 240 seconds per tool call on claude.ai and Desktop, 25,000 tokens by default in Claude Code `[A1][A7]`. No result-size or tool-count limit was found for ChatGPT or Grok Bot.
3. **Connection.** Claude and ChatGPT state that a custom remote server connects without vendor review or a directory listing; the Grok Bot pages mention no such condition. Claude: any plan, by URL (Free: one connector; Team and Enterprise: an Owner adds it) `[A2]`. ChatGPT: documented for Business and Enterprise/Edu (admins and owners; delegated by role on Enterprise/Edu), with read/fetch only on Pro, on the web only `[O1]`. Grok Bot: the user, or a Bot, adds a server by URL, subject to the team's MCP policy `[X2][X4]`.
4. **Acting without a prompt.** All three offer schedules and event triggers on some surface. Claude: scheduled tasks in Claude Code (schedule, API call, GitHub event) and in Cowork `[A8][A9]`. ChatGPT: scheduled tasks, event-triggered tasks, and MCP Events, where the MCP server itself delivers events to ChatGPT by webhook `[O5][O7]`. Grok Bot: routines on a schedule, on an app event, on an email, or on a webhook call `[X3]`.

## Comparison

| | Claude (claude.ai, Desktop, mobile, Cowork) | ChatGPT (web) | Grok Bot |
| - | - | - | - |
| Tools | Supported **V** `[A1]` | Supported **V** `[O3][O4]` | Supported **V** `[X2][X5]` |
| Resources | Supported, text and binary **V** `[A1]` | UI resources for MCP Apps supported **V** `[O3]`; plain resources **NV** | **NV** |
| Prompts | Supported **V** `[A1]` | **NV** | **NV** |
| Server instructions | Used by Claude Code **V** `[A7]`; chat apps **NV** | Used **V** `[O4]` | **NV** |
| Elicitation | Chat apps **NV**; Claude Code supported **V** `[A7]` | Documented for server authors **V** `[O4]` | **NV** |
| Sampling | Not supported **V** `[A1]` | **NV** | **NV** |
| Resource subscriptions | Not supported **V** `[A1]` | **NV** | **NV** |
| Events pushed by the server | Claude Code channels only **V** `[A7]` | MCP Events (draft extension), webhook delivery **V** `[O5]` | **NV** |
| Result formats | Text and image tool results; text and binary resources **V** `[A1]` | `structuredContent`, `content`, `_meta` hidden from the model **V** `[O4]` | **NV** |
| Result size | About 150,000 characters; 240 s per call **V** `[A1]` | **NV** | **NV** |
| Number of tools | **NV** (Claude Code: no fixed cap **V** `[A7]`) | **NV** | **NV** |
| Transport | Streamable HTTP; legacy HTTP+SSE, being deprecated **V** `[A1]` | Streamable HTTP **V** `[O2][O3]` | Remote HTTPS URL, or a command on the Bot's computer **V** `[X2]` |
| Auth | OAuth (CIMD, DCR or own client), request headers (beta), or none **V** `[A2]` | OAuth 2.1 (CIMD, DCR or predefined client), no auth, optional mTLS **V** `[O9]` | OAuth client ID and secret, HTTP headers **V** `[X2]` |
| Who adds a custom server | The user; on Team/Enterprise an Owner **V** `[A2]` | Business: admins and owners; Enterprise/Edu: also members granted the permission; Pro: read/fetch only **V** `[O1]` | The user or a Bot, unless the team MCP policy blocks it **V** `[X2]` |
| Vendor approval or listing | Not required **V** `[A1][A3]` | Not required for private use; workspace publishing needs an admin **V** `[O1]` | None stated **V** `[X2]` |
| Schedules | Claude Code and Cowork scheduled tasks **V** `[A8][A9]` | Scheduled tasks **V** `[O7]` | Routines **V** `[X3]` |
| Event triggers | Claude Code: API call, GitHub event **V** `[A8]` | Gmail, Slack, GitHub events; MCP Events **V** `[O5][O7]` | Slack, GitHub, Linear, Sentry, PagerDuty, email, webhook **V** `[X3]` |

## Claude (Anthropic)

Anthropic documents four surfaces separately. The chat apps (claude.ai, Desktop, mobile) and Cowork share one connector infrastructure; Claude Code and the API differ.

### 1. Features

Chat apps and Cowork, custom connectors:

- A remote server can expose tools, prompts and resources. Tool results may be text or images; resources may be text or binary. **V** `[A1]`
- Not supported: resource subscriptions, sampling, and "advanced or draft" capabilities. **V** `[A1]`
- Claude's client is described as implementing a subset of the MCP specification. **V** `[A1]`
- MCP Apps (interactive UI rendered in the conversation) are supported. **V** `[A1][A4]`
- Every tool must declare the `readOnlyHint` and `destructiveHint` annotations. **V** `[A11]`
- Elicitation, roots, logging, progress and `list_changed` notifications are not mentioned in either the supported or the unsupported list. **NV**

Claude Code:

- Tools; resources, referenced with `@` mentions and through built-in list and read tools; prompts, exposed as commands. **V** `[A7]`
- `list_changed` notifications refresh tools, prompts and resources in an interactive session; only the tool list in non-interactive mode and the Agent SDK. **V** `[A7]`
- Elicitation is answered in interactive sessions. On connections to claude.ai connectors Claude Code declares form mode only. **V** `[A7]`
- `roots/list` is answered everywhere. **V** `[A7]`
- Channels: an MCP server can push messages into a running session. **V** `[A7]`
- Sampling. **NV**

Claude API (Messages API, MCP connector, beta):

- Only tool calls are supported. For prompts or resources the documentation points to client-side SDK helpers instead. **V** `[A6]`

### 2. Limits

| Limit | claude.ai and Desktop | Claude Code |
| - | - | - |
| Tool result size | About 150,000 characters **V** `[A1]` | 25,000 tokens by default, configurable with `MAX_MCP_OUTPUT_TOKENS`; warning above 10,000 tokens **V** `[A1][A7]` |
| Tool call timeout | 240 seconds per call **V** `[A1]` | Configurable with `MCP_TOOL_TIMEOUT` **V** `[A1]` |

Further Claude Code limits, all **V** `[A7]`:

- A text result longer than 50,000 characters is saved to a file and replaced by a file reference. A tool can raise this with `_meta["anthropic/maxResultSizeChars"]`, up to 500,000 characters.
- A response from an HTTP or SSE server larger than 16 MiB fails.
- No fixed cap on tools per server; the context window is the practical limit. Tool definitions are deferred through tool search by default.
- Server instructions and descriptions of tools loaded upfront are cut at 4,096 characters; descriptions of tools loaded through tool search at 16,384.

Other limits:

- Free plan: one custom connector. **V** `[A2]`
- Request headers on a custom connector: up to four (beta, limited availability). **V** `[A2]`
- A cap on the number of tools or connectors on claude.ai for paid plans. **NV**
- API: the server URL must start with `https://`. No tool-count or result-size limit is stated. **V** `[A6]`

### 3. Connection

- Available on Free, Pro, Max, Ultra, Team and Enterprise plans. **V** `[A2]`
- Free, Pro, Max, Ultra: the user adds the connector to their own account by entering a name and the server URL. **V** `[A2]`
- Team and Enterprise: an Owner adds the connector for the organization; on Enterprise a member with a custom role that manages the organization's libraries can too. Each member then connects with their own account. **V** `[A2]`
- A custom connector is added with no review by Anthropic. Review applies only to a directory listing. Both run on the same runtime. **V** `[A1][A3]`
- A custom connector is never suggested in chat and is labelled "Custom". **V** `[A3]`
- An install link can prefill the add-connector dialog; the user still confirms. **V** `[A3]`
- Claude connects from Anthropic's cloud, so the server must be reachable from the public internet, or through an MCP tunnel. **V** `[A2][A5]`
- Transport: Streamable HTTP; the legacy HTTP+SSE transport is supported and being deprecated. **V** `[A1]`
- Authentication: OAuth with Claude's published client identity (CIMD), Dynamic Client Registration, or a client ID the user registered; fixed request headers (beta); or no sign-in. Callback URL: `https://claude.ai/api/mcp/auth_callback`. Authorization specifications followed: 2025-03-26, 2025-06-18, 2025-11-25. **V** `[A1][A2]`
- Authentication settings cannot be changed after the connector is added. **V** `[A2]`
- Users set each tool to always allow, needs approval, or blocked. Organizations can set per-tool controls. **V** `[A4][A7]`
- Claude Code: connectors added in claude.ai are loaded automatically when the user signs in with a claude.ai subscription, and not when an API key or a third-party provider is active. Servers can also be added directly with `claude mcp add` at local, project or user scope. Organizations can restrict servers with allowlists and denylists. **V** `[A7]`
- API: the caller passes the server URL and an OAuth access token on each request, and handles the OAuth flow itself. Beta headers `mcp-client-2026-09-15` and `mcp-client-2025-11-20`. Not eligible for zero data retention. **V** `[A6]`

### 4. Acting without a human prompt

- **Claude Code cloud scheduled tasks** (formerly routines; research preview). A saved prompt, repositories and connectors, run on Anthropic's cloud. Triggers: a schedule (recurring or one-off), an HTTP POST to the task's endpoint, or a GitHub repository event. Plans: Pro, Max, Ultra, Team, Enterprise. All of the user's connected connectors are included by default. Schedules more frequent than hourly are rejected. Limits: 100 scheduled runs per hour per account; 30 manual or API runs per hour per task. **V** `[A8]`
- **Cowork scheduled tasks.** Hourly, daily, weekly, weekdays, or manual. Paid plans. Run remotely, unless the task needs local files or apps. Runs can use the connectors set up in Cowork. **V** `[A9]`
- **Claude Code channels.** An MCP server acting as a channel pushes messages into a session, so Claude reacts to external messages or webhook events. **V** `[A7]`
- **Research.** During research Claude can call connector tools without further approval. **V** `[A5]`
- A schedule or trigger in the plain claude.ai chat app, outside Cowork and Claude Code. **NV**
- Claude Tag routines in Slack. **NV**

## ChatGPT (OpenAI)

OpenAI's documentation changed shape during 2026: custom MCP servers are now added from **Plugins**, and a connected server becomes a plugin. Developer Mode is no longer required. **V** `[O1]`

### 1. Features

- An MCP server can expose tools, resources, prompts and instructions; plugins "primarily use tools". **V** `[O3]`
- Tool results: `structuredContent`, `content` (text or other MCP content), and `_meta`, which is hidden from the model. **V** `[O4]`
- Server `instructions` returned at initialization are used by ChatGPT and Codex alongside tool metadata. **V** `[O4]`
- Tool annotations (`readOnlyHint`, `destructiveHint`, `openWorldHint`) drive confirmation behavior. **V** `[O4]`
- Elicitation is documented as available to server authors. **V** `[O4]`
- A server can return an optional UI resource for clients that support MCP Apps. **V** `[O3]`
- **MCP Events.** ChatGPT implements webhook delivery from the draft MCP Events specification. A server advertises an `events` capability and implements `events/list`, `events/subscribe` and `events/unsubscribe`. ChatGPT subscribes with a callback URL and signing secret; the server POSTs events to it. Polling and streaming delivery are not supported. Available in Work chats on the web, in the desktop app with Cloud selected, and with dots. **V** `[O5]`
- What ChatGPT does with MCP prompts and with plain (non-UI) resources. **NV**
- Sampling, roots, resource subscriptions, `list_changed`. **NV**
- `search` and `fetch` tools are no longer required. They are still needed for a server to serve as a company knowledge source. **V** `[O1][O4]`
- Tool definitions do not update on their own in a workspace. After an admin approves a server, ChatGPT uses a frozen snapshot of its tools until an admin refreshes it; new actions are off by default. **V** `[O1]`

OpenAI API (Responses API, `mcp` tool):

- The API imports the server's tool list and calls tools. Supports `allowed_tools`, `require_approval`, `defer_loading`, and an `authorization` token. Transports: Streamable HTTP or HTTP/SSE. **V** `[O8]`
- Resources and prompts are not mentioned. **NV**

### 2. Limits

- Result size, number of tools, tool call timeout. **NV**
- MCP Events: one event per request, body at most 256 KiB. **V** `[O5]`
- Server instructions: the most important details should be in the first 512 characters. **V** `[O4]`
- Deep research can use custom servers for read/fetch actions only. **V** `[O1]`
- Custom servers cannot be connected on mobile. **V** `[O1]`
- App selection applies to the message where it is used, not the whole conversation. **V** `[O1]`

### 3. Connection

- Added from **Plugins** with the server's public HTTPS endpoint (typically ending in `/mcp`), or through Secure MCP Tunnel for a private server. The user configures authentication and acknowledges a risk warning. **V** `[O1][O2]`
- Plans: custom server connections with full MCP support are documented for Business and Enterprise/Edu on the web. Pro users can connect custom servers with read/fetch permissions only. **V** `[O1]`
- Business: only admins and owners can create and publish. **V** `[O1]`
- Enterprise/Edu: admins and owners can create; admins can grant the "Create plugins with MCPs" permission to selected members by role. It is off by default on Enterprise. Only admins and owners can publish to the workspace. **V** `[O1][O6]`
- Plus, Go and Free accounts. **NV**
- No OpenAI review is needed to connect and use a server privately or inside a workspace. Review applies to the public plugin directory, which also requires a stable public HTTPS endpoint and domain verification. **V** `[O1][O4]`
- Transport: Streamable HTTP. **V** `[O2][O3]`
- Authentication: OAuth 2.1 per the MCP authorization specification, with CIMD (preferred), Dynamic Client Registration, or a predefined client; a tool can be declared `noauth`. ChatGPT also presents an OpenAI-managed client certificate, so a server can verify it with mTLS. **V** `[O9]`
- ChatGPT connects from published IP ranges. **V** `[O4]`
- Write actions may require confirmation depending on app permissions; some may be blocked. **V** `[O1]`

### 4. Acting without a human prompt

- **Scheduled tasks.** One-time or recurring. Available from Free to Enterprise. Active tasks: 3 on Free and Go, 5 on Plus, 10 on Business and Edu, 15 on Pro and Enterprise. Free: at most daily. Paid: up to hourly. **V** `[O7]`
- **Event-triggered tasks.** Run in Work and respond to Gmail, Slack or GitHub activity. Plus, Pro, Business, Enterprise, Edu; not Free or Go. Up to 30 runs per hour and 720 per day across all of a user's event-triggered tasks. An action needing approval pauses the task. **V** `[O7]`
- **MCP Events.** A plugin's MCP server can deliver events that start work in the subscribed chat (see Features). Not every plugin supports events. **V** `[O5][O6]`
- Scheduled tasks can use supported connected apps. **V** `[O7]`
- Whether a scheduled task can call a custom MCP server. **NV**
- Workspace Agents and Codex automations exist as separate products. **NV**

## Grok Bot (xAI)

Grok Bot signs in with a Cursor account and runs on Cursor's cloud. Its connector policy, OAuth token storage and admin dashboard are Cursor's. **V** `[X4][X5][X6]`

Already verified before this research and not re-derived: Grok Bot supports custom tooling and can interact with any MCP server or API `[X1]`; routines start on a schedule, on an event, or on a webhook POST, and a session can be started by another Bot `[X8]`.

### 1. Features

- A connected app or MCP server gives a Bot tools. The model sees only tool results. **V** `[X2][X5]`
- Resources, prompts, sampling, elicitation, roots, notifications. **NV**
- Cursor's own MCP documentation lists tools, prompts, resources, roots, elicitation and MCP Apps as supported by Cursor. No Grok Bot page says this table applies to Grok Bot. **NV** for Grok Bot `[X7]`

### 2. Limits

- Result size, number of tools, accepted result formats. **NV**
- Routines: up to 50 per Bot; schedules at least five minutes apart; 300 runs per hour across a user's routines and 200 per routine. **V** `[X3]`
- Approvals for work that started without the user expire after about 30 minutes. **V** `[X6]`

### 3. Connection

- In the desktop app the user chooses **Add a custom connector**, with type Remote HTTPS (server URL, optional HTTP headers, optional OAuth client ID and secret) or Command (a command and arguments run on the Bot's cloud computer). **V** `[X2]`
- A Bot can itself add an MCP server that is not in the catalog, by URL or by command. A server on a private network or with a private certificate authority needs the user's approval first. **V** `[X2]`
- A connected app is account-wide: every Bot the user runs can use it. **V** `[X2]`
- Remote MCP servers are reached from Cursor's servers, not from the Bot's computer. OAuth tokens stay on Cursor's connector backend. **V** `[X5]`
- Teams: Grok Bot inherits the team's Cursor connector policy. A blocked server cannot be saved or shows as disabled by the team admin. The MCP allowlist is Enterprise only. With an allowlist network policy, an optional switch also holds remote MCP servers to that list. **V** `[X2][X4][X5]`
- No xAI or Cursor review, and no listing, is mentioned as a condition for adding a custom server. **V** `[X2]`
- Access: paid plans (The X Subscription Premium, Super, Ultra; legacy paid Cursor plans; linked SuperGrok plans). On Cursor Enterprise an admin enables it. **V** `[X6]`
- Adding a custom connector from the mobile app. **NV**

### 4. Acting without a human prompt

- Routines run in the cloud on a schedule, or when something happens: a Slack message; a GitHub, Linear, Sentry or PagerDuty event; an email to the Bot's own address; or a webhook call. **V** `[X3]`
- A webhook trigger exposes a URL and a bearer key; an HTTP POST with an optional JSON body starts a run. **V** `[X3]`
- Event triggers come from Cursor account integrations, which are separate from the connected apps. **V** `[X3]`
- Email written during a routine waits as a draft until the user sends it. **V** `[X2]`
- A trigger that originates from a custom MCP server. **NV**

## Not verified

| Claim | What was read | Why it does not settle it |
| - | - | - |
| Claude chat apps: elicitation, roots, logging, progress, `list_changed` | `[A1]` | The page lists what is supported and what is not; these appear in neither list. |
| Claude chat apps: use of server instructions | `[A1][A7]` | Only Claude Code's handling is described. |
| Claude chat apps: cap on tools or connectors for paid plans | `[A1][A2][A4][A5]` | Only the Free-plan cap of one custom connector is stated. |
| Claude Code: sampling | `[A7]` | Not found in the part of the page read in full; the first part was read through a summarizer only. |
| Claude: schedule or trigger in the plain chat app | `[A4][A5][A9]` | Scheduling is documented for Cowork and Claude Code only. |
| Claude Tag routines | `[A10]` | Seen in the documentation index; the page was not read. |
| ChatGPT: MCP prompts and plain resources | `[O3][O4][O9]` | The docs say servers can expose them and that plugins primarily use tools; they do not say whether ChatGPT reads or ignores them. |
| ChatGPT: sampling, roots, resource subscriptions, `list_changed` | `[O9]` | No mention in the combined Plugins documentation. |
| ChatGPT: result size, tool count, tool call timeout | `[O9]` | A search of the combined Plugins documentation for numeric limits found none for these. |
| ChatGPT: custom servers on Plus, Go, Free | `[O1][O6]` | `[O1]` names Business, Enterprise/Edu and Pro only. `[O6]` says availability depends on plan and role without listing them. |
| ChatGPT: scheduled tasks calling a custom MCP server | `[O7]` | The page names Gmail, Slack and GitHub and "supported apps". |
| ChatGPT: Workspace Agents, Codex automations | Index entries only | The pages were not read. |
| OpenAI API: resources and prompts | `[O8]` | The guide describes tool import and tool calls only, with no statement about other features. |
| Grok Bot: every MCP feature other than tools | `[X1]` to `[X6]` | The Grok Bot pages describe connected apps only as providers of tools. |
| Grok Bot: whether Cursor's MCP support table applies | `[X7]` | The table is about Cursor; Grok Bot pages link Cursor for policy, not for protocol support. |
| Grok Bot: result size, tool count, result formats | `[X1]` to `[X6]` | Not stated. |
| Grok Bot: custom connector from mobile | `[X2]` | The steps are given for the desktop app only. |
| Grok Bot: triggers from a custom MCP server | `[X3]` | Only the listed trigger sources are documented. |

Reading caveats:

- `[A5]` and `[A9]` were read through a summarizing fetch tool, so their wording was not seen in full. Every claim taken from `[A5]` except the Research one is also confirmed in `[A2]`.
- `[O1]`, `[O6]` and `[O7]` show a relative date only ("updated 16, 15 and 12 days ago"). `[X1]` is dated 2026-11-07 and `[X5]` 2026-11-10. The other pages carry no date.
- `[X8]` was not re-read; it is listed because the prior verification relied on it.

## Sources

Anthropic:

- `[A1]` Build an MCP server for Claude: https://claude.com/docs/connectors/building
- `[A2]` Add a connector that isn't in the directory: https://claude.com/docs/connectors/custom/remote-mcp
- `[A3]` Directory connectors vs custom connectors: https://claude.com/docs/connectors/building/directory-vs-custom
- `[A4]` Get started with connectors: https://claude.com/docs/connectors/getting-started
- `[A5]` Get started with custom connectors using remote MCP: https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
- `[A6]` MCP connector (Claude API): https://platform.claude.com/docs/en/agents-and-tools/mcp-connector
- `[A7]` Connect Claude Code to tools via MCP: https://code.claude.com/docs/en/mcp
- `[A8]` Automate work with scheduled tasks (Claude Code): https://code.claude.com/docs/en/routines
- `[A9]` Schedule recurring tasks in Claude Cowork: https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork
- `[A10]` Claude documentation index: https://claude.com/docs/llms.txt
- `[A11]` Model Context Protocol (MCP): https://claude.com/docs/connectors/building/mcp

OpenAI:

- `[O1]` Connecting custom MCP servers in ChatGPT: https://help.openai.com/en/articles/12584461
- `[O2]` Connect and test your plugin: https://developers.openai.com/plugins/deploy/connect-chatgpt
- `[O3]` MCP server (concepts): https://developers.openai.com/plugins/concepts/mcp-server
- `[O4]` Build an MCP server: https://developers.openai.com/plugins/build/mcp-server
- `[O5]` MCP Events: https://developers.openai.com/plugins/build/mcp-events
- `[O6]` Plugins in ChatGPT: https://help.openai.com/en/articles/20001256-plugins-in-chatgpt
- `[O7]` Scheduled tasks in ChatGPT: https://help.openai.com/en/articles/10291617-scheduled-tasks-in-chatgpt
- `[O8]` MCP servers (Responses API): https://developers.openai.com/api/docs/guides/tools-connectors-mcp
- `[O9]` Combined Plugins documentation, including Authentication: https://developers.openai.com/plugins/llms-full.txt

xAI and Cursor:

- `[X1]` Grok Bot overview: https://docs.x.ai/grok-bot/overview
- `[X2]` Use the computer and apps: https://docs.x.ai/grok-bot/computer-and-apps
- `[X3]` Skills and routines: https://docs.x.ai/grok-bot/skills-routines-and-automations
- `[X4]` Grok Bot for teams and enterprises: https://docs.x.ai/grok-bot/teams-and-enterprises
- `[X5]` Grok Bot security: https://docs.x.ai/grok-bot/security
- `[X6]` Grok Bot FAQ: https://docs.x.ai/grok-bot/faq
- `[X7]` Cursor, Model Context Protocol (MCP): https://cursor.com/docs/mcp
- `[X8]` Designing Grok Bot: https://x.ai/news/designing-grok-bot

MCP specification:

- `[M1]` Specification, revision 2025-11-25: https://modelcontextprotocol.io/specification/2025-11-25
- `[M2]` Specification, latest (revision 2026-07-28): https://modelcontextprotocol.io/specification/latest
