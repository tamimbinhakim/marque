# Roadmap

> Concrete milestones this repository tracks, organized against the five-phase migration strategy in [`spec/context/03-migration.md`](../spec/context/03-migration.md). Amended by Process-category MIPs; never by commit without a MIP.

---

## Phase 1 — Foundation (Year 0–2)

**Goal:** A stable wire format and at least two independent, unaffiliated provider implementations that a self-hoster can actually run against Gmail without losing correspondence.

### Spec milestones

- [x] MIP-0001 (MIP process) — **Active** (this repo).
- [x] Founding specification complete, reorganized into `spec/overview` + `spec/protocol` + `spec/context` tracks — **Done** (v0).
- [x] Normative terminology reference merged into [`spec/protocol/01-terminology.md`](../spec/protocol/01-terminology.md) — **Done** (v0).
- [x] CDDL normative wire format (`schemas/cddl/envelope.cddl`) — **Done** (v0).
- [x] Whitepaper / executive summary ([`docs/whitepaper.md`](./whitepaper.md)) — **Done** (v0).
- [x] eIDAS 2.0 ↔ Marque compliance mapping ([`docs/compliance/eidas-mapping.md`](./compliance/eidas-mapping.md)) — **Done** (v0).
- [x] MVP surface ([`spec/overview/05-mvp.md`](../spec/overview/05-mvp.md)) — **Done** (v0).
- [x] Informational architecture Internet-Draft (`draft-tamim-marque-arch-00`) — **Done** (v0, not yet submitted).
- [ ] Individual Internet-Draft `draft-tamim-marque-core-00` submitted to datatracker.ietf.org.
- [ ] Individual Internet-Draft `draft-tamim-marque-did-mail-00` submitted.
- [ ] Individual Internet-Draft `draft-tamim-marque-arch-00` submitted.
- [ ] Content migration to six stub drafts: `proof`, `mbs`, `bridge`, `antispam`, `rooms`, `storage`.
- [ ] IETF BoF proposal for WG `mpc` (Marque Protocol Core).
- [ ] Liaison statements to ETSI ESI, W3C DID WG, MIMI WG.
- [ ] IANA provisional registrations for `did:mail`, `core.*` block types, MBS algorithm codepoints, `marque://` URI scheme, `.marquebox` media type.
- [ ] Formal ETSI conformance self-assessment against **EN 319 521** (provider policy + security) using the **TS 119 524-1/-2** test suites (Phase-2 milestone; tracked in [`docs/compliance/eidas-mapping.md`](./compliance/eidas-mapping.md) §7).

### Implementation milestones

- [ ] **At least two Apache-2.0 independent interoperable implementations** from unaffiliated groups — the IETF standards-track maturity bar. Example implementations are welcome and expected (Mailroom on the provider side; a TypeScript web client, native mobile clients, native desktop clients on the client side). None is privileged. Implementers are unrestrained in how they differentiate — AI, UX, search, discovery, storage — subject only to passing the conformance suite at the wire.
- [ ] **SMTP bridges shipped day one.** Tier-zero operation. Buggy bridges lose users in a week.
- [ ] Conformance test suite covering all `core.*` block types and the MLS async transport profile.
- [ ] `did:mail` resolver library (JS, Rust, Python, Go).
- [ ] JSON Schemas published for envelope, DID doc, and every `core.*` block.

### Community milestones

- [ ] Public mailing list `marque-discuss@marqueproto.org`.
- [ ] Monthly community call.
- [ ] First implementer summit (virtual).

**Target adoption: 5 000–10 000 active DIDs.**

---

## Phase 2 — First commercial providers (Year 2–3)

**Goal:** At least two unaffiliated privacy-first incumbents shipping opt-in Marque support; one QTSP running QERDS-mode Marque in Italy.

### Spec milestones

- [ ] Post-WG adoption: draft renamed `draft-ietf-mpc-core-NN` and stable; core MIPs in `Final` status.
- [ ] ETSI conformance assessment of Marque against EN 319 521 (policy + security) + EN 319 522 (semantic + bindings) using the TS 119 524-1/-2 test suites.
- [ ] Cross-implementation interop test matrix published quarterly.

### Implementation milestones

