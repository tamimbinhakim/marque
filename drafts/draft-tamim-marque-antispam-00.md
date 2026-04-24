---
title: "Economic and Cryptographic Anti-Spam for Marque"
abbrev: "marque-antispam"
docname: draft-tamim-marque-antispam-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - spam
  - anti-abuse
  - proof-of-work
  - argon2
  - privacy-pass
  - prompt-injection
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC9106:    # Argon2
  RFC9578:    # Privacy Pass issuance

informative:
  RFC9420:
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"
  CAMEL:
    title: "CaMeL: Defeating Prompt Injections by Design"
    target: "https://arxiv.org/abs/2503.18813"

--- abstract

This document specifies Marque's anti-abuse policy surface. It replaces
content-based filtering — which has reached a ceiling against
LLM-generated abuse — with economic and cryptographic mechanisms
composing as an AND-of-ORs policy.

The mechanisms are: verified sender identity via DID-anchored signatures;
memory-hard proof-of-work via Argon2id {{RFC9106}}; refundable bonds with
multi-currency support and a 14-day appeal window; provider-vouched
bootstrap tokens for new identities; Privacy-Pass rate-proof tokens
{{RFC9578}}; web-of-trust and federated reputation gossip; and structural
data-vs-instruction separation at the content layer following the
CaMeL dual-LLM approach {{CAMEL}} to defend AI assistants against
prompt injection.

--- middle

# Introduction

This draft is the Internet-Draft form of the anti-spam section of the
Marque founding specification {{MARQUE-FOUNDING}}.

## Scope

- The AND-of-ORs anti-spam policy composition.
- Bootstrap-token issuance for new users (cold-start mitigation).
- Bond mechanics: currency, minimum/maximum, appeal process, reputation-
  poisoning mitigation.
- AI-assistant-safe content parsing via `ai_instruction` and
  `public_untrusted` flags, allowlist-gated `core.agent_instruction` blocks.
- Honest dystopia-risk mitigations.

## Out of scope

- Content-layer semantics (covered by `draft-tamim-marque-mbs`).
- Legal-tier proof (covered by `draft-tamim-marque-proof`).

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/08-anti-spam.md` and `spec/protocol/02-identity.md §2.8`
(new-user cold start) on completion of review.

# IANA Considerations

- Bootstrap-token media type.
- Bond control-frame codepoints.

# Security Considerations

TBD — migrate from founding specification, especially the explicit
rejection of proof-of-humanity mandates and the data-vs-instruction
separation requirements against prompt injection.

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
