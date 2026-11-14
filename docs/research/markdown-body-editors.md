# Which web editors can edit a Markdown body, keep marks that carry an id and tell which stretch a change touched

Research for issue #23. All sources were read on **2026-11-14**. Only primary sources were used: each project's own documentation, API reference and repository, the packages it publishes, and the npm registry for release dates and licences. Every claim carries a source tag such as `[P1]`; the tags resolve in [Sources](#sources). Nothing was run.

Status marks: **V** = verified, read in a primary source on the date above. **NV** = not verified; the reason is given in [Not verified](#not-verified).

Terms follow `GLOSSARY.md`: body, passage, stretch, reference, mention, anchor, thread, conflict. Two words need care, because the editors use them differently from the glossary:

- *The two marks of an anchor* are the pair of marks the glossary describes, one at the start and one at the end of a stretch. Several editors also have a *ranged mark*: a property laid over a run of text, with no characters of its own. This file always says "ranged mark" for the editors' kind.
- *Document* is what an editor holds while it runs. For most editors here it is a tree of their own, not the Markdown text of the body.

This file builds on [the findings on comment anchors](https://github.com/TommyBez/agent-intelligence/blob/research/comment-anchors/docs/research/comment-anchors.md) (issue #15), which already read how Tiptap, Lexical, ProseMirror and Yjs keep a comment attached. What was read there is cited here as `[#15]` and not repeated.

## Short answer

1. **What the editor keeps.** Only CodeMirror 6 keeps the Markdown text itself: its document is "a flat string" `[CM2]`, so text the human did not write is not touched by construction. Every other editor read keeps a tree of its own and makes Markdown from it on the way out: ProseMirror `[P3]`, Tiptap `[T1]`, Lexical `[L1]`, Milkdown `[M1]`, BlockNote `[B1]`, Plate `[S1]` and MDXEditor `[X1]`. None of these documents that Markdown comes out unchanged where the human did not write. Three say the opposite in their own words: BlockNote calls both directions "lossy" `[B1][B2]`, Lexical's newer Markdown package says "Round-trips are close, not exact" `[L3]`, and MDXEditor reports a change on load caused by "additional whitespace, bullet symbols different than the configured ones" `[X3]`. For ProseMirror and Tiptap the source of the serializer shows fixed output forms (one bullet character, one emphasis character, fenced code only) `[P4][T6]`.
2. **Something that carries an id.** Every editor read can hold an inline thing with an id. Ready-made: Tiptap's Mention node (`id`, `label`) `[T7]`, Plate's mention, written to Markdown as `[display text](mention:id)` `[S1]`, and Lexical's `MarkNode` and `MarkStateExtension`, which lay thread ids over a run of text `[L6]`. No editor read ships a pair of point marks. The tree editors offer a ranged mark with an id, or an inline node that behaves as one unit; CodeMirror leaves the marks as characters in the text and can draw them as something else `[CM4]`. How each one is written to Markdown has to be written by the application in every case except Tiptap's and Plate's mentions. Lexical is the only one that documents in detail what happens when a human writes inside, at the edge of, or over a marked run `[L6]`.
3. **What a change reports.** CodeMirror reports each change as ranges of the old and the new text (`fromA`, `toA`, `fromB`, `toB`) `[CM1]`; since the text is the Markdown, that is the stretch of the body. ProseMirror and Tiptap report steps with the same four numbers, but as positions in their tree `[P1][T4]`; nothing read turns those into offsets in the Markdown. Lexical reports which nodes are dirty, by key, with no range `[L4]`. BlockNote reports changed blocks `[B3]`. Milkdown and MDXEditor hand over the whole Markdown string `[M2][X3]`.
4. **A change from outside.** CodeMirror and ProseMirror both take a replacement of one range as a transaction and move the human's selection through it by default `[CM2][P1]`. Both also ship a small module, with no Yjs, that keeps the human's unsent changes and rebases them over changes that arrive from a central authority `[CM5][P2]`. Tiptap has a command that replaces a range with Markdown `[T2]`. MDXEditor, Milkdown's `replaceAll` and Plate's `setValue` replace the whole document `[X2][M3][S3]`. With Yjs, every tree editor shares its tree, not the Markdown; only the CodeMirror binding shares a text `[Y1][Y2]`.
5. **Licence and upkeep.** All are MIT except BlockNote (MPL-2.0, with GPL-3.0 or commercial `xl-` packages) `[B5]`. Tiptap's core, Markdown, Mention and Collaboration packages are MIT; Comments, snapshots, parts of its AI toolkit and its hosted collaboration server need a paid plan `[T10][T11]`. Every project had a release in the three months before the reading date; the two central-authority modules (`prosemirror-collab` and `@codemirror/collab`) were last released in 2023 `[N1]`. Tiptap's Markdown package is marked beta `[T1]` and Lexical's `@lexical/mdast` experimental `[L3]`.

## Comparison

What the editor keeps and what comes back out as Markdown:

| | What it keeps | Markdown in and out | What is documented about fidelity |
| - | - | - | - |
| CodeMirror 6 | The text, "a flat string" **V** `[CM2]` | Nothing is converted; the Markdown package only parses for display **V** `[CM6]` | Changes replace stated ranges only **V** `[CM2]` |
| ProseMirror | A tree defined by a schema **V** `[P2]` | `prosemirror-markdown`: markdown-it in, a serializer out **V** `[P3]` | No statement. The serializer writes fixed forms **V** `[P4]`; the size of the difference **NV** |
| Tiptap | ProseMirror tree, as JSON **V** `[T1]` | `@tiptap/markdown`, beta: MarkedJS in, per-extension handlers out **V** `[T1]` | "Test round-trip conversion" **V** `[T2]`; fixed forms in the source **V** `[T6]`; comments not supported **V** `[T1]` |
| Lexical | A tree of nodes in an editor state **V** `[L5]` | `@lexical/markdown` (transformers) or `@lexical/mdast` (experimental) **V** `[L1][L3]` | "close, not exact"; syntax preservation "partial" for `@lexical/markdown` **V** `[L3]` |
| Milkdown | ProseMirror tree **V** `[M1]` | remark in and out **V** `[M1]` | No statement; output set by remark options **V** `[M2][R1]` |
| BlockNote | Blocks, as JSON **V** `[B2]` | Its own "minimal" parser; `blocksToMarkdownLossy` **V** `[B1]` | "lossy" both ways **V** `[B1][B2]` |
| Plate | Slate tree, as JSON **V** `[S1]` | remark in and out **V** `[S1]` | Custom elements round-trip through MDX **V** `[S1]`; untouched Markdown **NV** |
| MDXEditor | Lexical tree **V** `[X1]` | mdast in and out; "accepts and emits markdown as a string" **V** `[X1]` | Markdown can change on load **V** `[X3]` |

Something that carries an id, and what a change reports:

| | A reference (inline, with an id) | An anchor (two marks, or a ranged mark with an id) | What a change reports |
| - | - | - | - |
| CodeMirror 6 | Characters in the text, drawn as a widget by a replacing decoration **V** `[CM1][CM4]` | Characters in the text; ranges can be made atomic **V** `[CM4]` | Ranges of old and new text, with the inserted text **V** `[CM1]` |
| ProseMirror | Inline node with attributes, `atom` **V** `[P1]` | Ranged mark with attributes, or two inline nodes **V** `[P1]`; nothing ties two nodes together **NV** | Steps with ranges in tree positions **V** `[P1]` |
| Tiptap | Mention node: `id`, `label` **V** `[T7]` | Custom ranged mark **V** `[T9]`; Comments extension (paid) not written to Markdown **V** `[T1]` | The transaction; a `delete` event for a node or ranged mark **V** `[T4][T5]` |
| Lexical | Custom node; `token` text or decorator node **V** `[L8]` | `MarkNode`, `MarkStateExtension`: ids over a run **V** `[L6]` | Dirty node keys, tags **V** `[L4]`; added and removed ids, documented on `main` **V** `[L6]` |
| Milkdown | Custom node with Markdown rules **V** `[M4]` | Custom ranged mark with Markdown rules **V** `[M4]` | Whole Markdown before and after; or the ProseMirror transaction **V** `[M2][M5]` |
| BlockNote | Custom inline content with props **V** `[B4]`; in Markdown **NV** | Comments extension **V** `[B6]`; in Markdown **NV** | Blocks inserted, updated, deleted, moved, with a source **V** `[B3]` |
| Plate | Mention, as `[text](mention:id)` **V** `[S1]` | Comment as a text mark, overlapping allowed **V** `[S2]`; in Markdown **NV** | Value on change **V** `[S3]`; Slate operations with path and offset **V** `[S4]` |
| MDXEditor | Directive or JSX with a custom editor **V** `[X4]` | **NV** | Whole Markdown string **V** `[X3]` |

A change from outside, licence and upkeep (latest release as listed by the npm registry on the reading date `[N1]`):

| | One range replaced from outside | Collaboration layer | Licence | Latest release | React |
| - | - | - | - | - | - |
| CodeMirror 6 | Transaction; selection mapped by default **V** `[CM2]` | `@codemirror/collab` (central authority) **V** `[CM5]`; `y-codemirror.next` shares a `Y.Text` **V** `[Y2]` | MIT **V** `[CM7]` | `@codemirror/view` 6.43.15, 2026-10-19; `@codemirror/state` 6.7.6, 2026-09-22; `@codemirror/collab` 6.1.1, 2023-09-14 | No React package of its own in what was read **NV** |
| ProseMirror | Transaction; selection mapped by default **V** `[P1]` | `prosemirror-collab` (central authority) **V** `[P2]`; `y-prosemirror` shares a tree **V** `[Y1]` | MIT **V** `[P3]` | `prosemirror-view` 1.42.9, 2026-10-31; `prosemirror-markdown` 1.13.9, 2026-10-22; `prosemirror-collab` 1.3.1, 2023-05-17 | No React package of its own in what was read **NV** |
| Tiptap | `insertContentAt` a range, Markdown accepted **V** `[T2]` | Collaboration extension on Yjs **V** `[T8]` | MIT for the packages read; paid: Comments, snapshots, hosted server **V** `[T10][T11]` | `@tiptap/core`, `@tiptap/markdown`, `@tiptap/react` 3.34.0, 2026-11-13 | `@tiptap/react`, React 17 to 19 **V** `[N1]` |
| Lexical | `editor.update` on nodes **V** `[L5]`; the caret **NV** | `@lexical/yjs` **V** `[L7]` | MIT **V** `[L9]` | `lexical`, `@lexical/markdown`, `@lexical/react` 0.53.0, 2026-10-28 | `@lexical/react`, React 18 or later **V** `[N1]` |
| Milkdown | ProseMirror transaction **V** `[M5]`; `replaceAll` replaces everything **V** `[M3]` | `@milkdown/plugin-collab` on Yjs **V** `[M6]` | MIT **V** `[N1]` | `@milkdown/kit` 7.23.0, 2026-10-27 | `@milkdown/react` **V** `[N1]` |
| BlockNote | By block id **V** `[B7]` | Yjs **V** `[B6]` | MPL-2.0; `xl-` packages GPL-3.0 or commercial **V** `[B5]` | `@blocknote/core` 0.57.0, 2026-10-30 | `@blocknote/react`, React 18 or 19 **V** `[N1]` |
| Plate | Slate transforms; `setValue` replaces everything **V** `[S3]` | `@platejs/yjs` on slate-yjs **V** `[S5]` | MIT **V** `[S6]` | `platejs` 53.5.2, 2026-10-30 | React 18 or later **V** `[N1]` |
| MDXEditor | `setMarkdown` replaces everything; `insertMarkdown` at the cursor **V** `[X2]` | None documented **V** `[X2]` | MIT **V** `[X5]` | `@mdxeditor/editor` 4.5.0, 2026-11-02 | React 18 or 19 **V** `[N1]` |

The application declares React 19.3.0 and Next.js 16.4.5 in `package.json`. Every React range above admits React 19. Whether any of these editors works under the React Compiler, which the application also declares, was not looked up. **NV**

## CodeMirror 6

### What it keeps

- "CodeMirror, being a text editor, treats the document as a flat string." **V** `[CM2]`
- A change is a value that describes "precisely which ranges of the old document are being replaced by which bits of new text". **V** `[CM2]`
- So for a body edited in CodeMirror the Markdown text is the document. Nothing is parsed into another form and written back, and a change touches only the ranges it names. This follows from the two statements above; no page states it about Markdown as such.
- `@codemirror/lang-markdown` "implements Markdown language support for the CodeMirror code editor": a parser for strict CommonMark or for GFM, a keymap and commands, and a way to extend the parser with new syntax. **V** `[CM6]` Its parser is incremental and does not do everything "a conforming CommonMark parser is expected to do"; it does not validate link references. **V** `[CM6]`
- Nothing in the pages read turns the Markdown into formatted text for display: the package is language support for a code editor, and what else is drawn is up to decorations the application adds. **V** for what the package says it is `[CM6]`; that nothing more exists **NV**.

### Something that carries an id

- The characters of a reference or of the marks of an anchor are part of the text. The editor has no document node for them.
- A *replace decoration* "replaces the given range with a widget, or simply hides it". A *widget decoration* shows a DOM element at a position. A *mark decoration* styles a range. Decorations are drawn by the view and do not change the document. **V** `[CM1][CM4]`
- The project's own example does this for placeholders: it "replaces placeholder names like `[[this]]` with widgets, and makes the editor treat them as atoms". The `atomicRanges` facet makes "cursor motion" skip such ranges, and the example says they are backspaced "out in one step". **V** `[CM4]` `atomicRanges` "does not prevent direct programmatic selection updates from moving into such regions". **V** `[CM1]`
- A set of decorations kept in editor state is mapped through each change (`underlines.map(tr.changes)` in the example). **V** `[CM4]` A mark decoration and an inline replace decoration are not `inclusive` by default: content inserted at the edge does not become part of the range. **V** `[CM1]`
- New inline syntax can be added to the Markdown parser (`parseInline`, `defineNodes`), which is how a reference or a mark written in a syntax of the product's own would be recognised. **V** `[CM6]`
- What happens when a human writes around, through or removes one end: these are ordinary edits of text. A mark whose characters are deleted is gone from the text; a mark made atomic is deleted as one unit. **V** for the mechanism `[CM4]`. Nothing ready-made shows a reference with a live title or keeps the two marks of an anchor together; that is application code. **NV**

### What it reports about a change

- `EditorView.updateListener` registers "a function to be called every time the view updates". It receives a `ViewUpdate` with `changes` ("The changes made to the document by this update"), `startState`, `state`, `transactions` and `docChanged`. **V** `[CM1]`
- `ChangeSet.iterChanges` calls a function for each changed range with "the range in the original document (`fromA`-`toA`) and the range that replaces it in the new document (`fromB`-`toB`)", and the inserted text. `iterChangedRanges` gives the ranges without the text. **V** `[CM1]`
- A transaction can carry annotations. `Transaction.userEvent` describes what the user did (typing, pasting, undo); `Transaction.remote` marks "a change made by some other actor, not the user". **V** `[CM1]`
- Positions are offsets in the document string. Since that string is the body, the stretch a change touched is read directly from the change. This follows from the above.

### A change from outside

- Without a collaboration layer: a change is dispatched as a transaction (`{from, to, insert}`). "when there are document changes, but no explicit new selection, the selection will be implicitly mapped through these changes". **V** `[CM2]`
- A position can be mapped through a change with a side (`assoc`), and the mapping can report a deletion (`MapMode.TrackDel`: "Return null if deletion happens across the position"). A change set can be mapped over another that started from the same document: "This provides a basic form of operational transformation". **V** `[CM1]`
- `@codemirror/collab`, with no Yjs: "the editor will accumulate local changes, which can be sent to a central service. When new changes are received from the service, they can be applied to the state with `receiveUpdates`." **V** `[CM1]`
- How that module treats text the human has not sent yet: "If there are unconfirmed changes present, operational transformation is used to transpose the remote changes across the unconfirmed local ones, and vice versa, so that the remote changes can be applied to the peer's current document, and the updated local changes can be submitted to the server as if they came after the remote changes." **V** `[CM5]`
- The authority is application code: "you must implement the authority (which can be very simple)". When updates arrive with an old version number it can "either reject the updates, or rebase and accept them" (`rebaseUpdates`). **V** `[CM5][CM1]`
- With Yjs: `y-codemirror.next` "binds a Y.Text to a CodeMirror editor", with remote cursors and shared undo as separate plugins. **V** `[Y2]`

### Licence and upkeep

- MIT. **V** `[CM7]`
- Latest releases: `@codemirror/view` 6.43.15 (2026-10-19), `@codemirror/state` 6.7.6 (2026-09-22), `@codemirror/lang-markdown` 6.5.2 (2026-08-04), `@lezer/markdown` 1.8.1 (2026-10-19), `@codemirror/collab` 6.1.1 (2023-09-14), `y-codemirror.next` 0.3.6 (2026-08-18). **V** `[N1]`
- The packages read declare no dependency on React. A React wrapper from the CodeMirror project was not found in the pages read. `@uiw/react-codemirror` is a third-party wrapper (MIT, 4.25.12, 2026-09-24); only its registry entry was read. **V** for the registry entry `[N1]`, **NV** for the rest.

## ProseMirror

### What it keeps

- A document is a tree of nodes that conforms to a schema; inline content is a flat sequence of text nodes that carry ranged marks. **V** `[P2]`
- `prosemirror-markdown` "implements a ProseMirror schema that corresponds to the document schema used by CommonMark, and a parser and serializer to convert between ProseMirror documents in that schema and CommonMark/Markdown text". The parser "uses markdown-it to tokenize a file". It is described as a "non-core" module. **V** `[P3]`
- The default parser parses "unextended CommonMark, without inline HTML". **V** `[P3]`
- The README makes no statement about whether Markdown comes out as it went in. **V** `[P3]`
- The serializer's source (version 1.13.9) writes fixed forms, whatever the input used: `*` for emphasis and `**` for strong; headings as `#` lines; every code block fenced with backticks; a rule as `---` and a bullet as `*` unless the node has an attribute that says otherwise, and the default schema gives it none; ordered items numbered from the list's start with `.`; a hard break as a backslash before the newline. It also escapes characters that would otherwise start Markdown syntax. The default schema keeps whether a list is tight and where an ordered list starts. **V** `[P4]`
- So a body written with `-` bullets, `_` emphasis, an indented code block or a `Heading` underlined with `===` comes back in the other form, in every place, whether the human edited there or not. This is read from the source; it was not run, and the full list of differences is not known. **NV**
- A serializer and parser can be configured for another dialect: the parser takes a markdown-it instance and a table of tokens; the serializer takes a function per node and an opening and closing string (or function) per ranged mark. **V** `[P3][P4]`

### Something that carries an id

- Node types and mark types have attributes (`attrs`), so both can carry an id. **V** `[P1]`
- For a reference: an inline node with `atom` set "doesn't have directly editable content and should be treated as a single unit in the view". How a node is drawn can be replaced with a node view. **V** `[P1]` A reference shown with a title that changes later is application code. **NV**
- For an anchor, as a ranged mark: a mark type is `inclusive` by default, "active when the cursor is positioned at its end", and it is exclusive "with marks of the same type" by default; a schema can let several marks of one type coexist "as long as they have different attributes". **V** `[P1]` `[#15]`
- For an anchor, as two inline nodes, one at each end: the schema allows it, and each node is a unit that can be deleted alone. Nothing read ties two nodes together or reports that one of a pair is gone. **NV**
- Writing to Markdown: the node's serializer function or the mark's opening and closing functions receive the node or mark, so they can write its id. Reading from Markdown: a markdown-it rule has to produce a token, and the token table maps it to a node or mark with `getAttrs`. **V** `[P3][P4]` No such syntax exists in the package; it is application code.

### What it reports about a change

- Every change to the state is a transaction. A transaction is a transform: it has `steps`, `docChanged`, the documents before each step and a `mapping`. **V** `[P1]`
- Each step has a step map, whose `forEach` gives every changed range as `oldStart`, `oldEnd`, `newStart`, `newEnd`. `Transform.changedRange()` returns "a single range, in post-transform document positions, that covers all content changed by this transform"; it ignores "changes that add/remove marks without replacing the underlying content". **V** `[P1]`
- `prosemirror-changeset` "condenses a number of step maps down to a flat sequence of replacements", each with `fromA`, `toA`, `fromB`, `toB`, and can be "built up incrementally". **V** `[P5]`
- A transaction carries metadata; the view marks paste, cut and drop, and code can add its own. **V** `[P1]`
- All these positions are positions in the tree (a count of tokens), not offsets in the Markdown. `MarkdownSerializer.serialize` returns the string and nothing else. **V** `[P1][P4]` Nothing read gives the stretch of Markdown that a step touched. **NV**

### A change from outside

- Without a collaboration layer: a transaction that replaces a range. Its selection "defaults to the editor selection mapped through the steps in the transaction". Position mapping "is used, for example, for updating the selection when the document changes". **V** `[P1]`
- `prosemirror-collab`, with no Yjs: "a central authority which determines in which order changes are applied". When the authority has accepted someone else's changes first, the editor has to "rebase its local changes on top of those from the other editor, and try to submit them again". **V** `[P2]`
- `receiveTransaction` creates "a transaction that represents a set of new steps received from the authority". Its option `mapSelectionBackward` makes "content inserted at the cursor" end up "after the cursor"; "Users usually prefer this, but it isn't done by default". `sendableSteps` gives the steps not yet confirmed. **V** `[P1]`
- A rebased step can fail: "some steps don't make sense anymore when, for example, the content they applied to has been deleted". **V** `[P2]`
- The changes exchanged are steps on the tree. A replacement of a stretch of Markdown would first have to become a change to the tree. How was not looked for beyond the parser. **NV**
- With Yjs: `y-prosemirror` "maps a Y.XmlFragment to the ProseMirror state". **V** `[Y1]` What is shared is the tree, not the Markdown. Its README advises relative positions "for anything that outlives a single transaction or leaves the local editor — cursors, comments, annotations, stored selections". **V** `[Y3]`

### Licence and upkeep

- MIT. **V** `[P3]`
- Latest releases: `prosemirror-model` 1.25.13 (2026-11-08), `prosemirror-transform` 1.12.5 (2026-11-04), `prosemirror-view` 1.42.9 (2026-10-31), `prosemirror-markdown` 1.13.9 (2026-10-22), `prosemirror-changeset` 2.4.6 (2026-11-02), `prosemirror-state` 1.4.4 (2025-10-23), `prosemirror-collab` 1.3.1 (2023-05-17). **V** `[N1]`
- The project's repositories have moved from GitHub to `code.haverbeke.berlin`; the GitHub copy of `prosemirror-markdown` was at 1.13.4 when read. **V** `[P3]`
- The packages read declare no dependency on React. React bindings for bare ProseMirror were not looked at. **NV**

## Tiptap

Tiptap is a framework over ProseMirror. What is said above about ProseMirror's tree, steps and position mapping holds for it.

### What it keeps

- A ProseMirror document, read and written as JSON. **V** `[T1]`
- `@tiptap/markdown` "provides bidirectional Markdown support": it parses "Markdown strings into Tiptap's JSON format" and serializes back. Its pages are tagged beta and open with: "The markdown extension is a early release and can be subject to change or may have edge cases that may not be supported yet." **V** `[T1]`
- The path is Markdown, the MarkedJS lexer, tokens, each extension's parse handler, JSON; and back through each extension's render handler. **V** `[T1]`
- Stated limits: "Comments are not supported yet ... Be cautious when parsing Markdown content into a document that contains comments as they may be lost if replaced by Markdown content", and one child node per table cell. **V** `[T1]`
- On fidelity the documentation says only: "Test round-trip conversion to ensure your custom Markdown content survives parse → serialize", and "Include all needed extensions or content may be lost". **V** `[T2]`
- The render handlers in the source write fixed forms: strong as `**`, emphasis as `*`, a bullet as `- `, a rule as `---`, a code block fenced with backticks, a hard break as two spaces and a newline. **V** `[T6]` So Markdown in another form comes back changed. Not run. **NV**
- HTML inside Markdown is parsed with the extensions' HTML rules; HTML with a tag no extension knows is kept as literal text. **V** `[T2][T6]`
- A second, paid package (`@tiptap-pro/extension-export-markdown`) exports a file "using the `@tiptap/markdown` extension". **V** `[T12]` An older community package, `tiptap-markdown` (MIT, 0.9.0, 2025-09-08), exists; only its registry entry was read. **V** `[N1]`

### Something that carries an id

- The Mention node is inline, `atom`, not selectable, with attributes `id`, `label` and the trigger character. It is drawn as the trigger character followed by the label, or the id when there is no label. **V** `[T7]`
- In Markdown the Mention node is written as a shortcode, not as a link: `[@ id="…" label="…"]`. The helper that does it, `createInlineMarkdownSpec`, "Creates a Markdown specification for inline nodes using shortcode syntax (`[nodeName]`)"; its documented example is `Hey [mention id="user123" label="John"]!`. **V** `[T3][T7]`
- The label is an attribute stored in the document. A label that follows a later rename is application code (a node view). **NV**
- Each extension can define its own Markdown: `parseMarkdown`, `renderMarkdown` and a `markdownTokenizer` for syntax MarkedJS does not know. The documentation has guides for a custom ranged mark and a custom inline node. **V** `[T2][T3]` So another syntax for a reference, or one for the marks of an anchor, can be written; none is shipped.
- A ranged mark has the ProseMirror options. Tiptap's page says "By default, marks are not inclusive", while the ProseMirror reference it points to says the option "Defaults to true". **V** for both statements `[T9][P1]`; which holds in Tiptap **NV**.
- The editor emits a `delete` event: "Content was deleted from the editor (either a node or mark)", with the range before and after, the transaction, and `partial`, "Whether the deletion was partial". **V** `[T4][T5]`
- Tiptap's Comments extension keeps its anchor as an `inlineThread` ranged mark with a thread id `[#15]`. It needs a subscription `[T10]`, and by the limit quoted above it is not written to Markdown `[T1]`. **V**

### What it reports about a change

- `update` fires "When there is a change in the content" and carries the editor, the ProseMirror transaction and the transactions plugins appended. `transaction` fires on every state change. **V** `[T4][T5]`
- `editor.getMarkdown()` returns the whole content as a string. **V** `[T2]` Positions in the transaction are tree positions, as in ProseMirror. A map from them to offsets in the Markdown was not found. **NV**

### A change from outside

- `insertContentAt({ from, to }, value, { contentType: 'markdown' })` replaces a range with parsed Markdown. Its option `updateSelection` "controls if the selection should be moved to the newly inserted content". `setContent` replaces the whole content. **V** `[T2][T13]` The range is given in tree positions.
- The Collaboration extension syncs through Yjs and "comes with its own history extension". **V** `[T8]` Content the local schema cannot show is removed by default, and "These changes are synced, so other users lose that content too"; a setting keeps it and pauses editing instead. **V** `[T8]`
- Tiptap's hosted or on-premises collaboration server has an endpoint to inject content from a server: "Users can still collaborate in real-time as content is injected." It takes JSON or a Yjs update, can target nodes by an attribute, and needs the Team plan. **V** `[T14]` It does not take a passage of Markdown.

### Licence and upkeep

- "Tiptap's open source is published on GitHub under the MIT license. When you need advanced features like comments or AI commands drop in paid Tiptap extensions." **V** `[T10]`
- "A Tiptap account is required to access Pro extensions. Select extensions such as Snapshots, Comments, and some features of AI Toolkit also require an active subscription." **V** `[T11]`
- MIT in the registry: `@tiptap/core`, `@tiptap/markdown`, `@tiptap/react`, `@tiptap/extension-mention`, `@tiptap/extension-collaboration`, `@tiptap/y-tiptap`. All at 3.34.0, released 2026-11-13, except `@tiptap/y-tiptap` 3.3.0 (2026-11-10). **V** `[N1]`
- `@tiptap/react` accepts React 17, 18 or 19. The Next.js page uses a client component and `immediatelyRender: false` to avoid a hydration mismatch. **V** `[N1][T15]`

## Lexical

### What it keeps

- An editor state: a tree of nodes and a selection. Nodes point to each other by key. **V** `[L5]`
- `@lexical/markdown` converts with a list of *transformers* passed to import and export (`$convertFromMarkdownString`, `$convertToMarkdownString`). **V** `[L1]` Lexical's own comparison describes its parser as "bespoke regular expressions" and its syntax preservation as "partial". **V** `[L3]`
- Its export changes text to protect it: "Literal brackets and guarded `<`/`&` use numeric references so escaping does not change adjacent emphasis flanking." **V** `[L2]`
- `@lexical/mdast` is a second package on micromark and mdast, for CommonMark and GFM. It is "marked `@experimental` and may change between any two Lexical releases"; "`@lexical/markdown` ... remains the supported default". **V** `[L3]`
- Its documentation has a section titled "Round-trips are close, not exact". It keeps "the literal syntax of each construct" on the nodes: the list bullet, the ordered delimiter, the fence style, setext or ATX headings, the hard-break style, soft line breaks, the rule marker, `*` or `_` for emphasis ("one of each for the whole document"), and the link style. "Some forms are normalized", such as bold and italic together (`**_both_**` comes back as `_**both**_`) and GFM tables, which are padded. **V** `[L3]`

### Something that carries an id

- Ready-made for an anchor: `@lexical/mark`. "Comment threads, review suggestions and similar annotations mark ranges of text with an id." Two ways: `MarkNode`, "an inline `ElementNode` that wraps the marked text", and `MarkStateExtension` (experimental), which stores the ids "on the TextNodes, LineBreakNodes and inline DecoratorNodes it covers, with no wrapper element". **V** `[L6]` `[#15]`
- These are ranges with an id, not two point marks. A pair of point marks would be two custom inline nodes; nothing read covers that. **NV**
- What happens when a human edits, as documented for `MarkStateExtension`: "Typing inside a marked run adds to the TextNode that carries the ids"; "text inserted exactly at the edge of a run stays outside it, as with `MarkNode`"; "Copying or cutting text carries its ids, and deleting it deletes them"; text that replaces a selection gets "the ids that all of the replaced text had, so retyping a commented word keeps the comment (`MarkNode` loses the comment when all of its text is replaced)". **V** `[L6]`
- Overlap: "Text covered by two comments is a single TextNode carrying both ids. With `MarkNode`, overlapping comments become sibling wrappers". **V** `[L6]`
- A copy carries the id to a second place: "pasting copied commented text elsewhere puts the same id in a second place". **V** `[L6]`
- When all the text of a thread is deleted the documentation advises: "mark the thread orphaned rather than deleting it: an undo, a redo or a paste of the commented text brings the id back". **V** `[L6]`
- With the Yjs binding, "Concurrent changes to the ids of the same text keep one collaborator's set"; and with the older binding, adding an id splits a text node, which syncs "as a delete and a re-insert", so "typed text can move and concurrent comments can duplicate text". **V** `[L6]`
- To Markdown: no transformer for these ids is documented. In the export source of `@lexical/markdown`, an element node with no transformer is exported as its children, and a decorator node as its text content. **V** `[L10]` So a `MarkNode` is not written to Markdown by default. Whether `MarkStateExtension` ids are written by either Markdown package. **NV**
- For a reference: a custom node. A text node in `token` mode "acts as immutable node, can't change its content and is deleted all at once"; a decorator node is drawn by a React component. **V** `[L8]` A *text match transformer* gives a custom node a Markdown form: a regular expression for import and a function for export. **V** `[L1][L10]`

### What it reports about a change

- `registerUpdateListener` is called "when Lexical commits an update to the DOM", with `editorState`, `prevEditorState`, `tags`, `dirtyElements` (keys of element nodes marked dirty), `dirtyLeaves` (keys of other dirty nodes) and `mutatedNodes`. **V** `[L4]`
- `registerMutationListener` reports nodes of one class as created, updated or destroyed. `registerTextContentListener` reports the new text content. **V** `[L4]`
- An update carries tags, among them `COLLABORATION_TAG`, `HISTORIC_TAG` (undo and redo) and `PASTE_TAG`. **V** `[L5]`
- The documentation on `main` also has `registerMarkIDsListener`, called "after each commit that changes which ids of a flavor are in the document", with the ids `added` and `removed`. **V** `[L6]` It is not among the exports of the published `@lexical/mark` 0.53.0 that were read. **V** `[L11]`
- None of these gives a range of text. Which stretch of the Markdown a change touched is not reported. **V** for the absence in the pages read `[L4][L5]`.

### A change from outside

- A change is made inside `editor.update()`, on nodes. "Lexical's selection is part of the `EditorState`", and a point of a selection is a node key and an offset in that node. **V** `[L5]`
- `editor.setEditorState` "replaces current state with the one passed"; a state can be cloned "with custom selection", for example `null` "to prevent focusing the editor". **V** `[L5]`
- What happens to the human's caret when an update from outside replaces the node the caret is in. **NV**
- With Yjs: `@lexical/yjs` and, for React, `CollaborationPlugin`. The FAQ says: "It's recommended to treat the Yjs model as the source of truth." **V** `[L7]` What is shared is the node tree.
- No module was found that takes changes from a central authority without Yjs. **NV**

### Licence and upkeep

- MIT. **V** `[L9]`
- Latest release 0.53.0 (2026-10-28) for `lexical`, `@lexical/markdown`, `@lexical/mdast`, `@lexical/mark`, `@lexical/yjs`, `@lexical/react`; nightly builds are published. Still below version 1. **V** `[N1]`
- `@lexical/react` asks for React 18 or later. **V** `[N1]`

## Milkdown

- Built on ProseMirror and remark. Parsing: "Markdown text → Remark AST", "Remark AST → ProseMirror Schema", "Schema → ProseMirror Document". Serializing goes back the same way to "Markdown text". **V** `[M1]`
- The output form is set by the options of remark's serializer, which Milkdown exposes (`remarkStringifyOptionsCtx`; the example sets `bullet`, `fences`, `incrementListMarker`). **V** `[M2]` That serializer chooses one form per construct: by default `*` for bullets, emphasis and strong, fenced code "always", `*` for rules, no setext headings. **V** `[R1]` Milkdown's own defaults for these options. **NV**
- No statement about Markdown coming out as it went in. **V** `[M1][M2]`
- An id: a custom node or ranged mark is defined with a ProseMirror schema plus `parseMarkdown` and `toMarkdown` rules that match and build remark nodes; remark plugins can be added (`$remark`), and the documented example uses directives. **V** `[M4]`
- A change: the listener plugin gives `markdownUpdated(ctx, markdown, prevMarkdown)` (two whole strings, with the warning that it "will influence the performance for large documents") and `updated(ctx, doc, prevDoc)`. **V** `[M2][M3]` "You can use the entire ProseMirror API in Milkdown", `prosemirror-changeset` included. **V** `[M5]`
- From outside: the `replaceAll` macro replaces the whole content; `insert` puts Markdown at the selection. **V** `[M3]` A replacement of one range is a ProseMirror transaction. **V** `[M5]`
- Collaboration: `@milkdown/plugin-collab`, "powered by Y.js", through `y-prosemirror`. **V** `[M6]`
- MIT; `@milkdown/kit`, `@milkdown/react` and `@milkdown/plugin-collab` 7.23.0 (2026-10-27). **V** `[N1]`

## BlockNote

- "BlockNote builds directly on two awesome projects; Prosemirror ... and Tiptap." Its document is an array of blocks, each with an id that "will keep the same ID from when it's created until it's removed". **V** `[B5][B2][B8]`
- Markdown: "The functions to export to Markdown are considered 'lossy'; some information might be dropped", and the same sentence for import. "BlockNote ships a minimal Markdown parser"; supporting every Markdown dialect "is not a goal of BlockNote". Export: "children of blocks which aren't list items are un-nested and certain styles are removed". **V** `[B1]`
- It recommends its own JSON for storage, "the most durable format & guaranteed to be lossless". **V** `[B2]`
- An id: custom inline content has a `propSchema`, "data stored with your inline content in the document", and HTML rules for export and paste. **V** `[B4]` How custom inline content is written to Markdown. **NV**
- A change: `onChange(editor, { getChanges })`; `getChanges()` lists blocks inserted, deleted, updated or moved, each with a source (`local`, `paste`, `drop`, `undo`, `redo`, `yjs-remote`). `onBeforeChange` can cancel a change. **V** `[B3]` The unit is the block.
- From outside: blocks are updated, replaced or removed by block id. **V** `[B7]` Collaboration is through Yjs. **V** `[B6]` Comments are an extension with a thread store. **V** `[B6]`
- Licence: "The majority of BlockNote is licensed under the MPL-2.0 license ... The XL packages ... are licensed under the GPL-3.0. If you cannot comply with this license and want to use the XL libraries, you'll need a commercial license." **V** `[B5]` The exporters to PDF, DOCX, ODT and email are `xl-` packages. **V** `[B2]`
- `@blocknote/core` and `@blocknote/react` 0.57.0 (2026-10-30); React 18 or 19. **V** `[N1]`

## Plate

- Built on Slate; its document is a Slate value (JSON). **V** `[S1]`
- `@platejs/markdown` converts through remark: "Supports CommonMark, with GFM ... available via `remark-gfm`". **V** `[S1]`
- "Round-Trip Serialization: Preserves custom elements through MDX syntax during conversion cycles." Custom elements with no Markdown form "(e.g., underline, mentions)" are written as MDX elements and parsed back. **V** `[S1]` This is about custom elements. Whether untouched standard Markdown comes out as it went in is not stated. **NV**
- Serializing uses remark's serializer, "with Plate setting emphasis to `_` and resource links to `false`". **V** `[S1]`
- Raw HTML in Markdown "stays editable source text ... written back to Markdown unchanged". **V** `[S1]`
- A reference: mentions are written as a link that carries an id. "The `remarkMention` plugin uses the **[display text](mention:id)** format"; "When serializing, all mentions use the link format". **V** `[S1]` This is the closest ready-made form to a reference as the glossary defines it; the display text is stored, not looked up.
- An anchor: comments are "text marks with inline annotations", and "multiple comments on the same text" are supported. **V** `[S2]` How a comment mark is written to Markdown is not in the Markdown page. **NV** That page has an option that wraps blocks as `<block id="...">content</block>` "to enable AI comment tracking". **V** `[S1]`
- A change: `onChange({ editor, value })` and `onValueChange({ value })`. **V** `[S3]` Underneath, Slate records every change as operations with a path and an offset (`insert_text`, `remove_node`, …), kept in `editor.operations`. **V** `[S4]`
- From outside: "Plate is not a normal controlled text input. The editor owns content, selection, history, plugin state, and normalization." `editor.tf.setValue(value)` "replaces the document"; "Use transforms for external changes". **V** `[S3]` Slate's documentation on `main` has `Editor.withRemoteChanges`: inside it, for an expanded selection, "text inserted at either edge stays outside the selected content". **V** `[S4]` Whether it is in the published release. **NV**
- Collaboration: `@platejs/yjs`, using Yjs and slate-yjs. **V** `[S5]`
- MIT; `platejs` 53.5.2 (2026-10-30), `@platejs/markdown` 53.5.3 (2026-11-04), `@platejs/yjs` 53.5.6 (2026-11-11); React 18 or later. **V** `[N1][S6]` The README lists a template hosted at `pro.platejs.org`; what it costs was not read. **NV**

## MDXEditor

- "an open-source React component that lets users author markdown documents in a WYSIWYG manner ... there is no intermediate HTML representation of the markdown document. The component accepts and emits markdown as a string." **V** `[X1]`
- Inside, it is Lexical, with its own conversion "between the Markdown Abstract Syntax Tree (MDAST) and the Lexical AST"; it uses `@lexical/markdown` "only for the provided markdown shortcuts". **V** `[X1]`
- Markdown can change without the human writing. `onChange` has a second argument, `initialMarkdownNormalize`, "set to true if the change is triggered when the initial markdown is set. This can happen due to variety of reasons - for example, additional whitespace, bullet symbols different than the configured ones, etc." The initial Markdown is trimmed by default. Output options are those of remark's serializer. **V** `[X3][R1]`
- It has a source mode: a plugin "allows the user to switch to editing the markdown source of the document or to compare it to the initial version". **V** `[X4]`
- An id: directives (`::youtube[Video of a cat in a box]{#01ab2cd3efg}`) and JSX components can have custom editors. **V** `[X4]` A ranged mark with an id. **NV**
- A change: `onChange(markdown, initialMarkdownNormalize)` gives the whole string and "is not throttled". **V** `[X3]`
- From outside: `setMarkdown` replaces the content; `insertMarkdown` inserts "at the current cursor position". The `markdown` property is read once, "like the textarea `defaultValue`". **V** `[X2][X3]` Where the caret goes after `setMarkdown`. **NV**
- No collaboration is mentioned in its documentation. **V** `[X2]`
- It "does not support server rendering"; on Next.js it is loaded with `dynamic` and `ssr: false`. **V** `[X2]`
- MIT; `@mdxeditor/editor` 4.5.0 (2026-11-02); React 18 or 19. **V** `[X5][N1]`

## Sources

CodeMirror:

- `[CM1]` CodeMirror Reference Manual: https://codemirror.net/docs/ref/
- `[CM2]` CodeMirror System Guide: https://codemirror.net/docs/guide/
- `[CM4]` Example: Decorations: https://codemirror.net/examples/decoration/
- `[CM5]` Example: Collaborative Editing: https://codemirror.net/examples/collab/
- `[CM6]` `@codemirror/lang-markdown` 6.5.2 README: https://unpkg.com/@codemirror/lang-markdown@6.5.2/README.md and `@lezer/markdown` 1.8.1 README: https://unpkg.com/@lezer/markdown@1.8.1/README.md
- `[CM7]` codemirror/dev, `LICENSE`: https://github.com/codemirror/dev/blob/main/LICENSE

ProseMirror:

- `[P1]` ProseMirror Reference manual: https://prosemirror.net/docs/ref/
- `[P2]` ProseMirror Guide: https://prosemirror.net/docs/guide/
- `[P3]` `prosemirror-markdown` 1.13.9 README: https://unpkg.com/prosemirror-markdown@1.13.9/README.md (project home: https://code.haverbeke.berlin/prosemirror/prosemirror-markdown)
- `[P4]` `prosemirror-markdown` 1.13.9 source, `src/to_markdown.ts`, `src/from_markdown.ts`, `src/schema.ts`: https://unpkg.com/prosemirror-markdown@1.13.9/src/to_markdown.ts
- `[P5]` `prosemirror-changeset` 2.4.6 README: https://unpkg.com/prosemirror-changeset@2.4.6/README.md

Tiptap (documentation read from its repository, ueberdosis/tiptap-docs, commit 9a96bbbc3d; source from ueberdosis/tiptap, commit 93574965ba):

- `[T1]` Markdown, Introduction: https://tiptap.dev/docs/editor/markdown
- `[T2]` Markdown, Basic Usage and Editor API: https://tiptap.dev/docs/editor/markdown/getting-started/basic-usage and https://tiptap.dev/docs/editor/markdown/api/editor
- `[T3]` Markdown, Utilities and guides: https://tiptap.dev/docs/editor/markdown/api/utilities
- `[T4]` Events: https://tiptap.dev/docs/editor/api/events
- `[T5]` `packages/core/src/types.ts` (event payloads): https://github.com/ueberdosis/tiptap/blob/main/packages/core/src/types.ts
- `[T6]` Markdown handlers in the source: `packages/extension-bold`, `-italic`, `-heading`, `-horizontal-rule`, `-list`, `-code-block`, `-hard-break`, `-link`, and `packages/markdown/src/MarkdownManager.ts`: https://github.com/ueberdosis/tiptap/tree/main/packages
- `[T7]` Mention: https://tiptap.dev/docs/editor/extensions/nodes/mention and `packages/extension-mention/src/mention.ts`
- `[T8]` Collaboration extension: https://tiptap.dev/docs/editor/extensions/functionality/collaboration
- `[T9]` Mark API and Node API: https://tiptap.dev/docs/editor/extensions/custom-extensions/create-new/mark and https://tiptap.dev/docs/editor/extensions/custom-extensions/create-new/node
- `[T10]` Editor overview: https://tiptap.dev/docs/editor/getting-started/overview
- `[T11]` How to integrate Tiptap Pro extensions: https://tiptap.dev/docs/guides/pro-extensions
- `[T12]` Conversion, Export Markdown from your editor: https://tiptap.dev/docs/conversion/export/markdown/editor-extension
- `[T13]` `insertContentAt` command: https://tiptap.dev/docs/editor/api/commands/content/insert-content-at
- `[T14]` Collaboration, Inject content REST API: https://tiptap.dev/docs/collaboration/documents/content-injection
- `[T15]` Install, Next.js: https://tiptap.dev/docs/editor/getting-started/install/nextjs

Lexical (documentation and source read from facebook/lexical, branch `main`, commit 35e70c9662):

- `[L1]` `@lexical/markdown` README, `main`: https://github.com/facebook/lexical/blob/main/packages/lexical-markdown/README.md
- `[L2]` `@lexical/markdown` 0.53.0 README, as published: https://unpkg.com/@lexical/markdown@0.53.0/README.md
- `[L3]` Markdown with @lexical/mdast: https://lexical.dev/docs/serialization/markdown-mdast and `@lexical/mdast` 0.53.0 README: https://unpkg.com/@lexical/mdast@0.53.0/README.md
- `[L4]` Listeners: https://lexical.dev/docs/concepts/listeners
- `[L5]` Editor State, Updates, Selection, Key Management: https://lexical.dev/docs/concepts/editor-state , https://lexical.dev/docs/concepts/updates , https://lexical.dev/docs/concepts/selection , https://lexical.dev/docs/concepts/key-management
- `[L6]` Comments and Annotations: https://lexical.dev/docs/collaboration/comments
- `[L7]` Collaboration, React and FAQ: https://lexical.dev/docs/collaboration/react and https://lexical.dev/docs/collaboration/faq
- `[L8]` Nodes: https://lexical.dev/docs/concepts/nodes
- `[L9]` `LICENSE`: https://github.com/facebook/lexical/blob/main/LICENSE
- `[L10]` `packages/lexical-markdown/src/MarkdownExport.ts` and `MarkdownTransformers.ts`: https://github.com/facebook/lexical/tree/main/packages/lexical-markdown/src
- `[L11]` `@lexical/mark` 0.53.0, `dist/MarkStateExtension.d.ts`: https://unpkg.com/@lexical/mark@0.53.0/dist/MarkStateExtension.d.ts

Yjs bindings:

- `[Y1]` `y-prosemirror` 1.3.7 README: https://github.com/yjs/y-prosemirror/tree/v1.3.7#readme
- `[Y2]` `y-codemirror.next` 0.3.6 README: https://unpkg.com/y-codemirror.next@0.3.6/README.md
- `[Y3]` `y-prosemirror` README on `main` (documents the unreleased `@y/prosemirror`): https://github.com/yjs/y-prosemirror/blob/master/README.md

Milkdown (documentation read from Milkdown/website, commit 3cd83521d8):

- `[M1]` Architecture Overview and Why Milkdown: https://milkdown.dev/docs/guide/architecture-overview
- `[M2]` FAQ: https://milkdown.dev/docs/guide/faq
- `[M3]` Interacting with Editor and Macros: https://milkdown.dev/docs/guide/interacting-with-editor and https://milkdown.dev/docs/guide/macros
- `[M4]` Composable Plugins and plugin examples: https://milkdown.dev/docs/plugin/composable-plugins
- `[M5]` ProseMirror API: https://milkdown.dev/docs/guide/prosemirror-api
- `[M6]` Collaborative Editing: https://milkdown.dev/docs/guide/collaborative-editing

remark:

- `[R1]` `mdast-util-to-markdown` README, Options: https://github.com/syntax-tree/mdast-util-to-markdown#options

BlockNote (read from TypeCellOS/BlockNote, commit 58a40d2f55):

- `[B1]` Markdown Export and Markdown Import: https://www.blocknotejs.org/docs/features/export/markdown and https://www.blocknotejs.org/docs/features/import/markdown
- `[B2]` Format Interoperability: https://www.blocknotejs.org/docs/foundations/supported-formats
- `[B3]` Events: https://www.blocknotejs.org/docs/reference/editor/events
- `[B4]` Custom Inline Content: https://www.blocknotejs.org/docs/features/custom-schemas/custom-inline-content
- `[B5]` README, License: https://github.com/TypeCellOS/BlockNote/blob/main/README.md
- `[B6]` Collaboration and Comments: https://www.blocknotejs.org/docs/features/collaboration
- `[B7]` Manipulating Content: https://www.blocknotejs.org/docs/reference/editor/manipulating-content
- `[B8]` Document Structure: https://www.blocknotejs.org/docs/foundations/document-structure

Plate and Slate (Plate read from udecode/plate, commit e401aaf96b; Slate from ianstormtaylor/slate, branch `main`):

- `[S1]` Plate, Markdown: https://platejs.org/docs/markdown
- `[S2]` Plate, Comment: https://platejs.org/docs/comment
- `[S3]` Plate, Controlled value: https://platejs.org/docs/controlled
- `[S4]` Slate, Operations, Editor API and RangeRef API: https://docs.slatejs.org/concepts/05-operations and https://docs.slatejs.org/api/nodes/editor
- `[S5]` Plate, Collaboration (Yjs): https://platejs.org/docs/yjs
- `[S6]` Plate README and `LICENSE`: https://github.com/udecode/plate

MDXEditor (read from mdx-editor/editor, commit 0d0cfbaabc):

- `[X1]` Overview: https://mdxeditor.dev/editor/docs/overview
- `[X2]` Getting started: https://mdxeditor.dev/editor/docs/getting-started
- `[X3]` `src/MDXEditor.tsx` (component properties and their descriptions): https://github.com/mdx-editor/editor/blob/main/src/MDXEditor.tsx
- `[X4]` Directives; Diff/source mode: https://mdxeditor.dev/editor/docs/custom-directive-editors and https://mdxeditor.dev/editor/docs/diff-source
- `[X5]` README, License: https://github.com/mdx-editor/editor/blob/main/README.md

Registry and earlier findings:

- `[N1]` npm registry, package metadata (latest version, its date, licence, peer dependencies), one request per package: https://registry.npmjs.org/
- `[#15]` Findings on comment anchors: https://github.com/TommyBez/agent-intelligence/blob/research/comment-anchors/docs/research/comment-anchors.md

## Not verified

| Claim | Sources checked | Why it stays unverified |
| - | - | - |
| Any tree editor: that Markdown comes out unchanged where the human did not write, and exactly what changes | `[P3][P4][T1][T2][T6][L1][L3][M1][S1][X3]` | No project states it. The fixed output forms were read in source or documentation; nothing was run, so the full list of differences for a real body is not known. |
| ProseMirror, Tiptap, Milkdown: the stretch of Markdown that a step touched | `[P1][P4][T2][M2]` | Steps give tree positions; the serializers return a string only. No map between the two was found in the pages read. Not looked for in third-party packages. |
| ProseMirror, Lexical: an anchor held as two point nodes, and what happens when one is removed | `[P1][L6][L8]` | Both document ranged marks and single inline nodes. Nothing covers a pair. |
| Any editor: a reference shown with the current title of what it points to | `[P1][T7][L8][CM4][S1]` | Each documents how to draw a node or widget. The stored label is static in Tiptap's Mention and Plate's mention. Looking a title up is application code. |
| Tiptap: whether a ranged mark is inclusive by default | `[T9][P1]` | Tiptap's page and the ProseMirror reference it links to say opposite things. |
| Tiptap: how much Markdown changes on a round trip | `[T2][T6]` | Read from the render handlers; the package is beta and was not run. |
| Lexical: whether `MarkStateExtension` ids are written by `@lexical/markdown` or `@lexical/mdast` | `[L1][L3][L6][L10]` | Not stated. The export source was read only for nodes without a transformer. |
| Lexical: what happens to the caret when an update from outside replaces the node it is in | `[L5]` | The selection is described; this case is not. |
| Lexical: a way to take changes from a central authority without Yjs | `[L5][L7]` | None found in the pages read. |
| Lexical: which parts of the comments page are in the published 0.53.0 | `[L6][L11]` | The page was read on `main`. The published type file has `MarkStateExtension`, `$addMarkID`, `$removeMarkID`; `defineMarkExtension` and `registerMarkIDsListener` were not in it. |
| CodeMirror: a ready-made way to show a reference with a title or to keep the two marks of an anchor | `[CM1][CM4]` | The pieces are documented (decorations, atomic ranges, parser extensions). No package that does it was looked for. |
| CodeMirror, ProseMirror: a React binding from the project itself | `[CM1][P1][N1]` | None in the pages read. Third-party bindings were not read. |
| Milkdown: its default serializer options | `[M2][R1]` | The FAQ shows how to set them, not what they are. |
| BlockNote: how custom inline content and comments are written to Markdown | `[B1][B4][B6]` | Not stated. |
| Plate: untouched standard Markdown on a round trip; comment marks in Markdown | `[S1][S2]` | The round-trip claim is about custom elements. The Markdown page does not mention comment marks. |
| Plate and Slate: whether `Editor.withRemoteChanges` is released; what the `pro.platejs.org` template costs | `[S4][S6]` | Slate's documentation was read on `main`; the template's page was not opened. |
| MDXEditor: a ranged mark with an id; the caret after `setMarkdown` | `[X2][X3][X4]` | Not stated. |
| Every editor: behaviour under the React Compiler; behaviour on Next.js 16 | `[N1][T15][X2]` | Only peer dependency ranges and two installation pages were read. |
| Every editor: what a human actually sees and feels when one range is replaced under the caret | all | Nothing was run. |

Reading caveats:

- Pages were fetched as HTML and reduced to text, or read as the Markdown source that the project's documentation repository holds. Nothing was read through a summarizing model.
- The documentation of Tiptap, Lexical, Milkdown, BlockNote, Plate and MDXEditor was read from the repositories that hold it, at the commits given in [Sources](#sources). The published sites are built from those files; the site pages themselves were not opened, and the page addresses given are where each file is published.
- Lexical's files were read on `main`, which is ahead of the published 0.53.0. For the two statements that differ, the published README is cited (`[L2]`) and the difference in exports is noted (`[L11]`).
- ProseMirror's and CodeMirror's GitHub repositories are no longer the projects' homes; the GitHub copy of `prosemirror-markdown` was five patch releases behind. For those packages the files inside the published npm package were read instead (`[P3][P4][P5][CM6]`).
- `[Y3]` documents a new, unreleased version of the Yjs binding for ProseMirror; the released one is `[Y1]`.
- Release dates and licences come from one registry request per package, on the reading date. The registry's `latest` tag is what is reported.
- `[P4]`, `[T5]`, `[T6]`, `[L10]`, `[L11]` and `[X3]` are source or type files. They show what the code does, not what the project promises.
- Not looked at: Toast UI Editor (latest release 3.2.2, 2023-02-17, by the registry), Remirror, bare Slate without Plate, Quill, Monaco, `@uiw/react-md-editor`, the community `tiptap-markdown` package, Automerge and Liveblocks bindings, and any paid editor.

## The two kinds of editor set against what is already decided

This section sets the findings against decisions already recorded in `docs/adr`. It does not choose an editor. Where it reasons beyond a source, it says so.

### An editor whose document is the Markdown text

Only CodeMirror 6 among those read.

- *The body is the document.* What the human did not write is not rewritten, because nothing is serialised `[CM2]`. A reference and the marks of an anchor are in the text as the agent reads them.
- *The stretch a change touched is reported in the same units as the body*: offsets in the text `[CM1]`. A change from outside to a stretch is a change of the same kind `[CM2]`.
- *Unsent text and a change from outside are covered by a documented module* with a central authority and no Yjs `[CM5]`. The authority is application code; whether it fits "checked only against the stretch it touches" (ADR 0005) was not worked out. Reasoning, not a source.
- *What the human sees is source text unless the application draws it otherwise.* Showing a reference with its title and hiding the marks of an anchor are decorations the application writes `[CM4]`. ADR 0006 asks that the editor "show an anchor as something other than its marks, and keep the marks when a human edits around them"; here the marks are kept because they are text, and they are lost the way any text is lost. Reasoning.
- *It is not a formatted-text editor.* Headings, lists and emphasis appear as Markdown syntax with styling. Whether that is acceptable belongs to the later ticket.

### An editor whose document is a tree, with Markdown made from it

ProseMirror, Tiptap, Lexical, Milkdown, BlockNote, Plate, MDXEditor.

- *The human edits formatted text.* References and anchors can be nodes or ranged marks that the editor draws as it likes `[P1][T7][L6]`.
- *The Markdown is produced, not kept.* Each project either says the round trip is not exact or shows fixed output forms in its source `[B1][L3][X3][P4][T6]`. By ADR 0004 an agent quotes a passage exactly as it read it. If opening and saving a body changes a bullet or an emphasis character in a stretch the human did not write, that stretch no longer matches what an agent read, and by ADR 0005 it counts as changed by the human. Reasoning from the ADRs; the size of the effect was not measured.
- *A change is reported in the tree's units* (steps and positions, dirty nodes, blocks) `[P1][L4][B3]`. To check a change against the stretch of Markdown it touches, the application would have to turn those into stretches of Markdown, or compare the Markdown before and after. Nothing read does this. Reasoning.
- *An anchor as a ranged mark is the documented form*, with its edge and overlap rules `[P1][L6]`. The glossary's anchor is two marks in the text, each of which can be removed alone; a ranged mark has no ends that can be removed separately. How the two map onto each other, in both directions, is application code in every editor read. Reasoning.
- *A change from outside is applied to the tree.* ProseMirror and Tiptap map the selection through it `[P1][T2]`; ProseMirror has a central-authority module `[P2]`; the others document Yjs, which shares the tree `[Y1][L7][M6][B6][S5]`. ADR 0005 records that "a structure built for writing shared live" was not weighed.

### What holds for both

- Every editor read is maintained and under a licence that allows commercial use; the paid parts found are Tiptap's comments, snapshots and hosted server, and BlockNote's `xl-` packages when GPL-3.0 cannot be met `[T10][T11][T14][B5]`.
- No editor read ships the product's reference (a link with an id, shown with a current title) or its anchor (two marks with a thread id). Plate's mention link and Tiptap's Mention node are the nearest to the first; Lexical's mark packages and Tiptap's paid Comments are the nearest to the second, as ranged marks `[S1][T7][L6]`.
- Nothing here was run. Each behaviour marked **NV** above is a question a prototype would answer.
