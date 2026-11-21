# 03 — Commands

This section defines the mandatory commands and their behavior.
Every VCO implementation MUST provide these with these names and
semantics.

## 3.1 init

vco init

Creates `.vco/` in the current directory with:

- `config.json` with a fresh `repo_id` and `version: "0.1"`.
- `refs.json` with `{"branches": {}, "HEAD": "main"}`.
- `tracked.json` with `{"files": []}`.
- empty `objects/` and `drafts/`.

Fails if `.vco/` already exists.

## 3.2 track

vco track <path>...

Adds paths to `tracked.json`. Paths are relative to the repository
root. Adding an already-tracked path is a no-op. Fails if a path
does not exist.

## 3.3 save

vco save "<message>" [--only <path>...]

The core operation. Steps:

1. Read `tracked.json`. If `--only` is given, use those paths instead.
2. For each path, read the file and compute the blob hash.
3. Write any new blob objects to `objects/`.
4. Build a change object:
   - `parent` = `[current branch head]`, or `[]` if the branch is empty.
   - `tree` = map of all tracked paths to their blob hashes.
5. Compute the change hash and write it to `objects/`.
6. Update `refs.json`: set the current branch to the new change hash.

Rules:

- `message` MUST be non-empty. Implementations SHOULD reject
  messages that are only whitespace.
- `save` takes everything tracked. There is no staging area.
- If nothing changed since the parent, `save` MAY be a no-op with a
  warning, or MAY create an empty change. Implementations MUST
  document which.

## 3.4 log

vco log [--limit N]

Walks from the current branch head along `parent` links and prints,
newest first:

<hash>  <message>  <time>

For a merge (two parents), both parents are walked. Implementations
MUST NOT print the same change twice.

## 3.5 diff

vco diff [<change>]

Without arguments: compares the working copy against the current
branch head, per tracked file.

With a change hash: compares that change's tree against its first
parent.

Output is a unified diff of text files. Binary files are reported
as `Binary files differ`.

## 3.6 branch

vco branch <name>
vco branch            # list branches

Creates a branch pointing at the current head. Fails if the name
already exists or is invalid (must match `[a-zA-Z0-9._/-]+`).

## 3.7 switch

vco switch <name>

Changes `HEAD` to `<name>` and updates the working copy to match the
branch head's tree.

Rules:

- If the working copy has unsaved changes, the implementation MUST
  create a draft (see 3.10) before switching, and MUST tell the user.
- Files tracked in the target but absent locally are created.
  Files tracked locally but absent in the target are deleted.
- Untracked files are never touched.

## 3.8 merge

vco merge <branch>

Merges `<branch>` into the current branch.

Steps:

1. Find the lowest common ancestor (LCA) of the two heads.
2. For each path in either tree, three-way merge:
   - unchanged in both → keep.
   - changed in one → take that side.
   - changed in both → conflict.
3. If no conflicts: build a change with
   `parent: [current_head, other_head]`, write it, update the branch.
4. If conflicts: present them to the user interactively. The user
   resolves each region. No `<<<<<<<` markers are written to files
   unless the user asks for them.

The merge change's `tree` MUST contain the resolved result.

## 3.9 push / pull

vco push <remote> <branch>
vco pull <remote> <branch>

Defined in 04-protocol.md. In short:

- `push` sends objects the remote lacks, then updates the remote ref.
- `pull` fetches the remote ref, downloads missing objects, and
  merges (or fast-forwards) into the local branch.

## 3.10 draft

vco draft
vco draft list
vco draft apply <id>

`vco draft` snapshots the working copy into `.vco/drafts/<id>.json`.
Drafts are not history and are never pushed. They exist so that
unsaved work is never lost on `switch` or `merge`.

## 3.11 Exit codes

- `0` — success.
- `1` — user error (bad arguments, missing file).
- `2` — repository error (corrupt object, missing parent).
- `3` — conflict (merge stopped, user must resolve).

These codes are part of the spec so that scripts can rely on them. 
