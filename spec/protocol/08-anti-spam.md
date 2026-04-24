# Anti-spam in the AI age

> Content filtering has reached a ceiling against LLM-generated abuse. Marque shifts filtering from content semantics to economic and cryptographic identity — composed as an AND-of-ORs policy where no single mechanism wins.

**You will learn**

- Why content classifiers cannot keep up with personalized LLM phishing.
- The eight anti-abuse layers and their composition.
- How verified sender identity makes SPF / DKIM / DMARC / ARC obsolete.
- Proof-of-work tuning, refundable bonds with appeals, reputation gossip, recipient-controlled policy.
- The protocol-level defense against AI-assistant prompt injection — impossible to retrofit into SMTP.
- Honest statement of the risks the economic anti-spam model introduces.

---

## 8.1 The content-filtering ceiling

Traditional content filtering has reached a ceiling against LLM-generated abuse.

- A 2026 HAL study on 7 700 participants found LLM-generated personalized spear phishing nearly **triples click rate** versus generic phishing.
- Bethany et al. 2024's 11-month, 9 000-person university study showed LLM lateral phishing evades existing anti-phishing infrastructure.
- NDSS 2025 measured LLM attacks at **92.5% emotional manipulation** versus 56.5% traditional and **96.8% personalization** versus 56.2%.
- IBM's 2024 report: LLMs make phishing **96% faster** to produce.
- A spammer runs a 7B open-weights model on a consumer GPU for **sub-cent marginal cost per message.**

Bayesian filters, SpamAssassin rules, and even transformer-based classifiers depend on static patterns or distributional anomalies — both collapse when every message is unique, grammatically perfect, contextually plausible, and interleaved into hijacked legitimate threads.

Marque shifts filtering from content semantics to economic and cryptographic identity.

## 8.2 AND-of-ORs composition

Eight layers compose. No single mechanism wins on its own.

## 8.3 Verified sender identity — the mandatory baseline

The sender's cryptographic key is their identity. The message is signed by a DSK bound to a DID. **Spoofing is structurally impossible.** SPF, DKIM, DMARC, and ARC become obsolete — they are overlays attempting to provide, at the transport layer, a property Marque provides at the identity layer.

This alone eliminates the majority of email phishing: mail that purports to be from `bank@example.com` but is not actually signed by that identity is rejected at the envelope layer before any content inspection.

## 8.4 Proof-of-work — Argon2id, tuned asymmetrically

