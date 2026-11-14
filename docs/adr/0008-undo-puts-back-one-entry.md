# An undo puts back one entry, as a change like any other

The first version needs a history that shows who did what, and a way to put content back. A human's changes leave the editor by themselves and are many and small, and an agent changes a body one passage at a time, so a single change is too small a thing to read or to undo. So an entry of the history of a body is an unbroken run of changes by one member: it ends when another member changes the body, after a pause, or when an agent acts through another connection. For anything that is not a body, an entry is a single change. The product keeps every change as it arrives, and the entry is what a member reads and undoes.

An undo puts back what one entry changed, as it was before that entry. It is a change like any other: it is made by the member who undoes, it is checked against the stretches it touches ([0005](0005-change-is-checked-against-the-stretch-it-touches.md)), and it is recorded as an entry of its own. Later changes to other stretches stay. Where someone has since changed a stretch of the entry, it is a conflict: an agent's undo is refused whole, and a human is shown those stretches and decides. A history only grows, and no member changes or removes an entry.

Decided in [What does the history record, and what can be undone?](https://github.com/TommyBez/agent-intelligence/issues/14). Nothing was run for this decision; it is reasoning only.

## Considered options

- **Every change is an entry.** Rejected because the history of a page a human wrote becomes hundreds of entries, for a human and for an agent that reads it, and undoing one of them puts back a few words.
- **Versions taken at intervals of time.** Rejected because a version taken while two members write holds the work of both, and no longer shows who did what.
- **No undo; a member puts the old text back with ordinary changes.** Rejected because repairing a page after ten wrong passages by an agent takes ten changes by hand. It would add nothing to build.
- **Restore the whole body as it was at a point in the history.** Rejected because it removes from the body what everyone wrote after that point, in stretches the undone change never touched.
- **Refuse the undo of a field whose value has changed since.** Rejected because a field that holds one value already takes the last change to arrive, and the history keeps the value that was replaced.

## Consequences

- A burst of changes by one agent to one body is one entry, so one undo repairs the page and leaves the work of others in place.
- An entry on a body can touch several stretches, and it is undone whole. One stretch changed since is enough for an agent's undo to be refused.
- An undo needs no rule of access of its own: whoever could make the change as an ordinary change can undo the entry. Any editor undoes an entry on a body, a title, a field of a task or the state of a thread, whoever made it; only its author undoes a change to the text of a comment.
- Undoing an entry on a field puts back the value from before, whatever the value is now. A task moved from to do to in progress and then to done goes back to to do when the first entry is undone.
- An undo gives notifications as any change does: one that puts a mention back gives its member a notification again, in the name of the member who undoes.
- An undo that puts both marks of an anchor back brings the anchor back, unless the thread has another anchor by then.
- How long the pause is that ends a run is not decided, and is left to the spec.
- Whether a page, a task or a comment can be deleted was not decided here; [0009](0009-deletion-destroys-nothing.md) decides that it can. The deletion is an entry and is undone like any other, and a change that arrives for something deleted is refused as in a conflict.
