  # How VCO Works

This document explains the flow without normative language.
For the rules, see 01–05.

## One sentence

VCO stores history as a linked list of JSON objects, where each
`save` is a new object pointing to its parent, and branches are
just lines in `refs.json`. Sync is exchanging those objects over HTTP.

## On disk

```
myproject/
├── .vco/
│   ├── config.json      # repo id, format version
│   ├── refs.json        # {"branches": {"main": "9c1e5f3"}, "HEAD": "main"}
│   ├── tracked.json     # ["parser.py", "utils.py"]
│   ├── objects/
│   │   ├── 3f2a9c1.json   # change: add parse_date
│   │   ├── 8e4d7b2.json   # change: handle errors
│   │   ├── 9c1e5f3.json   # change: merge timezone-support
│   │   └── a1b2c3d.json   # blob: parser.py content
│   └── drafts/
│       └── draft-7a1.json # unsaved snapshot
└── parser.py
└── utils.py
```

Every file in `objects/` is JSON. `cat`, `grep`, `jq` work without
any VCO tool.

## What happens on `vco save`

1. Read `tracked.json` → list of files.
2. For each file, compute SHA-256 of its content.
3. Compare with the parent change's tree.
4. For changed files, write a blob object → `objects/<blob_hash>.json`.
5. Build a change object:
   ```
   { type, parent, author, time, message, tree }
   ```
6. Compute id = SHA-256(canonical JSON of the change).
7. Write `objects/<change_id>.json`.
8. Update `refs.json`: `{"branches": {"main": "<change_id>"}}`.

No staging area. No index. One step.

## History graph

```
        ┌──────────┐
        │ 3f2a9c1  │  add parse_date
        └────┬─────┘
             │
        ┌────▼─────┐
        │ 8e4d7b2  │  handle errors
        └────┬─────┘
             │
      ┌──────┴──────┐
      │             │
 ┌────▼─────┐  ┌────▼──────────┐
 │ 9c1e5f3  │  │ timezone-head │  branch timezone-support
 │  merge   │◄─┤ (timezone)    │
 └──────────┘  └───────────────┘
      ▲
      │
   refs.json: {"branches": {"main": "9c1e5f3"}, "HEAD": "main"}
```

## Sync

```
CLIENT A                    SERVER                    CLIENT B
   │                          │                          │
   │  POST /objects           │                          │
   │  [3f2a9c1, 8e4d7b2]      │                          │
   ├─────────────────────────►│                          │
   │                          │  stored objects          │
   │  POST /refs              │                          │
   │  {"branch":"main",       │                          │
   │   "old":"8e4d7b2",       │                          │
   │   "new":"9c1e5f3"}       │                          │
   ├─────────────────────────►│                          │
   │                          │  GET /refs               │
   │                          │◄─────────────────────────┤
   │                          │  {"main":"9c1e5f3"}      │
   │                          ├─────────────────────────►│
   │                          │  GET /objects/9c1e5f3    │
   │                          │◄─────────────────────────┤
   │                          ├─────────────────────────►│
   │                          │  object                  │
```

## Git vs VCO in one picture

```
GIT:   working copy → [index] → objects (binary) → remote
                        ↑
                  forgot add = pain

VCO:   working copy → objects (JSON) → HTTP server
                        ↑
                  no intermediate layer
```                │
