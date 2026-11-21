# VCO — Version Control Open

**VCO** is an open specification for version control systems.
Not a product. A protocol.

Any implementation that follows this specification is compatible
with any other. History is stored as human-readable JSON objects,
synchronized over plain HTTP.

## Status

- **Version:** 0.1 (draft)
- **Stability:** experimental — the spec may change until 1.0
- **License:** [CC BY 4.0](LICENSE)
- **Name policy:** [TRADEMARK.md](TRADEMARK.md)

## Why VCO

Git won. But git is a product, not a standard: its object format
is not an open specification that anyone can implement and remain
compatible. VCO defines the format, the protocol, and the commands,
so that clients and servers from different vendors interoperate.

Three problems VCO solves:

1. **No staging area.** `vco save` takes everything tracked. One step.
2. **Readable history.** Objects are JSON. `cat`, `grep`, `jq` work without tools.
3. **No vendor lock-in.** Any server implementing four HTTP endpoints is compatible.

## Documentation

| File | Contents |
|---|---|
| [00-intro.md](00-intro.md) | Axioms: why, for whom, how |
| [01-model.md](01-model.md) | Terms: change, blob, ref, workspace |
| [02-objects.md](02-objects.md) | JSON format, canonicalization, SHA-256 |
| [03-commands.md](03-commands.md) | init / track / save / log / diff / branch / switch / merge |
| [04-protocol.md](04-protocol.md) | HTTP: /refs, /objects |
| [05-extensions.md](05-extensions.md) | Namespace x-* |
| [HOW-IT-WORKS.md](HOW-IT-WORKS.md) | Diagrams and step-by-step flow |
| [COMPARISON.md](COMPARISON.md) | Git vs VCO |
| [EXAMPLE.md](EXAMPLE.md) | Real scenario: parser.py |

## Reference implementation

- [vco-project/reference](https://github.com/vco-project/reference) — Python, MIT
- [vco-project/server](https://github.com/vco-project/server) — Go, MIT
- [vco-project/tests](https://github.com/vco-project/tests) — conformance tests

## License

Specification text: CC BY 4.0.
Reference code: MIT.
The name "VCO" and the logo are covered by [TRADEMARK.md](TRADEMARK.md). 
