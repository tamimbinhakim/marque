# Marque

> **Registered communication, by default.**

**Marque** is a cryptographically signed, federated messaging protocol proposed as the successor to SMTP and IMAP. It replaces email over a 10–20 year transition while bridging bidirectionally to legacy mail from day one — so no correspondent is ever left behind.

Marque treats identity as a portable [W3C DID][did] the user owns, not an address the provider rents. Content is wrapped in [MLS (RFC 9420)][mls] for end-to-end encryption by default. Legal proof is a tiered option that maps to [eIDAS 2.0 QERDS][eidas]. HTML's frozen 2005 subset is replaced with a typed, signed block format. Non-repudiable timestamps are anchored in Bitcoin via [OpenTimestamps][ots] at zero marginal cost while keeping message content, routing, and identity entirely off-chain.

> [!NOTE]
> **Status — pre-RFC founding specification.** Target track: IETF Standards Track with ETSI ESI and W3C DID WG liaisons. Target IETF submission: Q3 2026. Spec v1.0 target: 2028. Expect frequent updates until the first WG draft. The repository [CHANGELOG](./CHANGELOG.md) tracks every revision.

> [!TIP]
> If any acronym below (KERI, QERDS, FROST, ML-DSA, OTS, …) is unfamiliar, the [glossary and citation hub](./docs/glossary.md) defines every term with a one-line meaning and a link to the authoritative source.

---

## Read the spec

Start at [`spec/00-overview.md`](./spec/00-overview.md) — ten minutes, three reading paths.

| Audience                       | Reading path                                                                                                                           | Time       |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| **Executive / funder**         | [`docs/whitepaper.md`](./docs/whitepaper.md)                                                                                           | 15 minutes |
| **Product person**             | `overview/01`, `02`, `03`, `04`                                                                                                        | 30 minutes |
| **Implementer**                | `overview/02`, then [`overview/05`](./spec/overview/05-mvp.md) for the implementation-sequencing plan, then `protocol/01–10` in order  | 4 hours    |
| **IETF reviewer**              | [`drafts/draft-tamim-marque-arch-00`](./drafts/draft-tamim-marque-arch-00.md), then `protocol/02`, `03`, `05`, `07`, then `context/05` | 2 hours    |
| **QTSP / trust-list operator** | [`docs/compliance/eidas-mapping.md`](./docs/compliance/eidas-mapping.md), then `protocol/07-legal-proof.md`                            | 1 hour     |

The specification is organized in three tracks:

```
spec/
├── 00-overview.md              Marque in 10 minutes
├── overview/                   Product, UX, adoption
├── protocol/                   Normative wire formats, identity, crypto, content, legal proof
└── context/                    Blockchain scope, commerce, migration, naming, risks, related work
```

---

## Core commitments

| Commitment                | What Marque commits to                                                                                                                                                                                  |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Portable identity**     | A [`did:mail`](./spec/protocol/02-identity.md) the user owns — a long-lived keypair with a [KERI][keri]-style signed rotation log. Provider change is an entry edit; no correspondent updates anything. |
| **End-to-end encryption** | Encrypted by default via [MLS][mls], with a per-message [deniability-vs-non-repudiation choice](./spec/protocol/05-cryptography.md) and a [post-quantum hybrid][mlkem] cipher suite.                    |
| **Legal proof**           | Native and tiered: see the [`ProofEnvelope`](./spec/protocol/07-legal-proof.md). Maps to [eIDAS Article 43 / 44][eidas] (ERDS / [QERDS][eidas]). Anchored in Bitcoin via [OpenTimestamps][ots].         |
| **Commodity providers**   | Dumb encrypted mailboxes. Store envelopes, not conversations. Never read content. Never own conversation state. See [`provider-role.mmd`](./docs/diagrams/provider-role.mmd).                           |
| **Typed content**         | The [Marque Block Spec (MBS)](./spec/protocol/06-content.md) — typed, signed, schema-validated blocks. Not HTML tables. Deterministic rendering, native accessibility.                                  |
| **Anti-spam**             | [Economic and cryptographic](./spec/protocol/08-anti-spam.md): verified identity, [Argon2id][argon2] proof-of-work, refundable bonds, gossip reputation. Content filtering has hit its ceiling.         |
| **Fair competition**      | Providers compete on features, not address lock-in. Portable identity forces it. See [`spec/context/02-commercial-model.md`](./spec/context/02-commercial-model.md).                                    |
| **SMTP interop**          | Bidirectional [SMTP bridge](./spec/protocol/09-interop.md), opportunistic, clearly labeled, for 10–20 years. See [`smtp-bridge.mmd`](./docs/diagrams/smtp-bridge.mmd).                                  |

