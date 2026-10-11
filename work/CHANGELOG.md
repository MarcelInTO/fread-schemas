# fread work — version history

The XML serialization of a **work's** structure: divisions, blocks, anchors and
illustrations. Companion works use this same schema, since a companion work is
itself a base work — it differs in how it is rendered and in referencing anchors
in another work, not in content shape. Layers and bridges are shaped differently
and get their own schemas alongside this one, each under its own subject path.

The version a file was written against is stamped on its root element as
`export-schema="fread-work/<major>.<minor>"`, and the schema it validates
against is named in `xsi:noNamespaceSchemaLocation` as an absolute URL under
<https://schema.fread.now/>.

## What the two numbers mean

**MAJOR** changes when a file written against an earlier version can no longer
be imported as it stands — an element renamed, an attribute's meaning changed,
a structural rule tightened. Re-export the work and redo the edits.

**MINOR** changes when the addition is one an older file survives unchanged: a
new optional attribute, a new optional element, a new member of an open enum.
`ingest-work import` accepts any file whose major matches and whose minor is
at or below its own, so an edit in flight is not invalidated by a tool upgrade.
It refuses a *higher* minor, because that file may carry an attribute this
build would silently drop.

The `export-state` an import checks a file against fingerprints the work, not
the build that wrote it: the version stamp and the comment at the head of the
file are left out, so an older minor's export of a work that has not changed
since still imports. One version difference does move it, deliberately: a
minor that adds an attribute to the export changes the fingerprint of every
work holding a value for it, and an older file of such a work is refused as
stale, since it cannot carry the value. (Until 2026-10-11 the stamp was in the
fingerprint, so every minor bump refused every export in flight; changing the
fingerprint refused them once more. `fread/fread-issues#564`.)

A consequence worth stating: the published `.../work/<major>/` alias
always serves the newest schema in that major line, and because minors are
additive it still validates every older file in the line. The exact
`<major>.<minor>/` URL is what an export stamps, so a file always names the
schema it was actually written against.

Versions before 4.0 predate this rule — every change was breaking, because the
importer compared the version string for exact equality. They are listed with a
`.0` minor for consistency.

## 4.9

The plain-text projection lists each block's **sentence anchors**, on a
work opted in to them: one `<sentence anchor-id ref c0 c1/>` per sentence,
nested in its block after any `<illustration>`, in order of its index. `c0`
and `c1` are UTF-16 offsets into the block's plain text, `c1` exclusive, and
the ref is the block's ref and `.s` and the index, from 0 (`ch1.p3.s0`). The
root names the splitter version that cut them, `sentence-splitter`
(`fread-sentence/1`).

