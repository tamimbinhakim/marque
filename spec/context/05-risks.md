# Risks, open questions, and Mailroom integration

> Known risks with explicit mitigations, open protocol questions that a Working Group will need to resolve, and the boundary between Marque-the-protocol and Mailroom-the-reference-MTA.

**You will learn**

- The six known risks Marque accepts, and what bounds each.
- The six open protocol questions requiring WG resolution.
- How Marque and Mailroom relate — HTTP / nginx, not HTTP / Chrome.

---

## 1. Known risks with explicit mitigations

### 1.1 Key-loss identity death

Cannot be fully prevented. Social-recovery guardians plus provider-as-one-default-guardian plus paper-backup opt-in covers most cases, but **a cohort of users will lose everything**. This is the price of self-sovereign identity; the mitigation is to make the default recovery paths maximally redundant for non-expert users, as specified in [`protocol/02 §2.7`](../protocol/02-identity.md#27-recovery).

### 1.2 Handle squatting

Inherited from DNS (UDRP applies) but novel for the portable-tier registry namespace. Requires a **UDRP-analog arbitration process** rather than Handshake-style blockchain auctions. Operational work for the IMIF chartered body.

### 1.3 Spam / reputation cold-start

New DIDs are unknown to recipients' reputation systems. Addressed by **provider-issued bootstrap quota** ([`protocol/02 §2.8`](../protocol/02-identity.md#28-new-user-cold-start)) plus cross-signed social proofs, but remains an adoption friction until the first cohort of well-reputed providers exists.

### 1.4 Registry governance capture

The IMIF portable-tier registry is a single point of governance. Bounded by:

- Mandatory operator rotation.
- Open audits.
- Multi-operator replication (3–7 operators).

The ICANN precedent shows imperfect-but-workable 25-year neutrality. Marque's registry cannot be worse-governed than ICANN has been; if it approaches ICANN's track record, portability holds.

### 1.5 Bridge reliability

The SMTP bridge is load-bearing for Phase 1–4. Buggy bridges churn early adopters back to Gmail within a week. Mitigation: treat bridge operation as tier-zero SRE, with uptime SLOs published in capability records ([`protocol/09 §9.7`](../protocol/09-interop.md#97-bridge-as-tier-zero-operation)).

### 1.6 Gatekeeper clone with incompatible semantics

Apple's RCS adoption without E2EE interop is the template. Mitigation: mandatory conformance tests, no spec-optional extensions at the interop surface, and a charter-level "minimum two unaffiliated providers at every milestone" rule.

## 2. Open protocol questions requiring WG resolution

### 2.1 MLS post-quantum ciphersuite

MLS lacks a registered post-quantum ciphersuite as of RFC 9420. **Liaison with the MLS WG is needed for codepoint allocation.**

### 2.2 FROST threshold at QTSPs

No eIDAS-listed QTSP currently offers threshold FROST signing. **An ETSI TR may be needed**, with sequential independent qualified seals as the Certified-tier fallback until FROST is blessed.

### 2.3 Sealed Sender versus Qualified evidence

Sealed-sender privacy conflicts with Qualified-tier evidence requirements that the recipient's provider know the sender.

**Resolution:** Qualified tier reveals sender identity under provider confidentiality, analogous to registered-mail postal carriers knowing both parties. Specified normatively in [`protocol/05 §5.5`](../protocol/05-cryptography.md#55-metadata-privacy-tiers).

### 2.4 Mixnet versus Qualified

Mixnet routing disturbs timestamps-and-receipts chains. **MIXED privacy tier and QUALIFIED auth-mode are mutually exclusive per message** ([`protocol/05 §5.5`](../protocol/05-cryptography.md#55-metadata-privacy-tiers)).

### 2.5 Cross-jurisdictional archival

Cross-jurisdictional archival across 20-year TSA failures, algorithm deprecations, and CA collapses is operationally hard.

**Mechanism** (`ProofEnvelope`-B-LTA) is specified in [`protocol/07 §7.7`](../protocol/07-legal-proof.md#77-qtsp-revocation-and-long-term-verifiability). **Operational playbook** will be a companion BCP document.

### 2.6 The `did:mail` method choice

The custom method adds to the DID zoo. The alternative is publishing a profile of **`did:webs`** with a mail sub-profile — less namespace pollution, more implementation reuse.

**Decision tabled for the WG.**

## 3. Integration with Mailroom

**Mailroom** (Documents 1–2 of this research series) is the **AI-native MTA** — the sending and receiving infrastructure that sits underneath, hosts mailboxes, applies AI triage and compliance policy, and operates the SMTP bridges during the transition.

| Marque | Mailroom |
|---|---|
| Defines wire protocol | Implements the MTA role |
| Defines identity model | Hosts mailboxes |
| Defines cryptographic envelope | Provides AI-assistant sandboxing ([`protocol/08 §8.8`](../protocol/08-anti-spam.md#88-ai-assistant-safe-content-parsing)) |
| Defines content schema | Issues provider co-signatures and Attested-tier attestations ([`protocol/07`](../protocol/07-legal-proof.md)) |
| Defines legal-proof semantics | Operates the SMTP bridge ([`protocol/09`](../protocol/09-interop.md)) |
| — | Monetizes per the commercial model ([`context/02`](./02-commercial-model.md)) |

A reference **Mailroom deployment alongside the reference protocol implementation** in Phase 1 is how the ecosystem bootstraps. **Mailroom is the first Marque provider.**

The relationship is the **HTTP / nginx** relationship, not **HTTP / Chrome**. Marque is the protocol; Mailroom is one excellent implementation. The protocol's permanent guarantee is that any conformant implementation interoperates with any other — **no single MTA, including Mailroom, can capture the address space, the identity layer, or the trust tier.**

This is the architectural commitment that makes the commercial model sustainable and the migration strategy possible.

---

Continue to [**Related work**](./06-related-work.md).