Marque is a composition of components that work in production today — [Bluesky AT][atproto], [MLS][mls], [Sigstore][rekor], [OpenTimestamps][ots], [PEC][pec], [Hyperswarm][hyperswarm], [SimpleX][simplex], [MIMI][mimi], [FROST][frost], [Argon2id][argon2], [BLAKE3][blake3], [QUIC][quic] — wrapped in an opinionated architecture with firm red lines against on-chain content, per-message gas, wallet-only identity, single-provider capture, server-owned conversation state, and spec-optional extensions. Every component above is defined and linked in the [glossary and citation hub](./docs/glossary.md).

---

## Repository layout

```
marque/
├── spec/               Specification (overview / protocol / context)
├── drafts/             IETF Internet-Draft source (2 complete, 1 informational, 6 stubs)
├── mips/               Marque Improvement Proposals — process and template
├── schemas/            JSON Schema + CDDL for envelopes, DIDs, blocks
│   ├── cddl/           Normative CBOR wire format
│   ├── _defs/          Shared schema fragments
│   └── blocks/         One file per core.* MBS block type
├── examples/           Worked envelopes, DID documents, ProofEnvelopes, MBS messages
├── docs/               Whitepaper, FAQ, glossary, architecture overview, roadmap, editors
│   ├── compliance/     eIDAS ↔ Marque mapping for QTSPs and trust-list operators
│   └── diagrams/       Mermaid sources
└── .github/            Issue templates, PR template, CODEOWNERS
```

### Diagrams at a glance

- [Identity stack](./docs/diagrams/identity-stack.mmd) — three-tier key hierarchy and the DID document.
- [Message path](./docs/diagrams/message-path.mmd) — P2P, store-and-forward, and mixnet delivery.
- [Provider role](./docs/diagrams/provider-role.mmd) — what the dumb encrypted mailbox does and refuses to do.
- [ProofEnvelope](./docs/diagrams/proof-envelope.mmd) — how Casual, Attested, and Certified compose.
- [SMTP bridge](./docs/diagrams/smtp-bridge.mmd) — inbound discovery and encryption upgrade.
- [MIP lifecycle](./docs/diagrams/mip-lifecycle.mmd) — Idea → Draft → Review → Last Call → Final.

All six are rendered inline in [`docs/architecture-overview.md`](./docs/architecture-overview.md) and [MIP-0001](./mips/mip-0001-process.md).

---

## Getting involved

- **Read the [spec overview](./spec/00-overview.md)** first — ten minutes, three audience-specific reading paths.
- **Read the [FAQ](./docs/faq.md)** — most recurring questions have short answers.
- **Propose a change** via a [Marque Improvement Proposal](./mips/README.md). Start from [`mips/mip-template.md`](./mips/mip-template.md).
- **File an issue** using one of the [issue templates](./.github/ISSUE_TEMPLATE) — bug report, spec clarification, or MIP stub.
- **Contribute** per [CONTRIBUTING.md](./CONTRIBUTING.md).
- **Report a security issue** privately per [SECURITY.md](./SECURITY.md).
- **Governance** is documented in [GOVERNANCE.md](./GOVERNANCE.md).

---

## Related work

Marque composes from proven systems. Each reference below is a debt we acknowledge, detailed in [`spec/context/06-related-work.md`](./spec/context/06-related-work.md):

