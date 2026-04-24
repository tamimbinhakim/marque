---
title: "Marque Rooms — Shared Mailboxes, Teams, and Mailing Lists"
abbrev: "marque-rooms"
docname: draft-tamim-marque-rooms-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - shared-mailbox
  - mailing-list
  - delegation
  - threshold-signature
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC8058:    # One-click List-Unsubscribe
  RFC9420:    # MLS
  RFC9591:    # FROST

informative:
  ETSI-TS-119-461:
    title: "Electronic Signatures and Infrastructures (ESI); Policy and security requirements for trust service components providing identity proofing"
    target: "https://www.etsi.org/deliver/etsi_ts/119400_119499/119461/"
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies the **Room** primitive in Marque
{{MARQUE-CORE}} — a first-class identity kind for entities that are not
natural persons. Rooms carry their own `did:mail`, their own KEL, and
their own service endpoints, and distribute authority to act across a
roster of natural-person DIDs under explicit policy.

Five kinds are defined:

- `shared` — shared-inbox roles (`support@company.com`).
- `team` — project or thread-specific team identities.
- `list` — broadcast mailing lists with subscription flows.
- `delegation` — executive-assistant delegation in front of a principal.
- `org` — corporate identities bound to legal-person qualified
  certificates per ETSI TS 119 461 {{ETSI-TS-119-461}}.

Send-as authorization uses any-of, threshold (FROST {{RFC9591}}), or
owner-only policies. Role rotation, audit logs, and visibility tiers are
normative.

--- middle

# Introduction

This draft is the Internet-Draft form of the Rooms section of the
Marque founding specification {{MARQUE-FOUNDING}}.

## Scope

- Room DID document extensions (kind, roster, policy, parent, visibility).
- Send-as authorization (any-of, m-of-n threshold, owner-only).
- Five Room kinds and their semantics.
- Roster roles (owner, moderator, member, read-only, assistant).
- Role rotation and audit logs.
- Mailing-list subscription, double-opt-in, one-click unsubscribe per
  {{RFC8058}}, and reply-to policy flags.
- Rolling-epoch relaxation for lists > 50k subscribers.
- Handle conventions preserving email intuition.

## Out of scope

- Natural-person identity (covered by `draft-tamim-marque-did-mail`).
- Qualified-tier mechanics (covered by `draft-tamim-marque-proof`).

## Requirements Language

{::boilerplate bcp14-tagged}

# Status

**This is a stub draft.** Content will be migrated from
`spec/protocol/04-rooms.md` on completion of review.

# IANA Considerations

- Room-kind codepoint registry.
- Roster-role codepoint registry.
- Control-frame codepoints for `claim`, `handoff`, `subscribe`,
  `unsubscribe`, `roster-update`.

# Security Considerations

TBD — migrate from founding specification. Key concerns: delegation scope
enforcement, qualified-tier send-as restrictions for delegation kinds,
organizational handover on mergers/acquisitions.

--- back

# Acknowledgments
{:numbered="false"}

See the founding specification {{MARQUE-FOUNDING}}.
