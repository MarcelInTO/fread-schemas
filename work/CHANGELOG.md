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
