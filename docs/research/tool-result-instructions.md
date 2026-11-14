# What the MCP specification and the sample MCP clients say about text in a tool result that reads as an instruction

Research for issue #19. All sources were read on **2026-11-14**. Only primary sources were used: the MCP specification, its schema, its official documentation, blog and repository, and the documentation, help centres and published model guidelines of Anthropic, OpenAI and xAI. Every claim carries a source tag such as `[M3]`; the tags resolve in [Sources](#sources). Nothing was run.

Status marks: **V** = verified, read in a primary source on the date above. **NV** = not verified; the reason is given in [Not verified](#not-verified).

The sources are paraphrased throughout. One sentence is quoted word for word, in the section on Claude. Names of fields, of buttons and of documents are given as the sources spell them.

Terms follow `GLOSSARY.md`: agent, MCP client, connection, owner, member, content, body, comment. The MCP specification says *host* and *client* for what the glossary calls the MCP client, and *server* for what the product would be. "Result" means the result of a tool call. "Hint" means one of the four tool annotations `readOnlyHint`, `destructiveHint`, `idempotentHint` and `openWorldHint`.

The specification revision read is **2026-07-28**, which the versioning page names as the current one `[M2]`.

## Short answer

1. **How the text of a result is treated.** The specification does not say. It defines the result as content that the MCP client hands to the model, and it asks the MCP client to validate results before doing so, without saying what that means `[M3]`. The answer is in what each vendor publishes about its model. Anthropic and OpenAI both state that instructions found in a tool result are information and not commands `[A12][O9]`. xAI states that content from outside is marked as untrusted data when it is shown to the model and that only the user's own messages count as instructions `[X1][X2]`. All three say that these protections lower the risk and do not remove it `[A9][O7][X1]`.
2. **Whether a server can mark a result as written by a third party or as untrusted.** No. The specification has no such field. The annotations of a content item are `audience`, `priority` and `lastModified` and nothing else `[M4][M5]`. `_meta` can carry any key under the server's own prefix, but no key has a meaning an MCP client knows `[M6][M11]`. A proposal to add trust and sensitivity annotations to results (SEP-1913) is open and not accepted `[M10][M12]`. None of the three vendors documents a marking of its own. What they document instead is a way of writing the result: Anthropic advises saying in the tool description or in the structure of the result what the content is and where it came from `[A10]`.
3. **What the MCP client does by itself.** All three can stop and ask the human before a tool is called `[A4][O1][X3]`. Claude and Grok Bot let the human switch the question off for a tool or for a kind of action `[A4][X3]`; ChatGPT has settings for each app that say when it asks `[O10]`. In Claude and ChatGPT the hints change this. In Claude a tool marked read-only can run without a question and a tool marked destructive always gets one `[A1]`. In ChatGPT the hints inform when it asks, and a tool marked destructive gets explicit approval first `[O2][O3]`. Grok Bot does not mention the hints; a review model decides for each action whether to ask `[X2][X3]`. The specification calls the hints untrusted unless the server is trusted `[M3]`. The question protects a tool that writes. It does nothing about a result that only has to be read to mislead the agent.
4. **What each recommends to a server author whose results carry text written by others.** Very little that is specific. The specification says a server must sanitize its tool outputs and stops there `[M3]`; its security best practices page does not treat the subject `[M8]`. Anthropic's and OpenAI's rules for listed servers are about the server's own text: tool descriptions must not instruct the model `[A1][A8][O5]`. OpenAI tells server authors to assume that injected instructions will reach the server, to check every input and to require human confirmation for what cannot be undone `[O4]`. The fullest advice is Anthropic's, written for developers on its API and not for authors of MCP servers: keep third-party text inside the tool result, label its source, encode it as JSON, put no instructions of your own in a result, and give the model as little reach as the task needs `[A10]`. xAI publishes no rules for server authors; its one example of a server enforces its limits on the server and does not leave them to the model `[X10]`.

## Comparison

| | MCP specification 2026-07-28 | Claude | ChatGPT | Grok Bot |
| --- | --- | --- | --- | --- |
| Text of a result | Passed to the model. Client should validate first; the word is not explained **V** `[M3]` | Instructions in it are information, not commands **V** `[A12]`. Model trained to be sceptical of them **V** `[A10]` | Assumed untrusted, no authority unless delegated **V** `[O9]` | Marked as untrusted data for the model; only the user's messages are instructions **V** `[X1][X2]` |
| Marking a result as third-party or untrusted | No field. Proposal open **V** `[M5][M12]` | None documented **V** `[A7]` | None documented. `_meta` of a result is hidden from the model **V** `[O3]` | None documented **V** `[X1][X8]` |
| Content annotation `audience` | Defined: `user`, `assistant` **V** `[M4]` | Not mentioned **NV** | Not mentioned **NV** | Not mentioned **NV** |
| Asking the human before a call | Should be possible to deny any call **V** `[M3]` | Per tool: always allow, needs approval, blocked **V** `[A4]` | Asks for writes depending on app permissions, context and impact **V** `[O1]` | A review model decides; rules can force a question **V** `[X3]` |
| Effect of the hints | Hints only; untrusted unless server trusted **V** `[M3][M5]` | Read-only runs without a question, destructive always asks **V** `[A1]` | Inform confirmation; destructive asks first **V** `[O2][O3]` | Not mentioned **NV** |
| Hints required | No, all optional **V** `[M5]` | `title` plus `readOnlyHint` or `destructiveHint`, for listing **V** `[A1][A8]` | `readOnlyHint`, `destructiveHint`, `openWorldHint`, for listing **V** `[O3][O5]` | Not mentioned **NV** |
| Advice to a server author | Sanitize outputs; not defined **V** `[M3]` | Rules on descriptions; API advice on labelling results **V** `[A1][A10]` | Assume injection reaches the server; confirm irreversible actions **V** `[O4]` | No rules; one example enforces limits on the server **V** `[X9][X10]` |

## MCP specification (revision 2026-07-28)

### 1. How the text of a result is treated

- A result holds unstructured `content` (text, image, audio, resource link, embedded resource) and, optionally, `structuredContent`, any JSON value. A tool that returns structured content should also return it as text. **V** `[M3]`
- The specification gives the result no level of trust. Its message flow shows the MCP client passing the result to the model to be processed. **V** `[M3]`
- Among the things an MCP client should do, the tools page lists validating results before passing them to the model. It does not say what to validate them against, apart from the output schema of the tool. **V** `[M3]`
- The only place the official material calls a result untrusted is the client best practices page, in the part on running several tool calls from generated code: results from one server are untrusted input to another, and output should be checked and cut before it goes back to the model. **V** `[M9]`
- The official blog states the problem plainly: models follow instructions in content and cannot reliably tell the user's instructions from ones an attacker put in a web page, an email or a calendar event. It adds that nothing in an annotation tells the model to ignore instructions it reads. **V** `[M11]`

### 2. Marking a result as written by a third party or as untrusted

- **Content annotations.** Every content item can carry `annotations` with three fields: `audience` (a list of `user` and `assistant`), `priority` (0 to 1) and `lastModified`. The specification presents them as hints an MCP client can use to filter, to order and to show dates. None says who wrote the content or whether it can be trusted. **V** `[M4][M5]`
- **`_meta`.** A result and each content item can carry `_meta`. Keys under a prefix whose second label is `modelcontextprotocol` or `mcp` are reserved; a server uses its own prefix in reverse-DNS form. The specification gives such keys no meaning. **V** `[M5][M6]`
- The blog says what follows from that: `_meta` suits a deployment where one organization runs both the server and the MCP client, and it is a way to try an idea before proposing it, but an MCP client off the shelf does not read a key it has never heard of. **V** `[M11]`
- **Structured or unstructured.** The two forms differ in shape only. The specification attaches no difference of trust to them. **V** `[M3]`
- **Tool annotations.** The four hints sit on the tool, not on a result. `openWorldHint` says the tool may deal with entities outside a closed domain; its default is true. The blog reads it as the hint that concerns what a result may bring back, and advises treating anything a tool considers external as a possible source of untrusted content. **V** `[M5][M11]`
- **Proposals.** The Tool Annotations Interest Group, chartered on 2026-04-20, lists as open questions whether annotations on results should be added and how annotations should relate to trust and to the human in the loop. **V** `[M10]`
- SEP-1913, Trust and Sensitivity Annotations, is an open pull request with the label `draft`, last updated on 2026-11-14. It is not part of revision 2026-07-28: the list of changes of that revision has nothing on tool annotations or on the trust of results. **V** `[M7][M12]`
- The summary of SEP-1913 describes annotations at the level of a result, among them a reuse of `openWorldHint` for data from untrusted or outside sources, a hint for detected suspicious patterns and a field for provenance. **NV** `[M12]`

### 3. What the MCP client does by itself

- The overview states as a principle that the MCP client must get the user's explicit consent before invoking any tool, and adds that the protocol cannot enforce this. **V** `[M1]`
- The tools page says there should always be a human in the loop who can deny a call, and that applications should show which tools are exposed, show when one is invoked and ask for confirmation. **V** `[M3]`
- An MCP client should also show the inputs of a call to the user before it is made, so that data is not sent out by malice or by accident. **V** `[M3]`
- The hints and their defaults: `readOnlyHint` false, `destructiveHint` true, `idempotentHint` false, `openWorldHint` true. A tool with no annotations is therefore taken to write, to be destructive and to reach outside. `destructiveHint` and `idempotentHint` mean something only when `readOnlyHint` is false. **V** `[M5]`
- All four are hints. An MCP client must consider them untrusted unless the server is trusted, and should never decide on the use of a tool from the hints of an untrusted server. **V** `[M3][M5]`
- The blog lists what an MCP client can do with each hint: skip the confirmation for a read-only tool, warn before a destructive one, retry an idempotent one, and look more closely at the output of an open-world one. It notes that most MCP clients treat the installation of a server as the sign of trust and make no further distinction. **V** `[M11]`

### 4. What it recommends to a server author

- A server must validate all tool inputs, implement access controls, limit the rate of calls and sanitize tool outputs. The word sanitize is not explained. **V** `[M3]`
- The security best practices page covers authorization, token handling, request forgery, state handles, local servers and scopes. It has no section on instructions inside results, on content written by third parties or on prompt injection. **V** `[M8]`
- The index of the official documentation lists no page whose title or description names prompt injection or untrusted content. **V** `[M13]`
- The blog's advice to server authors: set `readOnlyHint` true on tools that only read, `destructiveHint` false on tools that only add, `openWorldHint` false on tools with a closed domain; do not rely on a hint for anything that has to hold, which needs controls outside the model. **V** `[M11]`

## Claude (Anthropic)

### 1. How the text of a result is treated

- Claude's constitution puts tool call results in a class it calls conversational inputs, with documents and search results. It says that instructions inside such inputs are to be treated as information and not as commands that must be followed, and that Claude should use judgment about how far to trust an input: more for an established tool, less for an unreliable source. **V** `[A12]`
- Anthropic's API documentation says the same of the trained model: "Claude is trained to treat instructions that appear inside tool results with appropriate skepticism" `[A10]`. **V**
- The same page warns that Claude may ignore or flag instructions a developer puts in a tool result, because it treats the content of results as untrusted data. **V** `[A10]`
- For connectors, the user-facing pages say only that Claude has protections of its own against prompt injection, that these attempt to block such attacks, and that the user should still watch tool inputs and outputs. **V** `[A2][A5][A9]`
- For the computer use and browser use tools, Anthropic says it runs extra classifiers on what the tools return. The pages read do not say this of connector results. **NV** `[A10]`

### 2. Marking a result as written by a third party or as untrusted

- The page that lists what Claude's client supports names text and image results. It does not mention content annotations, `audience`, `structuredContent` or `_meta` on a result. **V** `[A7]`
- No Anthropic page read documents a field by which a server tells Claude that part of a result was written by someone else. **V** `[A1][A3][A7]`
- The advice Anthropic gives developers on its API is to make the nature and the source of the content explicit, in the tool description or in the structure of the result, for example that it is the body of an email from an unknown sender. It says this helps Claude judge how far to trust directives in it. **V** `[A10]`
- The same page advises wrapping third-party strings in a JSON object, so that the text cannot be mistaken for the structure around it. **V** `[A10]`
- In Claude Code, a server can set `_meta["anthropic/requiresUserInteraction"]` to `true` on a tool, and Claude Code then asks on every call of that tool, also in the permission modes that normally skip the question; in the mode that never asks it denies the call. This is on the tool, not on a result, and is documented for Claude Code only. **V** `[A14]`

### 3. What the MCP client does by itself

- Claude can ask for approval before it uses a connector's tool. The human chooses **Allow once** or **Always allow**. **V** `[A4]`
- Under the connector's tool permissions the human sets **Always allow**, **Needs approval** or **Blocked** for a group of tools or for one tool. **V** `[A4]`
- The hints decide the automatic permissions: a read-only tool can run without confirmation at each call, and a destructive tool always prompts. **V** `[A1]`
- For a listing in the directory every tool needs a `title` and the applicable hint, `readOnlyHint` or `destructiveHint`; the orientation page says every tool must declare both. `openWorldHint` and `idempotentHint` are not asked for. **V** `[A1][A3][A8]`
- The submission portal groups a server's tools into read-only and write by their annotations. **V** `[A6]`
- During Research, Claude can call connector tools without further approval. Anthropic advises turning off tools that write when Research is used with a custom connector. **V** `[A9]`
- Claude Code uses `readOnlyHint` and `idempotentHint` to decide whether it may repeat a call that was pending when a connection closed, and runs read-only calls in parallel. **V** `[A14]`

### 4. What it recommends to a server author

- The checklist for the directory rejects tool descriptions that tell Claude how to behave: that make it call tools the user did not ask for, interfere with other tools, pull instructions from outside sources, hide or encode instructions, or try to override system instructions. The policy behind it says the same. Both speak of the server's own descriptions, not of content the server returns. **V** `[A1][A8]`
- The checklist asks for a read tool and separate write tools, never one tool for both, and for results of a reasonable size. **V** `[A1]`
- The submission includes an acknowledgment on prompt injection among seven. Its text was not seen. **NV** `[A6]`
- For developers on the API, Anthropic's list is: deliver third-party content only inside tool results; say what it is and where it came from; encode it as JSON; put no instructions of your own in a result; limit the model's access to data and actions; screen tool output with a small model before the main model acts on it; test with content that contains injected instructions. **V** `[A10][A11]`
- To users Anthropic says: connect only servers you trust, review approval requests, use **Always allow** only for trusted servers, block tools you do not need. **V** `[A2][A5][A9]`

## ChatGPT (OpenAI)

### 1. How the text of a result is treated

- OpenAI's Model Spec of 2026-10-29 has a rule to ignore untrusted data by default. Tool outputs, with quoted text, attachments and multimodal data, are assumed to hold untrusted data and have no authority by default: instructions in them are information, not instructions to follow. **V** `[O9]`
- Authority can be delegated to a tool output by an instruction that does have authority, and a user can delegate it implicitly; the Model Spec's example is a coding task where project files hold instructions the user expects to be followed. The model is to judge by the tool call that produced the output, its level of trust, how much the user knows and controls of it, and how the instructions relate to what the user asked. **V** `[O9]`
- The Model Spec recommends that developers give the model enough information about each tool to judge its trustworthiness and side effects. **V** `[O9]`
- The API guide says that malicious MCP servers may include hidden instructions, that OpenAI has safeguards of its own to help detect and block them, and that inputs and outputs should still be reviewed. **V** `[O7]`

### 2. Marking a result as written by a third party or as untrusted

- ChatGPT shows `structuredContent` and `content` to the model, and both appear in the conversation. `_meta` on a result goes only to the server's UI component and is hidden from the model. **V** `[O2][O3]`
- No OpenAI page read documents a field by which a server marks part of a result as third-party or untrusted, and none mentions the content annotation `audience`. **V** `[O2][O3]`
- `openWorldHint` is defined by OpenAI as true for a tool that reaches the public internet or open-ended outside entities, and allowed to be false for a tool limited to a bounded private account or workspace, even one hosted elsewhere. It is on the tool, not on a result. **V** `[O2][O5]`
- A server's UI component is told to treat `structuredContent` as untrusted input when it renders it. This concerns the component, not the model. **V** `[O3]`

### 3. What the MCP client does by itself

- For actions that write or modify, ChatGPT may ask for confirmation, depending on the app's permissions, the context of the action and its possible impact. Some especially risky actions may be blocked and not offered for approval. **V** `[O1]`
- Each app has **Permissions** settings that set when ChatGPT asks before using it. OpenAI adds that asking for confirmation does not replace limits on what the app or the connected account can reach. **V** `[O10]`
- The hints help ChatGPT choose its confirmation and safety behaviour. `destructiveHint` tells it to get explicit approval first. The reference says the hints only influence how the call is presented to the user. **V** `[O2][O3]`
- `readOnlyHint`, `destructiveHint` and `openWorldHint` must be set as explicit booleans on every tool of a listed server; `idempotentHint` is optional. A value cannot be corrected by a written justification; the server has to change it. **V** `[O3][O5][O6]`
- Adding a custom MCP server requires acknowledging a risk warning. In a workspace, ChatGPT keeps a frozen snapshot of an approved server's tools; new tools are off until an admin turns them on. **V** `[O1]`
- Deep research can use custom servers for read and fetch only. **V** `[O1]`
- In the API, each MCP tool call needs the developer's approval by default (`require_approval`). **V** `[O7]`
- Codex, a separate MCP client from OpenAI, has an approval mode `writes` that prompts for tools not marked read-only. **V** `[O11]`

### 4. What it recommends to a server author

- Assume that prompt injection and malicious inputs will reach the server; check every input on the server even when the model supplied it; keep audit logs; require human confirmation for operations that cannot be undone; review tool descriptions. **V** `[O4]`
- Set the hints to match what the tool does. They inform the MCP client's safeguards and do not replace authorization, validation or confirmation in the server. **V** `[O2][O5]`
- Model-readable fields of a listed server must not manipulate how the model chooses or uses other tools. **V** `[O5]`
- A result must hold only data relevant to the request, with no secrets and no internal identifiers that are not needed. **V** `[O2][O5]`
- For developers on the API: connect only trusted servers, require approval for sensitive actions, be careful with URLs that come back in tool output, keep untrusted input out of developer messages, and use structured outputs so that free text does not carry instructions from one step to the next. **V** `[O7][O8]`
- No OpenAI page read gives advice specific to a server whose results hold text written by its own users. **V** `[O4][O5]`

## Grok (xAI)

The glossary names Grok Bot as the sample MCP client. xAI also documents custom MCP connectors on grok.com and remote MCP tools in its API; they are noted where they differ.

### 1. How the text of a result is treated

- The security page says that content a Bot reads from outside, which it lists as web pages, app results and command output, can try to steer it, and that outside content is marked as untrusted data when it is presented to the model. How it is marked is not said. **V** `[X1]`
- Only the user's own messages count as instructions. Text from somewhere else, such as an email or a web page, does not count as the user's request when an action is reviewed. **V** `[X2][X3]`
- For email the same is stated in detail: each message a Bot reads is marked as from the user or from someone else, and instructions inside an email never count as permission to send or to act. **V** `[X4]`
- xAI says these controls reduce the risk from malicious content and do not eliminate it. **V** `[X1][X2]`

### 2. Marking a result as written by a third party or as untrusted

- Nothing is documented. None of the xAI pages read mentions content annotations, `audience` or `_meta` on a result. **V** `[X1][X4][X7][X8]`
- The tool annotations appear once, in an example for the API: its tools carry `readOnlyHint`, which the example calls a hint for clients that nothing relies on. No page on Grok Bot or on grok.com connectors mentions them. **V** `[X10]`
- An Enterprise team can have every model request, with the content the Bot has gathered, checked by its own security server before the model sees it (inference hooks, in beta). This is a control of the team that uses Grok Bot, not of the server author. **V** `[X6]`

### 3. What the MCP client does by itself

- Auto Review, a separate review model that is on by default, checks each action a Bot is about to take, calls to connected apps included. It lets the action go, asks for approval or denies it. Most actions go ahead on their own. **V** `[X1][X2][X3]`
- An approval card shows the operation and its inputs, with **Allow once**, **Always allow** and **Deny**. Some cards carry a warning, for example that the action cannot be undone or that it shares data outside the chat. **V** `[X3]`
- The human can write rules in plain words: an **Ask first** rule always stops a matching action, an **Allow automatically** rule lets it through when the reviewer finds no other reason to stop. **V** `[X1][X3]`
- Auto Review does not review every side effect; memory writes are given as an example. xAI calls it model-based and a complement to least privilege and explicit approval, not a replacement. **V** `[X1][X3]`
- What needs approval depends on the tool, the risk of the action and the rules. The hints are not named as an input. **V** `[X5]`
- In xAI's API the `require_approval` parameter for remote MCP tools is not supported; a developer limits tools with `allowed_tools`. **V** `[X7]`
- The page on grok.com connectors describes adding a custom MCP server and says nothing about approvals. **V** `[X8]`

### 4. What it recommends to a server author

- xAI's documentation has no page of rules or review criteria for authors of MCP servers. **V** `[X9]`
- Its one example of a server, a database that only reads, enforces its limits on the server and says they are not left to the model. **V** `[X10]`
- To users xAI says: state in the request what the Bot may change and where it must stop, keep **Ask first** rules on anything that cannot be undone, and approve no action whose target or effect is unclear. **V** `[X1][X2][X3]`

## What the findings mean for a server whose results carry text written by other members

This section sets the findings against the product's situation. It does not choose.

- **The defence the vendors describe sits in the model and in the MCP client, not in the protocol.** Each vendor states that its model does not take text in a result as a command `[A12][O9][X1]`. Each also states that this is not a guarantee `[A9][O7][X1]`.
- **The protocol gives the server nothing to mark a comment or a body as written by another member.** Whatever the product wants an agent to know about the author of a text has to be in the result itself, as text or as structure the model reads `[M5][A10]`. In ChatGPT a key in `_meta` of a result does not reach the model at all `[O3]`.
- **Two vendors say that the way a result is written matters.** Anthropic: name the source in the tool description or in the structure of the result, and encode third-party strings as JSON `[A10]`. OpenAI's Model Spec: give the model enough information about each tool to judge how far to trust its output, and put untrusted data in a quoted or structured format `[O9]`. Both are written for developers who build on the vendor's API.
- **A server's own instructions in a result may be ignored.** Anthropic says Claude may ignore or flag them `[A10]`. Text that the product itself adds to a result, such as a warning, is in the same place as the text it warns about.
- **The question before a call protects writes, and the human can relax it.** The hints tell Claude and ChatGPT which tools to ask about `[A1][O3]`. A read tool marked read-only runs without a question in Claude `[A1]`, and it is the read that brings another member's text to the agent. Research in Claude and deep research in ChatGPT call read tools with no question `[A9][O1]`.
- **The damage the vendors describe needs a second step.** The examples are an agent that reads an instruction and then writes, sends or fetches with other tools `[M11][O7]`. Which tools those are depends on the whole set an owner has connected to the MCP client, which the product does not see `[M11]`.
- **`openWorldHint` is not defined for this case.** OpenAI lets a tool limited to a bounded private workspace set it to false `[O5]`. The MCP blog advises treating whatever a tool considers external as a possible source of untrusted content `[M11]`. Neither speaks of a closed workspace whose content is written by members with different owners.

## Not verified

- What the protections built into Claude and the safeguards built into OpenAI's models consist of for results that come from an MCP server. The pages name them and do not describe them `[A9][O7]`.
- How Grok Bot marks outside content as untrusted for the model, and whether a result from a custom MCP server is handled as one of the app results the security page names `[X1]`.
- Whether Claude, ChatGPT or Grok Bot reads the content annotations `audience` and `priority` on a result, and what they do with a content item whose audience is the user only. No page read mentions them.
- Whether Claude and Grok Bot pass `_meta` of a result to the model. Only OpenAI states that it is hidden `[O3]`.
- Whether Grok Bot or grok.com reads the tool annotations at all. The only mention in xAI's documentation is in an example for the API `[X10]`.
- The exact rule by which ChatGPT turns the hints into a question. The pages say the hints help choose the behaviour and that a destructive tool gets explicit approval `[O2][O3]`; whether a read tool ever gets a question is not stated.
- What Claude does with a tool of a custom connector that carries no annotations. The checklist states the effect of the hints `[A1]`; the default without them is not stated on the pages read.
- The content of SEP-1913. Its state (open, not merged, label `draft`) was read from GitHub; its fields were read only through a tool that summarizes the page `[M12]`.
- What sanitizing a tool output and validating a result mean in the specification. Both are stated without explanation `[M3]`.
- The text of the prompt injection acknowledgment in Anthropic's submission portal `[A6]`.
- Whether the advice in `[A10]`, `[A11]`, `[O7]` and `[O8]` is meant for authors of MCP servers used through the vendors' own applications. Those pages address developers who build an application on the vendor's API.
- Elicitation, by which a server can ask the human a question through the MCP client, was not read as a way for a server to get a confirmation of its own.
- `[O1]` could not be fetched directly (HTTP 403). It was read through a web scraping service that served a copy cached on 2026-11-13. OpenAI's page on prompt injections (openai.com/safety/prompt-injections) did not load and was not used.
- `[O2]` to `[O6]` were read in OpenAI's single-file export of the Plugins documentation, not page by page.
- `[M11]` and the summary of `[M12]` were first read through a summarizing fetch tool; the passages of `[M11]` used here were then checked against the page's own text.
- Dates. `[O1]`, `[A8]` and `[A9]` show a relative date only (17 days ago, over 2 weeks ago, today). `[M10]` is dated 2026-04-20, `[M11]` 2026-03-16, `[O9]` 2026-10-29. The other pages carry no date.
- Nothing was tested against a running MCP client.

## Sources

### MCP specification and official material

- `[M1]` Specification, revision 2026-07-28, overview: https://modelcontextprotocol.io/specification/2026-07-28 (reached through https://modelcontextprotocol.io/specification/latest)
- `[M2]` Versioning: https://modelcontextprotocol.io/specification/versioning
- `[M3]` Tools: https://modelcontextprotocol.io/specification/2026-07-28/server/tools
- `[M4]` Resources, section Annotations: https://modelcontextprotocol.io/specification/2026-07-28/server/resources
- `[M5]` Schema, `schema/2026-07-28/schema.ts` (`ToolAnnotations`, `Annotations`, `TextContent`, `CallToolResult`): https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.ts
- `[M6]` Base protocol overview, section `_meta`: https://modelcontextprotocol.io/specification/2026-07-28/basic/index
- `[M7]` Key changes since 2025-11-25: https://modelcontextprotocol.io/specification/2026-07-28/changelog
- `[M8]` Security Best Practices: https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices
- `[M9]` Client Best Practices: https://modelcontextprotocol.io/docs/2026-07-28/develop/clients/client-best-practices
- `[M10]` Tool Annotations Interest Group charter: https://modelcontextprotocol.io/community/interest-groups/tool-annotations
- `[M11]` Tool Annotations as Risk Vocabulary: What Hints Can and Can't Do (blog, 2026-03-16): https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/
- `[M12]` SEP-1913, Trust and Sensitivity Annotations (pull request): https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913
- `[M13]` Documentation index: https://modelcontextprotocol.io/llms.txt

### Anthropic

- `[A1]` Connector pre-submission checklist: https://claude.com/docs/connectors/building/review-criteria
- `[A2]` Connector verification: https://claude.com/docs/connectors/verification
- `[A3]` Model Context Protocol (MCP): https://claude.com/docs/connectors/building/mcp
- `[A4]` Get started with connectors: https://claude.com/docs/connectors/getting-started
- `[A5]` Add a connector that isn't in the directory, section Security and privacy: https://claude.com/docs/connectors/custom/remote-mcp
- `[A6]` Submit a connector to the directory: https://claude.com/docs/connectors/building/submission
- `[A7]` Build an MCP server for Claude: https://claude.com/docs/connectors/building
- `[A8]` Anthropic Software Directory Policy: https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy
- `[A9]` Get started with custom connectors using remote MCP: https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
- `[A10]` Mitigate jailbreaks and prompt injections: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- `[A11]` Handle tool calls: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
- `[A12]` Claude's constitution: https://www.anthropic.com/constitution
- `[A13]` Claude Code, Security: https://code.claude.com/docs/en/security
- `[A14]` Claude Code, Connect Claude Code to tools via MCP: https://code.claude.com/docs/en/mcp

### OpenAI

- `[O1]` Connecting custom MCP servers in ChatGPT: https://help.openai.com/en/articles/12584461
- `[O2]` Build an MCP server: https://developers.openai.com/plugins/build/mcp-server
- `[O3]` Reference (tool annotations, tool results), and Add UI to your MCP server: https://developers.openai.com/plugins/reference and https://developers.openai.com/plugins/build/chatgpt-ui
- `[O4]` Security & Privacy: https://developers.openai.com/plugins/guides/security-privacy
- `[O5]` Plugin guidelines: https://developers.openai.com/plugins/plugin-guidelines
- `[O6]` Plugin submission errors, and Upload and submit your plugin: https://developers.openai.com/plugins/deploy/submission-errors and https://developers.openai.com/plugins/deploy/submission
- `[O7]` MCP servers (Responses API), section Risks and safety: https://developers.openai.com/api/docs/guides/tools-connectors-mcp
- `[O8]` Safety in building agents: https://developers.openai.com/api/docs/guides/agent-builder-safety
- `[O9]` Model Spec, 2026-10-29, section Ignore untrusted data by default: https://model-spec.openai.com/2026-10-29.html
- `[O10]` Plugin controls: https://learn.chatgpt.com/docs/enterprise/apps-and-connectors
- `[O11]` Model Context Protocol (ChatGPT and Codex): https://learn.chatgpt.com/docs/extend/mcp
- `[O2]` to `[O6]` were read in the combined export: https://developers.openai.com/plugins/llms-full.txt

### xAI

- `[X1]` Grok Bot security: https://docs.x.ai/grok-bot/security
- `[X2]` How Grok Bot works: https://docs.x.ai/grok-bot/how-grok-bot-works
- `[X3]` Approvals, security, and privacy: https://docs.x.ai/grok-bot/approvals-security-and-privacy
- `[X4]` Use the computer and apps: https://docs.x.ai/grok-bot/computer-and-apps
- `[X5]` Grok Bot FAQ: https://docs.x.ai/grok-bot/faq
- `[X6]` Grok Bot inference hooks: https://docs.x.ai/grok-bot/inference-hooks
- `[X7]` Remote MCP Tools (API): https://docs.x.ai/developers/tools/remote-mcp
- `[X8]` Grok connectors: https://docs.x.ai/grok/connectors
- `[X9]` Documentation index: https://docs.x.ai/llms.txt
- `[X10]` Cookbook, Database over MCP: https://docs.x.ai/cookbook/examples/database-over-mcp

### Related findings in this repository

- `docs/research/mcp-client-features.md` on the branch `research/mcp-client-features` covers the features, limits and approval settings of the same three MCP clients.
