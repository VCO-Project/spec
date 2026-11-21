# 02 — Objects: Format, Canonicalization, Hashing

This section is normative. Any implementation that produces different
bytes for the same logical object is not VCO-compatible.

## 2.1 General rules

- Objects are JSON documents (RFC 8259).
- The hash of an object is the lowercase hex SHA-256 of its
  canonical form (RFC 8785).
- Objects are stored as `.vco/objects/<hash>.json`.
- Object files MAY be pretty-printed for human reading. The hash is
  always computed over the **canonical** form, not the stored form.

- An object's hash is computed over its core fields only. Extension
  fields are stored separately (see 05-extensions.md) and never
  affect the hash. This guarantees that a client which does not
  understand an extension can still verify and use the object.

## 2.2 Canonicalization

VCO uses **RFC 8785 — JSON Canonicalization Scheme (JCS)**.
Implementations MUST NOT invent their own canonical form.
Libraries exist for Python, Go, Rust, JavaScript, Java, and C.

Key consequences of JCS:

- Object keys are sorted lexicographically by UTF-16 code units.
- No insignificant whitespace.
- Strings are escaped per RFC 8785 (minimal escaping, UTF-8 for
  non-ASCII where allowed).
- Numbers use the ECMAScript shortest round-trip form.

## 2.3 Blob object
```json
{
  "type": "blob",
  "encoding": "utf-8",
  "content": "def parse_int(s):\n    return int(s)\n"
}
```
Fields:

- `type` — always `"blob"`.
- `encoding` — `"utf-8"` for text, `"base64"` for binary.
- `content` — the file content as a string. For `utf-8`, the literal
  text. For `base64`, the standard base64 encoding (RFC 4648) of the
  raw bytes.

Rules:

- A blob has no path, no name, no history.
- Two files with identical content share one blob.
- Empty file: `content` is `""`, encoding `"utf-8"`.

## 2.4 Change object
```json
{
  "type": "change",
  "parent": ["8e4d7b2..."],
  "author": "Jane Doe <jane@example.com>",
  "time": "2026-11-21T10:00:00Z",
  "message": "add parse_date",
  "tree": {
    "parser.py": "a1b2c3d...",
    "utils.py":  "d4e5f6a..."
  }
}
```
Fields:

- `type` — always `"change"`.
- `parent` — an array of change hashes:
  - `[]` — the first change in a repository.
  - `[h]` — a normal save.
  - `[h1, h2]` — a merge. Order is significant: `h1` is the branch
    being merged into, `h2` is the branch being merged from.
- `author` — `"Name <email>"`. Free-form, but the angle-bracket form
  is recommended.
- `time` — RFC 3339 timestamp in UTC, `Z` suffix.
- `message` — non-empty string. The spec does not enforce length,
  but conforming tools SHOULD warn above 200 characters.
- `tree` — a map from **relative path** (forward slashes, no leading
  slash) to blob hash. A path absent from `tree` is absent from the
  tracked set at this change.

Rules:

- The `tree` is the full tracked state, not a delta.
- A change with `parent: [h1, h2]` MUST have a tree that resolves the
  merge of the two parents (see 03-commands.md, merge).
- The first change in a repository has `parent: []`.

## 2.5 Ref file

`.vco/refs.json`:

{
  "branches": {
    "main": "9c1e5f3...",
    "timezone-support": "8e4d7b2..."
  },
  "HEAD": "main"
}

Rules:

- `branches` maps branch name to change hash.
- `HEAD` is the name of the current branch. Detached HEAD is not part
  of the core spec.
- A branch pointing to a hash that does not exist in `objects/` is
  invalid; implementations MUST reject it.

## 2.6 Tracked file

`.vco/tracked.json`:

{
  "files": ["parser.py", "utils.py"]
}

Rules:

- Paths are relative, forward slashes, no leading slash.
- Order is not significant; implementations MUST sort before hashing
  if they hash this file (the core spec does not require it).

## 2.7 Config file

`.vco/config.json`:
```json
{
  "version": "0.1",
  "repo_id": "uuid-or-random",
  "remotes": {
    "origin": "https://example.com/vco"
  }
} 
```

Rules:

- `version` — the spec version this repository follows.
- `repo_id` — opaque identifier, generated at `init`.
- `remotes` — optional map of name to URL.

## 2.8 Hash examples

Given the blob:
```json
{"type":"blob","encoding":"utf-8","content":"hello\n"}
```
its canonical form is the same string (keys sorted: `content`,
`encoding`, `type`). Its SHA-256 is the blob id.

Two implementations in different languages MUST produce the same
32-byte digest for this input. This is the conformance test for
canonicalization. 
