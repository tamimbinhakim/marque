# Commercial model

> Privacy alone does not monetize. The Skiff collapse is the proof. Marque treats portable identity and E2EE as free boring infrastructure (like TLS) and routes revenue to trust-layer services — compliance, legal-proof, transactional APIs — that advertising cannot cross-subsidize. The protocol is engineered against single-vendor capture, so commercial competition happens at the feature layer (AI, UX, search, discovery) rather than at the protocol surface.

**You will learn**

- Why Skiff failed and what that tells us about the revenue model.
- The ten provider revenue streams, ranked by margin and defensibility.
- Network-effect bootstrap: migration rewards, transactional beachhead, regulatory on-ramps.
- The market-compression prediction grounded in MNP precedent.
- The explicit anti-capture guardrails that keep the ecosystem fair.

---

## 1. The Skiff lesson

Skiff reached ~1M+ users on pure-privacy positioning with $14.2M VC (Sequoia-led). It was [acquired by Notion on 9 February 2024](https://techcrunch.com/2024/02/09/notion-acquires-privacy-focused-productivity-platform-skiff/) (Notion is also Sequoia-backed). Skiff services were shut down ~9 August 2024; email forwarding ran an additional six months and terminated on 9 February 2025.

The lesson is unambiguous: **privacy alone does not monetize**, because Proton, Tutanota, and Fastmail already occupy that shelf at commodity prices.

Marque therefore treats **E2EE, DID identity, and portable addresses as free, open, boring infrastructure** — exactly like TLS — and routes revenue to **trust-layer services** that require auditors, regulators, uptime SLAs, and customer-success teams that advertising revenue cannot cross-subsidize.

## 2. Current email-provider economics

Framing the target market:

| Segment | Typical pricing |
|---|---|
| Google Workspace ([2026 price list](https://workspace.google.com/pricing)) | Business Starter $7 (annual) / $8.40 (flex), Standard $14 / $16.80, Plus $22 / $26.40 per user/mo |
| Privacy-premium consumer ([Proton](https://proton.me/mail/pricing), Tutanota, [Fastmail](https://www.fastmail.com/pricing/)) | ~$3–5/mo |
| [HEY](https://www.hey.com/pricing/) | HEY for You $99/yr; HEY for Domains $12/user/mo (~$144/yr); premium 2-character `@hey.com` handles $999/yr |
| [Superhuman](https://help.superhuman.com/hc/en-us/articles/38456109456147-Pricing-Plans) | Starter $30/mo (or $25/mo annual); Business $40/mo |
| Transactional APIs (SendGrid, Postmark, Resend, Mailgun, SES, Loops) | ~$3–6 B annual market |
| Italian PEC (consumer retail) | €5–€10/yr for low-tier plans (e.g., [Aruba PEC Standard](https://www.aruba.it/en/pec-price-list.aspx) €5 first year / €9.90/yr renewal) |

The real upsell at the incumbent tier is **compliance and collaboration**, not storage.

## 3. Ten revenue streams

Ranked by margin and defensibility:

1. **Enterprise compliance suites** — eDiscovery, DLP, legal hold, retention, SSO/SCIM at $15–40/user/mo. *Where incumbents actually make money.*
2. **Legal-proof QERDS tier** — €0.20–0.80/message or €5–15/user/mo flat. *Competes against €3–8 physical raccomandata, not against email.*
3. **Transactional / developer APIs** — 3K/mo free, then $0.40/1K, with native E2EE and cryptographic receipts. *Differentiates from Resend and SendGrid.*
4. **Marketing / ESP features** — layered on the same API for the Klaviyo / Mailchimp tier.
5. **Consumer / SMB subscription** — $3–6/mo, beneath Proton and Fastmail.
6. **Custom domain / white-label** — resales to law firms and agencies.
7. **Concierge migration services** — $50–300/seat one-time. *The Skiff-avoidance lever, zeroing switching costs.*
8. **Premium anti-spam / anti-phishing** — adversarial-ML filtering, bounty-funded blocklists.
9. **Privacy-respecting AI assistants** — on-device or confidential-compute inference. Priced at Gemini / Copilot parity.
10. **Storage and eArchiving** — newly qualified under eIDAS 2.0, per-GB/month for regulated industries.

## 4. Network-effect bootstrap

### 4.1 Migration rewards

First-mover providers offer **6–12 months free** for users who port DIDs from Gmail or Outlook, funded by **reduced acquisition cost** versus the $30–80 consumer CAC and $200–1 000 B2B CAC currently spent on ad-words.

### 4.2 Transactional beachhead

**Developers adopt first** (as they did switching from SendGrid to Resend after the free-tier retirement). Every password-reset email delivered over Marque with a cryptographic receipt is a **recipient-education event**.

### 4.3 Regulatory on-ramps

eIDAS 2.0 QERDS mandates **cross-border interop**, so Marque plugs into existing trust-list infrastructure rather than competing with it. Italian PEC's **~2.5 billion messages per year** ([AgID supervised-services report](https://www.agid.gov.it/en/news/digital-identity-pec-qualified-electronic-signature-online-report-on-agid-supervised-services)) is a **precedent deployment, not a threat**.

## 5. Market prediction

The [Cho–Ferreira–Telang Mobile-Number-Portability study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2265104) across 15 EU countries predicts **4–8% price compression** and material consumer-surplus gains. Provider ARPU falls; **total addressable market expands** as switching friction disappears.

**Providers compete on bundles and vertical specialization** (law, health, finance, EU sovereignty) rather than address hostage-taking.

Marque does not compete with Gmail on price — it competes with **registered mail, DocuSign, and the compliance stack** — while being at least as cheap as Gmail on the base tier because the protocol itself is free.

## 6. Anti-capture guardrails

Commercial competition is healthy. A market that compresses to one dominant implementation — the pattern Gmail, Outlook, and iMessage each exemplify — is not. The protocol is designed with explicit rules that keep the ecosystem pluralistic rather than trusting any one operator to behave.

### 6.1 Example implementations exist; none is privileged

Marque expects and welcomes **example implementations** — the *Mailroom* MTA, a TypeScript web client, native mobile clients, and anything else builders want to ship. What Marque does **not** do is privilege any one of them. There is no "reference" client, server, or provider whose behavior defines the protocol; the conformance test suite (`tests/` under the `marque-protocol` GitHub organization, [`spec/protocol/10-conformance.md §10.4`](../protocol/10-conformance.md#104-test-vectors-and-conformance-suite)) is the normative artifact every implementation is measured against.

Implementers are **unrestrained** in what they build above the wire. They pick their language, architecture, threading model, UI framework, AI strategy, search strategy, storage engine, deployment topology, commercial posture, license (subject to test-vector license compatibility), and roadmap. Every implementation that passes the suite is conformant, and every conformant implementation is a peer.

The reason this matters: a "reference implementation" — even one nominally under a permissive license — accretes decision-making authority over the protocol to the maintainers of that codebase. When a reviewer asks "what does Marque actually do in edge case X," the answer must be **"read the spec and the test vectors,"** not **"read codebase Y."** Otherwise the protocol's canonical meaning drifts from the standards document to a maintainer's changelog, and the maintainer captures the ecosystem.

### 6.2 Competition happens at the feature layer, not at the protocol layer

Clients and providers are **expected to compete** on:

- AI triage and composition assistance (on-device, confidential-compute, or transparently opt-in cloud).
- Inbox UX, thread visualization, notification strategy.
- Search (client-side by default; provider-side encrypted search is an optional opt-in).
- Discovery (address books, directory services, contact sync).
- Storage pricing, retention windows, archive tooling.
- SLA, uptime, geographic availability, compliance certifications.
- Bundled services (QERDS, DLP, eDiscovery, SSO/SCIM, transactional API).

They are **not allowed to compete** by:

- Shipping protocol extensions that only their own implementation interprets (the "embrace-extend-extinguish" pattern). Per [`protocol/10-conformance.md §10.6`](../protocol/10-conformance.md), extensions are registered, documented, and either interoperable or absent — never implementation-specific.
- Withholding features behind a proprietary codepoint when the same feature has a Marque-standard codepoint available.
- Holding a user's archive, social graph, or history hostage via an inadequate export format. `.marquebox` export is MUST-complete; see §6.4 below.

### 6.3 Substitutability of providers

Every user MUST be able to leave a provider in bounded time with nothing lost. The normative rules:

- **`.marquebox` export is complete.** Per [`spec/protocol/03-architecture.md §3.10`](../protocol/03-architecture.md) and [`drafts/draft-tamim-marque-storage-00`](../../drafts/draft-tamim-marque-storage-00.md), `.marquebox` carries the user's cleartext archive, ProofEnvelopes at their retention-appropriate depth, the KEL, the capability records, and contact data. A provider that refuses to emit `.marquebox` on demand is non-conformant.
- **`.marquebox` import is idempotent.** A receiving provider MUST accept a `.marquebox` produced by any conformant provider and produce an identical user-visible archive state.
- **Migration SHOULD complete in 7 days, MUST complete in 30 days.** See [`spec/protocol/03-architecture.md §3.11`](../protocol/03-architecture.md). A provider that blocks departing users with longer migration windows is non-conformant.
- **DID document ownership is the user's, not the provider's.** Changing providers is an entry edit in the DID document, never a re-issuance. Providers who tie the DID to their domain (instead of offering portable-tier `marque.id` fallback) are contributing to ecosystem concentration, not to it.
- **Multi-endpoint publication is MUST for Attested+.** A user with Attested or Qualified tier mail MUST list ≥2 provider endpoints; see [`spec/protocol/03-architecture.md §3.7`](../protocol/03-architecture.md). No single provider can hold a user's mail hostage by going offline or refusing service.
- **No forced DID deactivation.** A provider sunsetting its service MUST NOT deactivate a user's DID; see the provider-shutdown procedure in [`spec/protocol/03-architecture.md §3.8`](../protocol/03-architecture.md) (90-day and 30-day notice, export windows).

### 6.4 Client substitutability

Every client MUST be substitutable without loss of functionality. The normative rules:

- **No client-specific protocol extensions at the interop surface.** A client that invents new block types, new envelope fields, or new `ai_instruction` semantics that only it interprets is non-conformant unless those extensions are registered per [`protocol/10-conformance.md §10.6`](../protocol/10-conformance.md).
- **Fallback rendering is MUST-complete.** Every MBS block either renders on every conformant client, or carries a `fallback` string that every conformant client MUST show. The envelope additionally carries `alt_plaintext` as a last-resort universal rendering path.
- **Device keys and archive keys are exportable.** A user moving from one client to another with the same provider MUST be able to re-authorize the new client without losing access to past mail. Cross-device archive-key-encrypted cache deltas make this routine; see [`draft-tamim-marque-storage-00`](../../drafts/draft-tamim-marque-storage-00.md).

### 6.5 Anti-monopoly charter rules

At the WG-charter level and at every major Marque milestone:

- **Minimum two unaffiliated implementations** at every maturity gate: initial draft adoption, WG Last Call, Standards-Track publication, each subsequent major version.
- **Minimum two unaffiliated providers** for any feature that requires provider-side state (Qualified-tier QTSP coordination, confidential-compute AI inbox, provider-side encrypted search).
- **Conformance tests are mandatory, not optional.** An implementation claiming Marque conformance without a public self-test result URL in its capability record has not made the claim credibly.
- **Spec-optional extensions at the interop surface are not permitted.** See the *Gatekeeper cloning* killer in [`context/05-risks.md`](./05-risks.md) — this is the rule that prevents Apple/Google repeating the RCS-without-E2EE-interop pattern against Marque.

### 6.6 What this does and does not guarantee

These guardrails **do not guarantee** a pluralistic ecosystem — they can be eroded by monopolistic commercial behavior even against a technically compliant substrate. What they do is make the erosion visible. A provider violating §6.3 is demonstrably non-conformant, not merely unpopular. A client violating §6.4 can be called out in public.

The alternative — trusting that a 2030s-era duopoly will voluntarily preserve portability — is what gave us the 2025 state of SMTP.

---

Continue to [**Migration**](./03-migration.md).
