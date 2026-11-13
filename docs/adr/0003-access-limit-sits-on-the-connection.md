# An agent's access is its owner's, and the only limit sits on the connection

An agent is a declared persona: any agent of an owner can present the agent code of another, and a new name can be registered at any time. A limit placed on an agent would hold only as long as that owner's agents respected it. So the first version puts neither an access level of its own nor a joining step on the agent: a registered agent is a member of every workspace its owner is a member of, at its owner's access level, and the one limit an owner can set is to keep a connection a reader in a workspace. The connection is verified, because the product itself issues the authorization, so the limit holds for whatever agent acts through it, under any name.

Decided in [How does an agent join a workspace?](https://github.com/TommyBez/agent-intelligence/issues/8). It replaces one point of [What is the unit of access: the whole workspace or a single page?](https://github.com/TommyBez/agent-intelligence/issues/5), where an owner kept the agent itself a reader. That the product can tell connections apart rests on documentation only: no MCP client was connected to a test server.

## Considered options

- **A limit on the agent.** Rejected because it is declared: an agent kept a reader escapes it by registering under another name or by presenting the code of another agent of the same owner.
- **A limit on the agent and one on the connection.** Rejected because the one on the agent adds nothing that is verified.
- **An agent joins each workspace by an act of its owner.** Rejected in favour of membership that follows the owner. An agent never reaches further than its owner's connection does, so a joining step would guard nothing.
- **The owner approves each registration.** Rejected because a new name gains nothing. The approval that counts is authorizing the connection.
- **An agent tied to the connection it registered through.** Rejected because it reopens [ADR 0002](0002-agent-identity-is-declared.md), where an agent belongs to its owner and not to an MCP client.
- **One setting for the whole connection.** Rejected in favour of a setting per workspace, because an access level is defined per workspace.

## Consequences

- The same agent only reads through one connection and can write through another. As a member it counts at its owner's access level, so it can be the assignee of a task it cannot move forward from a connection kept a reader.
- Removing an agent tidies and does not protect: the name becomes free and an agent under another name is a member by default. Protection is keeping a connection a reader or revoking it.
- An agent cannot be removed from a single workspace. Whoever administers a workspace acts on the owner, not on one of the owner's agents.
- A limit or a revocation takes effect at once only if the product checks the connection on every request. With the defaults Better Auth documents, an authorization already issued can stay valid for up to an hour. This was not tested.
