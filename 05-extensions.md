# 05 — Extensions

This section defines how implementations add features without
breaking compatibility. The core spec is small on purpose; everything
else lives here.

## 5.1 Principle

An extension MUST NOT change the hash of a core object. A client that
does not understand an extension MUST still be able to verify and use
the object. Therefore extensions live in a **sidecar layer**, not
inside the object.

## 5.2 Sidecar file

For an object with hash `<hash>`, extensions are stored in:

.vco/objects/<hash>.x.json

The sidecar is a JSON object:

{
  "x-company-foo": { ... },
  "x-vco-largefiles": { ... }
}

Rules:

- Keys MUST start with `x-`.
- The sidecar is NOT hashed and NOT part of the object's identity.
- The sidecar MAY be absent. Absent sidecar = no extensions.
- Implementations MUST preserve sidecars they do not understand
  when copying or pushing objects.
- Sidecars are pushed and pulled alongside objects, but a server
  MAY drop sidecars it does not understand (documented behavior).

## 5.3 Naming

- `x-<vendor>-<feature>` — vendor-specific. Example: `x-acme-ci`.
- `x-vco-<feature>` — reserved for future official extensions.
  Implementations MUST NOT invent `x-vco-*` keys.

## 5.4 Core extensions reserved

The following `x-vco-*` keys are reserved for future versions of
this spec:

- `x-vco-largefiles` — external blob storage.
- `x-vco-signatures` — cryptographic signatures over changes.
- `x-vco-acl` — access control metadata.

They are not defined in 0.1. Implementations MUST ignore them.

## 5.5 Compatibility

A VCO implementation is **core-compatible** if it:

- reads and writes core objects per 02-objects.md;
- verifies hashes over core fields only;
- preserves unknown sidecars on copy.

It is **extension-compatible with X** if it additionally implements
the `x-*` extension X. Compatibility claims MUST name the extensions.
