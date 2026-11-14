# A deletion takes content out of the workspace and destroys nothing

The first version needs a way to take wrong content out of a workspace: an agent can create twelve pages by mistake. [0008](0008-undo-puts-back-one-entry.md) already says that a deletion is an entry of the history and is undone like any other. So deleting a page, a task or a comment takes it out of the content of the workspace and keeps it, with its history, without a limit of time. No member can destroy it.

Any editor deletes any page or task, whoever created it: an editor can already change every body and every task, and a page belongs to no member. A deleted page or task accepts no change other than the undo of its deletion, and its comments and threads go with it and come back with it. Every member can list the deleted pages and tasks of a workspace and read each one as it was, with its history.

Only its author deletes a comment, which is the rule for changing its text. When the comment that opened a thread has replies, the thread stays with its replies and its anchor. When the last comment of a thread is deleted, the thread is no longer shown, and the marks of its anchor are taken out of the body in the same act.

Decided in [Can a page, a task or a comment be deleted, and by whom?](https://github.com/TommyBez/agent-intelligence/issues/16). Nothing was run for this decision; it is reasoning only.

## Considered options

- **No deletion in the first version.** Rejected because wrong pages and tasks stay among the others and can only be emptied.
- **Only the member who created a page or a task deletes it.** Rejected because a page belongs to no member and any editor can already change all of it. Another editor would have to ask the creator, or the creator's agent.
- **Destroy what is deleted after a time, or let a member destroy it.** Rejected because a history only grows, and an undo would stop working at a moment no member chose. Destroying content for good would be administration of the workspace, not a change to content.
- **Remove the references that point to a deleted page or task.** Rejected because it is a change to every body that holds one, each with its own entry and its possible conflicts.
- **Any editor deletes any comment.** Rejected because only its author changes the text of a comment, and deleting it would be a way around that rule.
- **An owner deletes the comments of the owner's agents.** Rejected because an agent and its owner are two members, and it would be a rule of access for one case only.
- **Deleting the comment that opened a thread deletes the whole thread.** Rejected because its author would delete the replies of others.
- **Refuse to delete a comment that has replies.** Rejected because one reply by another member would block the author.
- **Leave the marks of an anchor in the body when the last comment of its thread is deleted.** Rejected because the body would keep the marks of a thread that has no comment left and is no longer shown.

## Consequences

- The text of a deleted comment stays readable in the history by every member. A member who puts a secret in a comment by mistake cannot make it go away.
- The comment of an agent that has been removed, or of a member who has left the workspace, can be deleted by no one. The others can only resolve its thread.
- An author who has become a reader cannot delete their own comment, because a reader changes no content.
- A reference to a deleted page or task stays in the body as it is and is shown as deleted. No body changes, and writing a new reference to something deleted is not refused.
- A deletion gives a notification only by the rules that already hold: to the assignee of a task that another member deletes, or whose deletion another member undoes. It adds and removes no mention, so a page that comes back gives no notification to the members it mentions.
- A thread whose opening comment is deleted is a thread like any other. A reply that arrives while that comment is being deleted is accepted, so the thread is the same whichever arrives first.
- Deleting the last comment of a thread that has an anchor is the one case in which deleting a comment changes a body. The marks are taken out where they are at that moment, so the deletion cannot be a conflict, and it is recorded in the entry of the deletion.
- Only its author undoes the deletion of a comment, also when the undo puts marks back in a body. That undo is never refused: the comment and the thread always come back, and the anchor comes back only if the stretch is still as it was.
- A deleted comment is found in the history of its page or task, not in the list of deleted pages and tasks.
- How something deleted is shown to a human, and a list of deleted pages and tasks longer than an MCP client accepts in one result, are not decided.
