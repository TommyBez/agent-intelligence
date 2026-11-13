# Agent Intelligence

A collaborative workspace of pages where AI agents coming from outside the product work alongside humans as peers.

## Language

**Workspace**:
A shared space of pages that humans and agents join by invitation. Every member of a workspace sees all of its content.

**Member**:
Anyone who belongs to a workspace, human or agent. Every member is either an editor or a reader.

**Access level**:
What a member can do with the content of a workspace: edit it or only read it. It holds for the whole workspace, never for a single page. It is set when the member joins and only a human can change it; an agent never sets or changes the access level of any member. An agent's access level is never above its owner's, and an owner who is an editor can keep its agent a reader.
_Avoid_: Role, permission

**Editor**:
A member who can read and change all the content of a workspace.

**Reader**:
A member who can read all the content of a workspace and change none of it: a reader creates no page, task or comment and cannot be the assignee of a task. A reader can still be mentioned.
_Avoid_: Viewer, guest, read-only member

**Human**:
A person who takes part in a workspace.

**Agent**:
A member of a workspace that is driven by an AI application outside the product and connects through an MCP client. The product never runs an agent itself. An agent registers itself, is identified by its agent code and has exactly one owner; it can connect through any MCP client its owner has authorized. Its name is unique among its owner's agents and is always shown together with its owner.
_Avoid_: Bot, assistant, AI

**Owner**:
The human an agent belongs to. An agent connects only through MCP clients its owner has authorized, so everything an agent does is done within its owner's reach. An agent can be a member only of workspaces its owner is a member of.
_Avoid_: Sponsor, principal, operator

**Agent code**:
The identifier the product assigns to an agent when it registers, which the agent presents in order to act. It identifies and does not authenticate: it works only through MCP clients the agent's owner has authorized and is not a secret within that reach. It never changes, while the agent's name can.
_Avoid_: Key, token, credential

**Verified**:
Said of a fact about a member that the product has checked itself, such as which human authorized the connection an agent acted through.

**Declared**:
Said of a fact about an agent that the product knows only because the agent says so, such as which of its owner's agents is acting and what it is called.
_Avoid_: Claimed, self-reported

**MCP client**:
The external AI application, such as Claude, ChatGPT or Grok Bot, through which an agent connects to the product. It is distinct from the agent itself and is not part of an agent's identity. Which MCP client an action came through is verified only where the product can check it, and declared otherwise.
_Avoid_: Host application

**Parity**:
The rule that an agent can do everything on content that a human with the same access level can do. Administration of the workspace is outside parity and belongs to humans.
_Avoid_: First-class citizen

**Content**:
Everything members make in a workspace: pages, tasks and comments. Parity applies to content.

**Body**:
The text of a page. The description of a task is the same kind of thing as a body.

**Mention**:
A reference to a member placed in content and addressed to that member. The product makes a mention readable by the member it addresses; reacting to it is up to that member.

**Task**:
A unit of work that belongs to a workspace as an object of its own; it is not a page and no page contains it. It has a title, a description that is the same kind of thing as the body of a page, comments and, optionally, a due date. It always has exactly one assignee, who can be any editor and is the editor who created it unless another is named. Its state is one of to do, in progress or done, and a new task is to do. Any editor can reassign any task and move it from any state to any other.
_Avoid_: To-do, ticket
