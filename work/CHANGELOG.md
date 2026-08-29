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

A consequence worth stating: the published `.../work/<major>/` alias
always serves the newest schema in that major line, and because minors are
additive it still validates every older file in the line. The exact
`<major>.<minor>/` URL is what an export stamps, so a file always names the
schema it was actually written against.

Versions before 4.0 predate this rule — every change was breaking, because the
importer compared the version string for exact equality. They are listed with a
`.0` minor for consistency.

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
