# Marque specification

> **Registered communication, by default.**

Pre-RFC founding specification. Target track: IETF Standards Track with ETSI ESI and W3C DID WG liaisons. Companion to the _Mailroom_ AI-native MTA specification (Documents 1–2 of this series).

Start at [`00-overview.md`](./00-overview.md) — ten-minute product overview with three audience-specific reading paths.

---

## Structure

```
spec/
├── 00-overview.md              Marque in 10 minutes
├── overview/                   Product, UX, adoption — read for orientation
│   ├── 01-what-marque-replaces.md
│   ├── 02-what-marque-is.md
│   ├── 03-how-it-feels.md
│   ├── 04-when-to-adopt.md
│   └── 05-mvp.md               First-slice implementation-sequencing plan (twelve-week)
├── protocol/                   Normative specification — read to implement
│   ├── 01-terminology.md
│   ├── 02-identity.md
│   ├── 03-architecture.md
│   ├── 04-rooms.md
│   ├── 05-cryptography.md
│   ├── 06-content.md
│   ├── 07-legal-proof.md
│   ├── 08-anti-spam.md
│   ├── 09-interop.md
│   └── 10-conformance.md
└── context/                    Supporting material — read for rationale
    ├── 01-blockchain-scope.md
    ├── 02-commercial-model.md
    ├── 03-migration.md
    ├── 04-naming.md
    ├── 05-risks.md
    └── 06-related-work.md
```

Every file opens with a one-sentence abstract and a **"You will learn"** bullet list so you can decide in ten seconds whether to keep reading. Every file ends with a handoff to the next.

## Reading paths

### Executive / funder — 15 minutes

- [`docs/whitepaper.md`](../docs/whitepaper.md) — proposal-style summary, no RFC 2119 keywords.

### Product person — 30 minutes

- [`00-overview`](./00-overview.md)
- [`overview/01–05`](./overview/)

### Implementer — 4 hours

- [`overview/02`](./overview/02-what-marque-is.md) — orientation
- [`overview/05-mvp`](./overview/05-mvp.md) — twelve-week first-slice implementation-sequencing plan
- [`protocol/01–10`](./protocol/) — in order

### IETF reviewer — 2 hours

- [`drafts/draft-tamim-marque-arch-00`](../drafts/draft-tamim-marque-arch-00.md) — Informational architecture draft
- [`protocol/02`](./protocol/02-identity.md) — identity
- [`protocol/03`](./protocol/03-architecture.md) — architecture
- [`protocol/05`](./protocol/05-cryptography.md) — cryptography
- [`protocol/07`](./protocol/07-legal-proof.md) — legal proof
- [`context/05`](./context/05-risks.md) — risks and open questions

### QTSP / trust-list operator — 1 hour

- [`docs/compliance/eidas-mapping.md`](../docs/compliance/eidas-mapping.md) — eIDAS 2.0 + ETSI ESI ↔ Marque mapping
- [`protocol/07-legal-proof.md`](./protocol/07-legal-proof.md) — normative Qualified-tier semantics

## RFC 2119 language

Normative keywords — **MUST**, **MUST NOT**, **SHALL**, **SHOULD**, **SHOULD NOT**, **MAY**, **RECOMMENDED**, **OPTIONAL** — appear in uppercase and carry the meanings of [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174). Lowercase uses carry their ordinary English meaning.

## Companion documents

| Resource                                                                                    | Purpose                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Whitepaper (`docs/whitepaper.md`)](../docs/whitepaper.md)                                  | Proposal-style summary for executives, funders, and the press. Non-normative.                                                                                                                                      |
| [eIDAS mapping (`docs/compliance/eidas-mapping.md`)](../docs/compliance/eidas-mapping.md)   | eIDAS 2.0 + ETSI ESI ↔ Marque side-by-side for QTSPs and national trust-list operators.                                                                                                                            |
| [Internet Drafts (`drafts/`)](../drafts/)                                                   | Individual submissions to the IETF datatracker. Two normative complete (core, did-mail); one informational complete (arch); six stubs awaiting content migration.                                                  |
| [MIP process (`mips/`)](../mips/)                                                           | Marque Improvement Proposals — the process by which this specification evolves.                                                                                                                                    |
| [JSON Schemas (`schemas/`)](../schemas/)                                                    | Machine-readable schemas for envelopes, DID documents, capability records, ProofEnvelopes, and `core.*` blocks. Canonical CBOR wire format lives in [`schemas/cddl/envelope.cddl`](../schemas/cddl/envelope.cddl). |
| [Examples (`examples/`)](../examples/)                                                      | Worked envelopes, DID documents, capability records, ProofEnvelopes, MBS messages.                                                                                                                                 |
| [Architecture overview (`docs/architecture-overview.md`)](../docs/architecture-overview.md) | Inline Mermaid diagrams of identity stack, message path, provider role, ProofEnvelope composition, SMTP bridge, MIP lifecycle.                                                                                     |
| [FAQ (`docs/faq.md`)](../docs/faq.md)                                                       | Recurring questions with short answers and deep-links.                                                                                                                                                             |

## Status

- **Lexicon and terminology reference** — merged into [`protocol/01`](./protocol/01-terminology.md).
- **CDDL normative wire format** — [`schemas/cddl/envelope.cddl`](../schemas/cddl/envelope.cddl).
- **19 original flat-numbered Parts** (before the 2026-04-23 restructure) are preserved in git history as the `c67dba4` baseline and its five thematic commits.
- **`draft-tamim-marque-core-00`**, **`draft-tamim-marque-did-mail-00`**, **`draft-tamim-marque-arch-00`** — complete, awaiting IETF submission. The first two are Standards-Track; `arch` is Informational and frames the companion drafts.
- **Six stub Internet-Drafts** — `proof`, `mbs`, `bridge`, `antispam`, `rooms`, `storage` — frontmatter and scope only; normative content migration pending.
- **Non-spec companion artifacts** — [`docs/whitepaper.md`](../docs/whitepaper.md) (executive summary) and [`docs/compliance/eidas-mapping.md`](../docs/compliance/eidas-mapping.md) (eIDAS conformance mapping) complete.

## Version

Pre-charter. Each MIP merge bumps a revision counter on `CHANGELOG.md`. The spec is marked `pre-RFC` throughout. Post-IETF charter, version tags align with Internet-Draft revisions.
