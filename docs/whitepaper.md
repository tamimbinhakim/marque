# Marque — a protocol proposal to replace email

> Pre-RFC founding specification. April 2026.
> Target track: IETF Standards Track. ETSI ESI and W3C DID WG liaisons in preparation.
> Status: research complete, first interoperable implementations pending, IETF submission Q3 2026.

> [!TIP]
> Every acronym in this whitepaper — KERI, MLS, ERDS, QERDS, FROST, OpenTimestamps, ML-DSA, RFC 3161, CAdES, PEC, AgID — is defined with a one-line meaning and an authoritative-source link in the [glossary and citation hub](./glossary.md). Open it in a second tab if you want footnotes.

## The elevator pitch

Email is broken in ways its forty-year history of patches has failed to fix. Marque is a cryptographically signed, federated, peer-to-peer-capable messaging protocol proposed as its successor. It keeps everything about email that worked — asynchronous, addressable, federated, vendor-neutral — and fixes what did not: identity is portable cryptographic, content is end-to-end encrypted by default, rich content is a typed signed block log rather than frozen-2005 HTML, legal proof is native and tiered, anti-spam is economic and cryptographic rather than AI-beats-AI content classification, and the bridge to legacy SMTP is bidirectional and honestly labeled throughout a ten-to-twenty year transition.

One sentence: **Marque is a federation of dumb encrypted mailboxes with portable cryptographic identities, a typed content layer, and a tiered legal-proof envelope — wrapped in a bidirectional SMTP bridge for the transition.**

## Why now

Three independent pressures converge in 2025–2027 that make the patch path no longer credible:

**1. Two operators now gatekeep legitimate mail.** Gmail and Microsoft together control the reputation data that decides whether an independently-run mail server delivers at all. Self-hosting against Gmail has become structurally infeasible, regardless of how correctly SPF, DKIM, DMARC, ARC, BIMI, MTA-STS, and TLS-RPT are configured. Independent providers survive on tolerated deliverability; regulators have begun to notice.

