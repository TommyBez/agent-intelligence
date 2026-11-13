# An anchor is a pair of marks in the body

A thread can be attached to one stretch of a body, and it has to stay attached while humans and agents change that body. A body is Markdown text that an agent reads whole and changes one passage at a time ([0004](0004-agent-changes-a-passage.md)). So the first version writes the anchor in the body itself: two marks, one at the start and one at the end of the stretch, each carrying the id of the thread. The marks are part of the text, so the anchor stays with the stretch through every change and nothing is recomputed. An agent sees what is commented in the one read it already makes, and an agent that rewrites a stretch decides whether the anchor goes into the new text, by writing the marks or leaving them out.

An anchor lasts only while both marks are in the body. A change that removes either of them is accepted, and the thread stays on its page or task without an anchor, keeping the stretch as it was just before the change.

Decided in [What is a comment, and what can it be attached to?](https://github.com/TommyBez/agent-intelligence/issues/13), on [findings](https://github.com/TommyBez/agent-intelligence/blob/research/comment-anchors/docs/research/comment-anchors.md) about how other applications keep a comment attached to a part of a text. The findings were read in primary sources. Nothing was run and no agent was connected, so whether a real agent carries the marks in a passage it quotes is not known.

## Considered options

- **A comment attached to the whole page or task, carrying a quote of the part it is about.** Rejected because the quote does not follow the text: once the text changes, the comment no longer points at anything.
- **A position kept outside the body, which the product moves at every change**, as in Google Docs. Rejected because when an agent replaces a passage that contains an anchored stretch, the product knows only the old text and the new, and has to guess where the anchor went. An agent would also not see what is commented in the body it reads. It would have kept the body free of marks, and no writer could have damaged an anchor.
- **A quote of the stretch, found again by searching**, as in Hypothesis. Rejected because the search can fail or match in several places.
- **One mark written as a link around the stretch**, as a reference is. Rejected because in CommonMark a link cannot contain another link: it could not enclose a stretch that holds a reference or a mention, and two anchors could not overlap.
- **Refusing a change that removes a mark.** Rejected because it gives an agent a second kind of refusal and lets a comment block a rewrite.

## Consequences

- This corrects a consequence recorded in 0004. A part of a body has an identifier where a thread is anchored, and that identifier is made when the anchor is set. Blocks with ids set in advance stay rejected.
- Setting an anchor is a change to the body. It follows the rule of every other change ([0005](0005-change-is-checked-against-the-stretch-it-touches.md)) and has no rule of its own. Whether the history shows it is not decided.
- An agent must reproduce the marks exactly in a passage that crosses them. An anchor set after an agent read the body makes a passage that crosses it no longer found, until the agent reads the body again. 0004 records the same for a reference whose target is renamed.
- An anchor can be lost without anyone meaning to: an agent that rewrites a stretch and leaves the marks out removes it. The thread is kept, without an anchor.
- Replacing a whole body is not part of the form (0004), so a change never touches the marks outside the passage it quotes. A whole-body replacement is what made Atlassian's MCP server drop the markers of Confluence's inline comments.
- Resolving a thread does not change the body, and the marks of a resolved thread stay in it.
- A thread that loses its anchor keeps the stretch as it was just before the change. Most of the applications read keep the copy made when the thread was created.
- The editor for humans has to show an anchor as something other than its marks, and keep the marks when a human edits around them. How it does that is not decided.
- The characters a mark is written with are left to the spec.
