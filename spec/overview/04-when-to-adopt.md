# When to adopt

> Honest about timelines. HTTPS took ten years with Google's backing and Let's Encrypt's subsidy; IPv6 is at 28 years and roughly half-done. Marque will take about as long, and this page says so.

**You will learn**

- The five adoption phases and what each looks like.
- Which users should adopt in each phase.
- The signals that will accelerate or kill the transition.

---

## Realistic trajectory

| Phase                             | Years | Target adoption                  | What it feels like                                                                                                                                            |
| --------------------------------- | ----- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1 — Foundation**                | 0–2   | 5 000–10 000 active handles      | Privacy forums, self-hosters, protocol enthusiasts. Rough edges. SMTP bridge is load-bearing.                                                                 |
| **2 — First providers**           | 2–3   | 500 000–2 000 000                | At least two unaffiliated privacy-first incumbents ship opt-in support. First QTSP runs QERDS-mode Marque in Italy. Developer SDKs.                           |
| **3 — Regulatory and enterprise** | 3–5   | 10–50 million                    | eIDAS QERDS listings in Italy, France, Germany. Fortune 500 compliance drivers replace DocuSign-plus-email stacks. AI-assistant inboxes.                      |
| **4 — Majority of new signups**   | 5–10  | 100–500 million                  | iOS Mail, Gmail app, Outlook add native support. Default-on for new accounts at two or more top-10 providers. The Gmail/Yahoo moment for Marque.              |
| **5 — Equilibrium**               | 10–20 | Majority of human correspondence | Major providers announce SMTP receive-only. SMTP becomes the Ethernet-over-copper of messaging — ubiquitous for machine-to-machine, unfashionable for humans. |

## Who should adopt, and when

**Now, if you are:**

- A **privacy researcher** or self-hoster who wants cryptographic correspondence and is willing to tolerate sharp edges.
- A **protocol enthusiast** who wants to shape the standard during Phase 1 while it is still malleable.
- A **developer** targeting the transactional-API beachhead: every password-reset email delivered over Marque with a cryptographic receipt is a recipient-education event, and there's a real business reason to want that.

Expect to maintain a parallel `@gmail.com` identity indefinitely for people who don't have Marque yet.

**Phase 2, if you are:**

- A **privacy-conscious professional** — journalist, lawyer, consultant — whose correspondence benefits from verified sender identity and where registered-grade proof of delivery is occasionally useful.
- A **small business** that sends contracts, invoices, or confidential documents and currently pays separately for DocuSign or certified mail.
- An **EU-focused service provider** who sees QERDS as a commercial differentiator coming.

**Phase 3, if you are:**

- A **Fortune 500 or EU-regulated enterprise** where compliance (eDiscovery, DLP, legal hold) is a line item. The commercial case here is replacing the DocuSign-plus-email compound stack with a native tier.
- A **regulated industry** (law, health, finance) with a qualified electronic signature mandate.
- A **government communications** operator handling citizen correspondence under eIDAS-scoped regulation.

**Phase 4+, if you are:**

- Everyone else. Wait until your OS mail app ships Marque natively and your contacts' providers are defaulting Marque-on. At that point adoption is one-tap.

## The three accelerants to watch

### eIDAS 2.0 QERDS implementing acts (2025–2027)

Regulation (EU) 2024/1183 is in force. Implementing acts land through 2027. QERDS interoperability is mandated for qualified services. Marque inherits a regulatory moat and EU reference deployment with no Gmail-equivalent resistance. **This single accelerant compresses roughly five years of organic adoption into two.** No parallel exists in pure market-path adoption.

### A headline AI-assistant breach via SMTP prompt injection

EchoLeak (CVE-2025-32711) was the first. There will be more. The industry's content-filter approach cannot scale, and sooner or later a corporate or political incident will force "protocol-level data-vs-instruction separation" onto every CISO's Q4 plan. Marque has that separation by design; SMTP cannot.

### Apple, Android, or a major client shipping Marque

When iOS Mail or the Gmail app ships native Marque and prefers Marque when both endpoints support it, adoption inverts. This is the Phase 4 trigger. Every history suggests it will happen 5–8 years after the first privacy-first provider ships (Phase 2 entry).

## The three killers

### A broken bridge in the first two years

If early-adopter messages to `@gmail.com` correspondents go missing, or legacy-SMTP replies don't reach Marque inboxes, early adopters churn back to Gmail within a week. **Bridge reliability is tier-zero** during Phase 1. Any provider shipping a bridge must run it like a bank, not like a hobby project.

### A gatekeeper cloning the design with incompatible semantics

Apple's RCS adoption without E2EE interop is the template: an incompatible "Marque-like" protocol from Google or Apple that uses the name and vocabulary without the portability guarantees. Mitigation: mandatory conformance tests, no spec-optional extensions at the interop surface, and the multi-provider minimum-two rule enforced at charter level.

### A cryptographic crisis requiring in-protocol rotation

If MLS is broken, or `did:mail` is compromised, or Argon2 is beaten, Marque needs ciphersuite negotiation that works. TLS 1.3 solved this; Marque v1.0 ships the same mechanism so a rotation is a codepoint allocation, not a re-specification.

## One number to set expectations

**Spec v1.0 target: 2028.** First commercial deployments 2028–2029. eIDAS QERDS listing 2029–2030. Phase-4 majority-of-new-signups tipping point realistically 2033–2036.

Honesty about timelines is baked into the proposal, because it has to be. A successor to a forty-year-old protocol doesn't ship in eighteen months.

---

You've now read the product overview. The rest of the specification is technical.

- Implementers should continue to [**Overview / The first-slice surface**](./05-mvp.md) for the twelve-week implementation-sequencing plan (first slice of the full surface, not a product cut), then on to [**Protocol / Terminology**](../protocol/01-terminology.md).
- Reviewers and strategists may find [**Context / Migration**](../context/03-migration.md) and [**Context / Risks**](../context/05-risks.md) most useful next.
