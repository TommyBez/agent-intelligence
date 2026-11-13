# A task is an object of its own, not a page

Notion is the benchmark for what a task carries, and in Notion a task is a page inside a database. Here a task is an object of its own in the workspace: it is not a page, no page contains it, and it does not turn into one. Two reasons decided it. Notion-style databases are out of the first version, and a page with an assignee and a state is the first piece of one. Agents work through MCP tools only, so "which tasks are assigned to me" needs an answer that does not depend on reading pages.

Decided in [What is a task, and where does it live?](https://github.com/TommyBez/agent-intelligence/issues/6).

## Considered options

- **A page with an assignee and a state**, as in Notion. Rejected because it gives a page properties, which is where databases begin, and because listing tasks and listing pages would overlap.
- **A part of the body of a page.** Rejected because finding a task would mean reading bodies, an edit to a body could remove a task, and a task would depend on the form in which an agent reads and writes a page.

## Consequences

- The description of a task is the same kind of thing as the body of a page. Whatever is decided for a body, such as the form in which an agent reads and writes it and how simultaneous edits are handled, has to hold for both.
- A task does not appear among pages and has no sub-pages. It relates to pages only through references written in content.