A work is opted in by `ingest-work sentences`, which pins the splitter on
`base_work.sentence_splitter` and mints the anchors. The text aligner reads
them here, so a bridge row can end on an anchor the ingest keeps, rather
than on a sentence the aligner cut for itself (TranslationBridges §9.2,
fread/fread-issues#520).

Only the projection carries them, because their offsets index the plain
text and only the projection carries that. The authored export is
unchanged, and so is its `export-state`: the projection still carries the
authored tree's. A reader of an older projection sees none, and a reader
that takes a block's text as its own text and the tails of its children
(`(b.text or "") + "".join(c.tail or "" for c in b)`) reads the same text,
since an export writes nothing between a block's children.

## 4.8

`knownBlockKind` gained `argument-line`: one line of an argument set in
verse. An `argument` is a block of prose, and a book that opens on a summary
in couplets — Chapman's *Iliad* sets each book's argument so, under "THE
ARGUMENT" and "ANOTHER ARGUMENT" — holds its argument as verse lines, one
block each. Typed `argument`, those lines would read as a stack of separate
paragraphs, and nothing stored would tell them from an argument of several
prose paragraphs; typed `verse-line`, they read as the poem. A run of
`argument-line` blocks is one argument set as a stanza: the argument's
register, the verse line's lineation.

Like `argument`, it is the operator's to assign (`ingest-override set --kind
argument-line`); no parser emits it. The text aligner leaves it out with
`argument`, as apparatus (TranslationBridges §4).

Found on Chapman's *Iliad* (51355), whose 310 argument lines were
`verse-line` (fread/fread-issues#553, R8 of #506).

## 4.7

`knownDivisionKind` gained `chant`: a numbered division of a long poem
that the work itself heads "Chant", the French word for one. Like `canto`
and `stave`, it composes its word with the division's `number`
("Chant 1"), so the reader prints the word the book prints. It is its
own kind rather than `book` or `canto` because neither English word fits
every work that uses it: Homer's chants are books, and Dante's, Ariosto's
and Byron's are cantos.

The Gutenberg parser assigns it from a head that opens "Chant" and a
numeral; an operator can assign it with `ingest-override set --kind
chant`.

Found on Leconte de Lisle's *Iliade* (14285), whose 24 chants arrived as
chapters with no number (#508).

## 4.6

`knownBlockKind` gained `tabular`: text set in fixed-width columns — a
table, a list in columns, a genealogy drawn in type — kept verbatim, each
line's own indentation and the spaces inside it included, so the reader can
draw it in a fixed-width face and the columns line up. Gutenberg sets most
of its tables this way, as plain text in a `<pre>` or a `div.pre`, and
until now the Gutenberg parser split them into one verse line per row or
stored them as a paragraph, which the reader drew with the spaces
collapsed. Distinct from `preformatted`, where the leading indentation is
the shape (Alice's Mouse's Tale) and the body face keeps it.

The Gutenberg parser recognizes it by the column gap — three or more spaces
inside a line on at least half of a block's lines (`tabular.go`); an
operator can assign it with `ingest-override set --kind tabular`.

Found on The Wealth of Nations (3300), whose price tables arrived run
together (#411).

## 4.5

`knownBlockKind` gained `transcriber-note`: the transcriber's own matter, as
distinct from the author's `footnote` — the DP template's cover credit ("The
cover image was created by the transcriber and is placed in the public
domain", 2,112 works on the mirror, mostly a bare `<p>`), a note on the
transcription. It is the signal table's Apparatus role made real (the
2026-09-10 decision recorded on the `transcriber-note` row): typed and kept,
never dropped at ingest, because a drop is the one thing a re-ingest cannot
undo on a detached work; the reader sets it as apparatus, and an operator
can delete or retype it. The Gutenberg parser recognizes the cover credit by
shape (`transnote.go`); the class row (`div.transnote`, `p.tn`, `div.mynote`,
…) stays Seed until a fixture verifies it, and lands on the same kind.

Found on the illustrated Adventures of Sherlock Holmes (48320), whose title
page opened on the credit as a prose line.

## 4.4

`knownBlockKind` gained the title page's own stack: `title`, `subtitle` and
`byline`. The Gutenberg parser assigns them from the signal → semantics
table (`fread/fread-issues#276`) — the source's classes (`p.author`,
`div.title-main`, `div.byline`) where a template spells them, and the stack's
shape where it does not: the first heading is the title, a heading that
reads "By …" or names a creator the package lists is the byline, and what
sits between is a subtitle. Before this the reader took the first block of a
title-page division as the title and every later one as a byline line, which
put Huckleberry Finn's subtitle "(Tom Sawyer's Comrade)" in the byline. A
`subtitle` is the subtitle of the ENCLOSING division, so under a chapter head
it is that chapter's (`#141`).

Three kinds in one bump rather than three bumps, per the minting rule
(workspace memory `feedback_minting_a_content_kind`). The table's other
targets — footnote, sidenote, caption, imprint, transcriber-note — are still
on inert seed rows and join the enumeration when their rows are verified.

## 4.3

The root gained `projection`, marking a file as a **read-only view** of the
work that `ingest-work import` refuses outright. One value so far:
`plain-text`, written by `ingest-work export --plain-text`, in which every
block's content is `content_block.text` — the derived form the reader shows and
its character offsets index — instead of the authored `text_html`. Blocks in a
projection carry no `text-form` marker, since every one is plain and the
marker's meaning ("this block has no HTML form") would be false.

