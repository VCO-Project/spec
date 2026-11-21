# 04 — Synchronization Protocol

This section defines how VCO repositories exchange objects over HTTP.
The protocol is intentionally small: four endpoints, JSON bodies.

## 4.1 Endpoints

GET  /refs
GET  /objects/<hash>
POST /objects
POST /refs

All bodies are `application/json`. All hashes are lowercase hex
SHA-256.

## 4.2 GET /refs

Returns the remote's refs:
```json
{
  "branches": {
    "main": "9c1e5f3..."
  }
}
```

No `HEAD` is returned — the remote has no working copy in the core
spec. Implementations MAY add `x-head` for server-side tools.

## 4.3 GET /objects/<hash>

Returns the raw stored object (pretty-printed or canonical — the
client MUST re-canonicalize before hashing). 404 if absent.

## 4.4 POST /objects

Body: a JSON array of objects.
```json
[
  {"type": "blob", "encoding": "utf-8", "content": "..."},
  {"type": "change", "parent": ["..."], "tree": {...}, ...}
]
```
The server MUST verify each object's hash matches its content.
It MAY reject the whole batch or individual objects. Response:

{"accepted": 3, "rejected": []}

Rejected objects include a `reason` string.

## 4.5 POST /refs

Body:

{
  "branch": "main",
  "old": "8e4d7b2...",
  "new": "9c1e5f3..."
}

Updates branch `main` from `old` to `new`. This is a **compare-and-
swap**: if the current remote value is not `old`, the server rejects
with 409 Conflict. This prevents two clients from overwriting each
other silently.

Response: 200 on success, 409 on conflict, 404 if the branch does not
exist and `old` is not `null`.

## 4.6 Push algorithm (client side)

1. `GET /refs` → remote heads.
2. Compute the set of objects reachable from the local branch head
   but not from the remote head.
3. `POST /objects` with that set (batched, e.g. 100 at a time).
4. `POST /refs` with `old` = remote head, `new` = local head.
5. On 409: fetch, merge locally, retry.

## 4.7 Pull algorithm (client side)

1. `GET /refs` → remote head.
2. Walk from the remote head, skipping objects already local.
3. `GET /objects/<hash>` for each missing object.
4. If the local branch can fast-forward, move it. Otherwise merge
   (03-commands.md, 3.8) and push the merge.

## 4.8 Authentication

Not part of the core spec. Implementations MAY use HTTP Basic, Bearer
tokens, or anything else. The spec only requires that unauthenticated
access, if allowed, is explicit in the server config.

## 4.9 Conformance

An implementation is **VCO-protocol-compatible** if it:

- serves all four endpoints with the exact JSON shapes above;
- verifies object hashes on `POST /objects`;
- implements compare-and-swap on `POST /refs`.

A conformance test suite lives in `vco-project/tests`. 
