# Cryptography

> Primitives, MLS profile, auth-mode tiers, privacy tiers, padding, and the crypto building blocks consumed by the legal-proof and anti-spam layers.

**You will learn**

- The baseline classical suite, the post-quantum hybrid default, and the FIPS-agility story.
- The MLS profile for asynchronous mail — KeyPackage pools, epoch checkpoints, thread semantics.
- The three auth modes (`CASUAL` / `SIGNED` / `QUALIFIED`) and their chip vocabulary.
- The three privacy tiers (`CASUAL` / `SEALED` / `MIXED`) and their mutual-exclusion rule with Qualified.
- Padding, cover traffic, and honest statement of what is not achievable.
- Cryptographic primitives provided to the anti-spam policy.

---

## 5.1 Identity primitives

### 5.1.1 Baseline classical suite

**Ed25519 for signatures** ([RFC 8032](https://datatracker.ietf.org/doc/html/rfc8032)) and **X25519 for key agreement** ([RFC 7748](https://datatracker.ietf.org/doc/html/rfc7748)).

Ed25519's determinism eliminates the nonce-reuse catastrophes that have repeatedly broken ECDSA (Sony PS3, Java `SecureRandom`, multiple Bitcoin thefts). Public keys are 32 bytes, signatures 64 bytes, no ASN.1/DER ambiguity. Verification is roughly 70 kilocycles on modern x86.

**P-256 + ECDSA** is available as a FIPS dual-track for regulated deployments requiring FIPS 140-3 certification. The wire format carries an `AlgorithmID` octet so dual-track is a runtime negotiation, not a fork.

### 5.1.2 Post-quantum hybrid (default)

| Role | Algorithm | Reference |
|---|---|---|
| Key agreement | `X25519MLKEM768` | [`draft-ietf-tls-ecdhe-mlkem`](https://datatracker.ietf.org/doc/draft-ietf-tls-ecdhe-mlkem/) |
| Signatures | `Ed25519 + ML-DSA-65` | [FIPS 204 — Module-Lattice-Based Digital Signature Standard](https://csrc.nist.gov/pubs/fips/204/final), published 14 August 2024 |
| Archival signatures | `SLH-DSA-SHA2-128f` | [FIPS 205 — Stateless Hash-Based Digital Signature Standard](https://csrc.nist.gov/pubs/fips/205/final), published 14 August 2024 |

Verification succeeds only when both component signatures verify — security degrades to `max(classical, post-quantum)`. Cloudflare's ~35% of edge TLS traffic using X25519MLKEM768 as of early 2025 is the deployment proof.

The **IANA Marque Algorithms registry** assigns opaque byte tags so new primitives enter via codepoint allocation without re-specification. Crypto agility without algorithm sprawl.

### 5.1.3 Three-tier key hierarchy

See [`protocol/02 §2.3`](./02-identity.md#23-the-three-tier-key-hierarchy). Summary:

| Tier | Lifetime | Storage | Purpose |
|---|---|---|---|
| **IRK** | Years | Hardware token or cold storage | Signs KEL events and DSK certificates. |
| **DSK** | Months | Per-device hardware-backed storage | Signs messages; authenticates TLS. |
| **MLS ratchet** | Ephemeral | Volatile | Forward-secret message encryption. |

## 5.2 End-to-end encryption via MLS

Marque uses **MLS ([RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420))** as its E2EE foundation. MLS was chosen over Signal's Double Ratchet for three reasons:

1. **TreeKEM** provides O(log n) group operations versus Sender Keys' O(n), making 50 000-member groups practical.
2. **Epochs** are first-class objects that map naturally to "thread" semantics users already understand.
3. MLS has **multiple audited independent implementations** (OpenMLS, AWS `mls-rs`, Cisco `mlspp`, Wire) and first-class production deployment.

### 5.2.1 Asynchronous extensions

Email's defining property is that recipients may be offline for weeks. Two extensions address this:

**KeyPackage pools.** Each identity maintains a pool of at least **32 fresh KeyPackages** at each listed provider, with at least one flagged `last_resort`. Clients MUST upload replacements whenever the pool drops below 16. Providers MUST NOT distribute `last_resort` KeyPackages unless the pool is exhausted.

**Epoch checkpoints.** A sending client MAY request its provider to persist an HPKE ([RFC 9180](https://datatracker.ietf.org/doc/html/rfc9180))-encrypted `GroupContext` snapshot against an offline member's latest KeyPackage. On return, the offline member fetches the snapshot and leaps to the current epoch without processing intervening commits.

The [MIMI working group's `draft-ietf-mimi-protocol`](https://datatracker.ietf.org/doc/draft-ietf-mimi-protocol/) provides the concrete HTTPS-over-MLS transport Marque adopts.

### 5.2.2 CC, BCC, threads

- **CC** maps to visible MLS members.
- **BCC** uses the `external_senders` extension with parallel two-member groups per BCC recipient.
- **Threads** are persistent MLS groups; replies are application messages in the current epoch.
- The MLS `group_id` for a new thread is the SHA-256 hash of the concatenation of all founding members' DIDs in canonical byte order.

Operational semantics — Reply vs Reply-All rules, Forward flavors, BCC reply-exposure prevention, thread membership authorization — live in [`protocol/06`](./06-content.md).

## 5.3 Deniability versus non-repudiation

Signal's deniable authentication and legal registered-mail non-repudiation are in cryptographic tension. Marque resolves this with a **per-message user-mode switch** carried as `auth_mode`.

| Mode | Authentication | Chip | Legal weight |
|---|---|---|---|
| `CASUAL` | Standard MLS MAC. Any group member could have forged any transcript. | *(no chip)* | Deniable. |
| `SIGNED` | Adds Ed25519 (+ optional ML-DSA) signature over canonical message hash. Sender's DSK is non-repudiable. | **Signed** | Non-repudiable under sender's DID. |
| `QUALIFIED` | Adds a qualified certificate from a QTSP bound to the IRK under CAdES-B-B or CAdES-B-T. | **Registered** | eIDAS qualified signature. |

The same thread MAY mix modes. Clients render the mode prominently per the [terminology chip table](./01-terminology.md#4-tier-chips).

This is the resolution the OTR / Signal / S-MIME community has debated for twenty years: **give users the choice; don't bake one property into the protocol**.

## 5.4 Non-repudiation envelope

A `ProofEnvelope` wraps `SIGNED` or `QUALIFIED` messages. Full specification in [`protocol/07`](./07-legal-proof.md). Summary of composition:

- Canonical SHA-256 hash of a TLS-presentation-syntax serialization of the message.
- Sender's signature under the chosen algorithm.
- DSK certificate chain to the IRK.
- One or more [RFC 3161](https://datatracker.ietf.org/doc/html/rfc3161) qualified timestamps.
- Optional FROST ([RFC 9591](https://datatracker.ietf.org/doc/html/rfc9591)) threshold signatures from notary relays.
- OpenTimestamps `.ots` proof for Bitcoin-anchored long-term verification.
- Periodic CAdES-B-LTA archive timestamps re-anchoring the envelope as primitives deprecate.

**FROST was chosen over BLS** because it produces stock 64-byte Ed25519-compatible signatures verifiable by any existing implementation — critical for eIDAS trust-list tooling interoperability.

## 5.5 Metadata privacy tiers

Three tiers balance privacy against UX cost.

| Tier | What providers see | Transport |
|---|---|---|
| **`CASUAL`** | Sender and recipient DIDs plaintext behind TLS 1.3. | QUIC. |
| **`SEALED`** (recommended default) | Recipient pseudonym only (Signal Sealed Sender applied to MLS envelopes). Sender's provider sees sender DID; recipient's provider does not. | QUIC. |
| **`MIXED`** | Sphinx-wrapped, Nym mixnet or Tor onion. 10–30 second per-hop latency. | Nym / Tor. |

**Nym was chosen over Tor onions** for mail specifically because async latency tolerance (seconds to minutes) makes Loopix mixing's per-hop delay invisible, while Tor's low-latency design surrenders to global passive adversaries of exactly the type threatening legal-sensitive long-lifetime mail.

### Mutual exclusion with Qualified tier

**`MIXED` is mutually exclusive with `QUALIFIED` auth-mode.** The Qualified tier requires identifiable provider and QTSP co-signatures, which Sealed/Mixed pseudonymization blocks.

Clients MUST refuse to compose a message with both `privacy_tier: MIXED` and `auth_mode: QUALIFIED`. Providers MUST reject deposits with both fields set, returning error `tier-incompatible`.

Implementations MUST NOT silently downgrade the tier selected by the sender. If a provider cannot honor the requested tier (e.g. no mixnet peering), it MUST reject the deposit with a structured error.

## 5.6 Padding and cover traffic

### Pad buckets

Every envelope is padded to one of six fixed sizes before transmission. An attacker sees bucket, not exact size.

| Bucket | On-wire size |
|---|---|
| `b4k` | 4 KiB |
| `b16k` | 16 KiB |
| `b64k` | 64 KiB |
| `b256k` | 256 KiB |
| `b1m` | 1 MiB |
| `b4m` | 4 MiB |

A sender MUST select the smallest bucket that fits the fully encoded envelope. A receiver MUST reject envelopes whose on-wire size does not exactly equal the nominal bucket size.

Padding bytes MUST be cryptographically random and MUST be placed in a trailing byte-string field **outside the signed envelope map**, so padding does not interact with the envelope signature and no padding oracle is exposed.

Envelopes exceeding 4 MiB MUST split attachments into content-addressed blobs referenced by the MBS layer and carried out-of-band.

### Cover traffic

**Providers MUST emit server-to-server cover traffic** at a Poisson-distributed rate calibrated to make egress rates to every peer uniform regardless of real user activity. Without this, traffic analysis against provider pairs reveals busy-hour patterns.

**Client-side cover traffic is OPTIONAL.** Signal's and Threema's deployments demonstrate mobile battery abandonment when mandatory; we refuse to mandate it.

## 5.7 What is not achievable

Marque does not conceal:

- **Endpoint compromise.** A compromised endpoint learns everything. Inherent to any E2EE system.
- **Typing cadence** on long-polled connections.
- **DID resolution via DNS** — resolvers SHOULD support DoH/DoT; Mixed-tier users MUST.
- **The existence of Marque traffic** between two providers at the IP layer.
- **Group-membership observation against ≥40% of Nym mix nodes** by a state adversary.
- **Qualified-tier sender identity to the recipient's provider** — the legal-tier requires identifiability by design; this is a known trade-off, analogous to registered-mail postal carriers knowing both parties.

We do not overclaim.

## 5.8 Anti-spam cryptographic primitives

The crypto layer provides four primitives for the economic anti-spam policy in [`protocol/08`](./08-anti-spam.md):

| Primitive | Construction | Purpose |
|---|---|---|
| **Verified sender identity** | DSK chain to DID-anchored IRK. | Closes the entire spoofing class. SPF / DKIM / DMARC / ARC become obsolete. |
| **Proof-of-work tokens** | Argon2id ([RFC 9106](https://datatracker.ietf.org/doc/html/rfc9106)) with 64 MiB memory cost. | ASIC- and GPU-resistant. Asymmetric tuning: 50 ms for contacts, 5–15 s for strangers. |
| **Bond tokens** | Commit-reveal with optional Chaumian blind signatures. | Refundable deposits; privacy-preserving. |
| **Rate-proof tokens** | Privacy Pass VOPRF ([RFC 9578](https://datatracker.ietf.org/doc/html/rfc9578)) from sender's provider. | Per-epoch rate attestation; also used for new-user bootstrap. |

## 5.9 Cryptographic agility

The `cipher_suite` field is a negotiated identifier not protected by the envelope signature. An active adversary MAY attempt a downgrade attack by rewriting the field in transit.

Marque defends against this by:

- **Binding `cipher_suite` into the MLS `GroupContext` extension field.**
- **Requiring receivers to reject** envelopes whose advertised suite does not match the MLS-level context.

Specification of the binding mechanism is deferred to the MLS cipher-suite codepoint registered in the IANA Marque Algorithms registry.

---

Continue to [**Content**](./06-content.md).
