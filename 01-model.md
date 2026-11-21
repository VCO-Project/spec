# 01 — Model and Terms

This section defines the objects and the working model. All later
sections use these terms with exactly these meanings.

## 1.1 Repository

A repository is a directory containing a `.vco/` folder. The folder
holds all history, refs, and configuration. There is no separate
"bare" repository in the core spec; a bare repository is simply one
without a working copy.

## 1.2 Object

An object is an immutable, content-addressed JSON document stored in
`.vco/objects/<hash>.json`. There are two object types in the core:

- **blob** — the content of a single file.
- **change** — a point in history.

The hash of an object is the SHA-256 of its canonical JSON form
(see 02-objects.md). The hash is the object's identity. Two objects
with the same hash are the same object, byte for byte.

## 1.3 Blob

A blob holds the content of one file. It does not know its path or
its history. The same content in two different files is one blob.

## 1.4 Change

A change is a node in history. It contains:

- `parent` — a list of zero, one, or two change hashes. Zero for the
  first change. One for a normal save. Two for a merge.
- `author`, `time`, `message` — metadata.
- `tree` — a map from file path to blob hash, representing the full
  state of the tracked files at this point.

A change is a **logical snapshot**: it references every tracked file,
not a delta. Implementations may store deltas physically.

## 1.5 Ref

A ref is a named pointer to a change. The core defines:

- **Branch refs** — `refs.json` maps branch names to change hashes.
- **HEAD** — the name of the current branch.

A branch is a single line in `refs.json`. There are no separate
files per branch in the core spec.

## 1.6 Workspace

The workspace is the directory the user edits, together with
`tracked.json` (the list of tracked paths). The workspace is not
part of history until `vco save`.

## 1.7 Draft

A draft is a snapshot of the workspace that is not part of history.
Drafts are stored in `.vco/drafts/`. They survive `switch` and
`merge` and are never pushed. A draft becomes history only via
`vco save`.

## 1.8 History graph

The history graph is a directed acyclic graph (DAG) of change objects,
where edges point from a change to its parents. A branch is a path
through this graph from its head back to the first change. Merge
creates a node with two parents. The graph is the same model as git,
fixed in the standard.

## 1.9 Canonical JSON

Canonical JSON is the byte sequence defined by RFC 8785 (JCS).
All hashes are computed over canonical JSON, never over pretty-printed
JSON. This guarantees that the same logical object yields the same
hash in every implementation and every language.

## 1.10 Remote

A remote is an HTTP endpoint implementing the VCO protocol
(see 04-protocol.md). A repository may have zero or more remotes
configured in `.vco/config.json`.
