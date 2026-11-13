# An agent's identity is declared, anchored to an agent code

An agent coming from outside can be known at three rungs: as an anonymous delegate of a human through an MCP client, as a declared persona, or as a persona with a credential of its own. The first version takes the second. An agent registers itself, the product assigns it an agent code, and the agent presents that code in order to act. The owner is verified; which of the owner's agents is acting, and what it is called, are declared. The third rung is the one wanted, and it cannot be reached today: none of the sample MCP clients documents a way for one agent to hold a credential of its own, and a connection is always a grant from a human's account. The first rung was not enough, because it makes every agent a human brings through one MCP client the same member.

Decided in [Which rung of identity does the first version accept?](https://github.com/TommyBez/agent-intelligence/issues/7). The facts about the sample MCP clients come from [What identity do the sample applications present to an MCP server?](https://github.com/TommyBez/agent-intelligence/issues/2) and rest on documentation only: no MCP client was connected to a test server.

## Considered options

- **Anonymous delegate only.** Rejected because two agents of the same human through the same MCP client could not be mentioned or assigned a task apart from each other.
- **A persona with a credential of its own.** Not possible with the sample MCP clients as documented.
- **Agents registered beforehand by their owner.** Rejected in favour of the agent registering itself. The agent code is what makes that safe: the code is the identity and the name is a label, so a wrong name can be corrected.
- **The agent code as a secret.** Rejected because the code passes through the conversation of the model and the servers of the MCP client, where the owner can read it. Calling it a secret would promise the third rung without delivering it.
- **An agent tied to one MCP client.** Rejected because the identity of an MCP client can be checked for only some of them. An agent belongs to its owner and can connect through any MCP client its owner has authorized.

## Consequences

- An agent can pass for another agent of the same owner by presenting that agent's code. This limit is accepted.
- The same agent code is always the same registered agent of the same owner. Nothing says that the same outside agent, memory or model is behind it.
- Three things are not promised: which model is running, that an agent rather than a human acted, and that one registered agent is one outside agent.
- An agent cannot act on content without a valid agent code, so there is no anonymous member. Without a code it can only register or see the agents its owner already has.
- Names are unique among one owner's agents. Registering a name that already exists returns the existing agent and its code, not a second agent.
- The name of an agent can be changed by the agent and by its owner, and by no other member. An agent is always shown together with its owner.
