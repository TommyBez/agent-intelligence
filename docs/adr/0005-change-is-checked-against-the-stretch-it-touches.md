# A change is checked against the stretch it touches

A human and an agent can change the same body at the same time, and neither may lose work; seeing others type live is not required. So the first version checks a change only against the stretch of the body it touches, and the rule is the same for humans and agents. A change counts for the stretches it changes and is applied to the body as it is when the change arrives, so what others changed elsewhere stays. Only a conflict stops a change: a change that touches a stretch someone else changed after its author last read it.

In a conflict neither text is dropped unseen. An agent's change is refused, nothing is changed, and the agent reads the body again. A human's change is not applied to that stretch: the human keeps their text, is shown the stretch as it now is and decides, while the rest of the change is applied. A human's change leaves the editor by itself shortly after the human stops writing, and a page a human has open takes in others' changes by itself, so the copy a human works from is seconds old and a conflict covers a few words.

Decided in [How are simultaneous edits kept from losing work?](https://github.com/TommyBez/agent-intelligence/issues/11). Nothing was run for this decision. The agent's side rests on the simulation recorded in [0004](0004-agent-changes-a-passage.md); the human's side is reasoning only.

## Considered options

- **Refuse any change made from an older version of the page.** Rejected because a change is then refused for something that happened in a part it does not touch. An agent would read the whole body again after every change by anyone else, and a human would redo work that nothing had touched.
- **The last change to arrive wins.** Rejected for a body because it loses work without anyone seeing it. It is kept for a field that holds one value, where there is nothing to merge.
- **In a conflict the human's text wins and the other is left to the history.** Rejected because the other text leaves the page without a human having seen it, and because it rests on what the history keeps, which is not decided.
- **In a conflict the text already in the body wins and the human's is discarded.** Rejected because it loses the human's work.
- **The refusal given to an agent carries the current body.** Rejected because every refusal then weighs as much as the whole page, and only Claude publishes a limit on the size of a result. It would save the agent one call.
- **The human saves with a button.** Rejected because the copy a human works from can then be an hour old, and a conflict covers whole paragraphs.

Not weighed: a structure built for writing shared live, which merges every keystroke. The map rules seeing others type live out of scope, so it was not compared with this decision.

## Consequences

- Two changes that are each accepted can leave a page that says two different things. A human changes the launch date in the first paragraph from March to April; an agent that read the page before then replaces a passage in the third paragraph, written with March in mind. Both changes stay, and the product does not notice.
- Two changes to different words of the same paragraph are not a conflict, for humans as for agents.
- An agent sees a conflict as a passage that is no longer found. It cannot tell that from a passage it quoted wrongly.
- A field that holds one value takes the last change to arrive: the title of a page, and the title, state, assignee and due date of a task.
- The editor for humans has to send a change by itself, know which stretches the human changed, take in others' changes while the page is open and show a conflict. How it does any of that is not decided.
- A page a human has open shows each change by someone else whole, after it is applied, with no cursor and no sign of who is present. That is what keeps this short of seeing others type live.
