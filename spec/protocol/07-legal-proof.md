# Legal proof

> Tiered non-repudiation of origin, delivery, content, and time. Marque's Qualified tier is designed for eIDAS 2.0 QERDS conformance from day one, with OpenTimestamps Bitcoin anchoring for zero-cost long-term verifiability.

**You will learn**

- Why PEC works and De-Mail didn't — the empirical case for legal-proof mail.
- How Marque's three tiers map to eIDAS Art. 43 and Art. 44.
- The `ProofEnvelope` composition: signature, certificate chain, timestamps, FROST, OpenTimestamps.
- Proof objects: submission, delivery, receipt, content, identity.
- Non-delivery states and machine-readable error surface.
- Read-receipt obligation at Qualified tier.
- QTSP revocation handling and long-term verifiability via CAdES-B-LT.
- Cost-quotation requirement before Qualified sends.
- Retention per tier, amendment, and retraction.

---

## 7.1 The empirical record

Legal-tier electronic mail has been attempted many times. Two examples define the outcome boundaries.

**Italy's PEC** (*Posta Elettronica Certificata*) has roughly 16 million active accounts, mandatory for all Italian businesses and professionals, processing about 2.5 billion messages per year with full legal equivalence to paper registered mail under DPR 68/2005. Italian Court of Cassation rulings consistently affirm PEC delivery receipts as full proof of dispatch and delivery time. **PEC works because regulation mandated it.**

**Germany's De-Mail**, lacking a mandate, collapsed. Deutsche Telekom withdrew in 2022, United Internet in March 2024, German customs in September 2024. A voluntary legal-tier service without regulatory scaffolding does not reach adoption escape velocity.

**France's Lettre Recommandée Électronique** under eIDAS and **Swiss IncaMail** both deploy at working scale.

**The US Electronic Postmark** never achieved meaningful adoption.

The lesson: regulation is necessary but not sufficient. Marque is designed so the protocol-level substrate is ready the day regulation lands.

## 7.2 The eIDAS 2.0 anchor

Marque's legal-proof tier is designed for **eIDAS 2.0 QERDS** conformance from day one.

[Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) entered into force on 20 May 2024 and strengthens cross-border QERDS interoperability via:

