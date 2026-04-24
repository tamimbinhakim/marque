# Marque in ten minutes

**Marque is a cryptographically signed, federated messaging protocol proposed as the successor to email.** It replaces SMTP and IMAP over a 10–20 year transition while bridging bidirectionally to legacy email from day one, so no correspondent is ever left behind.

This document is the shortest honest description of Marque. It exists so a reader can decide in ten minutes whether to keep reading.

---

## In one paragraph

Marque treats identity as a portable cryptographic object the user owns, not an address the provider rents. It encrypts message content end-to-end by default using MLS ([RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420)), produces [eIDAS 2.0 QERDS](https://eur-lex.europa.eu/eli/reg/2024/1183/oj)-compliant legal proof as a tiered option, replaces HTML's frozen 2005 subset with a typed and signed block format, and anchors non-repudiable timestamps in Bitcoin via [OpenTimestamps](https://opentimestamps.org) at zero marginal cost — while keeping message content, routing, and identity entirely off-chain. Providers become commodity relays that compete on service quality, compliance, and storage, not on owning the user's address.

## Why it exists

Email's structural defects are now critical and cannot be patched by another bolt-on:

- **Gmail and Microsoft have effectively killed self-hosting** through IP-reputation gatekeeping.
- **Account suspension vaporizes decades of correspondence** with no human recourse.
- **HTML rendering remains trapped in Outlook's Word engine** from 2005.
- **Zero-click AI prompt injection** ([CVE-2025-32711 / EchoLeak](https://nvd.nist.gov/vuln/detail/CVE-2025-32711)) is now a weaponized attack class.
- **AI-generated phishing** is ~83% of analyzed phish, and content filtering has reached its ceiling.

The bolt-ons — SPF, DKIM, DMARC, ARC, BIMI, MTA-STS, TLS-RPT, List-Unsubscribe, Autocrypt, S/MIME, OpenPGP — are historical evidence that the 1982 SMTP substrate cannot absorb modern identity, cryptography, rich content, legal proof, AI-assistant safety, or provider competition.

## Core commitments

| | |
|---|---|
| **Identity is portable** | A long-lived keypair published as `did:mail`, with a KERI-style signed rotation log. Your address survives any provider change the way a phone number survives a carrier change. |
| **Content is end-to-end encrypted** | MLS groups, per-message deniability / non-repudiation choice, post-quantum hybrid by default. |
| **Legal proof is native and tiered** | Casual / Signed / Registered, mapping to eIDAS ERDS and QERDS, with OpenTimestamps Bitcoin anchoring for zero-cost long-term verification. |
| **Providers are dumb encrypted mailboxes** | Store envelopes, not conversations. Never read content. Never own conversation state. |
| **Rich content is typed, signed, schema-validated blocks** | Not HTML tables. Deterministic rendering, native accessibility, render-capability negotiation. |
| **Anti-spam is economic and cryptographic** | Verified sender identity, proof-of-work, refundable bonds, recipient-controlled policy — not AI-beats-AI content classification. |
| **SMTP interop is bidirectional and labeled honestly** | `bridged · encrypted`, `email`, and `Registered` chips make provenance visible at a glance. |

## Reading paths

Pick one. The rest of the spec is structured so each path produces a complete mental model without reading anything else.

### Executive / funder — 15 minutes

Read [`docs/whitepaper.md`](../docs/whitepaper.md) — one document, proposal style. The only artifact in the repository written without any RFC 2119 keywords.

### Product person — 30 minutes

Read the four `overview/` files:

1. [**What Marque replaces**](./overview/01-what-marque-replaces.md) — the problem statement.
2. [**What Marque is**](./overview/02-what-marque-is.md) — the architectural commitment list.
3. [**How it feels**](./overview/03-how-it-feels.md) — a walkthrough from a user's perspective.
4. [**When to adopt**](./overview/04-when-to-adopt.md) — timeline and signals.

### Implementer — 4 hours

Read [`overview/02`](./overview/02-what-marque-is.md) to orient, then [`overview/05`](./overview/05-mvp.md) for the twelve-week implementation-sequencing plan (the first slice of the full surface, not a feature cut), then the full `protocol/` sequence:

1. [Terminology](./protocol/01-terminology.md)
2. [Identity](./protocol/02-identity.md)
3. [Architecture](./protocol/03-architecture.md)
4. [Rooms](./protocol/04-rooms.md)
5. [Cryptography](./protocol/05-cryptography.md)
6. [Content](./protocol/06-content.md)
7. [Legal proof](./protocol/07-legal-proof.md)
8. [Anti-spam](./protocol/08-anti-spam.md)
9. [SMTP interop](./protocol/09-interop.md)
10. [Conformance](./protocol/10-conformance.md)

### IETF reviewer — 2 hours

- [`drafts/draft-tamim-marque-arch-00`](../drafts/draft-tamim-marque-arch-00.md) — the Informational architectural frame for the normative companion drafts. Start here.
- [`protocol/02`](./protocol/02-identity.md), [`03`](./protocol/03-architecture.md), [`05`](./protocol/05-cryptography.md), [`07`](./protocol/07-legal-proof.md) for the technical core.
- [`context/05`](./context/05-risks.md) for known risks and open WG questions.

### QTSP / trust-list operator — 1 hour

- [`docs/compliance/eidas-mapping.md`](../docs/compliance/eidas-mapping.md) — side-by-side eIDAS 2.0 + ETSI ESI ↔ Marque mapping.
- [`protocol/07-legal-proof.md`](./protocol/07-legal-proof.md) — normative `ProofEnvelope` and Qualified tier semantics.

## Status

Pre-RFC founding specification. Target track: IETF Standards Track with ETSI ESI and W3C DID WG liaisons. Companion to the *Mailroom* AI-native MTA specification (Documents 1–2 of this series).

## Non-goals

- **Not a cryptocurrency.** No protocol token, no per-message gas, no wallet requirement.
- **Not a blockchain.** Bitcoin anchoring is used narrowly for timestamp non-repudiation; content, identity, and routing stay off-chain.
- **Not a client specification.** Marque defines wire format, identity, envelope, and block semantics. Client UX beyond the minimum rendering conformance is out of scope.
- **Not an MTA specification.** Provider behavior beyond the minimum interop surface is specified by companion documents (e.g. *Mailroom*).
- **Not a replacement for instant messaging.** Marque is async-first. Real-time chat remains better served by Matrix, Signal, or SimpleX.

---

Continue to [**Overview / What Marque replaces**](./overview/01-what-marque-replaces.md).