**2. Content filtering has hit its ceiling against LLM-generated abuse.** Recent industry phishing-trend reports (e.g., Hoxhunt's 2025–2026 series) put the AI-generated share of analyzed phishing in the ~80%+ range, with measurably higher click-through versus traditional templated phish. Running a larger classifier is no longer a strategy; the attacker's classifier is just as large.

**3. AI-assistant inboxes introduce prompt injection as a weaponized class.** [CVE-2025-32711 (EchoLeak)](https://nvd.nist.gov/vuln/detail/CVE-2025-32711) — a zero-click prompt-injection exfiltration via email against Microsoft 365 Copilot, disclosed by Aim Security, CVSS 9.3 — was the first of its kind. It will not be the last. The mitigation is a content format that separates instructions from data structurally. SMTP has no such format.

Add: **eIDAS 2.0 ([Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj)) entered into force 20 May 2024** and Qualified Electronic Registered Delivery Service (QERDS) implementing acts land through 2026–2027. The EU regulatory path creates a precedent-deployment slipstream for a standards-track protocol designed to carry QERDS semantics natively. This is the window.

## What Marque commits to

Seven commitments. Each one removes a class of SMTP failure.

**1. Identity is cryptographic and portable.** A user's identity is a [`did:mail`](../spec/protocol/02-identity.md) — a long-lived cryptographic keypair with a published [DID document][did] listing service endpoints, signed over by a [KERI][keri]-style key event log. Changing providers is an entry in the DID document; correspondents update nothing. The phone-number-portability analogy, applied correctly. Recovery is a configurable M-of-N guardian threshold; the provider is allowed to be one guardian but not sole guardian.

**2. Content is end-to-end encrypted by default.** Messages ride inside [MLS][mls] groups (RFC 9420, 2023). One group per thread. Post-compromise security, forward secrecy, `O(log n)` scaling to 50 000-member groups. The mandatory cipher suite is classical ([X25519][rfc7748] / [Ed25519][rfc8032]); a [post-quantum hybrid][hybrid-kem] (X25519+[ML-KEM-768][mlkem] / Ed25519 + [ML-DSA-65][mldsa]) is registered and will be promoted to mandatory when NIST deprecates the classical baseline.

**3. Legal proof is native and tiered.** Three tiers: Casual (no chip), Signed / Attested ([ERDS][eidas] grade, maps to eIDAS Article 43), Qualified / Registered ([QERDS][eidas] grade, maps to eIDAS Article 44). Composition: sender DSK signature + [RFC 3161][rfc3161] qualified timestamps + optional [FROST (RFC 9591)][frost] threshold signatures from notary quorum + [OpenTimestamps][ots] Bitcoin anchor for long-term verifiability at zero marginal cost + [CAdES-B-LTA][cades] archival timestamps for Qualified tier. [QTSP][eidas] co-signature for Qualified tier provides the legal-person binding required by eIDAS. Full structure: [`ProofEnvelope`](../spec/protocol/07-legal-proof.md).

**4. Providers are commodity encrypted mailboxes.** Providers store envelopes. Not conversations. Not membership state. Not threads. Storage cost scales as `envelopes × ciphertext_size`, not as `users × rooms × membership × history`. A provider holding 30 days of mail for a million users is one well-specced rack. Providers MUST NOT read content, MUST NOT own conversation state, MUST NOT see the social graph beyond immediate deposit metadata. This commodity-role design is what lets new providers enter the market without acquiring years of reputation data.

**5. Content is a typed signed block log.** The Marque Block Spec (MBS) replaces HTML email's frozen 2005 subset with JSON-based, schema-validated, reverse-DNS-namespaced blocks. Core blocks cover text, media, attachments, polls, schedules, forms, payment requests, reactions, replies, forwards, retractions, and the `core.signed_action` primitive that is the Marque-native DocuSign replacement. Rendering is deterministic. Accessibility is mandatory (in the schema, not in prose). Unknown blocks fall back to their `fallback` field; the envelope additionally carries `alt_plaintext` so no message is ever unreadable.

**6. Anti-spam is economic and cryptographic.** An AND-of-ORs of: verified sender identity (spoofing structurally impossible), Argon2id proof-of-work (fast for known contacts, slow for strangers), refundable bonds for stranger outreach (burned on spam report, 14-day appeal window), reputation gossip, recipient-controlled policy with machine-readable rejection reasons (silent drop-to-spam prohibited), and a per-user opt-in AI-assistant allowlist. At the content layer, structural data-versus-instruction separation defends AI-assistant inboxes against prompt injection.

**7. SMTP interop is bidirectional and honestly labeled.** A Marque provider ships an SMTP bridge from day one. Inbound SMTP with a discoverable sender key wraps into MLS and arrives with a `bridged · encrypted` chip; inbound without a discoverable key arrives with a plain `email` chip. Outbound to an SMTP-only recipient downgrades cleanly with an Autocrypt header for opportunistic upgrade, an authenticated download link for attachments (7-day default, per-recipient tokens, revocable, access-logged), and a Received-chain truncation to two outermost hops to reduce vendor-internal PII leakage. Legal-tier messages refuse to downgrade silently — they either fail closed or invoke a QERDS-to-legacy gateway.

## The shape of a message

A Marque message is:

* An outer **canonical CBOR envelope** carrying sender DID, recipient pseudonym, auth mode, privacy tier, cipher suite, [BLAKE3][blake3] content ID, pad bucket, and optional proof-envelope reference.
* An **MLS-encrypted payload** containing the [Marque Block Spec](../spec/protocol/06-content.md) document (the typed content).
* An optional [**ProofEnvelope**](../spec/protocol/07-legal-proof.md) retrieved out-of-band, carrying the tiered non-repudiation artifact (signatures, timestamps, Bitcoin anchor, optional QTSP qualified certificate).

Three actors: the **sender's client**, the **recipient's client**, and any number of **providers** acting as opaque relay mailboxes. A fourth role, **notary**, co-signs proof envelopes at the Attested and Qualified tiers. Providers and notaries are frequently the same entity in deployment but are architecturally distinct.

## Why this will take ten to twenty years

Because every protocol replacement at the internet scale does. HTTPS took ten years with Google's backing and Let's Encrypt's subsidy. IPv6 is at twenty-eight years and roughly half-done. Marque's trajectory:

| Phase                       | Years | Handles       | Shape                                                                                           |
| --------------------------- | ----- | ------------- | ----------------------------------------------------------------------------------------------- |
| **1. Foundation**           | 0–2   | 5K–10K        | Privacy forums, self-hosters, protocol enthusiasts. At least two independent, unaffiliated interoperable implementations. Bridge tier-zero. |
| **2. First providers**      | 2–3   | 500K–2M       | ≥2 unaffiliated privacy-first incumbents ship opt-in. First QTSP in Italy. Developer SDKs.       |
| **3. Regulatory/enterprise**| 3–5   | 10M–50M       | eIDAS QERDS listings (Italy, France, Germany). Fortune 500 compliance replaces DocuSign stacks.  |
| **4. Majority of new signups** | 5–10  | 100M–500M  | iOS Mail / Gmail app / Outlook ship native. Default-on at top-10 providers. The Gmail moment.     |
| **5. Equilibrium**          | 10–20 | Majority      | Major providers announce SMTP receive-only. SMTP becomes Ethernet-over-copper of messaging.      |

**One number to set expectations: spec v1.0 target is 2028; first commercial deployments 2028–2029; eIDAS QERDS listing 2029–2030; Phase-4 tipping point 2033–2036.** Honesty about timelines is baked into the proposal because it has to be.

## Honest about costs

> [!CAUTION]
> This is a **10–20 year standards proposal**, not a startup launch. The MVP requires two unaffiliated interoperable implementations, a conformance test suite, an operational SMTP bridge, and at minimum one QTSP partnership for Qualified-tier operation. Read this section before forming expectations about timeline, scope, or commercial readiness.

The honest tradeoffs of the design, in one place:

* **Marque is a 10–20 year transition, not a product launch.** There is no "flag day" when SMTP goes dark. The bridge is load-bearing for at least the first decade.
* **The MVP is large for a volunteer project.** At least two independent interoperable implementations from unaffiliated groups, a conformance test suite, and an operational bridge are minimum. Marque has no canonical or "reference" implementation — every implementation that passes the conformance suite is a peer, and clients and providers are expected to compete on features (AI triage, UX, search, discovery) rather than on protocol surface. A tightly scoped first-phase team per implementation is roughly five to eight engineers for twelve to eighteen months. See [`spec/overview/05-mvp.md`](../spec/overview/05-mvp.md) for the minimum three-month implementation plan.
* **QERDS-tier operation requires a QTSP.** The [Italian Agenzia per l'Italia Digitale (AgID)](https://www.agid.gov.it/en/news/digital-identity-pec-qualified-electronic-signature-online-report-on-agid-supervised-services) path is the concrete wedge; Italian PEC is already a regulated QERDS-equivalent system with **~16 million active accounts and ~2.5 billion messages per year** (AgID supervised-services report). Every other Qualified-tier deployment inherits from the Italian precedent or sets up a parallel QTSP partnership in its own jurisdiction.
* **Mobile battery budgets are real.** The default-recommended Sealed privacy tier is the strongest metadata reduction we believe is compatible with 2026-era mobile clients. Mixed tier (onion / mixnet) is available for higher-sensitivity correspondence but is not default-on.
* **We lose some convenience.** Qualified tier requires sender confirmation of per-message cost before commit (§6.7b of the founding specification). `@gmail.com` does not. This is a feature — it surfaces the cost of a registered-mail send rather than hiding it in a subscription — but it is also friction.

## Why this is not a blockchain, not a cryptocurrency, and not a chat app

**Not a blockchain.** Bitcoin anchoring via OpenTimestamps is used narrowly for zero-cost non-repudiable timestamping. Content, identity, routing, and conversation state stay off-chain. There is no consensus layer, no protocol token, no smart contract.

**Not a cryptocurrency.** No per-message gas, no wallet requirement, no token economics. Refundable bonds use existing fiat rails (USD / EUR / GBP) with an optional BTC mode.

**Not a chat app.** Marque is async-first. Real-time chat remains better served by [Signal](https://signal.org), [Matrix](https://matrix.org), or [SimpleX][simplex]. The MLS substrate is shared with those systems but the addressing model (persistent DIDs, not per-session Signal numbers) and the async-delivery extensions ([KeyPackage][mls] pools, epoch checkpoints for long-offline peers, delivery via dumb encrypted mailbox rather than push notification) are specific to Marque's use case.

**Not a client specification.** Marque specifies wire format, identity, envelope, content blocks, and the minimum rendering conformance. Client UX beyond that — how inboxes are laid out, how threads are visualized, how the Registered chip is rendered — is out of scope. Conformance is measured at the wire, not at the pixel.

## Who this is for

Two primary audiences. Everything else is a concrete form one of them takes.

### 1. Anyone who wants an inbox no one can spy into

If you use email — and in 2026 that's effectively everyone — Marque is for you. The default is end-to-end encrypted mail where **your provider sees opaque ciphertext, not content**; your identity is a cryptographic object **you own**, not an address the provider rents; and **leaving a provider is an entry edit in your DID document**, not a loss of correspondence. No provider, no employer, no advertiser, and — at the stated threat model — no state-level adversary can read what you send or receive. That is the pitch for every individual user.

The consumer story is not a side benefit; it is the foundation. A protocol that does not work for the one-person inbox does not end up in the enterprise, the hospital, the law firm, or the parliament. Marque starts at the individual and layers everything else on top of the same wire.

### 2. Governments and regulators that need to send proper legal mail

Marque's Qualified tier carries [eIDAS 2.0](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) QERDS-grade non-repudiation: cryptographic proof of sender, recipient, content, delivery, and time, independently verifiable — no trusted third party required — and archivable for decades. This is the successor substrate to Italy's PEC, France's LRE, Germany's (now-defunct) De-Mail, and the rest of the national electronic-registered-delivery systems: a standards-track, internationally-interoperable wire format instead of a patchwork of per-country silos.

Governments deploy it for citizen correspondence, tax notices, court summons, medical records, regulated notifications, and intra-administration mail. QTSPs operate it under existing trust-list accreditation. The EU Commission, drafting eIDAS 2.0 implementing acts through 2026–2027, gets an IETF-standards-track interoperability target to point a mandate at.

### Where these two audiences land in practice

The verticals below are not separate audiences — they are where the "private inbox" and "legal mail" pitches arrive with specific requirements on top:

- **Individuals, families, small businesses** — private correspondence, no provider spying, address that survives provider changes. The default Casual tier covers this.
- **Healthcare** — doctor–patient messaging, imaging transfer, referrals, prescriptions under US HIPAA, EU medical-records directives, the UK NHS confidentiality framework. Qualified-tier provenance, deterministic content rendering, hybrid archive-of-record.
- **Legal** — attorney–client privileged correspondence in the cryptographic record; `core.signed_action` as a DocuSign replacement; `.marquebox` as a complete eDiscovery export.
- **Banking, insurance, finance** — MiFID II / DORA correspondence obligations, non-repudiable signed instructions, bonded stranger outreach.
- **Journalism and human-rights work** — Mixed-tier metadata unlinkability for source protection; portable-tier identity that survives the loss of any single DNS domain; machine-readable rejection reasons instead of silent drops.
- **Research and regulated administration** — human-subjects correspondence with auditable non-delivery states; provenance for every notice sent under ICH E6(R3), 45 CFR 46, or equivalent.
- **Defense, intelligence, classified communications** — sovereign DID namespaces deployed inside air-gapped enclaves with their own provider(s); cryptographic identity + tiered proof + metadata tier compatible with cross-domain secure-messaging requirements.
- **Mail providers and transactional-API services** — the commercial channels that ship the primary audiences a working Marque experience. Privacy-first providers get a post-E2EE differentiator; transactional-API services (Resend, Postmark, SendGrid, Mailgun, Loops, or equivalents) get the Phase-1 recipient-education beachhead where every cryptographic-receipt email teaches another inbox what Marque looks like.

Full vertical-by-vertical deployment considerations are documented in the supporting specification; this section exists to tell a reader which paragraph in the rest of the document is about their problem.

## What we are asking for

Not money. Marque is being released under [CC-BY 4.0 (spec)](../LICENSE-SPEC) and [Apache 2.0 (future code)](../LICENSE) without a foundation, without a token, and without a commercial sponsor. It is a standards proposal, governed under the [MIP-0001 process](../mips/mip-0001-process.md) documented in this repository.

What we need, in order:

1. **IETF reviewers and Working Group participants.** Read the founding specification's protocol track ([spec/protocol/](../spec/protocol/)) and the individual drafts in [drafts/](../drafts/). Flag problems. The architecture draft (`draft-tamim-marque-arch-00`) is the ≤2-hour entry point.
2. **Implementers.** The [twelve-week first-slice plan](../spec/overview/05-mvp.md) is explicitly scoped so a small team can ship interoperable mail quickly. At least two independent, unaffiliated implementations are required under charter rule; one is insufficient. Third, fourth, and Nth implementations are explicitly welcome. Example implementations exist and more are expected — none is privileged. Implementers are unrestrained in how they build above the wire; competition happens at the feature layer (AI, UX, search, discovery), not at the protocol.
3. **QTSP partners.** The Italian PEC ecosystem is the first concrete wedge. A QTSP interested in running the first QERDS-tier Marque deployment would compress Phase 2 significantly.
4. **Enterprise pilot customers.** A Fortune 500 legal, HR, or compliance team willing to trial `core.signed_action` against a current DocuSign deployment.
5. **Government procurement contacts.** National administrations evaluating post-PEC citizen correspondence substrates in Italy, France, Germany, Spain, Netherlands, Finland, Estonia.

## How to engage

**Read the spec.** Start with [`spec/00-overview.md`](../spec/00-overview.md) — ten minutes. Three audience-specific reading paths branch from there. An explicit [MVP surface](../spec/overview/05-mvp.md) is provided for implementers who want the minimum conformant subset before committing to the full track.

**Open an issue.** The [GitHub issue templates](../.github/ISSUE_TEMPLATE/) cover protocol questions, proposed changes, and security concerns. Security disclosures follow [SECURITY.md](../SECURITY.md).

**Propose a MIP.** Substantive changes to the protocol follow the [Marque Improvement Proposal process](../mips/mip-0001-process.md). The process is intentionally IETF-compatible so MIPs graduate cleanly into Internet-Draft sections.

**Mailing list and community call.** `marque-discuss@marqueproto.org` (planned, Phase 1 milestone). Monthly community call (planned, Phase 1 milestone).

**Cite the founding specification as `MARQUE-FOUNDING`** in academic and industrial work. The canonical URL is `https://marqueproto.org/spec/` (planned); until that resolves, cite the GitHub repository at `https://github.com/tamimbinhakim/marque` (placeholder — update on publication).

## Contact

**Editor.** Tamim Bin Hakim — `tamimbinhakim.work@gmail.com`. Author line of record on the Internet-Drafts.

**Governance.** The [GOVERNANCE.md](../GOVERNANCE.md) document specifies the editor role, the MIP process, and the path to IETF WG charter.

---

*This whitepaper accompanies the Marque founding specification. Normative text lives in [`spec/`](../spec/); Internet-Draft text lives in [`drafts/`](../drafts/); this document is a non-normative executive summary.*

[did]: https://www.w3.org/TR/did-core/
[mls]: https://datatracker.ietf.org/doc/html/rfc9420
[mlkem]: https://csrc.nist.gov/pubs/fips/203/final
[mldsa]: https://csrc.nist.gov/pubs/fips/204/final
[hybrid-kem]: https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/
[rfc7748]: https://datatracker.ietf.org/doc/html/rfc7748
[rfc8032]: https://datatracker.ietf.org/doc/html/rfc8032
[keri]: https://arxiv.org/abs/1907.02143
[eidas]: https://eur-lex.europa.eu/eli/reg/2024/1183/oj
[ots]: https://opentimestamps.org
[rfc3161]: https://datatracker.ietf.org/doc/html/rfc3161
[cades]: https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/
[frost]: https://datatracker.ietf.org/doc/rfc9591/
[blake3]: https://github.com/BLAKE3-team/BLAKE3
[simplex]: https://simplex.chat
