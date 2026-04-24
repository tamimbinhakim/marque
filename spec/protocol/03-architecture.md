# Architecture

> The shape of the system. Federation of dumb encrypted mailboxes with an optional direct peer-to-peer fast path, a hybrid archive-of-record, and device-synchronized cleartext caches.

**You will learn**

- The three roles (client, provider, notary) and the hard invariants bounding each.
- The wire envelope in canonical CBOR.
- Delivery paths, metadata-privacy tiers, and the three-hop P2P fallback.
- Where messages live at each stage of their life: sender's Sent copy, recipient's mailbox, device archive.
- Cross-device sync, provider migration, and the `.marquebox` export.
- Provider obligations, shutdown, and termination procedures.

---

## 3.1 Three roles

**Clients** act on behalf of one or more DID-identified users. They construct envelopes, deposit them at provider endpoints, authenticate to providers for fetch, and maintain local MLS group state.

**Providers** accept envelopes addressed to pseudonyms that resolve to DIDs for which they are a listed service endpoint. They persist envelopes for at least their advertised retention, serve them to authenticated clients, and participate in federation gossip. A user's **home provider** is the lowest-priority `MarqueMailbox` entry in their DID document — the default deposit and retention target.

**Notaries** co-sign delivery receipts for the Attested and Qualified tiers. A notary MAY also act as a provider; the roles are distinct but compatible.

The **hard invariants**:

1. **Providers MUST NOT read envelope content.** They MAY read addressing and policy metadata as required for routing and anti-spam enforcement. They MUST NOT retain metadata beyond the retention policy advertised in their capability record.
2. **Providers MUST NOT own conversation state.** Group membership, thread continuity, and cryptographic epoch progression are client state. Persisting such state beyond the envelopes the provider is asked to retain for delivery is non-conformance.

These invariants bound provider storage to `Σ(envelopes × ciphertext_size)` — not the `Σ(users × rooms × membership × history)` cartesian product that makes server-stateful messaging systems fail at scale.

## 3.2 The envelope

The on-wire unit is a canonical CBOR map per [`schemas/cddl/envelope.cddl`](../../schemas/cddl/envelope.cddl) (normative). The JSON Schemas in `schemas/` are debugging projections.

Every envelope carries:

- **Version marker** — `"marque/1"`.
- **Recipient pseudonym** — a 32-byte HMAC of the recipient's MLS epoch secret and content hash (see §3.3).
- **Sender DID** — the originating `did:mail`.
- **Sender DSK reference** — the DID URL fragment of the signing DSK.
- **Auth mode** — `CASUAL`, `SIGNED`, or `QUALIFIED`.
- **Privacy tier** — `CASUAL`, `SEALED`, or `MIXED`.
- **Cipher suite identifier**.
- **Content ID** — BLAKE3-256 hash of the ciphertext.
- **Timestamp** — UTC Unix seconds (with optional millisecond extension).
- **Pad bucket** — one of six fixed sizes.
- **Optional legacy headers** (bridged messages only).
- **Ciphertext** — MLS `PublicMessage` or `PrivateMessage`.
- **Signature** — classical and optionally post-quantum.
- **Optional `proof_envelope_ref`** — REQUIRED when auth_mode is `QUALIFIED`.
- **Optional permission flags** — `forwardable`, `quotable`, `thread_id`, `sender_acting` (for Room sends).

Encoded envelopes are padded to the nominal bucket size in a trailing byte string **outside the signed map**, so the signature is independent of pad content and no padding oracle is exposed.

## 3.3 Recipient pseudonym

A sender computes:

```
recipient_pseudonym = HKDF-Expand(
  HKDF-Extract(
    salt = "marque:pseudonym/v1",
    ikm  = recipient_epoch_secret),
  info = sender_cert_ref || content_id,
  len  = 32)
```

