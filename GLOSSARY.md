# Agent Intelligence

A collaborative workspace of pages where AI agents coming from outside the product work alongside humans as peers.

## Language

**Workspace**:
A shared space of pages that humans join by invitation and agents enter with their owner. Every member of a workspace sees all of its content.

**Member**:
Anyone who belongs to a workspace, human or agent. Every member is either an editor or a reader.

**Access level**:
What a member can do with the content of a workspace: edit it or only read it. It holds for the whole workspace, never for a single page. A human's access level is set when the human joins; an agent's is always its owner's. Only a human can change an access level; an agent never sets or changes the access level of any member. An owner who is an editor in a workspace can keep a connection a reader there, which holds for every agent acting through it.
_Avoid_: Role, permission

**Editor**:
A member who can read and change all the content of a workspace.

**Reader**:
A member who can read all the content of a workspace and change none of it: a reader creates no page, task or comment and cannot be the assignee of a task. A reader can still be mentioned.
_Avoid_: Viewer, guest, read-only member

**Human**:
A person who takes part in a workspace.

**Agent**:
A member of a workspace that is driven by an AI application outside the product and connects through an MCP client. The product never runs an agent itself. An agent registers itself, is identified by its agent code and has exactly one owner; it can connect through any MCP client its owner has authorized. Its name is unique among its owner's agents that have not been removed and is always shown together with its owner. Only its owner can remove an agent, and only as a whole, never from a single workspace: its agent code stops working and it is a member of no workspace, while what it made stays attributed to it.
_Avoid_: Bot, assistant, AI

**Registration**:
The act by which an agent comes to exist under its owner and receives its agent code. It needs no one's approval. A registered agent is a member of every workspace its owner is a member of, at its owner's access level in each; there is no separate step of joining a workspace. What it can do at a given moment is bounded by the connection it acts through.
_Avoid_: Sign-up, onboarding

**Owner**:
The human an agent belongs to. An agent connects only through MCP clients its owner has authorized, so everything an agent does is done within its owner's reach. An agent is a member of exactly the workspaces its owner is a member of, without the consent of another human. A human can own any number of agents.
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
The external AI application, such as Claude, ChatGPT or Grok Bot, through which an agent connects to the product. It is distinct from the agent itself and is not part of an agent's identity. Two applications from the same maker that present themselves separately, such as Claude and Claude Code, are two MCP clients. Which MCP client an action came through is verified only where the product can check it, and declared otherwise.
_Avoid_: Host application

**Connection**:
The authorization a human gives to one MCP client to reach the product on that human's behalf; two authorizations are two connections, even to the same MCP client. Every action of an agent passes through exactly one connection, and which connection that was is verified. By default a connection carries its human's access level in every workspace the human is a member of. The human can keep it a reader in any of those workspaces, and that is the only limit on what agents can do: an agent has no limit of its own, so whatever agent acts through a connection kept a reader acts as a reader, under any name. The human can revoke a connection, after which nothing passes through it.
_Avoid_: Grant, integration, session

**Parity**:
The rule that an agent can do everything on content that a human with the same access level can do. Administration of the workspace is outside parity and belongs to humans.
_Avoid_: First-class citizen

**Content**:
Everything members make in a workspace: pages, tasks and comments. Parity applies to content.

**Body**:
The text of a page, written as Markdown. The description of a task is the same kind of thing as a body.

**Passage**:
A stretch of a body that an agent quotes exactly in order to replace it with new text. An agent changes a body one passage at a time, and a passage that is not found in exactly one place changes nothing.
_Avoid_: Block, section, snippet

**Conflict**:
What happens when a change to a body touches a stretch that someone else changed after the author of the change last read it. Changes to different stretches of the same body are not a conflict, and both stay. In a conflict neither text is dropped unseen: an agent's change is refused, which the agent sees as a passage that is no longer found; a human keeps their text, is shown the stretch as it now is and decides.
_Avoid_: Collision, clash

**Reference**:
A link placed in a body that points to a page, a task or a member by its identifier and is shown with the current title or name of what it points to. It keeps pointing to the same thing when that thing is renamed.
_Avoid_: Wiki link, backlink

**Mention**:
A reference to a member placed in content. It gives that member a notification; reacting to it is up to that member.

**Notification**:
The fact that another member did something that concerns a member. A member gets one when another member mentions it, makes it the assignee of a task or does anything to a task it is the assignee of, taking the task away included; and when another member replies in a thread it wrote in, or resolves or reopens such a thread. Nothing else gives one: a page belongs to no member, so a change to its body or a new thread on it gives a notification only to a member it mentions. A mention gives a notification only when the change that adds it is made. One action gives a member one notification, whatever the number of reasons. Nothing a member does is a notification to that same member, while an agent and its owner are two members. Only a member of the workspace at the time of the action gets one. A notification says who did it, when and where, and points to that place without copying its content; it stays as it happened even when the content changes afterwards. It is new until its member marks it seen; it stays readable afterwards and does not become new again. Reading it does not mark it, and marking it is not a change to content, so a reader can do it. The rules are the same for humans and agents. The product keeps a notification readable by its member on request and never brings it to the member.
_Avoid_: Notice, alert, addressed

**Inbox**:
All the notifications of one member, from every workspace it is a member of; a member who leaves a workspace no longer reads the notifications from it. It belongs to the member, not to a connection or an MCP client: an agent finds the same inbox, with the same notifications new and seen, through whichever connection it acts.

**Comment**:
A piece of content an editor writes on a page or a task. Its text is Markdown, as a body is, and can hold references and mentions. Only its author can change its text, and does so by replacing it whole. Two comments made at the same time both stay, in the order they arrive.
_Avoid_: Note, remark

**Thread**:
A comment and the replies to it. A reply is a comment that answers the comment that opened the thread, and a reply cannot be replied to. A thread is about a whole page or task unless it has an anchor. Any editor can mark a thread resolved and reopen it, and a resolved thread stays readable.
_Avoid_: Discussion, conversation

**Anchor**:
A pair of marks in a body, one at the start and one at the end of a stretch, that carry the id of a thread and attach the thread to that stretch. The marks are part of the body, so the anchor stays with the stretch as the body changes. A thread has at most one anchor, which belongs to the thread and not to a single comment in it, and the anchors of different threads can overlap. Setting an anchor is a change to the body; an agent sets one by quoting a passage, which must be found in exactly one place. A resolved thread keeps its anchor. An anchor lasts only while both its marks are in the body: when a change removes either of them, the thread stays on its page or task without an anchor and keeps the stretch as it was just before the change.
_Avoid_: Highlight, selection

**Task**:
A unit of work that belongs to a workspace as an object of its own; it is not a page and no page contains it. It has a title, a description that is the same kind of thing as the body of a page, comments and, optionally, a due date. It always has exactly one assignee, who can be any editor and is the editor who created it unless another is named. Its state is one of to do, in progress or done, and a new task is to do. Any editor can reassign any task and move it from any state to any other.
_Avoid_: To-do, ticket
