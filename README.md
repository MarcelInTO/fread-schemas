# fread-schemas

Published schemas for the [FRead](https://schema.fread.now) file formats, served at
**https://schema.fread.now/**.

Each format lives under a path named for its subject — `work/` today, with
`layer/` and `bridge/` to follow. All of it is **generated**: rendered from
the tooling that reads and writes each format, and pushed here automatically
when that tooling changes. Edit it there, not here.

## The one rule

A published version URL is **immutable**. Exported files name their exact
schema version and are validated against it long after they were written, so
changing what a version serves would retroactively rewrite what those files
claimed to be. A changed schema gets a new version; it never overwrites an old
one.

See [work/](work/) for the version history and what the two version numbers
promise.