- [Bluesky AT Protocol][atproto] — data-provider separation, account migration at scale.
- [MLS / RFC 9420][mls] — end-to-end group key agreement.
- [KERI][keri] — append-only key-event-log discipline for portable identity rotation.
- [Sigstore Rekor][rekor] and [OpenTimestamps][ots] — transparency and Bitcoin anchoring.
- [Italian PEC][pec] — legal-tier registered electronic mail at scale.
- [Hyperswarm][hyperswarm] — P2P UDP holepunching.
- [SimpleX][simplex] — metadata-private message queues.
- [MIMI WG][mimi] — async MLS transport.
- [FROST / RFC 9591][frost] — threshold Schnorr signatures.
- [Argon2id / RFC 9106][argon2] — memory-hard PoW.
- [BLAKE3][blake3] — content-addressed hashing.
- [QUIC / RFC 9000][quic] — modern transport.

---

## Companion: Mailroom

The **Mailroom** MTA (Documents 1–2 of this research series) is **an example** AI-native implementation of the provider role. It is not privileged; no implementation in the Marque ecosystem is. Implementers are unrestrained in how they differentiate — AI triage, UX, search, discovery, storage strategy, compliance posture — subject only to passing the conformance suite at the wire. Every implementation that passes the suite is a peer.

Marque defines the wire protocol; Mailroom is one of many expected provider implementations — the **HTTP / nginx** relationship, not **HTTP / Chrome**. A second, third, and Nth provider implementation, each from an unaffiliated group, is explicit charter-level policy.

---

## Authorship and AI collaboration

Marque — the specification, the Internet-Drafts, the schemas, the whitepaper, the eIDAS compliance mapping, the architecture diagrams, the MIP process, and the supporting prose — was researched and written by **Tamim Bin Hakim** in collaboration with **[Claude](https://claude.ai)** (Anthropic). The human is the author, architect, and decider; Claude was research partner, drafter, reviewer, and foil.

We credit Claude plainly — in commit `Co-Authored-By:` footers, in Acknowledgments sections, and here — because:

- A specification of this ambition is not research a single human completes in a reasonable window without capable tools. Using a capable AI is not a shortcut; it is how current work gets done.
- Pretending a document was wholly human-written when it wasn't is dishonest, and hiding AI contribution to avoid reviewer bias is a form of reviewer manipulation.
- Marque exists partly because of AI-created pressures on email (LLM-generated phishing, prompt injection into inbox assistants, AI-scale reputation-gating). Using an LLM to reason about the protocol that defends against those pressures is neither ironic nor contradictory — it is the honest division of labor.
- Responsibility for every normative MUST and every regulatory citation remains the author's. Claude is acknowledged; Claude is not a co-editor of record.

This convention is open to contributors who prefer to work without AI assistance and to those who prefer to credit their AI collaborator similarly. Either way, the `Co-Authored-By:` footer is the place to say so.

---

## License

Dual-licensed:

- Specification text under [Creative Commons Attribution 4.0 (CC-BY 4.0)](./LICENSE-SPEC).
- Reference code and schemas under [Apache-2.0](./LICENSE).

See [`LICENSE`](./LICENSE) for the full text.

---

[did]: https://www.w3.org/TR/did-core/
[mls]: https://datatracker.ietf.org/doc/html/rfc9420
[mlkem]: https://csrc.nist.gov/pubs/fips/203/final
[keri]: https://arxiv.org/abs/1907.02143
[eidas]: https://eur-lex.europa.eu/eli/reg/2024/1183/oj
[ots]: https://opentimestamps.org
[echoleak]: https://nvd.nist.gov/vuln/detail/CVE-2025-32711
[atproto]: https://atproto.com
[rekor]: https://docs.sigstore.dev/logging/overview/
[pec]: https://it.wikipedia.org/wiki/Posta_elettronica_certificata
[hyperswarm]: https://github.com/holepunchto/hyperswarm
[simplex]: https://simplex.chat
[mimi]: https://datatracker.ietf.org/wg/mimi/about/
[frost]: https://datatracker.ietf.org/doc/rfc9591/
[argon2]: https://datatracker.ietf.org/doc/rfc9106/
[blake3]: https://github.com/BLAKE3-team/BLAKE3
[quic]: https://datatracker.ietf.org/doc/html/rfc9000
