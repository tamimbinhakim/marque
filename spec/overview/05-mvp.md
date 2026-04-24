# The first-slice surface

> Marque is a full-surface protocol. This page is the implementation-sequencing plan — what an early implementation ships first so users get interop quickly — **not** a product cut. Every capability below the fold is load-bearing for the user-facing pitch.

**You will learn**

- How an early implementation sequences the full protocol in its first twelve weeks.
- Why nothing on this page is "optional" — the full feature set is the value proposition.
- The single rule for reading the rest of the specification with this sequencing in mind.

---

## Read this before the table

The full feature set — Qualified-tier legal proof, Rooms, Mixed-tier metadata privacy, post-quantum hybrid cryptography, FROST threshold signatures, `core.signed_action`, confidential-compute AI, multi-identity inbox — **is the point of Marque**. None of it is optional. Marque without the Qualified tier is not an email replacement for eIDAS-regulated correspondence; Marque without Rooms is not a team-inbox replacement; Marque without Mixed tier is not a threat-model-adequate system for journalism or human-rights work.

What this page does: it describes the **sequencing** an early implementation uses so it can ship interoperable mail quickly, and then layers the rest of the surface in a rational order. The capabilities off the first slice are **not "deferred"** in the sense of "we might add them later if there's demand." They are on the roadmap for every conformant implementation aiming at Phase-2 and beyond, and the conformance matrix in [`protocol/10-conformance.md`](../protocol/10-conformance.md) grades implementations against the full surface.

An implementation that ships only the first slice is **valid and conformant for the Casual interop tier** and a useful contribution. It is not "Marque-complete." The label a user reads in a client capability badge is **Tier: Casual** — not an asterisk, a first-class interop tier in the matrix, and the on-ramp to the higher tiers.

## The first-slice surface in one table

| Area              | First slice (weeks 0–12)                                                                                                                                          | Rest of the surface                                                                                                                                       |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Identity**      | `did:mail` resolution, three-tier key hierarchy (IRK → DSK → MLS), signed KEL, recovery via guardian threshold.                                                  | Portable-tier registry (`marque.id`), disposable handles, multi-identity unified inbox.                                                                   |
| **Transport**     | QUIC + TLS 1.3, DID-bound client certificates, long-poll fetch, multi-endpoint publication (≥2 for Attested+, SHOULD for Casual).                                 | Mix networks, onion routing, Privacy-Pass sealed sender.                                                                                                  |
| **Cryptography**  | MLS (RFC 9420) with `MLS_256_DHKEMX25519_AES256GCM_SHA512_Ed25519`, KeyPackage pools ≥32 with `last_resort`, epoch checkpoints for offline peers.                 | Post-quantum hybrid suite, FROST threshold signatures, `marque-archive-v1`.                                                                               |
| **Envelope**      | CBOR canonical encoding per `schemas/cddl/envelope.cddl`, six pad buckets, BLAKE3 `content_id`, `auth_mode=CASUAL or SIGNED`, `privacy_tier=CASUAL or SEALED`.    | `auth_mode=QUALIFIED`, `privacy_tier=MIXED`.                                                                                                              |
| **Content (MBS)** | `core.text`, `core.image`, `core.attachment`, `core.reply`, `core.forward` (Quoted flavor), `alt_plaintext`, deterministic rendering, required a11y metadata.    | `core.signed_action` (DocuSign replacement), `core.poll`, `core.form`, `core.payment`, `core.agent_instruction`, Attested and Certified forward flavors. |
| **Legal proof**   | `ProofEnvelope` at the Signed tier: sender signature + DSK chain + RFC 3161 timestamp + OpenTimestamps anchor.                                                   | Qualified tier (QERDS), QTSP co-signature, FROST notary relay, CAdES-B-LTA archive timestamps.                                                            |
| **Rooms**         | —                                                                                                                                                                 | All five kinds (`shared`, `team`, `list`, `delegation`, `org`). First-class feature for enterprises, teams, mailing lists, and executive assistants.      |
| **Storage**       | Hybrid archive-of-record (provider = ciphertext, device = cleartext), Sent-copy via sender self-group, archive-key-encrypted cross-device sync, `.marquebox` export. | Provider-side encrypted search, attachment GC at scale, confidential-compute AI inbox.                                                                    |
| **Anti-spam**     | Verified sender identity (inherent from `did:mail`), Argon2id proof-of-work, recipient policy with machine-readable rejection reasons.                           | Refundable bonds, reputation gossip, AI-assistant allowlist model, bond appeals.                                                                          |
| **SMTP bridge**   | Bidirectional bridge with inbound wrapping, outbound downgrade, authenticated download links, Autocrypt header emission on downgrade, Received-chain truncation to 2 hops, upgrade state machine. | —                                                                                                                                                         |
| **Conformance**   | Self-test matrix rows 1–14 in [`protocol/10-conformance.md`](../protocol/10-conformance.md).                                                                      | Self-test rows for Rooms, QERDS, mix-net, confidential-compute AI inbox, multi-currency bonds.                                                            |

