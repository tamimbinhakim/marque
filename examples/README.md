# Examples

Worked examples of Marque artifacts. Each file is **illustrative, not normative** — the normative sources are the spec under [`spec/`](../spec/) and the schemas under [`schemas/`](../schemas/).

> [!NOTE]
> Marque's wire format is **canonical CBOR** ([RFC 8949](https://datatracker.ietf.org/doc/html/rfc8949) §4.2 deterministic encoding). The JSON files below are a debugging projection: human-readable and easy to diff in code review, but **not the wire**. Implementations SHOULD treat the JSON projection as a debugging aid, not as a normative encoding.

## Contents

| File                                                                | What it shows                                                                                  | Spec section                                                                                |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [`did-document.json`](./did-document.json)                          | A `did:mail` DID document for `alice@example.com`.                                             | [`protocol/02-identity.md`](../spec/protocol/02-identity.md)                                |
| [`envelope.json`](./envelope.json)                                  | A minimal Marque envelope in JSON form (the wire is CBOR; this is the JSON projection).        | [`protocol/03-architecture.md §3.2`](../spec/protocol/03-architecture.md), [`schemas/cddl/envelope.cddl`](../schemas/cddl/envelope.cddl) |
| [`mbs-message.json`](./mbs-message.json)                            | A Marque Block Spec message with several `core.*` block types.                                 | [`protocol/06-content.md`](../spec/protocol/06-content.md), [`schemas/blocks/`](../schemas/blocks/) |
| [`proof-envelope.json`](./proof-envelope.json)                      | A `ProofEnvelope` at the Registered tier (signature chain, RFC 3161 timestamp, OTS receipt).   | [`protocol/07-legal-proof.md`](../spec/protocol/07-legal-proof.md)                          |
| [`capability-record.json`](./capability-record.json)                | A provider's `/.well-known/marque-provider.json` capability advertisement.                     | [`protocol/03-architecture.md §3.6`](../spec/protocol/03-architecture.md)                   |

## How to use these examples

**For spec readers** — pair each example with the spec section in the third column. Reading the prose plus the example together is faster than either alone.

**For implementers** — validate your CBOR encoder against these canonical forms. Re-encoding the JSON projection through a deterministic JSON-to-CBOR pipeline (per [RFC 8949 §4.2](https://datatracker.ietf.org/doc/html/rfc8949#section-4.2)) MUST produce byte-for-byte identical output to a reference encoder. The conformance suite in [`spec/protocol/10-conformance.md`](../spec/protocol/10-conformance.md) covers this.

**For debuggers** — when an envelope arrives that won't parse, project the bytes back to JSON, diff against the closest example here, and read the corresponding spec section. The encoding contract is one-to-one.

## Adding a new example

1. Place the file under `examples/` with a descriptive name (`mbs-message-with-attachment.json`, not `example2.json`).
2. Add a row to the table above with the spec section it illustrates.
3. Validate the JSON against the relevant schema in [`schemas/`](../schemas/) before committing.
4. Use the `add(examples)` commit type — see [`CONTRIBUTING.md`](../CONTRIBUTING.md#conventional-commits).
