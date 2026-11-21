# 00 — Introduction and Axioms

This section defines what VCO is, who it is for, and the principles
every other section must follow. If a later section contradicts an
axiom here, the axiom wins.

## A1. Version control is a protocol, not a product

Today, version control is tied to a vendor: a git repository lives
in git, an svn repository lives in svn. Changing tools means migrating.

VCO states: versions are a common exchange format, like HTTP.
A client from one vendor works with a server from another.

## A2. History must be readable without tools

Git hides history in binary pack files and requires `git log` to see it.
VCO: objects are text (JSON). They can be `cat`, `grep`, and `diff`'d
without any special software. The tool is optional for understanding.

## A3. Nothing is lost between saves

Git loses everything that is not committed (or stashed).
VCO: the working copy is a draft that the system sees.
Unsaved work does not disappear on switch or merge — it is kept
in a draft layer.

## A4. Less ceremony, more work

Staging area, index, detached HEAD, reflog — this is the price git
pays for flexibility. VCO: the core is small, each operation does
one understandable thing. Complexity only through optional extensions.

## B1. For developers tired of ceremony

The primary user is someone who runs `save`, `log`, `branch`, `merge`
every day and wants it to be predictable.

## B2. For tool authors

The spec is a contract. If you write a client, a server, a GUI, or a
CI integration, you read the spec and know what to do. No reverse-
engineering someone else's binary.

## B3. For organizations that need interoperability

If tomorrow's git hosting becomes expensive or shuts down, data is
not locked in. Any VCO implementation reads the same objects.

## B4. Not for everyone

VCO deliberately does not solve: large binaries (separate spec),
monorepos with millions of files (implementation detail), complex
access policies (server). This is not "git for everyone", it is
"a standard for those who need portability".

## C1. Logically a snapshot, physically anything

A change logically contains the state of the tree (references to blobs).
An implementation may delta-pack however it wants. The spec describes
*what*, not *how to store*.

## C2. One protocol — HTTP

No smart/dumb variants. `GET /refs`, `GET /objects/<hash>`,
`POST /objects`, `POST /refs`. That is all.

## C3. Small core, extensions via namespace

Mandatory commands: `init/track/save/log/diff/branch/switch/merge/push/pull`.
Everything else is `x-*`. The core does not grow.

## C4. Identifier is the hash of canonical JSON

The same change in any implementation yields the same id.
Canonicalization is part of the spec (RFC 8785).

## C5. Branch is a pointer, merge is a change with two parents

No magic. The history graph is a DAG of change objects, as in git,
but fixed in the standard.

## C6. Humans resolve conflicts, the system guides

On merge, the system shows conflicting regions step by step and asks
for a decision. No `<<<<<<<` markers in the file as the only way.
