# FAQ

> Questions that recur. Each answer is short; deep detail lives in the specification behind the deep-links.

---

## General

### Is Marque an email replacement or a Matrix / Signal replacement?

**Email replacement.** Marque is async-first and optimized for durable, long-lived, cryptographically provable correspondence. Real-time group chat is better served by Matrix, Signal, or SimpleX. The architectural choices (MLS over async transport per MIMI WG, legal-proof tiers, SMTP bridge) assume the email use case. See [`spec/00-overview.md`](../spec/00-overview.md).

### Is this a cryptocurrency project?

**No.** No protocol token, no per-message gas, no mandatory wallet. Bitcoin is used narrowly for timestamp non-repudiation via OpenTimestamps; content, identity, and routing stay off-chain. See [`spec/context/01-blockchain-scope.md`](../spec/context/01-blockchain-scope.md).

### Is this another decentralized-everything project?

**No.** The portable-tier registry is federated-neutral, not fully decentralized. Handshake and Namecoin have failed the adoption test. Marque takes the 95 / 5 trade: 95% of the benefit with 5% of the friction. See [`spec/protocol/02-identity.md`](../spec/protocol/02-identity.md).

### When will Marque ship?

**Spec v1.0 target: 2028.** First commercial deployments 2028–2029. eIDAS QERDS listing 2029–2030. Phase-4 majority-of-new-signups tipping point 2033–2036. Honesty about timelines is baked into the proposal; HTTPS took 10 years with Google's backing, IPv6 is at 28 and still half-done. See [`spec/context/03-migration.md`](../spec/context/03-migration.md).

## Identity

### What happens if I lose my key?

