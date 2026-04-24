---
title: "Bidirectional SMTP Bridge for Marque"
abbrev: "marque-bridge"
docname: draft-tamim-marque-bridge-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - smtp
  - bridge
  - interop
  - autocrypt
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC5322:
  RFC6376:    # DKIM
  RFC7489:    # DMARC
  RFC8058:    # One-click List-Unsubscribe

informative:
  RFC9420:
  AUTOCRYPT:
    title: "Autocrypt Level 1"
    target: "https://autocrypt.org/level1.html"
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies the bidirectional bridge between Marque
{{MARQUE-CORE}} and legacy SMTP {{RFC5322}} email. The bridge preserves
reply continuity with legacy correspondents throughout a projected
10–20 year transition period while labeling message provenance honestly
in the user interface.

Inbound SMTP messages are wrapped in MLS envelopes when a sender key is
discoverable via the four hybrid discovery methods defined herein
(DID resolution, DNS TXT, `.well-known`, Autocrypt header). Outbound
Marque messages to SMTP recipients downgrade with an Autocrypt header
advertising the sender's key for opportunistic upgrade on the next
round-trip.

The bridge sanitizes legacy Received chains, truncates vendor-internal
PII headers, and replaces E2EE-only attachments with time-limited
authenticated download links.

--- middle

# Introduction

This draft is the Internet-Draft form of the SMTP-interop section of
the Marque founding specification {{MARQUE-FOUNDING}}.

## Scope

- Inbound SMTP → Marque discovery, wrapping, and plaintext fallback.
- Outbound Marque → SMTP downgrade rules and Autocrypt injection.
- Received-chain sanitization and PII-header stripping.
- Authenticated download links for E2EE-only attachments.
- Round-trip upgrade state machine (UNKNOWN → AVAILABLE → MUTUAL → NATIVE).
- DNS records for coexistence (`_marque._tcp`, `_marque` TXT).
- UX labeling rules deferring to the Marque Lexicon.

## Out of scope

- SMTP itself.
- Autocrypt Level 1 — only Marque's profile is specified here.

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/09-interop.md` on completion of review.

# IANA Considerations

- `_marque._tcp` SRV record definition.
- `_marque` TXT record capability and policy grammar.
- HTTP header `Autocrypt` profile registration (referencing Autocrypt Level 1).

# Security Considerations

TBD — migrate from founding specification, especially §10.6 (bridge spam
and phishing mitigation) and the brand-impersonation detection rules.

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