where `recipient_epoch_secret` is the MLS `epoch_authenticator` of the current epoch for the group addressed by the envelope. The pseudonym binds the envelope to a specific recipient, sender, and content, preventing cross-recipient correlators from identifying identical-content envelopes.

At `CASUAL` privacy tier the sender MAY additionally transmit the recipient DID in clear; the pseudonym is still computed and included for integrity.

## 3.4 Delivery paths

A client attempts delivery paths in order; the first succeeds.

```
┌────────┐                                                  ┌────────┐
│ Sender │                                                  │ Recip. │
│ client │                                                  │ client │
└───┬────┘                                                  └───▲────┘
    │                                                           │
    │  (1)  Direct P2P — Hyperswarm-style UDP holepunching      │
    │       500 ms budget; both peers online, policy permits    │
    │ ────────────────────────────────────────────────────────▶ │
    │                                                           │
    │  (2)  Store-and-forward: deposit to 2-of-N providers      │
    │       from recipient's DID document                       │
    │  ┌──────────────┐   ┌──────────────┐                      │
    ├─▶│  Provider A  │   │  Provider B  │ ────────────────────▶│
    │  └──────────────┘   └──────────────┘                      │
    │                                                           │
    │  (3)  Metadata-sensitive: Sphinx-wrap, Nym mixnet or      │
    │       Tor onion; 10–30 s per hop latency                  │
    └───────────────────────────────────────────────────────────┘
```

**Content-addressed attachments** (BLAKE3 CIDs) deduplicate across recipients and support resumable partial fetches via the BLAKE3 Merkle tree.

## 3.5 Transport

Transport is **QUIC** ([RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000)) over **TLS 1.3** ([RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446)) — 0-RTT resumption, connection migration for mobile, stream multiplexing for both P2P and long-polled client-mailbox connections.

The TLS handshake uses **DID-derived self-signed certificates**: `SubjectPublicKeyInfo` is the DID key; the verifier resolves the DID, confirms the cert's public key appears in `verificationMethod`, and confirms the DSK's certificate chains to a KEL event at least one hour old (to bound rotation-race conditions).

Clients and servers MUST support the `X25519MLKEM768` key-share group. Classical-only `X25519` remains available for deployments that cannot yet ship the hybrid KEM.

Each QUIC connection carries:

- **Control stream** (QUIC stream 0): bidirectional length-prefixed CBOR control frames for capability discovery, KeyPackage publication, presence.
- **Deposit streams**: client-initiated unidirectional, one per envelope, closed after ack.
- **Fetch streams**: client-initiated bidirectional long-polls, closed by client.

Providers MUST support QUIC connection migration so mobile clients preserve long-poll continuity across network changes.

## 3.6 Provider obligations

A conformant Marque provider MUST:

