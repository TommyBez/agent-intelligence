# A notification is seen only when its member marks it

The product never wakes an agent: an agent reads its inbox on request, in a conversation that can end at any point. An agent has no memory the product can count on, and its identity is declared ([0002](0002-agent-identity-is-declared.md)), so it can take its work up again through another connection or another MCP client. So the product keeps, for each member, which notifications are new and which are seen, and reading the inbox changes nothing: a notification becomes seen only when its member marks it. A notification that was read and not acted on is still new for whoever takes the work up again.

Decided in [What is addressed to a member, and how does an agent read it?](https://github.com/TommyBez/agent-intelligence/issues/10). Nothing was run and no agent was connected, so whether a real agent marks a notification after acting on it is not known.

## Considered options

- **Reading the inbox marks what was read as seen**, as most inboxes do. Rejected because a conversation that ends between the reading and the action loses the notification, and no one knows it was lost.
- **The agent keeps its own place and asks for what came after it.** Rejected because the product cannot count on an agent's memory, and an agent that comes back through another MCP client has none.
- **New and seen kept for each connection or MCP client.** Rejected because an agent is not tied to a connection: the same notification would be new again everywhere else.

## Consequences

- An agent that never marks reads the same notifications as new every time. Marking is left to the agent.
- Marking is not a change to content. A reader can mark, and so can an agent acting through a connection kept a reader ([0003](0003-access-limit-sits-on-the-connection.md)).
- Any connection of an owner can read and mark the inbox of any of the owner's agents by presenting its agent code. Seen means that a notification was marked, not that the agent it was meant for read it.
- A notification marked seen does not become new again.
- A human has the same state, and opening a notification in the interface is how a human marks it. What the inbox looks like for a human is not decided.
- Whether the history records that a notification was marked is not decided.
