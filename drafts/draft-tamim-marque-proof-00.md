---
title: "Marque ProofEnvelope — Tiered Non-Repudiation"
abbrev: "marque-proof"
docname: draft-tamim-marque-proof-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - non-repudiation
  - eidas
  - qerds
  - frost
  - opentimestamps
  - timestamp
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC3161:
  RFC9591: # FROST

informative:
  RFC9420:
  ETSI-EN-319-122-1:
    title: "Electronic Signatures and Infrastructures (ESI); CAdES digital signatures"
    target: "https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/"
  ETSI-EN-319-522-2:
    title: "ETSI EN 319 522-2 — ERDS semantic contents"
    target: "https://www.etsi.org/deliver/etsi_en/319500_319599/31952202/"
  ETSI-EN-319-521:
    title: "ETSI EN 319 521 — ERDS provider policy and security requirements"
    target: "https://www.etsi.org/deliver/etsi_en/319500_319599/319521/"
  ETSI-TS-119-524:
    title: "ETSI TS 119 524 — Conformance assessment test suites for ERDS (EN 319 522 + EN 319 521)"
    target: "https://www.etsi.org/deliver/etsi_ts/119500_119599/"
  EIDAS-2:
    title: "Regulation (EU) 2024/1183 — eIDAS 2.0 (entered into force 20 May 2024)"
    target: "https://eur-lex.europa.eu/eli/reg/2024/1183/oj"
  OPENTIMESTAMPS:
    title: "OpenTimestamps"
    target: "https://opentimestamps.org"
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies the `ProofEnvelope` used by Marque {{MARQUE-CORE}}
to provide tiered non-repudiation of origin, delivery, content, and time
for messages carried over the Marque protocol. The ProofEnvelope wraps
a Marque envelope signed under `SIGNED` or `QUALIFIED` auth-mode with:

- the canonical message hash;
- the sender's signature under the negotiated cipher suite;
- the Device Signing Key certificate chain to the Identity Root Key;
- one or more {{RFC3161}} qualified timestamps;
- optional {{RFC9591}} FROST threshold signatures from notary relays;
- an OpenTimestamps {{OPENTIMESTAMPS}} Bitcoin-anchored long-term proof;
- periodic archive timestamps re-anchoring the envelope as primitives
  deprecate (CAdES-B-LTA pattern, {{ETSI-EN-319-122-1}}).

Three tiers map to user expectations and cost: Casual (no proof), Attested
(eIDAS ERDS, {{EIDAS-2}} Art. 43), and Certified (eIDAS QERDS, Art. 44).

--- middle

# Introduction

This draft is the Internet-Draft form of the legal-proof section of the
Marque founding specification {{MARQUE-FOUNDING}}. Content is in
preparation; see `spec/protocol/07-legal-proof.md` in the Marque
repository for the full text that will populate this draft.

## Scope

This document normatively specifies:

- The ProofEnvelope wire format (CBOR).
- The three tiers (CASUAL, SIGNED, QUALIFIED) and their evidence
  composition rules.
- The non-delivery state machine.
- Qualified-tier read-receipt obligation.
- QTSP revocation and long-term-verifiability handling (CAdES-B-LT).
- Cost-quotation requirement before Qualified-tier sends.

## Out of scope

- Identity (covered by `draft-tamim-marque-did-mail`).
- Envelope and transport (covered by `draft-tamim-marque-core`).
- Commercial arrangements between QTSPs and Marque providers.

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/07-legal-proof.md` and `spec/protocol/03-architecture.md`
(retention rules) on completion of review.

# IANA Considerations

TBD. Expected registrations: ProofEnvelope CBOR map keys, delivery-state
codepoints.

# Security Considerations

TBD — migrate from founding specification.

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
