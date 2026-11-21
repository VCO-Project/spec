
# Git vs VCO

VCO is not "git, but better". It is a standard, not a product.
This document lists what is the same and what is different.

## Same model

- History is a DAG of changes with parents.
- A branch is a pointer to a change.
- A merge is a change with two parents.
- A change references the full tracked state (logical snapshot).
- Content is addressed by hash.

If you understand git's graph, you understand VCO's graph.

## Differences

| | Git | VCO |
|---|---|---|
| Staging area | Yes (index), binary | No. `save` takes everything tracked |
| Objects | commit / tree / blob, binary, SHA-1 | change / blob, JSON, SHA-256 |
| Branch | File in `refs/heads/` | Line in `refs.json` |
| History on disk | Pack files (binary, after gc) | JSON files, readable with `cat` |
| Merge | Commit with two parents | Change with two parents (same) |
| Draft | No (stash is a workaround) | `draft` layer, never lost |
| Protocol | Smart HTTP / SSH / git:// | One: HTTP + JSON |
| Canonicalization | Implicit, per implementation | RFC 8785, fixed in spec |
| Extensions | Hooks, config, ad hoc | `x-*` sidecar, never changes hash |

## What VCO removes

- Staging area and the "forgot to add" class of bugs.
- Binary object storage as a requirement.
- Vendor lock-in of history format.

## What VCO keeps

- DAG history.
- Branches as pointers.
- Merge with two parents.
- Content addressing.

## What VCO does not solve (on purpose)

- Large binaries — separate spec.
- Monorepos with millions of files — implementation detail.
- Access control — server-side.
- Signing — reserved `x-vco-signatures`, not in 0.1.