- [ETSI EN 319 522](https://www.etsi.org/deliver/etsi_en/319500_319599/31952201/) — ERDS framework, semantic contents, formats, and bindings.
- [ETSI EN 319 521](https://www.etsi.org/deliver/etsi_en/319500_319599/319521/) — ERDS provider policy and security requirements.
- ETSI EN 319 532 — REM profile.
- [ETSI TS 119 524-1/-2](https://www.etsi.org/deliver/etsi_ts/119500_119599/) — ERDS conformance test suites (for EN 319 522 + EN 319 521).

Implementing acts land through 2026–2027. Marque publishes a formal profile mapping its envelope objects to EN 319 522-2 evidence semantic types, enabling CAB conformity assessment under EN 319 521 policy.

**This is the regulatory anchor that could accelerate adoption by five years versus a pure-market path.**

## 7.3 The three tiers

| Tier | Cost/msg | Sender signature | Provider role | Timestamp | eIDAS status | UI chip |
|---|---|---|---|---|---|---|
| **Casual** (`CASUAL`) | 0 | Optional MAC | TLS transport | — | Ordinary message | *(none)* |
| **Attested** (`SIGNED`) | ~€0.001 | Ed25519 + ML-DSA | Co-signs, transparency log | RFC 3161 + OpenTimestamps | ERDS (Art. 43) | **Signed** |
| **Certified** (`QUALIFIED`) | €0.10–€0.50 | Qualified signature | FROST threshold with QTSP | Qualified TSA (EN 319 422) | QERDS (Art. 44) | **Registered** |

A user-visible message either carries no chip (Casual), a **Signed** chip (Attested), or a **Registered** chip (Certified) — full lexicon in [`protocol/01 §4`](./01-terminology.md#4-tier-chips).

## 7.4 The ProofEnvelope

A `ProofEnvelope` wraps `SIGNED` or `QUALIFIED` messages. Canonical composition:

| Field | Presence | Description |
|---|---|---|
| `proof_version` | always | `"marque-proof/1"`. |
| `message_hash` | always | Canonical SHA-256 hash of a TLS-presentation-syntax serialization of the message. |
| `sender_signature` | always | Sender's signature under the chosen algorithm. |
| `sender_cert_chain` | always | DSK certificate chain to the IRK. |
| `qualified_certificate` | Qualified only | QTSP-issued qualified certificate bound to the DID per ETSI TS 119 461. |
| `rfc3161_timestamp` | Attested + Qualified | [RFC 3161](https://datatracker.ietf.org/doc/html/rfc3161) qualified timestamp. |
| `frost_threshold_signatures` | Optional (Attested); REQUIRED (Qualified) | [FROST](https://datatracker.ietf.org/doc/html/rfc9591) threshold signatures from notary relays. |
| `opentimestamps` | Optional | OpenTimestamps `.ots` proof for Bitcoin-anchored long-term verification. |
| `archive_timestamps` | Long-term profiles | CAdES-B-LTA archive timestamps re-anchoring every two years or on algorithm deprecation. |
| `delivery_receipt` | After delivery | Delivery state, recipient provider signature, transparency-log entry. |
| `trust_list_snapshot_ref` | Qualified | URL to the historical trust-list snapshot at signing time, per §7.7. |

**FROST ([RFC 9591](https://datatracker.ietf.org/doc/html/rfc9591)) was chosen over BLS** because it produces stock 64-byte Ed25519-compatible signatures verifiable by any existing implementation — critical for eIDAS trust-list tooling interoperability.

Full schema: [`proof-envelope.schema.json`](../../schemas/proof-envelope.schema.json).

## 7.5 Proof objects

The ETSI EN 319 522-2 semantic equivalents:

### Proof of submission

`SubmissionReceipt` signed by the sender's provider — analogous to PEC's *ricevuta di accettazione*. Attests that the sender's provider has accepted the message for onward delivery.

### Proof of delivery

`DeliveryReceipt` at one of the states defined in §7.6. FROST-threshold-signed at the Certified tier with at least one QTSP participant.

### Proof of receipt (read receipt)

Tier-dependent:

| Tier | Behavior |
|---|---|
| `CASUAL` | Off; no receipt produced. |
| `SIGNED` (Attested / ERDS) | Off by default; recipient MAY opt in per-sender or per-thread. |
| `QUALIFIED` (Certified / QERDS) | **Required for QERDS conformance.** Receipt produced on first open, or an explicit `REFUSED_TO_ACKNOWLEDGE` produced on decline. Off is not permitted, consistent with PEC's *ricevuta di avvenuta consegna*. |

At Qualified tier, the recipient client MUST surface the read-receipt obligation to the user at first receipt of a Qualified-tier message from a new sender, via a one-time consent modal. The user MAY decline; declining produces `REFUSED_TO_ACKNOWLEDGE`. The user **cannot silently consume** a Qualified message without producing either `ReadReceipt` or `REFUSED_TO_ACKNOWLEDGE`.

### Proof of content

The canonical hash binding (SHA-256 over TLS-presentation-syntax serialization; see §7.4).

### Proof of identity

The DID plus qualified certificate at the Certified tier, identity-proofed under [ETSI TS 119 461](https://www.etsi.org/deliver/etsi_ts/119400_119499/119461/).

## 7.6 Non-delivery states

Paper registered mail's full evidence catalog, extended to cover Marque's policy and cryptographic error surface:

| State | Meaning |
|---|---|
| `DELIVERED` | Recipient authenticated and fetched. |
| `STORED_FOR_RECIPIENT` | Persisted pending recipient online. |
| `STORED_BUT_UNFETCHED` | After 30 days pending recipient; escalates from `STORED_FOR_RECIPIENT`. |
| `REFUSED_TO_ACKNOWLEDGE` | Qualified tier only: recipient authenticated but declined the read-receipt obligation. Legally equivalent to paper registered mail's "refused at door." |
| `POLICY_REJECTED_BOND` | Recipient policy required a bond the sender did not post. |
| `POLICY_REJECTED_POW` | Recipient policy required PoW the sender did not furnish. |
| `POLICY_REJECTED_TIER` | Recipient policy requires a minimum tier the sender did not meet. |
| `UNDELIVERABLE` | All provider endpoints failed after the retry window. |
| `DID_UNRESOLVABLE` | Recipient DID cannot be resolved at any mirror. |
| `DID_DEACTIVATED` | Recipient DID's current KEL state is deactivated. |
| `RECIPIENT_PROVIDER_COMPROMISED` | Recipient's advertised providers are on a federation-gossip compromised list at send time. |
| `QTSP_UNAVAILABLE` | Qualified tier only: no configured QTSP notary reachable within the send-time budget. |

**Silent drop-to-spam is prohibited** at the protocol level. Every non-delivery MUST be machine-readable AND user-visible. The sender's client MUST surface the specific state to the user.

## 7.7 QTSP revocation and long-term verifiability

A QTSP's trust-list status may change between a message's signing and a later verification. QTSPs can lose qualified status under eIDAS for non-compliance, acquisition, or cessation. Marque's handling:

- **At verification time** the verifier consults the **historical trust-list snapshot** corresponding to the message's archive-timestamp — not the current trust-list. CAdES-B-LT captures the validation data (CRLs, OCSP responses, trust-list entries) at signing time; this is what survives later QTSP revocation.
- **Archive timestamps** (CAdES-B-LTA) re-anchor the validation envelope every two years or on algorithm-deprecation notice. Responsibility: the ProofEnvelope custodian ([`protocol/03 §3.9`](./03-architecture.md)).
- **Revocation of a QTSP after signing** does NOT invalidate messages signed while the QTSP was trusted, provided CAdES-B-LT validation data was captured correctly.
- **Cross-border recognition** beyond eIDAS jurisdictions: the ProofEnvelope is structurally jurisdiction-neutral; expert-witness presentation of the cryptographic chain is the recognized mechanism. National implementation profiles are future work.

## 7.8 Cost visibility at send time

A sender MUST see the total expected cost of a Qualified-tier send **before** committing.

The sender's client queries the recipient's capability record and the sender's own provider capability record, then presents:

- Sender-provider tier cost per `tiers.certified.cost_per_msg_usd`.
- Recipient-provider tier cost, if exposed.
- QTSP co-signing cost, visible from the sender-provider's configured QTSP partners.
- Archive-timestamp cost amortization, if charged.

Sends without a displayable pre-send cost quotation MUST NOT proceed at Qualified tier. The sender explicitly confirms the quoted total.

## 7.9 Retention per tier

Different parties retain different artifacts for different durations:

| Tier | Content retention | ProofEnvelope retention |
|---|---|---|
| Casual | Provider default (≥30 days) | — |
| Attested | Provider default; user MAY extend via pin | **7 years minimum.** Sender's provider primary; recipient's provider MAY mirror. |
| Certified (QERDS) | Provider default; user SHOULD extend to the full ProofEnvelope retention | **10 years minimum** (or per jurisdictional requirement). Sender's provider and issuing QTSP jointly; MUST survive either party's failure. |

Archive timestamps (CAdES-B-LTA) re-anchor every two years or on algorithm-deprecation notice. The re-anchoring custodian is normally the sender's provider for the first 10 years, then optionally the IMIF long-term-archive service for beyond.

**Attachment GC.** A provider MUST retain an attachment blob for at least as long as the longest `retention.minimum_days` of any envelope referencing it. Users MAY pin attachments to survive envelope expiry; pinned attachments count against the user's `total_mailbox_bytes_per_user` quota.

## 7.10 Append-only log and amendment

The edit-versus-immutability tension resolves via **append-only signed amendments**:

- A message is a signed append log, not mutable state.
- The message's `content_id` is the BLAKE3 root of the canonical log, not the current state.
- **Amendments** are signed `core.edit` blocks referencing the target block's hash.
- Recipients toggle between **"current"** and **"history"** views in the client.
- **Retraction** proves disavowal at a specific time without erasing the original — exactly the paper-mail analog where a letter of retraction does not un-send its predecessor.

A `core.retract` block carries a target `content_id`, an optional reason, and a `scope`:

| Scope | Meaning |
|---|---|
| `own_message` | Sender retracts their own message. Always permitted. |
| `thread_owner` | Room owner retracts any member's message. Room-backed threads only; recorded in the Room audit log ([`protocol/04 §4.12`](./04-rooms.md#412-audit-logs)). |

## 7.11 Multi-jurisdiction recognition

**eIDAS is EU.** Marque's Qualified tier is designed for cross-border admissibility within eIDAS jurisdictions under QERDS. Recognition elsewhere depends on local evidence rules; the `ProofEnvelope`'s cryptographic structure is jurisdiction-neutral and designed for expert-witness presentation.

Specific national profile documents — for US federal court admissibility, UK, Japan, Canada — are future work. The protocol does not prevent these; it provides the raw material.

---

Continue to [**Anti-spam**](./08-anti-spam.md).