**Argon2id** ([RFC 9106](https://datatracker.ietf.org/doc/html/rfc9106)) with 64 MiB memory cost. Memory-hardness prevents ASIC and GPU acceleration; the send cost scales with physical memory, which adversaries cannot amortize.

Difficulty is dictated by **recipient policy** advertised in the recipient's capability record:

| Relationship | Typical cost |
|---|---|
| Known contact | 50–100 ms |
| Stranger | 5–15 s |
| Low-reputation sender | Up to 60 s |

Low-end device mitigation: mobile clients MAY offload PoW to their home provider via a Privacy Pass VOPRF exchange. The cost exists; it is simply paid by the provider as part of the service.

## 8.5 Refundable bonds — the strongest single defense

Senders post **$0.01–$1 equivalent** to reach strangers. Refunded on engagement (recipient reply or mark-as-not-spam) or after a policy timeout; **burned on spam report** — subject to the appeals process below.

Implementations:

- Lightning micropayments.
- Provider-held escrow.
- Protocol-level IOUs settled in batches.

**Currency.** Bonds are denominated in `USD`, `EUR`, `GBP`, or `BTC` (satoshi). Multi-currency per capability record. Cross-currency bonds settle via provider-configured oracle pricing at burn time.

**Appeals.** A bond burn is reversible if:

1. The sender contests within 14 days via a `bond-contest` control frame.
2. The sender's provider posts a counter-challenge to the recipient's provider.
3. If the recipient does not re-confirm the spam designation within 7 days, the bond is refunded.
4. If re-confirmed, the bond remains burned; the dispute is logged on the federation reputation-gossip channel where serial false-positive reporters accrue negative reputation.

**Reputation-poisoning mitigation.** A recipient whose spam reports are contested at a rate above 30% over 90 days has their spam-report weight reduced proportionally on the gossip channel. Recipients cannot arbitrarily grief senders.

## 8.6 Web-of-trust reputation

Contacts-of-contacts trust scores plus **federated k-anonymous spam-signature gossip** between providers. This is *Google Safe Browsing for email, but privacy-preserving* — providers share hashes of high-confidence abuse signatures, not identifiable content.

Reputation is **decay-weighted** (recent activity matters more) and **per-domain**, so a single compromised account does not permanently taint an organization's reputation.

**Cartel mitigation.** Recipients override gossip when they choose; gossip is advisory, not mandatory. Multiple competing gossip networks are permitted; reputation from any one network is not monopoly.

## 8.7 Recipient-controlled explicit policy

Replaces opaque "went to spam" with **structured rejections**:

> `policy requires bond ≥ $0.10 or PoW ≥ 10 s`

Senders see reasons and can retry. See [`protocol/07 §7.6`](./07-legal-proof.md#76-non-delivery-states) for the non-delivery state enum.

A recipient's capability record expresses policy. Example:

```json
"anti_spam": {
  "argon2id": { "base_ms_strangers": 5000, "base_ms_contacts": 50 },
  "bond_minimum": 0.10,
  "bond_currencies": ["USD", "EUR", "BTC"],
  "reputation_gossip_enabled": true,
  "proof_of_humanity_required": false
}
```

Senders' clients consult the policy at compose time and present the applicable cost to the user.

## 8.8 AI-assistant-safe content parsing

**This is the defense SMTP completely lacks.** Three requirements:

### Structural separation of data and instructions

MBS blocks carry `ai_instruction: false` by default. Email assistants insert block content into their prompts **as data only**, with delimiters the model is trained to treat as untrusted — following DeepMind's 2025 [CaMeL dual-LLM approach](https://arxiv.org/abs/2503.18813).

### Signed instruction blocks

A trusted scheduling service explicitly marks and signs a `core.agent_instruction`. The recipient's assistant follows it only if the sender is **allowlisted** AND the instruction type is allowlisted.

**Allowlist model.** Per-user, maintained by the user's client, opt-in. Providers MAY seed the allowlist with well-known integrations (a specific calendar service, a known travel booking provider) that the user explicitly enables. **Providers MUST NOT force-add entries.** The allowlist is portable as part of `.marquebox` export.

### Content-type labeling

Blocks marked `public_untrusted: true` are always data, never instructions, regardless of signature. Default `public_untrusted: true` for any block whose provenance is not directly attributable to the sender's DID — forwarded content, quoted text from uncertified origin, archived public-list content all default `public_untrusted: true`.

This is a **protocol-level defense** against the CVE class dominating 2026–2030 as AI assistants become ubiquitous. **It is impossible to retrofit into SMTP / MIME** because RFC 5322 has no data/instruction separation.

## 8.9 Payment-optional cold outreach

Inverts the economic model for sales, recruiting, and similar contexts where recipients opt in — senders pay, refundable on engagement. Recipients set explicit "cold outreach requires payment of X" policies; senders posting payment flow naturally into the inbox rather than the spam folder.

## 8.10 Proof-of-humanity — explicitly not mandated

Worldcoin, BrightID, and Proof of Humanity are **rejected as a mandate**. Biometric centralization and exclusion of poor users create a two-class internet.

Proof-of-humanity is available as an **optional attestation tier** for narrow high-stakes contexts (legal notices, election communication, regulated financial advice) where a recipient may require verified humanity. Senders who refuse to produce it see the corresponding non-delivery state.

## 8.11 Honest dystopia risks

Every mechanism above has a failure mode. Marque acknowledges them explicitly.

| Risk | Mitigation |
|---|---|
| **Pay-to-email normalizes attention-as-commodity.** | Refundable-by-default bonds so legitimate outreach is free; cold outreach is only occasionally paid. |
| **PoW privileges fast CPUs / excludes low-end devices.** | Mobile clients offload PoW to their providers; providers compete on offload pricing. |
| **Reputation gossip becomes a soft blacklist cartel.** | Reputation decay, multiple competing gossip networks, recipient overrides. Per-domain rather than per-provider. |
| **Verified identity empowers stalkers.** | DID pseudonymity, multiple DIDs per natural person, provider-offered disposable rotating handles ([`protocol/02 §2.11`](./02-identity.md#211-disposable-and-linked-handles)). |
| **Bond-burn griefing by adversarial recipients.** | 14-day appeal window with counter-challenge; serial-false-positive weighting. |
| **Provider-seeded allowlists become default-on corporate integrations.** | Providers MUST NOT force-add entries; user explicitly enables. |

These are all real trade-offs. The design is not that they disappear — it is that the trade-off is visible, the mitigations are specified, and the user has agency.

---

Continue to [**SMTP interop**](./09-interop.md).
