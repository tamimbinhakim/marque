# Schemas

Machine-readable schemas for Marque envelopes, DID documents, capability records, and MBS blocks.

These are **Draft** schemas. They track the spec but are not themselves normative until an accompanying MIP lands them as the canonical reference. Expect breaking changes until v1.0.

## Contents

| Schema                                                             | Describes                                                     |
| ------------------------------------------------------------------ | ------------------------------------------------------------- |
| [`cddl/envelope.cddl`](./cddl/envelope.cddl)                       | **Normative CBOR wire format** (source of truth).             |
| [`_defs/common.schema.json`](./_defs/common.schema.json)           | Shared `$defs` (did_mail, blake3_256, auth_mode, …).          |
| [`envelope.schema.json`](./envelope.schema.json)                   | JSON projection of the Marque envelope.                       |
| [`did-mail.schema.json`](./did-mail.schema.json)                   | A `did:mail` DID document.                                    |
| [`capability-record.schema.json`](./capability-record.schema.json) | A provider's `/.well-known/marque-provider.json`.             |
| [`proof-envelope.schema.json`](./proof-envelope.schema.json)       | A `ProofEnvelope` for Attested or Certified tier.             |
| [`blocks/`](./blocks/)                                             | One file per `core.*` MBS block type.                         |

## Validating

These schemas are JSON Schema Draft 2020-12. Any compliant validator works:

```sh
# ajv-cli
npm i -g ajv-cli ajv-formats
ajv validate -s schemas/envelope.schema.json -d examples/envelope.json --spec=draft2020 -c ajv-formats

# Python jsonschema
pip install jsonschema
python -c "import json, jsonschema; \
  s = json.load(open('schemas/envelope.schema.json')); \
  d = json.load(open('examples/envelope.json')); \
  jsonschema.Draft202012Validator(s).validate(d)"
```

## Canonical encoding

Wire format is canonical CBOR. The CDDL source of truth is [`cddl/envelope.cddl`](./cddl/envelope.cddl); implementations MUST encode and decode against the CDDL. The JSON Schemas are the debugging/development projection, not the normative wire format.
