---
title: "Marque Storage, Archive, and Cross-Device Sync"
abbrev: "marque-storage"
docname: draft-tamim-marque-storage-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - archive
  - backup
  - gdpr
  - search
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC9180:    # HPKE
  RFC9420:    # MLS

informative:
  GDPR:
    title: "Regulation (EU) 2016/679 — General Data Protection Regulation"
    target: "https://eur-lex.europa.eu/eli/reg/2016/679/oj"
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies where and how Marque {{MARQUE-CORE}} messages
are stored, archived, searched, exported, and synchronized across a
user's devices.

The hybrid archive-of-record model places ciphertext at the provider
and cleartext on authorized devices, with cross-device synchronization
via HPKE-encrypted {{RFC9180}} cache deltas under a per-identity
archive key.

The sender's Sent-copy is preserved as a self-addressed MLS group
encryption. Provider migration uses the standardized `.marquebox`
export format. Search is client-side by default, with optional
provider-side encrypted search deferred to a companion draft.

GDPR {{GDPR}} right-to-erasure is reconciled with append-only signed
log semantics: recipients may delete local copies or request provider
deletion; senders cannot force deletion of recipient copies.

--- middle

# Introduction

This draft is the Internet-Draft form of the storage and archive
section of the Marque founding specification {{MARQUE-FOUNDING}}.

## Scope

- Hybrid archive-of-record (provider=ciphertext, device=cleartext).
- Sender's Sent-copy via self-addressed MLS group.
- Device-resident cleartext cache semantics and encryption-at-rest
  requirements.
- Cross-device archive sync via archive-key-encrypted cache deltas.
- New-device onboarding and MLS state transfer handshakes.
- Provider-migration procedure with 30-day dual-delivery window.
- `.marquebox` export format.
- Client-side search baseline.
- GDPR erasure reconciliation.
- ProofEnvelope retention per tier and attachment garbage collection.

## Out of scope

- The envelope format itself (covered by `draft-tamim-marque-core`).
- Provider-side encrypted search (deferred to `draft-tamim-marque-search`).

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/03-architecture.md` §3.8–§3.13 on completion of review.

# IANA Considerations

- `.marquebox` media type registration.
- Cache-delta control-frame codepoints.

# Security Considerations

TBD — migrate from founding specification. Key concerns: archive-key
storage on device (single most sensitive secret short of the IRK);
sync-conflict handling for high-stakes fields; secure deletion at
provider-migration endpoint.

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
