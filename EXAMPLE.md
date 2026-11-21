# Example: fixing a date parser

A real scenario, step by step, in both Git and VCO.

## The project
```
myproject/
└── parser.py
```
def parse_int(s):
    return int(s)

## Git
```
$ git init
$ git add parser.py
$ git commit -m "add parse_int"
```
# On disk:
```
.git/
├── index              # staging area, binary
├── objects/           # zlib-compressed, binary
│   ├── ab/cd123...    # commit
│   ├── ef/45678...    # tree
│   └── 12/34567...    # blob
└── refs/heads/main
```
Edit parser.py, add parse_date:
```
$ git add parser.py          # required!
$ git commit -m "add parse_date"
```
Forgot `git add`? Changes are not in the commit.

Branch:
```
$ git branch timezone-support
$ git checkout timezone-support
$ git add parser.py
$ git commit -m "timezone support"
$ git checkout main
$ git merge timezone-support
```
Log:
```
$ git log --oneline
9c1e5f3 merge timezone-support
8e4d7b2 timezone support
3f2a9c1 add parse_date
```
`cat .git/objects/...` gives nothing — zlib.

## VCO
```
$ vco init
$ vco track parser.py
$ vco save "add parse_int"
[main 3f2a9c1] add parse_int
```
# On disk:
```
.vco/
├── config.json
├── refs.json          # {"branches": {"main": "3f2a9c1"}, "HEAD": "main"}
├── tracked.json       # {"files": ["parser.py"]}
└── objects/
    ├── 3f2a9c1.json   # change
    └── a1b2c3d.json   # blob (parser.py content)
```
Edit parser.py, add parse_date:

$ vco save "add parse_date"
[main 8e4d7b2] add parse_date

No `add`. `save` takes everything tracked. For partial:
`vco save --only parser.py`.

Branch:
```
$ vco branch timezone-support
$ vco switch timezone-support
$ vco save "timezone support"
$ vco switch main
$ vco merge timezone-support
[main 9c1e5f3] merge timezone-support
```
Log:
```
$ vco log
9c1e5f3  merge timezone-support      just now
8e4d7b2  timezone support            2 minutes ago
3f2a9c1  add parse_date              5 minutes ago
```
And you can read it:

$ cat .vco/objects/8e4d7b2.json
```json
{
  "type": "change",
  "parent": ["3f2a9c1..."],
  "author": "You <you@example.com>",
  "time": "2026-11-21T14:10:00Z",
  "message": "add parse_date",
  "tree": { "parser.py": "a1b2c3d..." }
}
```
## Where the difference shows up

1. Forgot `add`.
   Git: changes are not in the commit; amend.
   VCO: impossible — `save` takes everything.

2. Inspect a commit.
   Git: `git show 8e4d7b2` — tool parses binary.
   VCO: `cat .vco/objects/8e4d7b2.json` — read with your eyes.

3. Move a repo to another host.
   Git: needs a git-compatible server.
   VCO: any server with four HTTP endpoints.

4. Close the laptop with unsaved work.
   Git: lost (or `git stash` if you remember).
   VCO: `vco draft` — snapshot in `.vco/drafts/`, never lost.