1. Accept envelopes for any recipient whose DID document lists it as a service endpoint.
2. Store envelopes for at least the retention period advertised in its capability record (MINIMUM 30 days).
3. Serve envelopes to authenticated clients via the pull API.
4. Not read, log, or transmit envelope content or recipient pseudonyms beyond what the protocol requires.
5. Participate in the transparency log for any delivery receipts it issues.
6. Publish a capability record at `/.well-known/marque-provider.json` describing retention, size limits, bridge policy, and supported cipher suites.
7. Issue bootstrap VOPRF tokens to newly-hosted DIDs per [protocol/02 §2.8](./02-identity.md#28-new-user-cold-start).
8. Support cross-device archive sync at the `device-sync` endpoint (§3.9).
9. Accept `.marquebox` imports at the `archive-import` endpoint (§3.10).
10. Retain a departing user's ciphertext for 30 days past DID de-listing (§3.11).
11. Issue a `retention-warning` event at least 30 days before any envelope's scheduled expiry.

A provider MAY additionally:

- Act as a FROST threshold signer for Attested or Qualified tier messages.
- Run a bridge to SMTP (see [protocol/09](./09-interop.md)).
- Act as one social-recovery guardian. A provider acting as a guardian MUST NOT be the sole authorizer of any recovery operation.
- Offer AI-assisted features subject to [protocol/08 §8.5](./08-anti-spam.md) data-separation requirements.
- Offer provider-side encrypted search per the companion `draft-tamim-marque-search` specification.

Full capability record schema: [`capability-record.schema.json`](../../schemas/capability-record.schema.json).

## 3.7 Multi-endpoint requirement

A DID document SHOULD list at least two `MarqueMailbox` service endpoints from distinct operators. Single-endpoint identities are permitted but a single point of failure: a provider outage silently drops inbound mail for the outage window.

Clients MUST warn a user creating a single-endpoint DID and SHOULD offer to add a secondary endpoint. For DIDs at the Attested or Qualified tier, two endpoints are **REQUIRED**; senders at these tiers MUST refuse to send to DIDs with fewer.

Multi-provider deposit means the recipient's multiple providers hold identical ciphertext. The recipient's client MUST deduplicate on `content_id` when fetching, so the user sees each message once.

## 3.8 Sender's Sent copy

A sender MUST persist a copy of each outbound envelope so the sender can later read, search, and present the message.

**Mechanism:** immediately prior to deposit, the client constructs a **self-addressed envelope copy** — the same content, re-encrypted under the sender's own **self-group**, a permanent single-member MLS group rooted at the sender's IRK for archival. The self-addressed copy is deposited at the sender's home provider with the `x-marque.sent-copy: true` flag and the original `content_id` unchanged (for cross-match with the recipient-addressed copy).

The self-group is established at DID creation and has no KeyPackage published for it — correspondents cannot discover or join it. A compromised correspondent cannot access the sender's Sent folder.

## 3.9 Archive-of-record and cross-device sync

Marque's archive is **hybrid**:

- The **provider stores ciphertext** (durable; survives device loss).
- The **device stores cleartext** (necessary because MLS epoch forward-secrecy means the provider's ciphertext becomes unreadable after the sending key is forgotten).
- The **cleartext is encrypted-at-rest** via OS key-store primitives.

Each authorized device maintains a local cleartext cache keyed by `content_id`: a per-thread ring buffer of the most recent N messages (default 1000, configurable), full-text indexed for search (§3.12), wiped on user sign-out or remote-wipe.

### Cross-device sync

Two or more authorized devices under the same DID MUST synchronize their cleartext caches. Each device publishes **cache deltas** — HPKE-encrypted bundles of new cleartext entries — to the home provider's `device-sync` endpoint. Peers fetch and decrypt on a configurable schedule (default 5 minutes while foregrounded).

The bundle is encrypted under a per-DID **archive key**, distinct from MLS group keys and DSK keys, derived from the IRK at identity creation, stored in hardware-backed storage on every authorized device. The archive key is the **single most sensitive secret** a user holds short of the IRK itself. Clients MUST wipe it on explicit sign-out.

Conflicts resolve last-write-wins per-field with a server-timestamp tiebreaker; clients SHOULD surface conflicts to the user on high-stakes fields (e.g., a Qualified-tier message marked as spam on one device but not another).

## 3.10 `.marquebox` — portable archive export

The normative export format is **`.marquebox`** — a ZIP container:

```
archive.marquebox/
├── manifest.json         archive metadata; DID; export time; envelope count; IRK signature
├── keys/
│   └── archive.jwk       archive key, wrapped under user passphrase (PBKDF2)
├── envelopes/
│   └── <content_id>.cbor canonical envelope per message
├── cleartext/
│   └── <content_id>.mbs.json   MBS cleartext, passphrase-wrapped (optional)
├── threads/
│   └── <group_id>.json   per-thread metadata, membership history, epoch log
└── proofs/
    └── <content_id>.proof.cbor  ProofEnvelope for Attested+ messages
```

Self-contained, deterministic (same input → byte-identical output), signed by the current IRK, encrypted under the user passphrase. Importable by any conformant provider at the `archive-import` endpoint. Satisfies GDPR data-portability obligations on user request within 30 days.

## 3.11 Provider migration, shutdown, termination

### User-initiated migration

1. **Announce** the new provider — add a second `MarqueMailbox` endpoint to the DID document with higher `priority`.
2. **Dual delivery window** — both providers accept inbound for at least 30 days.
3. **Export** from the old provider via `.marquebox` — MUST complete within 14 days of request.
4. **Import** to the new provider via `archive-import`.
5. **De-list** — publish an updated DID document removing the old endpoint.
6. **Old provider retention** — MUST retain ciphertext for 30 days past de-listing, then delete. User MAY elect immediate deletion.

### Provider shutdown

A provider ceasing operation MUST:

1. Announce shutdown at least **90 days** in advance via a `provider-shutdown` federation bulletin.
2. Make `archive-export` available at equal-or-greater capacity during the 90-day window, at no charge regardless of subscription status.
3. Notify every hosted DID's listed alternate endpoints so other providers can prompt re-delivery.
4. **Not publish `deactivate` KEL events on users' behalf** — a shutting-down provider MUST NOT deactivate users' DIDs.

### Account termination

A provider terminating a specific user's account (ToS, non-payment, compliance) MUST:

1. Provide at least **30 days' notice** (except where prohibited by legal process).
2. Allow export throughout the notice window.
3. Continue inbound delivery throughout the notice window.
4. Not deactivate the user's DID — the user retains the identity and MAY migrate.

## 3.12 Search

Search is **client-side by default**:

- Every device maintains a full-text index of its cleartext cache using a widely-implementable tokenizer (Unicode-segmenter + Porter stemmer for alphabetic languages; CJK ICU tokenization elsewhere).
- The index is encrypted-at-rest under the archive key.
- Queries are O(local cache). For archives larger than the cached subset, clients SHOULD surface "search may be incomplete."
- Queries MUST NOT leak to the provider.

**Provider-side encrypted search** (Searchable Symmetric Encryption under the user's archive-key-derived search key) is specified in a companion document `draft-tamim-marque-search`. It is **optional**; no conformant implementation requires it.

## 3.13 GDPR and append-only log semantics

Messages are append-only signed logs ([protocol/07 §7.8](./07-legal-proof.md)). This creates an apparent conflict with GDPR Article 17 (right to erasure).

**Resolution:**

- A recipient MAY delete their local cache of any message; this removes access for that recipient.
- A recipient MAY request their provider delete server-side ciphertext; the provider MUST comply within 30 days.
- A **sender cannot force deletion of a recipient's copy** — cryptographically impossible, and analogous to the legal principle that you cannot un-send a paper letter. The spec states this honestly.
- A `core.retract` amendment SHOULD be offered as the graceful-deletion UX; it does not remove the message but signals disavowal at a specific time, which is what most users actually want.

Erasure of personal data held by a provider (retention logs, reputation data, abuse fingerprints) is subject to the provider's data-retention policy, independent of message content.

## 3.14 Federation

Providers federate via a **pull-gossip protocol** — periodic polling of peer sets for:

- Transparency-log tree heads and consistency proofs.
- Anti-spam fingerprint bulletins (provider-agnostic hashes of high-confidence abuse envelopes).
- Reputation deltas.
- Provider-shutdown announcements.

The federation substrate does not require global consensus; eventual consistency is sufficient. Frame definitions are deferred to [`draft-tamim-marque-core`](../../drafts/draft-tamim-marque-core-00.md).

## 3.15 Scaling

| Dimension | Analysis |
|---|---|
| Provider storage | 30 days of mail for 1 M users at 100 msgs/user/day × 10 KB avg = **30 TB** — one well-specced rack. |
| Discovery | Inherits DNS's proven global performance envelope. |
| Group scale | MLS groups support up to **50 000 members** per [RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420). |
| Consensus | **None** — all sharding is embarrassingly parallel. |

---

Continue to [**Rooms**](./04-rooms.md).
