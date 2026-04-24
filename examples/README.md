# Examples

Worked examples of Marque artifacts. Each file is **illustrative, not normative** — the normative sources are in [`spec/`](../spec/) and [`schemas/`](../schemas/).

## Contents

| File | What it shows |
|---|---|
| [`did-document.json`](./did-document.json) | A `did:mail` DID document for `alice@example.com`. |
| [`envelope.json`](./envelope.json) | A minimal Marque envelope in JSON form (the wire is CBOR, this is the JSON projection). |
| [`mbs-message.json`](./mbs-message.json) | A Marque Block Spec message with several `core.*` block types. |
| [`proof-envelope.json`](./proof-envelope.json) | A `ProofEnvelope` at the Certified tier. |
| [`capability-record.json`](./capability-record.json) | A provider's `/.well-known/marque-provider.json`. |

## Encoding note

Marque's wire format is **canonical CBOR**. These examples are presented as JSON because JSON is easier to read for humans and diff in code review. Implementations SHOULD treat the JSON projection as a debugging aid, not as a normative encoding.

Field ordering in the JSON examples matches the CDDL definitions in [`schemas/`](../schemas/). When encoding to CBOR, the canonical form is deterministic.
