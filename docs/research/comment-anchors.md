# How other applications keep a comment attached to a part of a text

Research for issue #15. All sources were read on **2026-11-13**. Only primary sources were used: vendor documentation, official API references and machine-readable API descriptions, specifications, and the projects' own repositories. Every claim carries a source tag such as `[N1]`; the tags resolve in [Sources](#sources).

Status marks: **V** = verified, read in a primary source on the date above. **NV** = not verified; the reason is given in [Not verified](#not-verified).

Terms follow `GLOSSARY.md`: body, passage, reference, mention, agent, MCP client. "Thread" and "anchor" are used as issue #13 uses them: a thread can have an anchor, which attaches it to one stretch of a body. "Stretch" is used for the anchored text in every application, whatever the application calls it.

## Short answer

1. **Where the anchor is kept.** Three ways were found.
   - *A mark in the content that carries an id.* Word (range markers and a reference mark, the comment text in a separate part) `[W1][W2][W3]`, Confluence (a marker element in the storage format, an annotation mark in its JSON document format) `[C2][C3]`, Tiptap `[E6]`, Lexical `[E9]`. Notion shows its anchors to programs this way: in the Markdown it exchanges with agents the stretch is wrapped in a tag that lists discussion URLs `[N5]`. CriticMarkup also writes in the text, but it writes the comment itself there, with no id and no thread `[M1]`.
   - *A position kept beside the content, which the application moves at every change or ties to the history of edits.* Google Docs (a map of anchors with index ranges, beside the text) `[G4]`, the ProseMirror collaboration demo `[E3]`, Yjs relative positions `[E8]`, GitHub review comments (a line of a file at a commit) `[H1][H3]`.
   - *A quote of the stretch, found again by searching.* Hypothesis and the W3C Web Annotation model `[A1][A3]`. It is also how a program outside names a stretch when it creates an anchor in Confluence (the text and which occurrence) `[C1]`, and how Notion's suggested edits name theirs (text that must appear exactly once) `[N7]`.
2. **Following an edit.** A mark is part of the content, so it moves with the stretch and nothing is recomputed; it is lost when the stretch is written again without it. Atlassian states that replacing a whole body from Markdown drops the markers `[C6]`, and that cutting and pasting commented text in the Confluence editor counts as deleting it `[C3]`. A position outside the content has to be mapped through every change `[E1][E2]`; Google warns that an anchor's position "cannot be guaranteed between revisions" for anchors set through the Drive API `[G1]`, and the W3C model calls a position selector "very brittle" `[A1]`. A quote is found again approximately, and the search can fail `[A4][A6]`.
3. **When the stretch is removed.** Where the behaviour is documented, the thread is kept and marked as detached: "dangling" or resolved in Confluence `[C1][C3]`, "outdated" on GitHub `[H3]`, "orphan", now "unanchored", in Hypothesis `[A6][A7]`. Tiptap is the exception by default: it soft-deletes the thread, and offers a setting to keep it `[E5]`. Word and Google Docs are not documented on this point. Nearly every application also keeps a copy of the stretch as it was when the thread was made `[G2][G4][C1][H1][A1][E10]`.
4. **What a program outside sees.** Mostly a thread with a copy of the quoted text, and not always a usable position. The Drive API returns an opaque anchor string for a Google Doc `[G1]`; the Docs API returns index ranges and the quote `[G4]`. Notion's REST API returns the block and the discussion id, never the range, and cannot start a thread on selected text `[N1][N2]`; its Markdown for agents carries the mark inline `[N5]`. Confluence returns a marker reference and the original selection `[C1]`. GitHub returns line numbers, omitted when the comment is outdated `[H5]`. No application was found to document a Markdown export that keeps anchors: Notion documents comments for its HTML export only `[N10]`.
5. **Overlap and resolving.** Overlap is explicit in Google Docs (several threads on one anchor, several anchors on one location) `[G4]`, in Notion's Markdown (one tag lists several discussions) `[N5]` and in Lexical (one mark node holds several ids) `[E9]`. In ProseMirror two marks of one type exclude each other unless the schema says otherwise `[E2]`. Resolving is stored on the thread everywhere it is documented; the anchor is kept in Tiptap `[E5]` and its highlight is removed in Confluence `[C4]`.

## Comparison

Where the anchor is kept and how it behaves:

| | Where the anchor is kept | What is stored | How it follows an edit | When the stretch is removed |
| - | - | - | - | - |
| Notion | Internal storage **NV**. To programs: a tag around the stretch in Notion-flavored Markdown **V** `[N5]` | `<span discussion-urls="url,url">text</span>` **V** `[N5]`; in REST only a block id and a discussion id **V** `[N2]` | **NV** | Comment object has `original_content_deleted` **V** `[N3]`; its meaning is not explained **NV** |
| Google Docs | Beside the text: a `commentAnchors` map on the document tab **V** `[G4]` | Anchor id with index ranges; the thread holds the anchor id and the quote from creation **V** `[G4]` | Mechanism **NV**. Anchors set through the Drive API are immutable and not guaranteed between revisions **V** `[G1]` | **NV** |
| Word / OOXML | In the content **V** `[W1][W2]` | `commentRangeStart` and `commentRangeEnd` with an id, a `commentReference`; comment text in the Comments part **V** `[W1][W2][W3]` | Markers are elements of the content **V** `[W1]`; Word's own behaviour **NV** | **NV** |
| Confluence | In the content **V** `[C2][C3]` | Storage format: `ac:inline-comment-marker` tags around the stretch **V** `[C3]`. JSON format: `annotation` mark with `id`, type `inlineComment` **V** `[C2]` | Travels with the text when both tags are moved; a cut and paste in the editor counts as deletion (Data Center) **V** `[C3]` | Thread kept: resolved (Data Center) **V** `[C3]`; status `dangling` exists in the Cloud API **V** `[C1]` |
| GitHub review comments | Beside the content **V** `[H1]` | File path, commit, line or line range, side of the diff, the diff hunk; original values kept too **V** `[H1][H2]` | `line` is the current line, `originalLine` the line at creation **V** `[H3]`; how it is recomputed **NV** | Thread kept, flagged outdated **V** `[H3][H4]` |
| Hypothesis / W3C | Outside the document entirely **V** `[A1][A6]` | Several selectors for one stretch: range, position, quote with prefix and suffix **V** `[A1][A2][A3]` | Not followed: found again on each load, exact first, then approximate **V** `[A3][A4]` | Annotation kept as orphan (unanchored), quote shown struck through **V** `[A6][A7]` |
| ProseMirror (collab demo) | Beside the document, in plugin state **V** `[E3]` | Id, text, `from`, `to` **V** `[E3]` | Mapped through every transaction **V** `[E2][E3]` | The mapping can drop the range and report it **V** `[E2]` |
| Tiptap Comments | In the document **V** `[E6]` | `inlineThread` mark with `data-thread-id`; thread data in the collaboration document **V** `[E4][E6]` | Mark in the content **V** `[E6]` | Thread soft-deleted by default; configurable **V** `[E5]` |
| Yjs relative position | Wherever the application puts it **V** `[E8]` | A position attached to an element of the shared text **V** `[E8]` | Stays attached to that element **V** `[E8]` | **NV** |
| Lexical | In the editor state **V** `[E9]` | `MarkNode` holding a list of ids; the playground keeps threads, with a quote, in a separate store **V** `[E9][E10]` | Node in the content **V** `[E9]` | Playground: thread stays listed with its quote **V** `[E10]` |
| CriticMarkup | In the text **V** `[M1]` | `{==stretch==}{>>comment<<}`: the comment text itself, no id **V** `[M1]` | Plain text **V** `[M1]` | Not addressed **V** `[M1]` |

What is exposed, and the two side questions:

| | What a program outside sees | Overlapping stretches | Resolving |
| - | - | - | - |
| Notion | REST: block id and discussion id, no range; cannot start a thread on selected text **V** `[N1][N2]`. Markdown and MCP: the tag with discussion URLs **V** `[N5]`; MCP lists inline discussions, resolved ones included **V** `[N7]`. Markdown export: comments not mentioned **V** `[N10]` | One tag lists several URLs **V** `[N5]` | REST returns open comments only **V** `[N1]`; anchor **NV** |
| Google Docs | Drive API: opaque `anchor` string and `quotedFileContent` **V** `[G1][G2]`. Docs API: index ranges and `plainTextQuote` **V** `[G4]`. Markdown export **NV** | Several threads per anchor, several anchors per location **V** `[G4]` | Flag on the thread **V** `[G2][G4]`; anchor **NV** |
| Word / OOXML | The markers, to anything that reads the file **V** `[W5]`; Word JS API gives the range **V** `[W6]`; Pandoc turns them into spans on request **V** `[M4]` | **NV** | `done` in a separate part **V** `[W4]`; markers **NV** |
| Confluence | API: `inlineMarkerRef`, `inlineOriginalSelection`; to create, the text and which occurrence **V** `[C1]`. MCP: comment tools **V** `[C5]`; markers dropped by a whole-body Markdown write **V** `[C6]` | **NV** | Highlight removed **V** `[C4]`; marker in the stored body **NV** |
| GitHub review comments | REST and GraphQL: lines, commit, diff hunk **V** `[H1][H3]`. MCP: current lines omitted when outdated **V** `[H5]` | **NV** | `isResolved`, separate from `isOutdated` **V** `[H3]` |
| Hypothesis / W3C | The annotation with its selectors **V** `[A1]`; Hypothesis API **NV** | **NV** | Not part of the model read **NV** |
| ProseMirror | Whatever the application sends; the demo sends `from` and `to` **V** `[E3]` | Decorations are independent **V** `[E3]`; marks of one type exclude each other by default **V** `[E2]` | Not applicable |
| Tiptap Comments | REST API for threads; the anchor only by writing the mark into the document JSON **V** `[E6][E7]` | **NV** | Anchors kept **V** `[E5]` |
| Lexical | Serialized node with `ids`; excluded when content is copied as HTML **V** `[E9]` | List of ids on one node **V** `[E9]` | Not in the playground **NV** |
| CriticMarkup | The text itself **V** `[M1]` | Not addressed **V** `[M1]` | Not addressed **V** `[M1]` |

## Notion

### Where the anchor is kept

- A user comments inline by selecting any text, or on a whole block. A user cannot comment on several blocks at once, but can select text across several blocks and comment on it. **V** `[N9]`
- How Notion stores the anchor of a comment on selected text is not documented. **NV**
- In the format Notion exchanges with agents, the anchor is a mark in the text. Notion-flavored Markdown (`markdown_version: "v2"`) lists this inline tag: `<span color="..." discussion-urls="url,url">text</span>`. Notion says the specification "comes from the same source as the Notion MCP server's Markdown spec". **V** `[N5]`
- The specification gives the tag and nothing else about it: no sentence says when the attribute is returned, or what writing it does. **V** `[N5]` Whether the REST endpoint that returns a page as Markdown, or the MCP tool `notion-fetch`, includes the attribute by default. **NV**
- The older format (enhanced markdown, v1, deprecated) has a `<span>` with colour and underline only. **V** `[N12]`

### How it follows an edit

- Not documented for comments. **NV**
- Notion's Markdown is edited by search and replace: each `old_str` "must be a non-empty string that matches exactly one location" unless `replace_all_matches` is set; when nothing matches, the page is left unchanged and the error asks the caller to fetch the page again. **V** `[N6][N8]` What happens to a discussion when the replaced text carried its tag, and whether `old_str` has to include the tag. **NV**
- A neighbouring mechanism is documented. A suggested edit names its target by quoting it: `old_text` "must appear exactly once on the page". "If someone edits the text a suggestion covers, the suggestion becomes stale. A stale suggestion can't be read or accepted, but it can still be rejected." "One suggestion can't overlap another pending suggestion." **V** `[N7]` This is about suggestions, not comments.

### When the anchored text is removed

- The comment object has a required boolean `original_content_deleted`, described as "Whether this comment's original content was deleted." **V** `[N3]` The reference does not say whether "original content" is the text the comment was attached to or the comment's own text. **NV**

### What a program outside sees

- REST API: a comment has a `parent`, which is a page or a block, and a `discussion_id`. There is no field for a range of text. "Comments may only be parented by pages or blocks." **V** `[N2]`
- REST API: a comment can be created on a page, on a whole block, or as a reply to an existing discussion. "The public API does not support creating a new discussion anchored to a selected range of text inside a block." **V** `[N1]`
- REST API: the block reference has no comment or discussion field. **V** `[N11]`
- MCP server: `notion-get-comments` "Lists all comments and discussions on a page. Can include block-level and inline discussions, resolved threads, and full comment content." Discussions are identified by a `discussion://` URL. `notion-create-comment` supports "page-level comments, block-level comments (via content selection), and replies to existing discussions". `notion-set-discussion-resolved` resolves or reopens. **V** `[N7]`
- Whether `notion-get-comments` returns the text a discussion is attached to. **NV**
- Export: the HTML export can include comments "at both the page and block levels", resolved ones included. The section on the Markdown and CSV export does not mention comments. **V** `[N10]`

### Overlap and resolving

- The `discussion-urls` attribute takes a list, so one stretch can carry several discussions. **V** `[N5]` Two discussions on stretches that only partly overlap. **NV**
- A user resolves and reopens a comment; the comments pane filters by open or resolved. **V** `[N9]` The REST API lists open comments only and cannot retrieve resolved ones. **V** `[N1]` What happens to the anchor on resolving. **NV**

## Google Docs

### Where the anchor is kept

- Beside the text. With `commentsViewMode` set to include comments, `documents.get` returns `comments[]` on the document and, on each tab, `commentAnchors`: "The comment anchors in a document tab, keyed by anchor ID." The map sits at the level of the tab, next to other maps such as positioned objects, not on the text runs. **V** `[G3][G4]`
- A `CommentAnchor` is "One or more locations in the document that are tied to CommentThreads with the same anchorId", with an `anchorId` and `ranges[]`. **V** `[G4]`
- A `CommentThread` has `commentId`, `anchorId`, `headPost`, `replies`, `status` (open or resolved) and `plainTextQuote`: "The quoted text from the document when the comment was created". **V** `[G4]`
- The document and the thread are saved separately: a batch update can commit the change to the text while "the associated comments or suggestions might fail to save". **V** `[G3]`

### How it follows an edit

- The Docs API returns the current ranges when the document is read. How the editor moves them is not documented. **V** for the first `[G4]`, **NV** for the second.
- For an anchor set through the Drive API, Google says: "Anchors are immutable, and their position relative to the content of a document cannot be guaranteed between revisions. We recommend you use anchors in documents where the position doesn't change, such as image files or read-only documents." Google's editors do not show such comments as anchored: "they treat these comments as unanchored comments". **V** `[G1]`
- The Drive API describes an anchored comment as tied to "a specific location ... within a specific version of a document". **V** `[G1]`

### When the anchored text is removed

- Not documented in the pages read. **NV**

### What a program outside sees

- Drive API: `anchor`, "A region of the document represented as a JSON string", and `quotedFileContent`, "The file content to which the comment refers, typically within the anchor region." **V** `[G2]`
- For a Google Doc created in the editor, `anchor` "contains editor-specific internal anchor data", and the Drive API "treats this data as opaque and can't resolve internal document regions". Google points to the Docs API for anchored comments. **V** `[G1]`
- Docs API: index ranges (`startIndex`, `endIndex`) and the quote. A comment is created with `InsertCommentRequest`, which takes the text and a `range`. To get comments, suggestions must be returned inline and tab content included. **V** `[G3][G4][G5]`
- Whether an export of a Google Doc to Markdown keeps comments. **NV**

### Overlap and resolving

- "Multiple comment threads may be anchored to the same CommentAnchor." "Multiple anchors may refer to the same location." **V** `[G4]`
- A thread is resolved or reopened by a reply that carries an action; `resolved` is a read-only flag on the comment. **V** `[G1][G3]` What happens to the anchor on resolving. **NV**

## Microsoft Word / OOXML

### Where the anchor is kept

- In the content. Three elements share one id: `commentRangeStart` and `commentRangeEnd`, empty elements placed where the stretch starts and ends, and `commentReference`, placed in a run. The text of the comment is not in the body: it is a `comment` element in the separate Comments part, found by the same id. **V** `[W1][W2][W3][W5]`
- The rules, as quoted from ISO/IEC 29500-1: with both range elements, "the comment reference shall anchor the comment to the resulting range"; with neither, "the comment reference shall anchor the comment to its current location". A comment that no reference points to "can be ignored when loading the document". **V** `[W2][W3]`
- On a start with no end, the two pages read differ: one says it "shall be considered the single anchor point", the other that with only one of the two "the document is non-conformant". **V** `[W1][W2]`

### How it follows an edit

- The markers are elements of the content, so no position is stored and none is recomputed. **V** `[W1]` How Word treats the markers while a user edits. **NV**

### When the anchored text is removed

- Not documented in the pages read. Forum answers on Microsoft's sites say the comment is deleted with the text; they are not a primary source. **NV**

### What a program outside sees

- Any program that reads the file sees the markers and the Comments part; Microsoft's Open XML SDK has a class for each element. **V** `[W5]`
- Word JavaScript API: `Comment.getRange()` "Gets the range in the main document where the comment is on"; `resolved` reads and sets the state of the thread. **V** `[W6]`
- Pandoc, reading a .docx: by default comments are ignored. With `--track-changes=all` they are included, "wrapped in spans with `insertion`, `deletion`, `comment-start`, and `comment-end` classes", with author and time. **V** `[M4]` So converted to Markdown, a Word anchor becomes a pair of marks in the text. The exact Markdown produced was not checked. **NV**

### Overlap and resolving

- Start and end are separate elements paired by id. The pages read do not say whether two ranges may overlap. **NV**
- The state is kept apart from the content: a `commentEx` element in another part has `done`, "whether the associated comment is marked as done", and `paraIdParent` for a reply. **V** `[W4]` Whether Word touches the markers on resolving. **NV**

## Confluence

### Where the anchor is kept

- In the content. In the storage format (XHTML-based), the stretch is enclosed by an opening and a closing `ac:inline-comment-marker` tag. **V** `[C3]` The storage format reference page does not list this element. **V** `[C7]` The name of the attribute that carries the reference. **NV**
- In the Atlassian Document Format (JSON), the schema defines an `annotation` mark with two required attributes: `id`, a string, and `annotationType`, whose only value is `inlineComment`. **V** `[C2]`
- The comment itself is a separate object. Through the API it has `inlineMarkerRef`, "Property value used to reference the highlighted element in DOM", and `inlineOriginalSelection`, "Text that is highlighted". **V** `[C1]` That `inlineMarkerRef` equals the `id` of the mark. **NV**

### How it follows an edit

- Data Center: "Confluence compares Page versions after an update (previous vs. current) looking for changes." **V** `[C3]`
- Data Center: text cut from one place and pasted in another is identified as deleted, and the inline comment is resolved. Moving the text in the source editor together with "both the opening and closing 'ac:inline-comment-marker' tags" keeps the comment. **V** `[C3]`
- Cloud, through the MCP server: a user reported that updating a page made every inline comment dangling, even with the commented text unchanged. A reply from a person who says they are on the Confluence team: "this is a side effect of whole-body Markdown replacement — the body is regenerated, so inline-comment marker anchors get dropped." The reply says version 2 of the server answers this with a default body format that "is lossless and round-trips the existing body instead of rebuilding it from Markdown" and with edits that target one node by its id, both "designed to keep inline-comment anchors intact". A repository collaborator confirmed version 2 is available. **V** `[C6]` The behaviour itself was not tested here.

### When the anchored text is removed

- Data Center: the comment is resolved. "The comment itself is preserved and can be checked via Resolved comments, but it will not be visible in the Page's body anymore." **V** `[C3]`
- Cloud API: `resolutionStatus` is one of `open`, `reopened`, `resolved`, `dangling`, and "A dangling comment cannot be updated." **V** `[C1]` The API description does not define `dangling`. **NV**

### What a program outside sees

- REST API v2, reading: the comment, its resolution status, `inlineMarkerRef` and `inlineOriginalSelection`. **V** `[C1]`
- REST API v2, creating: the program does not give a position. It gives `textSelection`, "The text to highlight", `textSelectionMatchCount`, "The number of matches for the selected text on the page", and `textSelectionMatchIndex`, the zero-based occurrence to highlight. **V** `[C1]`
- A program that reads the body in the storage format or the JSON format sees the marker or the mark in the content. **V** `[C2][C3]`
- MCP server: tools to list, get, create ("Create or reply to a footer or inline comment"), update, and resolve or reopen comments. **V** `[C5]` What the page-reading tool returns for a marked stretch in each body format. **NV**

### Overlap and resolving

- Two inline comments on overlapping stretches. **NV**
- "You can resolve an inline comment whether you are editing or viewing content to remove the highlight." A resolved comment can be reopened. **V** `[C4]` Whether the marker stays in the stored body after resolving. **NV**

## GitHub pull request review comments

### Where the anchor is kept

- Beside the content. A review comment is made "on a portion of the unified diff". It stores `path`, `commit_id`, `line`, `start_line` for a range of lines, `side` and `start_side`, and `diff_hunk`, "The diff of the line that the comment refers to". It also stores `original_commit_id`, `original_line` and `original_start_line`. **V** `[H1][H2]`
- The unit is a line or a range of lines of a file, or the whole file (`subject_type`: `line` or `file`), not a stretch of text within a line. **V** `[H1]`
- The older `position` counts lines down from the first hunk header of the file in the diff. It "is closing down"; `line` replaces it. **V** `[H1]`

### How it follows an edit

- Two sets of values are kept. `line` is "The end line number on the file to which the comment applies"; `originalLine` is the line "to which the comment applied when it was first created". **V** `[H3]`
- How the current line is computed after new commits is not documented. **NV**

### When the anchored text is changed or removed

- The thread stays and is flagged. "Not using the latest commit SHA may render your comment outdated if a subsequent commit modifies the line you specify as the position." **V** `[H1]`
- `PullRequestReviewThread.isOutdated`: "Indicates whether this thread was outdated by newer changes." **V** `[H3]`
- The pull request shows "which conversations are unresolved, resolved, and outdated". **V** `[H4]`

### What a program outside sees

- REST and GraphQL: all the fields above, including the hunk of the diff the comment was made on. **V** `[H1][H3]`
- GitHub's MCP server: review threads "with nullable current and original line-range coordinates"; "Current coordinates are omitted when unavailable, such as for outdated comments." **V** `[H5]`

### Overlap and resolving

- Two threads on the same or overlapping lines. **NV**
- `isResolved` is a field of the thread, separate from `isOutdated`. Resolving collapses the conversation. **V** `[H3][H4]`

## Hypothesis and the W3C Web Annotation Data Model

### Where the anchor is kept

- Outside the document, which is not changed. An annotation has a target, and a selector says which part of the target is meant. **V** `[A1]`
- The model allows several selectors for the same part, "in order to maximize the chances that it will be discoverable later". **V** `[A1]`
- `TextQuoteSelector`: `exact`, "A copy of the text which is being selected", with `prefix` and `suffix`, the text just before and after, "to distinguish between multiple copies of the same sequence of characters". **V** `[A1]`
- `TextPositionSelector`: `start` and `end`, positions in the text of the document. **V** `[A1]`
- Hypothesis saves three selectors for each annotation: a range selector (paths to the elements, and offsets), a text position selector and a text quote selector. **V** `[A2][A3]` Its client takes 32 characters of context on each side for the prefix and the suffix. **V** `[A5]`

### How it follows an edit

- It does not follow edits. The stretch is found again each time the document is loaded. **V** `[A2][A3]`
- The client tries the selectors "in order, from simple to complex": the range, then the position, each accepted only if the text found equals the stored quote, then the quote alone. **V** `[A3]`
- The search for the quote is approximate. It allows a number of errors up to half the length of the quote, at most 256, and scores each candidate on its likeness to the quote, to the prefix, to the suffix and, as a tie-breaker, on its distance from the expected position. It returns nothing when no candidate is within the allowed number of errors, and the annotation is then not anchored ("Quote not found"); otherwise it takes the candidate with the highest score. **V** `[A4][A5]`
- The model on positions: the position selector "is very brittle with regards to changes to the resource. Any edits or dynamically transcluded content may change the selection". **V** `[A1]`
- The model on quotes: when several places match, "the selection SHOULD be treated as matching all of the matches". **V** `[A1]`

### When the anchored text is removed

- "When this happens, we don't get rid of or hide your annotation. Instead it goes into a category called 'Orphans'. The anchor text, crossed out, still appears on the annotation card." **V** `[A6]`
- Hypothesis now calls these unanchored annotations. **V** `[A7]`

### What a program outside sees

- The annotation as data, with its selectors: quote, prefix, suffix, positions. **V** `[A1]` The Hypothesis API was not read. **NV**

### Overlap and resolving

- Each annotation carries its own selectors. Nothing read says whether two may cover overlapping text. **NV**
- Resolving is not part of what was read. **NV**

## Editor frameworks

### ProseMirror

- ProseMirror has no comment feature. It has the two building blocks. **V** `[E1][E2]`
- *Positions mapped through changes.* "When you make a change to a document, positions pointing into that document may become invalid or change meaning." Each step gives a map that converts a position from before the step to after it. **V** `[E1]`
- A mapped position takes a side (`assoc`), "which determines in which direction to move when a chunk of content is inserted at the mapped position". The result can say the position was deleted, "completely enclosed in a replaced range". **V** `[E2]`
- The project's own collaboration demo keeps comments this way, outside the document: each comment is an id, a text and a range held as a decoration in plugin state, and the set is mapped through every transaction. Comments travel to the server as events with `id`, `from`, `to` and `text`. **V** `[E3]`
- A decoration says how it behaves at its edges: by default it "won't include the new content" inserted directly at its start or end. When a set is mapped, a callback is called "for each decoration that gets dropped as a result of the mapping". **V** `[E2]`
- *Marks in the document.* A mark type is "exclusive with marks of the same type" by default; a schema can let several marks of one type coexist "as long as they have different attributes". A mark is `inclusive` by default: active when the cursor is at its end. **V** `[E2]`

### Tiptap Comments

- The anchor is in the document: "An anchor is an inline thread mark or a block thread node." The mark is `inlineThread` with a `data-thread-id` attribute. The thread and its comments are kept in the collaboration document, not in the content. **V** `[E4][E5][E6]`
- Creating a thread from outside the editor takes two steps: create the thread through the REST API, then wrap the content in the mark in the document's JSON. "Calling the REST API alone is not enough. Without the inlineThread mark in the document, the editor has no way to know which content the thread belongs to". **V** `[E6][E7]`
- A program can ask for the ranges of each thread. "These positions describe the current document, not permanent anchors." "A thread can have several separate anchors". **V** `[E4]`
- When the stretch is removed: "By default, threads are marked as deleted when they are not referenced in a document anymore." A setting turns this off, and a callback "runs when a local edit removes a thread's last document anchor"; the documented example resolves the thread and keeps its comments. **V** `[E5]`
- Resolving: "Resolving, reopening, soft-deleting, or restoring a thread keeps its anchors." Deleting a thread for good "removes its anchors and preserves their text". **V** `[E5]`
- On comments imported from Word: "If that text is removed, the comment's anchor goes with it, the same as in any word processor." **V** `[E11]`
- Two threads on overlapping stretches. **NV**

### Yjs relative positions

- Yjs names comments as a use: positions "may represent cursor locations, selection ranges, or even assign a comment to a range of text". **V** `[E8]`
- A plain index is "invalidated as soon as a remote change manipulates the document". A relative position "is fixated to an element in the shared document and is not affected by remote changes": it stays attached to a character, on a chosen side. **V** `[E8]`
- It is converted back to an index when needed. The result is null "If the relative position cannot be referenced, or the type is deleted". **V** `[E8]`
- This is a position relative to the edit history: it only exists for a text held as a Yjs shared type. Where the application stores it is up to the application. **V** `[E8]`
- What it resolves to when the character it is attached to has been deleted. **NV**

### Lexical

- The anchor is a node in the editor state. `MarkNode` is an inline element that wraps content and holds a list of ids. Text typed just before or after it does not join it. A node with more than one id gets an extra "overlap" style. The node excludes itself when content is copied out as HTML (`excludeFromCopy`). **V** `[E9]`
- The playground keeps threads in a store separate from the editor state. A thread has an id, its comments and a `quote`: the selected text, cut to 100 characters, taken when the thread is made. **V** `[E10]`
- The playground tracks which mark nodes exist for each id. A thread whose marks are all gone stays in the list with its quote and is no longer linked to a place in the text. Deleting a thread unwraps its mark nodes. **V** `[E10]`

## Comments inside plain Markdown

### CriticMarkup

- Five marks: addition, deletion, substitution, comment `{>> <<}` and highlight `{== ==}`. **V** `[M1]`
- What is written in the text is the comment itself, not a pointer to it: `Lorem ipsum dolor sit amet.{>>This is a comment<<}`. To attach it to a stretch, the stretch is highlighted and the comment follows: `{==Truth is stranger than fiction==}{>>true<<}`. **V** `[M1]`
- There is no id, no thread, no reply and no author field. "A comment may include a note, time stamp, author initial or similar annotation." **V** `[M1]`
- Limits stated by the project: "Newlines should be avoided as much as possible within CriticMarkup tags", and a mark must wrap Markdown syntax completely, not half of it. **V** `[M1]`
- In real use: MultiMarkdown 6 has it built in, processes it before any other syntax, and adds that it "must be contained within a single block (e.g. paragraph, list item, etc.)". **V** `[M2]`

### Obsidian

- "You can add comments by wrapping text with `%%`. Comments are only visible in Editing view." Inline or as a block. "The `%%` syntax is specific to Obsidian. To hide text in other apps, use an HTML comment". **V** `[M3]`
- As described, this is hidden text at a point of the note. It is not attached to a stretch and has no thread. **V** `[M3]`

### Pandoc

- Pandoc's convention for a comment range, used when it reads Word files: spans with the classes `comment-start` and `comment-end`, carrying author and time. **V** `[M4]`

### CommonMark

- An HTML comment is valid raw HTML in CommonMark. **V** `[M5]`
- "Links may not contain other links, at any level of nesting." **V** `[M5]`

## Sources

Notion:

- `[N1]` Working with comments: https://developers.notion.com/guides/data-apis/working-with-comments
- `[N2]` Comment object: https://developers.notion.com/reference/comment-object
- `[N3]` Retrieve a comment (with the API schema): https://developers.notion.com/reference/retrieve-comment
- `[N4]` List comments: https://developers.notion.com/reference/list-comments
- `[N5]` Notion-flavored Markdown format: https://developers.notion.com/guides/data-apis/notion-flavored-markdown
- `[N6]` Working with markdown content: https://developers.notion.com/guides/data-apis/working-with-markdown-content
- `[N7]` Notion MCP, Work together in Notion, technical reference: https://developers.notion.com/guides/mcp/tools-comments-reference
- `[N8]` Notion MCP, Create and organize pages, technical reference: https://developers.notion.com/guides/mcp/tools-pages-reference
- `[N9]` Help, Comments, mentions & reactions: https://www.notion.com/help/comments-mentions-and-reminders
- `[N10]` Help, Export your content: https://www.notion.com/help/export-your-content
- `[N11]` Block: https://developers.notion.com/reference/block
- `[N12]` Enhanced markdown format (v1, deprecated): https://developers.notion.com/guides/data-apis/enhanced-markdown

Google:

- `[G1]` Drive API, Manage comments and replies (updated 2026-09-10): https://developers.google.com/workspace/drive/api/guides/manage-comments
- `[G2]` Drive API, comments resource: https://developers.google.com/workspace/drive/api/reference/rest/v3/comments
- `[G3]` Docs API, Work with comments and suggestions (updated 2026-09-30): https://developers.google.com/workspace/docs/api/how-tos/suggestions
- `[G4]` Docs API, documents resource (updated 2026-10-09): https://developers.google.com/workspace/docs/api/reference/rest/v1/documents
- `[G5]` Docs API, requests: https://developers.google.com/workspace/docs/api/reference/rest/v1/documents/request

Microsoft:

- `[W1]` CommentRangeStart class, quoting ISO/IEC 29500-1 §17.13.4.4: https://learn.microsoft.com/en-us/dotnet/api/documentformat.openxml.wordprocessing.commentrangestart
- `[W2]` CommentReference class, quoting §17.13.4.5: https://learn.microsoft.com/en-us/dotnet/api/documentformat.openxml.wordprocessing.commentreference
- `[W3]` Comment class, quoting §17.13.4.2: https://learn.microsoft.com/en-us/dotnet/api/documentformat.openxml.wordprocessing.comment
- `[W4]` CommentEx class, quoting MS-DOCX 2.5.39: https://learn.microsoft.com/en-us/dotnet/api/documentformat.openxml.office2013.word.commentex
- `[W5]` Insert a comment into a word processing document: https://learn.microsoft.com/en-us/office/open-xml/word/how-to-insert-a-comment-into-a-word-processing-document
- `[W6]` Word JavaScript API, Word.Comment: https://learn.microsoft.com/en-us/javascript/api/word/word.comment

Atlassian:

- `[C1]` Confluence Cloud REST API v2, machine-readable description: https://dac-static.atlassian.com/cloud/confluence/openapi-v2.v3.json (rendered at https://developer.atlassian.com/cloud/confluence/rest/v2/api-group-comment/)
- `[C2]` Atlassian Document Format schema, `@atlaskit/adf-schema` 57.7.42: https://unpkg.com/@atlaskit/adf-schema@57.7.42/dist/json-schema/v1/full.json
- `[C3]` Keep inline comments while editing Confluence Pages (Data Center, updated 2025-09-25): https://support.atlassian.com/confluence/kb/keep-inline-comments-while-editing-confluence-pages
- `[C4]` Comment on pages and blog posts (Cloud): https://support.atlassian.com/confluence-cloud/docs/comment-on-pages-and-blog-posts/
- `[C5]` Atlassian Rovo MCP Server, Supported tools: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/supported-tools/
- `[C6]` atlassian/atlassian-mcp-server, issue 54, with replies from Atlassian: https://github.com/atlassian/atlassian-mcp-server/issues/54
- `[C7]` Confluence Storage Format: https://confluence.atlassian.com/doc/confluence-storage-format-790796544.html

GitHub:

- `[H1]` REST API endpoints for pull request review comments: https://docs.github.com/en/rest/pulls/comments
- `[H2]` REST API description, schema `pull-request-review-comment`: https://github.com/github/rest-api-description
- `[H3]` GraphQL public schema: https://docs.github.com/public/fpt/schema.docs.graphql
- `[H4]` Commenting on a pull request: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/commenting-on-a-pull-request
- `[H5]` github/github-mcp-server, README: https://github.com/github/github-mcp-server/blob/main/README.md

Hypothesis and W3C:

- `[A1]` Web Annotation Data Model, W3C Recommendation, 2017-02-23: https://www.w3.org/TR/annotation-model/
- `[A2]` Hypothesis, Fuzzy Anchoring: https://web.hypothes.is/blog/fuzzy-anchoring/
- `[A3]` hypothesis/client, `src/annotator/anchoring/html.ts`, commit 72526e9c8a: https://github.com/hypothesis/client/blob/main/src/annotator/anchoring/html.ts
- `[A4]` hypothesis/client, `src/annotator/anchoring/match-quote.ts`, same commit: https://github.com/hypothesis/client/blob/main/src/annotator/anchoring/match-quote.ts
- `[A5]` hypothesis/client, `src/annotator/anchoring/types.ts`, same commit: https://github.com/hypothesis/client/blob/main/src/annotator/anchoring/types.ts
- `[A6]` Hypothesis, What are annotation "orphans" and where are they?: https://web.hypothes.is/help/what-are-orphans-and-where-are-they/
- `[A7]` Hypothesis, What are Unanchored Annotations?: https://web.hypothes.is/help/what-are-unanchored-annotations/

Editor frameworks:

- `[E1]` ProseMirror Guide: https://prosemirror.net/docs/guide/
- `[E2]` ProseMirror Reference manual: https://prosemirror.net/docs/ref/
- `[E3]` ProseMirror/website, `src/collab/client/comment.js`, commit a16b4ece45: https://github.com/ProseMirror/website/blob/master/src/collab/client/comment.js
- `[E4]` Tiptap Comments, Manage threads: https://tiptap.dev/docs/comments/core-concepts/manage-threads
- `[E5]` Tiptap Comments, Configure: https://tiptap.dev/docs/comments/core-concepts/configure
- `[E6]` Tiptap Comments, Create threads via API: https://tiptap.dev/docs/comments/integrate/create-thread-via-api
- `[E7]` Tiptap Comments, REST API: https://tiptap.dev/docs/comments/integrate/rest-api
- `[E8]` Yjs, Y.RelativePosition: https://docs.yjs.dev/api/relative-positions
- `[E9]` facebook/lexical, `packages/lexical-mark/src/MarkNode.ts`, commit 9052aa9c57: https://github.com/facebook/lexical/blob/main/packages/lexical-mark/src/MarkNode.ts
- `[E10]` facebook/lexical, playground `commenting/index.ts` and `plugins/CommentPlugin/index.tsx`, same commit: https://github.com/facebook/lexical/tree/main/packages/lexical-playground/src
- `[E11]` Tiptap Conversion, Comments: https://tiptap.dev/docs/conversion/content-types/structures-and-media/comments

Markdown:

- `[M1]` CriticMarkup/CriticMarkup-toolkit, README, commit ba9011ba65: https://github.com/CriticMarkup/CriticMarkup-toolkit/blob/master/README.md
- `[M2]` MultiMarkdown 6, CriticMarkup: https://fletcher.github.io/MultiMarkdown-6/syntax/critic.html
- `[M3]` Obsidian Help, Basic formatting syntax, Comments: https://help.obsidian.md/syntax (read from https://github.com/obsidianmd/obsidian-help)
- `[M4]` Pandoc User's Guide, `--track-changes`: https://pandoc.org/MANUAL.html (read from `MANUAL.txt` in https://github.com/jgm/pandoc)
- `[M5]` CommonMark Spec 0.31.2: https://spec.commonmark.org/0.31.2/

## Not verified

| Claim | Sources checked | Why it stays unverified |
| - | - | - |
| Notion: how an anchor on selected text is stored | `[N1]` to `[N12]` | Not documented publicly. |
| Notion: how an anchor follows an edit | `[N1]` to `[N9]` | Not stated. |
| Notion: what `original_content_deleted` refers to | `[N2][N3]` | The description is one sentence and does not say which content. |
| Notion: whether `discussion-urls` is returned by default, by REST or by `notion-fetch` | `[N5][N6][N8]` | The format lists the tag; no page says when it is returned. No Notion connection was called. |
| Notion: what an edit does to a discussion whose tag was inside the replaced text | `[N6][N8]` | Not stated. |
| Notion: partly overlapping discussions; the anchor on resolving | `[N5][N9]` | Not stated. |
| Google Docs: how the editor moves an anchor; what happens when the text is removed; the anchor on resolving | `[G1]` to `[G5]` | The API returns current ranges; the behaviour is not described. Only a forum and a third-party page spoke of removal. |
| Google Docs: comments in a Markdown export | None | Not looked up in a primary source. |
| Word: what Word does to a comment when its text is deleted, and to the markers on resolving | `[W1]` to `[W6]` | The file format is specified, the application's behaviour is not. Only forum answers were found. |
| Word: overlapping ranges | `[W1][W2]` | Not stated in the pages read. The full ISO/IEC 29500 text was not read, only the parts Microsoft quotes. |
| Pandoc: the exact Markdown written for a comment range | `[M4]` | The manual names the classes; Pandoc was not run. |
| Confluence: the attribute of the marker that carries the reference; that `inlineMarkerRef` equals the mark's `id` | `[C1][C2][C3][C7]` | The knowledge-base article names the tag only; the storage format page does not list it. |
| Confluence: the definition of `dangling` | `[C1]` | The value is listed without a description. A user's report in `[C6]` uses it for a comment that lost its anchor. |
| Confluence: whether the marker stays in the stored body after resolving; overlapping inline comments | `[C3][C4]` | Not stated. A forum thread says markers stay; it is not a primary source. |
| Confluence Cloud: what the editor does when commented text is cut and pasted | `[C3]` | The article is for Data Center only. |
| Confluence MCP: what the reading tool returns for a marked stretch; that version 2 keeps anchors | `[C5][C6]` | Stated as the design goal in `[C6]`; not tested, and the body format's documentation was not found. |
| GitHub: how the current line is recomputed; two threads on the same lines | `[H1]` to `[H4]` | Not stated. |
| Hypothesis: the API's form of an annotation; overlap | `[A1]` to `[A7]` | The API reference was not read. |
| Tiptap: overlapping threads; whether the mark survives a conversion to Markdown | `[E4]` to `[E7]` | Not stated in the pages read. |
| Yjs: what a relative position gives once its character is deleted | `[E8]` | The page covers only a position that cannot be referenced and a deleted type. |
| Lexical: resolving | `[E10]` | The playground has no resolved state. |

Reading caveats:

- Pages were fetched as HTML and reduced to text, or read as the Markdown or raw file the site serves. `[N9]` and `[N10]` were read through a fetch tool that returned the full page text. Nothing was read through a summarizing model.
- `[W1]` to `[W4]` are Microsoft's class pages, which quote the standards. The standards themselves (ISO/IEC 29500-1, MS-DOCX) were not opened.
- `[C6]` is an issue on Atlassian's repository. The report is a user's. The quoted reply is from a person who says elsewhere on the same repository that they are on the Confluence team; GitHub does not mark that account as a collaborator. A second reply, from an account that is marked as a collaborator, builds on it.
- `[C3]` is dated 2025 and applies to Data Center. `[A2]` is from 2013 and describes four strategies; the current code in `[A3]` and `[A4]` is what the section on Hypothesis follows.
- `[C2]`, `[A3]` to `[A5]`, `[E3]`, `[E9]`, `[E10]` and `[M1]` are source files; the version or commit read is given.
- Did not load: criticmarkup.com (the project's site; the README in its repository was read instead), and Tiptap's page of comment events (it renders in the browser only).
- Not looked at: Automerge (its documentation pages did not load), GitLab merge request discussions, Quip, Dropbox Paper, Coda, Liveblocks.

## The two forms set against the findings

This section states what the evidence shows for each form on the table in issue #13. It does not choose.

### Form 1: a mark in the body that carries the id of the thread

**Applications that work this way.** Word `[W1][W2]`, Confluence `[C2][C3]`, Tiptap `[E6]`, Lexical `[E9]`. Notion's Markdown for agents presents anchors this way, whatever Notion stores `[N5]`. Pandoc converts Word's anchors into this form `[M4]`. In all of them the mark carries only an id; the thread and its comments are kept elsewhere. CriticMarkup differs: it puts the comment itself in the text `[M1]`.

**What they show.**

- *Following an edit costs nothing.* The mark is content. Text inserted before it, a change inside it and a move of the stretch with its mark all leave the anchor in place, with no position to maintain. Confluence documents the move: text moved with both tags keeps its comment `[C3]`.
- *The anchor is lost whenever the stretch is written without the mark.* This is the failure Atlassian describes for its own MCP server: a body regenerated from Markdown drops the markers and the comments are left dangling `[C6]`. In the Confluence editor a cut and paste has the same effect `[C3]`. Atlassian's stated answer was a body format that round-trips without loss and edits that touch one node `[C6]`. Tiptap states the rule plainly: if the text is removed, "the comment's anchor goes with it" `[E11]`.
- *Every reader of the body sees the mark, and every writer has to carry it.* Notion's specification for agents includes the tag among the syntax an agent reads and writes `[N5]`. Lexical excludes its mark when content is copied out as HTML `[E9]`; Notion's Markdown export does not mention comments `[N10]`. An export or a rendering for humans has to decide the same.
- *The closest documented case to this product is Notion's.* Its Markdown carries the mark, and its edit is a quoted text that must match exactly one location, refused otherwise `[N5][N8]`. Notion does not document what happens to a discussion when an agent replaces text that carried the tag. **NV**
- *Removal of the whole stretch is easy to detect.* The mark is no longer in the body. Tiptap and Lexical's playground both work from that test `[E5][E10]`.
- *Overlap needs an answer in the syntax.* Notion lists several discussions on one tag `[N5]`; Lexical holds several ids on one node `[E9]`; ProseMirror excludes two marks of one type unless the schema allows them `[E2]`. For a mark written as a Markdown link, CommonMark says "Links may not contain other links, at any level of nesting" `[M5]`.
- *The mark has edges and limits of its own.* Lexical does not let text typed at the edge of a mark join it `[E9]`; a ProseMirror mark is active at its end by default `[E2]`. CriticMarkup asks that a mark hold no newline and wrap Markdown syntax completely, and MultiMarkdown that it stay within one block `[M1][M2]`. Word's markers have no such limit: they are separate start and end elements `[W1]`.
- *Resolving does not have to touch the body.* Tiptap keeps the anchor when a thread is resolved `[E5]`; Word keeps the state in another part of the file `[W4]`. Confluence removes the highlight `[C4]`.

**Reasoning from this product's own decisions, not from the sources.** By ADR 0004 an agent quotes a passage exactly as it read it. A mark inside a stretch is then part of what an agent has to quote, and it enters the body when a thread is anchored, after some agents have read the body. ADR 0004 already records the same effect for a reference whose target is renamed. By the CommonMark rule above, a mark written as a link could not wrap a stretch that contains a reference or a mention, since those are links, and one such mark could not sit inside another. None of this was tested.

### Form 2: a position kept outside the body, moved at every change

**Applications that work this way.** Google Docs `[G4]`, the ProseMirror collaboration demo `[E3]`, applications that use Yjs relative positions `[E8]`, GitHub review comments `[H1][H3]`. Hypothesis and the W3C model are the variant that keeps a quote in place of a position and searches for it `[A1][A3]`.

**What they show.**

- *The body carries nothing.* A program that reads it sees no trace of the threads, and a program that writes it cannot damage an anchor by leaving something out. To learn where a thread is attached, a program makes a separate read and gets a position, a quote, or both: index ranges and the quote in Google Docs `[G4]`, line numbers on GitHub `[H1]`, selectors in Hypothesis `[A1]`.
- *A position is valid for one version of the body.* Google ties an anchored comment to "a specific version of a document" `[G1]`; Tiptap, which also reports ranges, says they "describe the current document, not permanent anchors" `[E4]`; Yjs says an index is invalidated by any remote change `[E8]`. A program that reads a position and acts on it later acts on a body that may have changed.
- *Following an edit requires seeing every change as an operation with a place.* ProseMirror maps each position through each step, and needs a rule for text inserted exactly at an edge and a signal for a position whose text was deleted `[E1][E2]`. Yjs attaches the position to a character of a shared text, which requires the text to be held in that structure `[E8]`. Where an application writes an anchor without taking part in the editing, Google does not promise the position holds `[G1]`.
- *Where edits are not seen as operations, applications fall back on the quote.* GitHub keeps the original line and the hunk of the diff and flags the thread outdated when the line changes `[H1][H3]`. Hypothesis searches for the quote with its context, approximately, and gives up when nothing is within the allowed number of errors `[A4]`. The W3C model warns that a position alone is "very brittle" and that a quote can match in several places `[A1]`.
- *Removal is known only through the mapping or a failed search.* ProseMirror reports the dropped range `[E2]`; Hypothesis reports an orphan when the quote is not found `[A6]`.
- *Overlap is free.* Each anchor is its own range. Google states that several threads can share an anchor and several anchors a location `[G4]`.

**Reasoning from this product's own decisions, not from the sources.** By ADR 0004 an agent's change arrives as a passage and its replacement, so the product knows which stretch was replaced and by how much the text after it moved; inside the replaced stretch it knows only the old and the new text. By ADR 0005 the editor for humans already "has to ... know which stretches the human changed". Whether that is enough to move a position kept outside the body was not tested.

### What both forms share in the applications read

- The thread and its comments are kept apart from the content in every application except CriticMarkup. The two forms differ only in where the pointer to the stretch is.
- Nearly all of them keep a copy of the stretch as it was when the thread was made, whichever form they use: Google Docs `[G4]`, Confluence `[C1]`, GitHub `[H1]`, Hypothesis `[A1]`, Lexical's playground `[E10]`. It is what they show once the anchor is gone `[A6][E10]`.
- A program outside names the stretch for a new anchor by quoting it or by position, whichever form the application stores. Confluence, which stores a mark, takes the text and the number of the occurrence `[C1]`. Notion's suggested edits take a text that must appear exactly once `[N7]`. Google's Docs API takes index positions `[G5]`. Tiptap has the program write the mark into the document itself `[E6]`.
- Where it is documented, a thread whose stretch is removed is kept and given a state of its own, distinct from resolved: `dangling` in Confluence Cloud `[C1]`, outdated on GitHub `[H3]`, unanchored in Hypothesis `[A7]`. Confluence Data Center resolves it `[C3]` and Tiptap soft-deletes it by default `[E5]`.