Three-layer recovery per [`spec/protocol/02-identity.md §2.7`](../spec/protocol/02-identity.md#27-recovery):

1. **Primary** — device-resident key (TPM, Secure Enclave, hardware token).
2. **Social** — 3-of-5 trusted guardians with a 72-hour nullification window.
3. **Optional** — 24-word phrase for power users.

Your home provider MAY act as one guardian by default, giving you a familiar "password reset" path without losing portability. A provider cannot recover alone, regardless of its guardian weight.

### What's the difference between a provider-hosted and a portable-tier handle?

A **portable-tier handle** (`did:mail:marque.id:alice`) survives any provider change. Your handle never changes; no correspondent updates anything. This is the default for new consumer accounts.

A **provider-hosted handle** (`did:mail:acme.com:alice`) survives only while `acme.com` cooperates by serving the DID document. If `acme.com` expires or sells, the identity becomes unresolvable. Same risk class as email BYOD.

See [`spec/protocol/02-identity.md §2.10`](../spec/protocol/02-identity.md#210-the-portability-guarantee--and-its-limit).

### Can I have `did:mail:gmail.com:alice`?

Only if Google cooperates by serving `/.well-known/mail-did/alice/did.json`. They will not, at least not in Phase 1–2. The headline portability claim holds between cooperating providers. The mitigation is to make portable-tier the default.

### Can I still have multiple addresses?

Yes. Multiple DIDs per natural person are supported; providers may offer disposable rotating handles for privacy-sensitive use. See [`spec/protocol/02-identity.md §2.11`](../spec/protocol/02-identity.md#211-disposable-and-linked-handles).

### What if my provider shuts down like Skiff?

That's the whole point of portability. Your DID is yours; your handle survives provider shutdown. You point the `MarqueMailbox` service endpoint to a new provider; no contacts update anything. See [`spec/protocol/03-architecture.md §3.11`](../spec/protocol/03-architecture.md#311-provider-migration-shutdown-termination).

A provider shutting down MUST announce 90 days in advance and maintain export capacity throughout.

### Do I need a wallet?

**No.** Default resolution is DNS-rooted for provider-hosted handles, and the portable-tier registry for `marque.id` handles. ENS is an optional disaster-recovery fallback. See [`spec/context/01-blockchain-scope.md`](../spec/context/01-blockchain-scope.md).

## Cryptography

### Why MLS instead of Signal's Double Ratchet?

Three reasons per [`spec/protocol/05-cryptography.md §5.2`](../spec/protocol/05-cryptography.md#52-end-to-end-encryption-via-mls):

1. TreeKEM is O(log n) vs. Sender Keys' O(n) — 50K-member groups practical.
2. Epochs map naturally to "threads."
3. Multiple audited independent implementations in production (Cisco, Wire, Meta, OpenMLS).

### Is Marque post-quantum?

**Yes, by default.** `X25519MLKEM768` for key agreement, `Ed25519 + ML-DSA-65` for signatures. Classical-only is available as a FIPS dual-track for regulated deployments. See [`spec/protocol/05-cryptography.md §5.1.2`](../spec/protocol/05-cryptography.md#512-post-quantum-hybrid-default).

### Can I still have deniable messages?

Yes, per-message. The `auth_mode` flag chooses `CASUAL` (deniable MAC), `SIGNED` (non-repudiable, **Signed** chip), or `QUALIFIED` (eIDAS qualified signature, **Registered** chip). The same thread MAY mix modes. See [`spec/protocol/05-cryptography.md §5.3`](../spec/protocol/05-cryptography.md#53-deniability-versus-non-repudiation).

## Legal proof

### How does this compare to DocuSign?

For non-repudiable receipt of a specific document, the Certified tier is cheaper, faster, cryptographically stronger, and preserves the thread context DocuSign discards. Replaces the "email-plus-DocuSign" compound stack with a native `core.signed_action` primitive. See [`spec/protocol/07-legal-proof.md §7.3`](../spec/protocol/07-legal-proof.md#73-the-three-tiers).

### Is Certified-tier delivery admissible in court?

**In eIDAS jurisdictions, yes** — the design targets QERDS (Art. 44) conformance under [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj). Outside the EU, admissibility depends on local evidence rules; the `ProofEnvelope`'s cryptographic structure is jurisdiction-neutral and designed for expert-witness presentation. See [`spec/protocol/07-legal-proof.md §7.11`](../spec/protocol/07-legal-proof.md#711-multi-jurisdiction-recognition).

### Are read receipts required?

Tier-dependent. Off for Casual, opt-in for Signed, REQUIRED for Registered per QERDS. A Registered recipient either acknowledges or explicitly declines (`REFUSED_TO_ACKNOWLEDGE`). See [`spec/protocol/07-legal-proof.md §7.5`](../spec/protocol/07-legal-proof.md#75-proof-objects).

### Why Italian PEC as a reference?

PEC is the existence proof that mass-scale legal-tier mail works: 16M accounts, mandatory for Italian businesses, 2.5B messages/year, consistent Court of Cassation recognition. **It works because regulation mandated it.** See [`spec/protocol/07-legal-proof.md §7.1`](../spec/protocol/07-legal-proof.md#71-the-empirical-record).

## Content

### Can Marque render my HTML newsletter?

**Via the SMTP bridge, yes** — bridged-inbound messages render legacy HTML with an `email` chip. **Natively, no** — the Marque Block Spec replaces HTML with typed JSON blocks. Newsletter vendors migrate by targeting MBS via the transactional API. See [`spec/protocol/06-content.md`](../spec/protocol/06-content.md).

### Does Marque support attachments?

Yes. First-class, content-addressed via BLAKE3 CIDs, encrypted to the MLS group key, delta-sync via the BLAKE3 Merkle tree. No protocol-level size limit. See [`spec/protocol/06-content.md §6.4`](../spec/protocol/06-content.md#64-attachments-and-content-addressing).

### What happened to AMP for Email?

It failed because it was single-vendor-gated. Marque's WASM widgets learn the lesson — the sandbox is **technical, not social**; anyone can author a widget; capabilities are default-deny. See [`spec/protocol/06-content.md §6.3`](../spec/protocol/06-content.md#63-wasm-widgets).

### How does Forward work?

Three flavors:

- **Quoted** — always permitted. Forwarder signs; original sender's signature is NOT carried.
- **Attested** — preserves the original sender's signature and ProofEnvelope. Requires the original had `forwardable: attested`.
- **Certified** — preserves the Qualified chain-of-custody. Requires `forwardable: qualified_only` and both ends Qualified-capable.

See [`spec/protocol/06-content.md §6.6`](../spec/protocol/06-content.md#66-forward--three-flavors).

## Anti-spam

### Do I have to pay to email people?

**Only strangers whose policy requires a bond**, and bonds are refundable on engagement or timeout. Contacts you've corresponded with are free. New accounts get a bootstrap-token quota for their first 100 recipients. See [`spec/protocol/08-anti-spam.md §8.5`](../spec/protocol/08-anti-spam.md#85-refundable-bonds--the-strongest-single-defense).

### Why not just use better AI filters?

Because content filtering has reached a ceiling. Every message is unique, grammatically perfect, contextually plausible, and interleaved into hijacked legitimate threads. The NDSS 2025 and HAL 2026 studies are clear on this. See [`spec/protocol/08-anti-spam.md §8.1`](../spec/protocol/08-anti-spam.md#81-the-content-filtering-ceiling).

### Does Marque defend against EchoLeak-class prompt injection?

**Yes, protocol-level.** MBS blocks carry `ai_instruction: false` by default; `public_untrusted` defaults true for anything not directly attributable to the sender; signed instruction blocks require sender allowlisting. **It is impossible to retrofit into SMTP / MIME** because RFC 5322 has no data / instruction separation. See [`spec/protocol/08-anti-spam.md §8.8`](../spec/protocol/08-anti-spam.md#88-ai-assistant-safe-content-parsing).

### What if a recipient burns my bond unfairly?

You can contest via a 14-day appeal window. The recipient's provider issues a counter-challenge; if the recipient doesn't re-confirm the spam designation within 7 days, the bond is refunded. Serial false-positive reporters accrue negative reputation on the gossip channel. See [`spec/protocol/08-anti-spam.md §8.5`](../spec/protocol/08-anti-spam.md#85-refundable-bonds--the-strongest-single-defense).

## Migration

### How does this work with my Gmail address?

Create a `did:mail`, prove control of your Gmail via DKIM-signed challenge, publish `legacyEmail` as a DID service endpoint, configure Gmail forwarding. Inbound from old contacts continues to work; outbound to new contacts uses the DID. See [`spec/protocol/02-identity.md`](../spec/protocol/02-identity.md).

### Will @gmail.com people see my encrypted messages?

**No, because SMTP doesn't carry MLS envelopes.** Outbound Marque → SMTP downgrades to plain email with an amber pre-send modal warning the first time. An Autocrypt header advertises your key; the next round-trip may upgrade to encrypted SMTP. See [`spec/protocol/09-interop.md §9.3`](../spec/protocol/09-interop.md#93-outbound-marque--smtp).

### When should I adopt Marque?

- **Now** if you are a self-hoster, privacy researcher, or protocol enthusiast. Expect rough edges.
- **Phase 2 (2028–2029)** if you are a privacy-conscious professional.
- **Phase 3 (2029–2030)** if you are an enterprise with compliance drivers.
- **Phase 4+ (2031+)** if you wait for iOS Mail and Gmail to ship native clients.

See [`spec/overview/04-when-to-adopt.md`](../spec/overview/04-when-to-adopt.md).

## Storage and archive

### Where do my messages live?

A hybrid: the **provider stores ciphertext** (durable), and the **device stores cleartext** (necessary because MLS epoch forward-secrecy means the provider's ciphertext becomes unreadable after sending keys are forgotten). Both are encrypted-at-rest. Search is client-side. See [`spec/protocol/03-architecture.md §3.9`](../spec/protocol/03-architecture.md#39-archive-of-record-and-cross-device-sync).

### Can I export my archive?

Yes. The `.marquebox` format is a ZIP container with canonical envelopes, passphrase-wrapped cleartext, thread metadata, and ProofEnvelopes. Any conformant provider MUST accept `.marquebox` imports. This satisfies GDPR data-portability obligations. See [`spec/protocol/03-architecture.md §3.10`](../spec/protocol/03-architecture.md#310-marquebox--portable-archive-export).

### How does multi-device sync work?

Devices exchange HPKE-encrypted cache deltas via the home provider's `device-sync` endpoint. Encryption uses a per-DID archive key distinct from MLS group keys, held in hardware-backed storage on every authorized device. See [`spec/protocol/03-architecture.md §3.9`](../spec/protocol/03-architecture.md#39-archive-of-record-and-cross-device-sync).

## Rooms — shared mailboxes, teams, lists

### How does `support@company.com` work?

It's a **shared Room** — a first-class identity kind. Any roster member can send-as; inbound fans out to all roster members. Threads preserve Room identity across handoffs. See [`spec/protocol/04-rooms.md`](../spec/protocol/04-rooms.md).

### Can my assistant read my email?

Yes, via **delegation**. The delegation has a bounded scope (`read_only`, `triage`, or `full`), and every outbound message surfaces the acting assistant's identity ("Alice via assistant Bob"). Delegation cannot send at Qualified tier; legal-identity operations are not delegatable. See [`spec/protocol/04-rooms.md §4.9`](../spec/protocol/04-rooms.md#49-delegation-delegation-kind).

### What about mailing lists with 10 000 subscribers?

A `list`-kind Room. Owner (or moderator threshold) sends broadcast; subscribers receive. Rolling-epoch relaxation keeps MLS operational cost bounded at scale. One-click unsubscribe per RFC 8058. See [`spec/protocol/04-rooms.md §4.7`](../spec/protocol/04-rooms.md#47-mailing-lists-list-kind).

## Process and governance

### Who controls Marque?

Pre-charter: the editor team per [GOVERNANCE.md](../GOVERNANCE.md), with hard invariants against single-vendor capture. Post-charter: an IETF WG (proposed: *mpc*, Marque Protocol Core). The trademark will transfer to the Internet Mail Identity Foundation. See [GOVERNANCE.md](../GOVERNANCE.md).

### How do I propose a change?

Write a MIP. See [MIP-0001](../mips/mip-0001-process.md) for the process and [`mip-template.md`](../mips/mip-template.md) for the template.

### How do I get involved without writing a MIP?

- Open an issue using one of the [templates](../.github/ISSUE_TEMPLATE).
- Review open MIPs and leave substantive feedback.
- Write an independent interoperable implementation — Core MIPs require at least two independent implementations from unaffiliated groups to reach Final. Example implementations exist and more are welcome; none is privileged. Every implementation that passes the conformance suite is a peer, and implementers are unrestrained in how they differentiate above the wire (AI, UX, search, discovery).
- Run the conformance self-test matrix against any deployed Marque stack.

## Relationship to other systems

### How is this different from Nostr?

Nostr has no economic model for persistence — 95% of relays run at a loss. Marque's providers have paid revenue models ([`spec/context/02-commercial-model.md`](../spec/context/02-commercial-model.md)); users have a guaranteed-persistence home inbox. Identity is `did:mail` (portable, DID-based) vs. Nostr's `npub`/`nsec` (naked keypair; lose it, lose everything).

### How is this different from Bluesky AT?

Marque borrows the PDS pattern for mailbox providers and the `did:plc` nullification-window pattern for DID rotation, but rejects the firehose relay model — messaging does not need global public ordering. See [`spec/context/06-related-work.md §1.1`](../spec/context/06-related-work.md#11-bluesky-at-protocol--data-provider-separation-account-migration-at-scale).

### How is this different from Matrix?

Matrix stores conversation state on the server (`users × rooms × membership × history`). Marque stores envelopes only (`envelopes × ciphertext_size`). The scaling math is fundamentally different — Marque is 30 TB/rack for 1M users at email scale; Matrix Synapse *"will fail when used at nation scale"* per Element's own January 2025 statement.

### What about Proton / Tutanota / Fastmail?

They are natural first commercial adopters (Phase 2). They already occupy the privacy-premium shelf; Marque gives them a protocol that makes their privacy story cryptographic and portable rather than brand-dependent.
