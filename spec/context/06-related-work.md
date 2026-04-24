# Related work

> What Marque borrows, what it deliberately refuses, and why. Every architectural choice is traceable to a concrete prior system's success or failure.

**You will learn**

- The systems Marque studied and what it borrowed from each.
- The systems it refused to emulate, and why the refusal is load-bearing.
- Where to read more on each.

---

## 1. What Marque borrows

### 1.1 Bluesky AT Protocol — data-provider separation, account migration at scale

Bluesky's [AT Protocol](https://atproto.com) with a multi-PDS ecosystem (community trackers such as `bsky.jazco.dev` put third-party Personal Data Servers in the low thousands as of 2025) is the closest working model for separating identity from hosting. Marque borrows:

- The **PDS pattern** for mailbox providers.
- The **`did:plc` + nullification-window** pattern for account migration ([`protocol/02 §2.4`](../protocol/02-identity.md)). Note: `did:plc` is **not** a W3C-standardized DID method; it is maintained at [`did-method-plc/did-method-plc`](https://github.com/did-method-plc/did-method-plc) and Bluesky itself has stated it expects to evolve toward a less-centralized form over time.
- David Buchanan's adversarial PDS-migration documentation as the discipline template.

Marque does **not** borrow the firehose relay model — messaging does not need global public ordering.

### 1.2 MLS — RFC 9420

Messaging Layer Security ([RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420), published July 2023) is Marque's E2EE foundation. TreeKEM's O(log n) group operations, first-class epoch objects mapping to threads, and multiple audited independent implementations (OpenMLS, AWS `mls-rs`, Cisco `mlspp`, Wire — see the implementation directory at [messaginglayersecurity.rocks](https://messaginglayersecurity.rocks/)) made it the only viable choice for async group messaging at scale.

### 1.3 Sigstore Rekor and OpenTimestamps

Transparency logs with witness-signed tree heads ([Sigstore Rekor](https://docs.sigstore.dev/logging/overview/)) plus Bitcoin-anchored timestamps ([OpenTimestamps](https://opentimestamps.org)) give Marque global append-only guarantees for delivery receipts without running a cryptocurrency. See [`protocol/07`](../protocol/07-legal-proof.md).

### 1.4 Italian PEC — legal-tier registered electronic mail at scale

[PEC](https://it.wikipedia.org/wiki/Posta_elettronica_certificata) is the existence proof that a mass-scale legal-tier mail system can work: **~16 million active accounts, ~2.5 billion messages per year** ([AgID supervised-services report](https://www.agid.gov.it/en/news/digital-identity-pec-qualified-electronic-signature-online-report-on-agid-supervised-services)), mandatory for Italian businesses, consistent Court of Cassation recognition. **It works because regulation mandated it.** Marque's Certified tier targets eIDAS 2.0 QERDS so the regulation-to-protocol path is ready when implementing acts land.

### 1.5 Hyperswarm — P2P UDP holepunching

[Hyperswarm](https://github.com/holepunchto/hyperswarm) demonstrates that NAT-traversal P2P at the consumer-internet scale is operational. Marque's direct P2P fast path uses Hyperswarm-style holepunching with a 500 ms budget before falling back to store-and-forward.

### 1.6 SimpleX — metadata-private message queues

[SimpleX](https://simplex.chat)'s queue-per-contact metadata-privacy model informs Marque's Sealed privacy tier — specifically the use of recipient pseudonyms so providers learn delivery tokens, not social graphs.

### 1.7 MIMI WG — async MLS transport

The IETF [MIMI Working Group](https://datatracker.ietf.org/wg/mimi/about/) and `draft-ietf-mimi-protocol` provide the concrete HTTPS-over-MLS transport profile Marque adopts. Marque's async extensions (KeyPackage pools, epoch checkpoints) build on MIMI's foundation.

### 1.8 FROST — RFC 9591

[FROST](https://datatracker.ietf.org/doc/html/rfc9591) threshold Schnorr signatures under the (Ed25519, SHA-512) ciphersuite defined in RFC 9591 §6 produce 64-byte signatures that a stock RFC 8032 Ed25519 verifier accepts. Note: FROST signatures are **not deterministic** even in the Ed25519 ciphersuite — they are verifier-compatible, not algorithm-identical. This verifier compatibility is **critical for eIDAS trust-list tooling interoperability** and is why Marque chose FROST over BLS for notary co-signatures.

### 1.9 Argon2id — RFC 9106

Memory-hard proof-of-work ([RFC 9106](https://datatracker.ietf.org/doc/html/rfc9106)) is the only PoW construction that resists GPU and ASIC acceleration at consumer-memory costs. Marque uses Argon2id at 64 MiB for anti-spam PoW; asymmetric tuning (50 ms for contacts, seconds for strangers) is recipient-controlled.

### 1.10 BLAKE3 — content addressing

[BLAKE3](https://github.com/BLAKE3-team/BLAKE3) provides CIDs with incremental verification via Merkle chunks, enabling delta sync for attachments without retransmitting unchanged bytes.

### 1.11 QUIC — RFC 9000

[QUIC](https://datatracker.ietf.org/doc/html/rfc9000) with TLS 1.3 is the modern async transport for mobile clients — connection migration, 0-RTT resumption, stream multiplexing. Marque uses QUIC for client-provider and provider-provider connections.

### 1.12 CaMeL — dual-LLM prompt injection defense

DeepMind's 2025 [CaMeL dual-LLM approach](https://arxiv.org/abs/2503.18813) informs the data-vs-instruction separation in `ai_instruction` and `public_untrusted` block flags ([`protocol/08 §8.8`](../protocol/08-anti-spam.md#88-ai-assistant-safe-content-parsing)).

## 2. What Marque deliberately refuses

### 2.1 Matrix Synapse — server-side conversation state

Element's own January 2025 blog acknowledged community Synapse *"will fail when used at nation scale"* and paywalled a 500× scalability improvement into Synapse Pro. The structural cause is storing per-user-per-room conversation state on the server — a cartesian product that explodes whenever large federated rooms interact.

**Marque's hard invariant** is that servers store **envelopes, not conversations**. Storage scales as `envelopes × ciphertext_size`, not `users × rooms × membership × history`.

### 2.2 XMPP — spec-optional federation

XEP-by-XEP fragmentation produced incompatible clients; Google Talk's 2013 defection from federation collapsed the mass-market path.

**Marque refuses spec-optional extensions at the interop surface.** Conformance tests are mandatory. DID-portable identity ensures no single provider can trap users even if a future gatekeeper defects.

### 2.3 Nostr — no economic model for persistence

Roughly 95% of Nostr relays cannot cover costs; a recent census found 712 live relays, 625 free.

**Marque's "home provider" pattern** solves this — providers have paid revenue models ([`context/02`](./02-commercial-model.md)), and users always have a minimum-latency guaranteed-persistence inbox.

### 2.4 On-chain messaging (Mailchain, LedgerMail, Dmail, EtherMail)

See [`context/01`](./01-blockchain-scope.md). Cost, GDPR incompatibility, wallet-UX key-loss catastrophe, and sub-1000-TPS throughput make on-chain message content a non-starter for human correspondence.

### 2.5 Handshake / Namecoin — fully on-chain naming

Sub-$4M market cap (Handshake) demonstrates that fully on-chain naming does not attract mass adoption. Marque's portable-tier registry is **federated-neutral, not fully decentralized** — the 95/5 trade-off captures most of the benefit with a fraction of the friction.

### 2.6 AMP for Email — vendor-gated rich content

AMP for Email failed in 2019–2024 because it required Google-gated sender whitelisting, Microsoft never shipped it in Outlook, and developer apathy sealed the coffin. **A rich-content model gated by a single vendor will not survive.**

Marque's MBS and its WASM widget sandbox are **technically gated, not socially gated** — anyone can author a widget; the capability surface is so minimal that malicious widgets cannot do more than display ugly content.

### 2.7 PGP — per Moxie Marlinspike's 2015 verdict

PGP is [*"dead"*](https://moxie.org/2015/02/24/gpg-and-me.html). Key management UX failed at the consumer level; metadata leakage (subject lines unencrypted) is structural.

Marque treats E2EE as default baseline, not an opt-in power-user tier. Subject lines and body are encrypted together; metadata minimization is per-tier.

### 2.8 S/MIME — enterprise-only

S/MIME succeeds in specific enterprise contexts but has never become consumer infrastructure. Certificate issuance is opaque; revocation checking is fragile; client support is uneven.

Marque absorbs S/MIME's legal-tier use case (Qualified tier) while avoiding its certificate-and-revocation operational burden by binding qualified certificates to DIDs via the KEL.

### 2.9 Proof-of-humanity as a mandate

Worldcoin, BrightID, Proof of Humanity — all explicitly **rejected as a mandate** ([`protocol/08 §8.10`](../protocol/08-anti-spam.md#810-proof-of-humanity--explicitly-not-mandated)). Biometric centralization and exclusion of poor users create a two-class internet.

## 3. One-line summary of the synthesis

**Marque is not a research project.** It is a composition of components that work in production today (Bluesky AT, MLS, Sigstore, OpenTimestamps, PEC, Hyperswarm, SimpleX, MIMI, FROST, Argon2id, BLAKE3, QUIC), wrapped in an opinionated architecture with firm red lines against on-chain content, per-message gas, wallet-only identity, single-provider capture, server-owned conversation state, and spec-optional extensions.

---

You have reached the end of the specification. Return to the [**spec index**](../README.md) for the reading paths.
