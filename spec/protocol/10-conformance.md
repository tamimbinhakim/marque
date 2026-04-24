# Conformance

> What it means to be a conformant Marque implementation. A concise pointer to every MUST in the specification, the interop tiers, and the self-test procedure.

**You will learn**

- The three roles and their conformance obligations.
- The four interop tiers (Casual, Attested, Certified, Bridge).
- The self-test matrix every implementation SHOULD run.
- Where to find test vectors.

---

## 10.1 The three roles and their conformance

### Client conformance

A conformant Marque client MUST:

- Resolve `did:mail` via the three-path resolver (DNS primary, portable-tier fallback, chain-anchored disaster recovery).
- Verify KEL integrity and enforce trust-on-first-use on inception.
- Honor the 72-hour nullification window on every message from a rotating correspondent.
- Construct envelopes per the normative [CDDL](../../schemas/cddl/envelope.cddl) and sign with a valid DSK chain.
- Refuse `privacy_tier: MIXED` combined with `auth_mode: QUALIFIED` at compose time.
- Support the full MBS `core.*` block vocabulary for rendering; fall back to `core.text` or `alt_plaintext` for unsupported types.
- Render user-facing chips per [`protocol/01 §4`](./01-terminology.md#4-tier-chips).
- Maintain a device-resident cleartext cache and participate in cross-device sync via the `device-sync` endpoint.
- Surface non-delivery states as machine-readable AND user-visible.
- Support Reply, Reply-All, three Forward flavors, and honor `forwardable` / `quotable` flags.
- Surface the "You were BCC'd" banner and restrict Reply-All default on BCC receipt.
- Produce read receipts at Qualified tier or an explicit `REFUSED_TO_ACKNOWLEDGE`.
- Show pre-send cost quotation before Qualified-tier send.
- Warn before first outbound SMTP downgrade per recipient and cache consent for 90 days.

A conformant client SHOULD:

- Offer provider-as-guardian opt-out as a first-class setting.
- Surface KEL-rotation-in-progress banners for any thread involving a recovering DID.
- Participate in federation reputation gossip as a recipient MAY.
- Support handle-similarity warnings for inbound messages with lookalike DIDs.

### Provider conformance

The eleven provider MUSTs from [`protocol/03 §3.6`](./03-architecture.md#36-provider-obligations).

### Notary conformance

A conformant notary MUST:

- Verify each Attested or Qualified tier signing request against the sender's DID and DSK chain before co-signing.
- Produce FROST signatures verifiable by any Ed25519 implementation.
- For Qualified-tier: hold a qualified trust-list status issued by a recognized eIDAS supervisory body (or equivalent) and publish its QTSP identity in its capability record.
- Participate in the transparency log for every delivery receipt it co-signs.

## 10.2 Interop tiers

Implementations declare which interop tiers they support. Each tier has a separate MUST-matrix.

| Tier          | Description                                                        | Required MUSTs                                                                                                                  |
| ------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **Casual**    | Minimum viable Marque — native send/receive at `CASUAL` auth-mode. | Client MUSTs, provider MUSTs 1–10, full envelope/CDDL round-trip.                                                               |
| **Attested**  | Adds `SIGNED` auth-mode support and the Signed chip.               | All Casual MUSTs + ProofEnvelope generation and verification at `SIGNED` + `rfc3161_timestamp` + OpenTimestamps.                |
| **Certified** | Adds `QUALIFIED` auth-mode with QERDS conformance.                 | All Attested MUSTs + QTSP integration + FROST threshold + CAdES-B-LT + read-receipt obligation + cost quotation.                |
| **Bridge**    | Adds bidirectional SMTP bridging.                                  | All Casual MUSTs + SMTP ingress discovery + sanitization + authenticated download links + Autocrypt + round-trip state machine. |

An implementation MAY support any subset. Implementations claiming conformance MUST declare their tier set in their capability record (providers) or equivalent manifest (clients).

## 10.3 Self-test matrix

Every implementation SHOULD run the following self-tests prior to interop declaration:

### Envelope round-trip

- Encode a canonical envelope, decode, re-encode. Bytes MUST be identical.
- Mutate one bit of the envelope body and verify signature verification fails.
- Verify rejection of envelopes whose on-wire size doesn't match the nominal `pad_bucket`.
- Verify rejection of `QUALIFIED` envelope lacking `proof_envelope_ref`.
- Verify rejection of `MIXED` + `QUALIFIED` combination.

### Identity

- Resolve a portable-tier DID via the IMIF registry.
- Resolve a provider-hosted DID via `.well-known/mail-did`.
- Reject a KEL with inconsistent `prev_hash` chain.
- Reject a KEL with non-monotonic `seq`.
- Verify TOFU enforcement: a changed inception hash MUST fail.
- Verify nullification window: a rotation within 72 hours followed by a nullification MUST result in the original keyset being authoritative.

### MLS profile

- Create a thread between two DIDs; exchange 10 messages alternating `CASUAL` and `SIGNED`.
- Add a third member; verify the new epoch is reachable via epoch checkpoint.
- Remove a member; verify the removed member cannot decrypt post-remove messages.
- Exhaust a KeyPackage pool; verify `last_resort` distribution.

### Content

- Render each `core.*` block to the conformance test vectors; output MUST be byte-stable.
- Reject `core.image` lacking `a11y.alt` (unless `decorative: true`).
- Verify `core.forward` refuses the attested flavor when original `forwardable` is `no` or `quoted_only`.
- Verify BCC recipient's envelope carries `was_bcc: true`.

### Legal proof

- Generate and verify an Attested ProofEnvelope.
- Generate and verify a Qualified ProofEnvelope with FROST threshold.
- Verify CAdES-B-LT validation data captures correctly.
- Verify `delivery_receipt` transitions: `STORED_FOR_RECIPIENT` → `DELIVERED` or → `STORED_BUT_UNFETCHED` after 30 days.
- Verify `REFUSED_TO_ACKNOWLEDGE` path at Qualified tier.

### Anti-spam

- Generate Argon2id PoW at specified memory cost; verify acceptance.
- Post a bond, receive refund on recipient engagement.
- Contest a burned bond; verify the counter-challenge flow.
- Reject `ai_instruction: true` from a non-allowlisted sender.

### Bridge (Bridge tier only)

- Ingest SMTP from a sender with a discoverable DID; verify MLS wrapping.
- Ingest SMTP from a sender without a discoverable DID; verify plaintext-bridged labeling.
- Downgrade a Marque envelope to SMTP; verify attachment link lifetime and access control.
- Verify Received-chain truncation.
- Progress the upgrade state machine `UNKNOWN → AVAILABLE → MUTUAL → NATIVE`.

## 10.4 Test vectors and conformance suite

The canonical conformance test suite is maintained under the `marque-protocol` GitHub organization at `tests/` (not yet published as of founding spec v0). Test vectors are JSON + CBOR fixture files organized by the self-test matrix above. The suite is normative; implementations are measured against it, not against any particular codebase. Example implementations (client, provider, bridge) are welcome and expected, but none is privileged — every implementation that passes the suite is conformant, and implementers are unrestrained in how they differentiate above the wire.

A third-party test service — `conformance.marqueproto.org` — will publish pass/fail status for any implementation that runs the suite in CI and publishes its self-test results URL in its capability record.

## 10.5 Certification

Phase 1 has no formal certification. Phase 2 onward, providers claiming `Certified` (QERDS) interop tier MUST undergo a CAB conformity assessment against [ETSI EN 319 521](https://www.etsi.org/deliver/etsi_en/319500_319599/319521/) (provider policy and security requirements) and [ETSI EN 319 522](https://www.etsi.org/deliver/etsi_en/319500_319599/31952201/) (ERDS framework, semantic contents, formats, and bindings), using the [ETSI TS 119 524-1/-2](https://www.etsi.org/deliver/etsi_ts/119500_119599/) conformance test suites.

Conformance claims are self-declared and **verifiable by public test run** during Phase 1; independently audited during Phase 2+.

## 10.6 Extending the specification

**Spec-optional extensions are refused at the interop surface.** Conformance tests are mandatory; extensions live in a registered namespace.

New features enter via MIPs ([`../mips/mip-0001-process.md`](../../mips/mip-0001-process.md)). Any extension that introduces MUST or SHOULD language MUST be a Final MIP before an implementation can claim it is required for conformance.

---

This concludes the `protocol/` sequence. For supporting material, continue to [**Context / Blockchain scope**](../context/01-blockchain-scope.md) or jump to any entry in the [spec index](../README.md).