It exists for tools that compute positions over the reader's text — first the
audiobook alignment tool (`FreadClassicAudioAlignment.md` §6,
`fread/fread-issues#250`). The plain form is deliberately absent from an
ordinary export because it is derived, and a consumer cannot rebuild it: the
derivation has changed over the corpus's life, so tag-stripping `text_html`
disagrees with the stored text for a fifth of all blocks. Handing it out in a
file that can never be written back is what makes that safe — importing one
would replace every block's authored form with its tag-stripped text.

A projection's `export-state` is the **authored** export's for the same
revision, so an artifact computed over it can be matched to the file an
operator would edit. Optional attribute, closed enumeration (a projection is
something the tooling produces, so an unlisted value is a typo); older files
are unaffected.

## 4.2

The `knownBlockKind` and `knownDivisionKind` enumerations caught up with the
kinds the system already produces. Both are unions with `xs:string`, so every
one of these already validated and no file changes meaning — what changed is
that an editor now offers them, and `ingest-work import` no longer warns about
them (that warning is new in this version too, and is what made the drift worth
closing rather than noting).

Divisions gained `postscript`, `afterword`, `appendix`, `letter` and `section`.
The first three are what the shared classifier types a back-matter heading as;
`letter` is a Frankenstein-style framing letter, which the chap-padded parser
has emitted since `#126`; `section` is what the Folger parser calls an untyped
structural container, and the Sonnets carry two.

Blocks gained `list`, `definition-list` and `table` — the three container shapes
the Gutenberg block emitter has produced since `#116`. They are deliberately
absent from `ingest-override kinds`, which is an *operator's* menu and lists
only kinds the reader renders distinctly; this enumeration is the vocabulary of
what a work file may legitimately contain, which is a wider set. `#199`.

Nothing was added speculatively: every kind here is emitted by a parser today,
and all but `afterword` have live rows in the catalogue.

## 4.1

Illustrations gained `extent` and `depicts`: facsimile provenance — what portion
of the source edition a scan reproduces (`page` | `spread` | `detail`), and what
that portion holds (`division-open` | `title-page` | `text` | `plate` |
`binding`). Both optional, both open enums, and absent on anything that is not a
facsimile.

They are **facts about the scan, not instructions for showing it**. A facsimile
sits outside the core work, so how it is presented — full page or reduced,
before the chapter title or after — is an aesthetic and functional choice rather
than one any convention dictates. Recording what the scan *is* lets a renderer
derive that choice and revisit it later with no migration; recording the choice
itself would freeze one renderer's taste into every operator's file. That is why
these sit beside `role` rather than inside `presentation`, which is the
instruction object.

Neither is derivable from what is already recorded: a whole-page scan attached
to a chapter could equally be a facing plate rather than that chapter's opening
page.

## 4.0

Divisions gained `print-placement`: how the division head sat on the printed
page (`new-page` | `continues` | `own-leaf`), so a Part head sharing its page
with the chapter below it survives the round trip. Absent means unspecified and
reads as `new-page` — a distinct state, since no parser writes it and its
presence means an operator read the page.

Also the first version published to a URL. Earlier versions shipped only as a
file written out beside the book by `ingest-work schema --out`, which is
still how you work offline.

The format was named `fread-content-export` until publication, and files
carrying that stamp are still accepted — the old name denotes an identical
document shape. The version deliberately did not move for the rename, because
the version numbers what the document *contains*, and that did not change. The
new name says what the file holds rather than what was done to it, which is what
makes room for the layer and bridge schemas beside it.

## 3.0

Illustrations became round-trippable. They gained `item-id` (their real
identity — see `WorkIllustration.ItemID` for why blob-id could not serve),
`print-placement`, and `src` (the operator's insert handle, resolved against a
sidecar directory beside the file). Before this they exported but were silently
ignored on import: deleting one from the file did nothing, with no warning.

## 2.0

Block content became the AUTHORED form (`text_html`, falling back to `text` and
marked `text-form="plain"`). Illustrations began reporting the role in effect
plus a `role-pending` attribute when an override has not been applied yet.

## 1.0

Block content was `content_block.text` (plain). Lossy: 21% of the corpus
carries markup that lives only in `text_html`, so a round trip through 1.0
would have stripped emphasis and `<br>` lineation from every block touched.
