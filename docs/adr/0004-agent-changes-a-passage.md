# An agent changes a body one passage at a time

An agent works from a copy of a body that can be out of date by the time its change arrives, because a human or another agent may have changed the page in between. So the first version has an agent read a body whole, as Markdown text, and change it by quoting one passage and giving its replacement. The change is applied to the body as it is at that moment, and it is refused, with nothing changed, unless the quoted passage is found in exactly one place. A change then touches only the words it quotes, and a change that no longer fits is refused instead of overwriting what others did.

Decided in [In what form does an agent read and write a page?](https://github.com/TommyBez/agent-intelligence/issues/9), on a [prototype](https://github.com/TommyBez/agent-intelligence/blob/prototype/page-exchange-form/docs/prototypes/page-exchange-form.html) that simulated three forms. No MCP client was connected and no agent was run, so whether a real agent quotes a passage exactly enough for its change to be accepted is not known.

## Considered options

- **The agent sends the whole body back.** Rejected because, from an older copy, it silently undoes whatever changed in between. In the prototype it lost a human's change to another paragraph, lost a rewording of the same paragraph, and brought back a paragraph the human had deleted. It also sends the whole body for any change: 54,711 characters to fix one word on the long page tried, against 78.
- **Blocks with ids.** Rejected because the unit of a change becomes the block. Two changes inside the same block overwrite each other, where two passages in the same paragraph do not. It also needs an operation for each thing done to a block (update, insert, delete, move, split, merge), and it makes what a block is part of what every agent depends on. The need for those operations is reasoning: the prototype had only update and insert.

## Consequences

- No part of a body has an identifier set in advance. Whatever has to point at a part of a body has to bring its own: a thread does it with an anchor, a pair of marks written in the body when the thread is attached ([0006](0006-anchor-is-a-pair-of-marks-in-the-body.md)). Blocks would have given every part an identifier, and that was given up knowingly.
- Replacing a whole body in one call is not part of the form. Text is added by replacing a passage with itself followed by the new text, and a large rewrite is a passage as large as the rewrite.
- An agent must reproduce a passage exactly as it read it. A reference is handed out with the current title of what it points to, so in the prototype a passage that contains a reference stops matching once that thing is renamed, until the agent reads the body again.
- The form already keeps some simultaneous edits from losing work: what others changed outside the quoted passage stays. Whether that is enough is left to [How are simultaneous edits kept from losing work?](https://github.com/TommyBez/agent-intelligence/issues/11).
- An agent reads a body whole, so a body longer than an MCP client accepts in one result cannot be read this way. Only Claude publishes such a limit. What happens then is not decided.
