# Migration strategy

> Honest about timelines. HTTPS took ten years with Google's backing and Let's Encrypt's subsidy; IPv6 is at 28 years and half-done. Marque will take about as long, and this section says so.

**You will learn**

- How prior protocol migrations actually unfolded.
- The five Marque adoption phases and what each looks like.
- The top accelerants and the top killers.

---

## 1. Historical precedents

| Protocol                                                            | Time                  | Catalyst                                                                                                                     |
| ------------------------------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **HTTPS**                                                           | ~10 years (2014–2024) | [Let's Encrypt public beta Dec 2015, GA Apr 2016](https://letsencrypt.org/2015/11/12/public-beta-timing.html); Chrome _"Not Secure"_ labeling [Chrome 56 (Jan 2017) → Chrome 68 (Jul 2018)](https://blog.google/products/chrome/milestone-chrome-security-marking-http-not-secure/); [83 of top-100 HTTPS by Feb 2018](https://transparencyreport.google.com/https). |
| **IPv6**                                                            | ~27 years from RFC 2460 (Dec 1998); ~30 from RFC 1883 (1995) | [Google IPv6 adoption](https://www.google.com/intl/en/ipv6/) crossed 50% on 28 March 2026.                                                                                                                                                                                                        |
| **DNSSEC**                                                          | 30+ years below 5%    | Operator complexity offered no user-visible benefit.                                                                                                                                                                                                                                              |
| **DoH** ([RFC 8484](https://datatracker.ietf.org/doc/html/rfc8484)) | Rapid                 | Mozilla + Cloudflare default-on in US in 2019; Chrome 78 experiments Oct 2019.                                                                                                                                                                                                                    |
| **RCS vs SMS**                                                      | 10+ years             | Apple's November 2023 announcement; shipped in iOS 18 on 16 Sep 2024.                                                                                                                                                                                                                             |
| **Matrix in government**                                            | Verticalized          | [French Tchap](https://element.io/en/case-studies/tchap) ~400K MAU, [German BwMessenger](https://element.io/en/case-studies/bundeswehr) 100K+. No consumer spillover.                                                                                                                             |
| **Google + Yahoo bulk-sender mandate**                              | Fast                  | [Gmail/Yahoo Feb 2024](https://support.google.com/a/answer/14229414); [Microsoft enforcement 5 May 2025](https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730).     |

**Accelerants:** dominant-platform default-on, free compliance tooling, regulator-plus-competitor coordination, user-visible shaming signals.

**Killers:** backward-incompatibility forcing dual-stack pain, operator complexity without user benefit, single-vendor holdouts.

## 2. Phased rollout

### Phase 1 — Year 0–2: foundation

- Internet Draft.
- IETF working group with ETSI ESI and W3C DID liaisons.
- **At least two Apache-2.0 independent interoperable implementations** from unaffiliated groups (the IETF standards-track maturity bar). Example implementations are welcome and expected — none is privileged. Implementers are unrestrained in how they differentiate (AI triage, UX, search, discovery, storage strategy); every implementation passing the conformance suite is a peer.
- **SMTP bridges shipped day one** — bridge reliability is **tier-zero** because buggy bridges churn early adopters back to Gmail within a week.
- Early-adopter community among privacy forums, self-hosters, Delta Chat and Thunderbird and K-9 Mail developer overlap.

**Target adoption: 5 000–10 000 active handles.**

### Phase 2 — Year 2–3: first commercial providers

- 1–2 privacy-first incumbents (Proton, Fastmail, Tutanota, Mailbox.org, Posteo) adopt opt-in support.
- **Commit to minimum two unaffiliated live providers at every milestone** to defuse single-vendor-capture risk.
- First QTSP runs QERDS-mode Marque in Italy leveraging existing PEC infrastructure.
- SDKs in JavaScript, Python, Go, Rust, Swift, Kotlin.
- Transactional API at Resend price parity.

**Target adoption: 500 000–2 000 000 handles.**

### Phase 3 — Year 3–5: regulatory recognition and enterprise

- eIDAS 2.0 QERDS implementing acts through 2025–2027 list Marque's legal-proof tier on national trust lists in Italy, France, Germany.
- Fortune 500 compliance drivers replace DocuSign-plus-email stacks.
- AI-assistant support via confidential-compute inboxes.

**Target adoption: 10–50 million handles.**

### Phase 4 — Year 5–10: majority of new signups

- iOS Mail, Gmail app, Thunderbird, Outlook add native support, preferring Marque when both endpoints support it.
- **Default-on for new consumer accounts at ≥2 top-10 mailbox providers** is the Gmail/Yahoo moment for Marque — publish `downgrade=warn`, invert the network effect.

**Target adoption: 100–500 million handles.**

### Phase 5 — Year 10–20: equilibrium

- Major providers announce SMTP receive-only plans.
- Inbound bridge continues indefinitely for IoT and legacy-system senders.
- Possible regulatory mandate for QERDS-equivalent delivery in tax, legal, medical contexts.
- **SMTP is not extinct** — it becomes the Ethernet-over-copper of messaging. Ubiquitous for machine-to-machine, unfashionable for humans.

## 3. Top accelerant

**eIDAS 2.0 QERDS mandate.** The regulation is in force; implementing acts land through 2027; QERDS interoperability is mandated for qualified services.

Marque inherits a **regulatory moat and EU reference deployment** with no Gmail-equivalent resistance — **a five-year accelerant with no parallel in pure market-path adoption.**

## 4. Top killers and mitigations

| Killer                                                                                 | Mitigation                                                                                                       |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Bridge failure** isolating early adopters                                            | Treat bridges as tier-zero operations.                                                                           |
| **Single-provider capture** defeating the portability premise                          | Commit to multi-provider launch.                                                                                 |
| **Bridge-delivered spam** damaging new-protocol reputation                             | Stricter bridge policy ([`protocol/09 §9.6`](../protocol/09-interop.md#96-bridge-spam-and-phishing-mitigation)). |
| **Gatekeeper cloning** the design with incompatible semantics (RCS / iMessage pattern) | Mandatory conformance tests; refuse spec-optional extensions.                                                    |
| **Crypto-agility crises** requiring in-protocol rotation if MLS or `did:mail` breaks   | Ship ciphersuite negotiation in v1.0 like TLS 1.3.                                                               |

## 5. One number to set expectations

**Spec v1.0 target: 2028.** First commercial deployments 2028–2029. eIDAS QERDS listing 2029–2030. Phase-4 majority-of-new-signups tipping point realistically 2033–2036.

---

Continue to [**Naming**](./04-naming.md).
