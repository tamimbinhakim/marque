# What Marque is

> One page. The shape of the system, the commitments that define it, and the non-goals that bound it.

**You will learn**

- Marque's seven architectural commitments.
- The shape of a message and the role of each actor.
- What Marque deliberately is not.

---

## The one-sentence definition

Marque is a federation of dumb encrypted mailboxes with portable cryptographic identities, a typed content layer, and a tiered legal-proof envelope — wrapped in a bidirectional SMTP bridge for the transition.

Each of those seven words is a commitment. The rest of this document unpacks them.

## The shape of a message

```
sender client                                              recipient client
     │                                                            ▲
     │  (1) resolve recipient's did:mail                          │
     │  (2) fetch KeyPackage from their provider                  │
     │  (3) seal MLS envelope; sign with DSK                      │
     │  (4) deposit to ≥2 recipient-listed providers              │
     │                                                            │
     ▼                                                            │
 ┌────────────────┐        ┌────────────────┐        ┌────────────┴───┐
 │ sender's       │        │ recipient's    │        │ recipient's    │
 │ home provider  │        │ provider A     │        │ provider B     │
 └────────────────┘        └────────────────┘        └────────────────┘
                                   │                         │
                                   └──── pull / long-poll ───┘
                                              │
                                              ▼
                                  (ciphertext only;
                                   providers never
                                   read content)
```

Three actors: the **sender's client**, the **recipient's client**, and any number of **providers**. Providers store ciphertext envelopes; clients hold MLS state; nobody in the middle sees message content.

A message carries a `did:mail` identity for the sender, an MLS-encrypted payload for the recipients, and — for Attested or Registered tier — a `ProofEnvelope` with cryptographic non-repudiation. The outer envelope is canonical CBOR; the payload is the Marque Block Spec (MBS), a typed JSON block vocabulary.

## The seven commitments

### 1. Identity is cryptographic and portable

A user's identity is a `did:mail` — a long-lived keypair with a published DID document listing service endpoints. The keys live in a three-tier hierarchy: an **Identity Root Key** (years, cold storage), **Device Signing Keys** (months, hardware-backed per device), and **ephemeral MLS ratchet keys**.

Changing providers is changing an entry in the DID document. Your correspondents update nothing. This is the phone-number-portability analogy, applied correctly.

### 2. Content is end-to-end encrypted by default

Messages are carried inside [MLS (RFC 9420)](https://datatracker.ietf.org/doc/html/rfc9420) groups — one group per thread. MLS provides post-compromise security, forward secrecy, and O(log n) scaling to groups of 50 000. The default cipher suite is post-quantum hybrid: `X25519MLKEM768` for key agreement, `Ed25519 + ML-DSA-65` for signatures.

Each message carries an `auth_mode` choosing the cryptographic authentication style:

- **`CASUAL`** — standard MLS MAC, deniable.
- **`SIGNED`** — adds a sender signature; non-repudiable; rendered with a **Signed** chip.
- **`QUALIFIED`** — adds a qualified certificate and QTSP co-signature; legally binding; rendered with a **Registered** chip.

### 3. Legal proof is native and tiered

The `ProofEnvelope` composes the sender's signature with RFC 3161 qualified timestamps, optional FROST threshold signatures from notary relays, and an OpenTimestamps `.ots` proof anchored in Bitcoin. Qualified tier maps to eIDAS 2.0 QERDS ([Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj), Art. 44). Attested tier maps to ERDS (Art. 43).

The Bitcoin anchor is zero-cost per message (OpenTimestamps aggregates arbitrary numbers of hashes into one on-chain transaction per calendar cycle) and requires no trusted third party to verify.

### 4. Providers are commodity encrypted mailboxes

Providers store envelopes — not conversations. No per-user-per-room state. Storage scales as `envelopes × ciphertext_size`, not as `users × rooms × membership × history`. A provider holding 30 days of mail for a million users is a single well-specced rack.

The hard invariants: providers never read content, never own conversation state, never see the social graph beyond the immediate deposit metadata.

### 5. Content is a typed signed block log

The **Marque Block Spec (MBS)** replaces HTML email's frozen 2005 subset with JSON-based, schema-validated, reverse-DNS-namespaced blocks. Core blocks cover text (CommonMark + GFM), content-addressed media and attachments (BLAKE3 CIDs), polls, schedules, forms, payment requests, reactions, replies, forwards, retractions, and the `core.signed_action` primitive that replaces DocuSign.

Rendering is deterministic, accessibility is mandatory (not optional), and localization is native. Unsupported blocks render their `fallback` field; ultimately the envelope carries `alt_plaintext` so no message is ever unreadable.

### 6. Anti-spam is economic and cryptographic

Content filtering has reached a ceiling against LLM-generated abuse (2026 studies: near-tripled click rate over generic phishing, 96.8% personalization rate vs 56.2%). Marque shifts the battle ground:

- **Verified sender identity** — spoofing is structurally impossible, making SPF / DKIM / DMARC / ARC obsolete.
- **Proof-of-work** via Argon2id, tuned asymmetrically: fast for known contacts, slow for strangers.
- **Refundable bonds** for stranger outreach, burned on spam report (with a 14-day appeal window).
- **Recipient-controlled policy**, surfaced as machine-readable rejection reasons: no more opaque "went to spam."
- **Structural data-vs-instruction separation** at the content layer, defending AI-assistant inboxes against prompt injection.

### 7. SMTP interop is bidirectional and honestly labeled

A Marque provider bridges to SMTP from day one. Inbound SMTP messages with a discoverable sender key are wrapped in MLS and arrive with a `bridged · encrypted` chip; inbound without a discoverable key arrive with a plain `email` chip. Outbound to an SMTP-only recipient downgrades with an Autocrypt header for opportunistic upgrade on the next round-trip. Legal-tier messages never auto-downgrade — they either fail closed or invoke a QERDS-to-legacy gateway.

## Non-goals

- **Not a cryptocurrency.** No protocol token, no per-message gas, no wallet requirement.
- **Not a blockchain.** Bitcoin anchoring is used narrowly for timestamp non-repudiation; content, identity, and routing stay off-chain.
- **Not a client specification.** Marque defines wire format, identity, envelope, and block semantics. Client UX beyond the minimum rendering conformance is out of scope.
- **Not an MTA specification.** Provider behavior beyond the minimum interop surface is specified by companion documents (e.g. *Mailroom*).
- **Not a replacement for instant messaging.** Marque is async-first. Real-time chat remains better served by Matrix, Signal, or SimpleX.

## How the specification is organized

- **[`overview/`](../overview/)** — you are here. The product, the user perspective, adoption timeline.
- **[`protocol/`](../protocol/)** — normative wire formats, identity, crypto, content, legal proof, anti-spam, interop, conformance.
- **[`context/`](../context/)** — supporting material: blockchain scope, commercial model, migration strategy, naming, risks, related work.

Each file opens with *"You will learn"* so you can skip what you don't need.

---

Continue to [**How it feels**](./03-how-it-feels.md).