## The reading rule

When you encounter a **MUST** elsewhere in the specification that is not in the first-slice column above:

> It is a MUST at the interop tier that introduces it (Attested, Qualified, Rooms-enabled, Mixed-privacy, etc.), not an option. A first-slice implementation is conformant at the Casual tier only; it is **not** conformant at the higher tiers until it ships the MUSTs for those tiers.

Interop tiers are not "levels of optionality." They are named profiles in the conformance matrix, each with its own set of MUSTs. A Casual-tier implementation interoperates with Attested and Qualified senders by reading their ProofEnvelopes and rendering appropriate chips, but it cannot itself produce Attested or Qualified output. It has to ship the additional MUSTs to do that. Every Phase-2 implementation does.

## What this first slice is not

- **Not a toy.** It supports the 5 000–10 000 active-handle Phase-1 cohort. Every Casual-tier MUST is load-bearing for that scale.
- **Not a "lite" product.** There is one Marque. The first slice is how an implementation gets its feet under it; the full surface is what users adopt.
- **Not a license to skip the bridge.** The SMTP bridge is tier-zero. An implementation that ships the first slice without a bridge fails to interop with the legacy ecosystem the migration depends on.
- **Not a license to downgrade silently.** If a sender selects `auth_mode=SIGNED` and the recipient's provider does not support it, the deposit **MUST** fail with a machine-readable error — never silently fall back to Casual.

## A sensible twelve-week plan

A small team targeting the first slice should work in this order:

1. **Weeks 1–3** — `did:mail` resolver, DID document publication at a test domain, KEL append, three-tier key ceremony UI.
2. **Weeks 4–6** — MLS group creation and message exchange, KeyPackage pool, epoch checkpoints, BLAKE3 content IDs.
3. **Weeks 7–9** — Envelope CBOR encoder/decoder, QUIC transport, pad buckets, deposit + long-poll fetch against a peer implementation of the provider role.
4. **Weeks 10–12** — SMTP bridge: inbound wrapping, outbound downgrade with Autocrypt, authenticated download links, upgrade state machine, Received-chain truncation. `.marquebox` export. Self-test matrix rows 1–14.

The weeks-13+ roadmap layers the rest of the surface in whatever order the audience demands. A team building for the Italian PEC wedge reaches Qualified tier quickly; a team building for enterprise reaches Rooms + `core.signed_action` quickly; a team building for journalism reaches Mixed tier quickly.

## When each capability enters an implementation's roadmap

The full surface is on every conformant implementation's roadmap. The table below is the **realistic sequencing order** — which user signal typically pulls each capability forward, not whether the capability is "optional."

| Capability                   | Typical trigger for early promotion                                                                |
| ---------------------------- | -------------------------------------------------------------------------------------------------- |
| **Qualified tier (QERDS)**   | First QTSP partnership or enterprise customer asking for eIDAS-listed delivery.                    |
| **Rooms**                    | First user asking for a shared inbox, team handle, mailing list, or assistant delegation.          |
| **Refundable bonds**         | First stranger-outreach spam wave that verified identity + PoW alone cannot absorb.                |
| **`core.signed_action`**     | First enterprise customer asking to retire DocuSign.                                               |
| **Mix network (`MIXED`)**    | First threat model requiring metadata unlinkability (journalism, activism, sensitive health data). |
| **Confidential-compute AI**  | First enterprise customer asking for a regulated AI-assistant inbox.                               |
| **Post-quantum hybrid**      | NIST deprecates X25519, or a cryptanalytic break against the classical suite is announced.         |
| **Portable-tier registry**   | First user asking for a DNS-independent identity that survives domain loss.                        |
| **Multi-identity inbox**     | First user running personal + work + volunteer identities from one client.                         |
| **Multi-currency bonds**     | First deployment in a jurisdiction outside USD/EUR/GBP.                                            |

If a team ships the first slice and none of these triggers fire in year one, it has time to build the rest of the surface at a measured pace. If several fire early, the roadmap pulls itself forward. Either way, the destination is the same full surface.

---

Continue to [**Protocol / Terminology**](../protocol/01-terminology.md) for the normative sequence, or jump to [**Protocol / Conformance**](../protocol/10-conformance.md) for the full self-test matrix.