- [ ] 1–2 of {Proton, Fastmail, Tutanota, Mailbox.org, Posteo} shipping opt-in support.
- [ ] First QTSP running Certified tier (Italy, leveraging PEC infrastructure).
- [ ] SDKs in JavaScript, Python, Go, Rust, Swift, Kotlin.
- [ ] Transactional API at Resend price parity.
- [ ] Thunderbird, K-9 Mail, Delta Chat native support.

**Commit: minimum two unaffiliated live providers at every milestone.**

**Target adoption: 500K–2M DIDs.**

---

## Phase 3 — Regulatory recognition and enterprise (Year 3–5)

**Goal:** eIDAS QERDS listing in multiple jurisdictions; Fortune 500 compliance adoption displaces DocuSign-plus-email stacks.

### Regulatory milestones

- [ ] Marque listed on national trust lists: Italy, France, Germany.
- [ ] ETSI EN 319 522-2 profile published covering all Attested and Certified tier semantics.
- [ ] First national mandate (likely Italy) for Marque-compatible QERDS as PEC successor.

### Enterprise milestones

- [ ] eDiscovery, DLP, legal-hold, retention, SSO/SCIM integrations at parity with Google Workspace enterprise tier.
- [ ] Confidential-compute AI-assistant inbox (on-device or regulated TEE) at parity with Copilot/Gemini.

**Target adoption: 10M–50M DIDs.**

---

## Phase 4 — Majority of new signups (Year 5–10)

**Goal:** The Gmail/Yahoo 2024 bulk-sender moment for Marque — two or more top-10 providers default-on Marque for new consumer accounts.

### Client milestones

- [ ] iOS Mail native support.
- [ ] Gmail app native support.
- [ ] Outlook native support.
- [ ] Default preference: if both endpoints support Marque, use Marque.

### Provider milestones

- [ ] Default-on for new consumer accounts at ≥2 top-10 mailbox providers.
- [ ] `downgrade=warn` published by default for enterprise domains.

**Target adoption: 100M–500M DIDs.**

---

## Phase 5 — Equilibrium (Year 10–20)

**Goal:** SMTP receive-only for major providers; Marque is the default for human correspondence.

### Structural milestones

- [ ] Major providers announce SMTP receive-only (or inbound-only) plans.
- [ ] Possible regulatory mandates for QERDS-equivalent delivery in tax, legal, medical contexts.
- [ ] SMTP becomes the Ethernet-over-copper of messaging — ubiquitous for M2M, unfashionable for humans.

---

## Cross-phase work

### Security track

- Annual external cryptographic review of normative primitives.
- Post-quantum ciphersuite upgrade path (post-FIPS 204/205 and successor curves).
- Public advisory program per [SECURITY.md](../SECURITY.md).
- Coordinated-disclosure registry for cross-implementation vulnerabilities.

### Standards track

- Sync with MIMI WG on async MLS transport updates.
- Track MLS WG for post-quantum ciphersuite codepoints.
- Sync with W3C DID WG for `did:mail` method registration.
- Sync with ETSI ESI on QERDS profile updates.

### Ecosystem track

- Quarterly implementer summit.
- Annual Marque conference (co-located with IETF or FOSDEM).
- Funded migration tooling for existing email archives.
- Public-good QTSP for individuals who cannot afford commercial qualified certificates.

---

## Accelerants we're watching

| Signal                                                  | Why it matters                                              |
| ------------------------------------------------------- | ----------------------------------------------------------- |
| **eIDAS 2.0 QERDS implementing acts (2025–2027)**       | Compresses 5 years of organic adoption into 2.              |
| **EU sovereign-email procurement**                      | Government contracts willing to pay for Marque conformance. |
| **Major AI-assistant breach via SMTP prompt injection** | Forces the industry to a protocol-level fix.                |
| **iOS 19 or Android 17 shipping Marque**                | Phase 4 trigger.                                            |

## Killers we're tracking

| Signal                                               | Mitigation                                                                      |
| ---------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Bridge reliability failing at scale**              | Bridge engineering is tier-zero; run ops like a bank, not like a hobby project. |
| **Gmail cloning Marque with incompatible semantics** | Conformance tests are mandatory; no spec-optional interop.                      |
| **MLS or `did:mail` crypto crisis**                  | Ship ciphersuite negotiation; design rotation into both.                        |
| **Single-provider launch capture**                   | Charter-level rule: minimum two unaffiliated providers at every milestone.      |

---

_This roadmap is living. It is amended by Process-category MIPs._
