---
title: "Marque Block Spec (MBS) — Typed, Signed Content Blocks"
abbrev: "marque-mbs"
docname: draft-tamim-marque-mbs-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - content
  - blocks
  - schema
  - accessibility
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC8259:    # JSON
  RFC8949:    # CBOR
  RFC7595:    # URI scheme registration

informative:
  RFC5322:
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"
  COMMONMARK:
    title: "CommonMark Spec"
    target: "https://spec.commonmark.org/"
  WASMTIME:
    title: "Wasmtime — a standalone WebAssembly runtime"
    target: "https://wasmtime.dev/"

--- abstract

This document specifies the **Marque Block Spec** (MBS), the content layer
of Marque {{MARQUE-CORE}}. MBS replaces HTML email's frozen 2005 subset
with a typed, JSON-based, schema-validated block vocabulary registered
under reverse-DNS namespaces.

Core block types include structured text (CommonMark + GFM), content-
addressed media, attachments, polls, schedules, forms, payment requests,
reactions, replies, forwards, retractions, signed actions (the
DocuSign-replacement primitive), and an optional WebAssembly widget
sandbox {{WASMTIME}}.

MBS is designed for deterministic rendering, native accessibility,
render-capability negotiation, and structural separation of data from
AI-assistant instructions.

--- middle

# Introduction

This draft is the Internet-Draft form of the content-layer section of
the Marque founding specification {{MARQUE-FOUNDING}}. See
`spec/protocol/06-content.md` for the content vocabulary and the
semantics of `core.reply`, `core.forward`, and `core.retract`.

## Scope

- JSON envelope schema and CBOR encoding for MBS messages.
- The `core.*` block-type registry and schemas.
- Third-party reverse-DNS extension policy.
- WASM widget capability sandbox.
- Content-addressing via BLAKE3 CIDs for media and attachments.
- `marque://` URI scheme registration for attachment sources.

## Out of scope

- The Marque envelope itself (covered by `draft-tamim-marque-core`).
- Legal-tier wrapping (covered by `draft-tamim-marque-proof`).

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/06-content.md` on completion of review.

# IANA Considerations

- Registration of reverse-DNS `core.*` block-type namespace.
- Registration of `marque://` URI scheme per {{RFC7595}}.

# Security Considerations

TBD — migrate from founding specification, especially §7.3 (WASM widget
capability sandbox) and §8.2.7 (AI-assistant-safe content parsing).

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
