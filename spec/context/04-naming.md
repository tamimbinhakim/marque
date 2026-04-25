# Naming

> Why _Marque_, what it means, and why the tagline is _"A protocol for human correspondence in the age of AI."_

**You will learn**

- The naming precedents — continuity names, break names, replacement names.
- Why common candidates were rejected on trademark or tone grounds.
- The reasoning behind _Marque_ as the protocol name and _message_ as the unit.
- The tagline's three layers of meaning.

---

## 1. Precedent

Naming precedents split cleanly:

- **Continuity names** (HTTP → HTTP/2 → HTTP/3; IPv4 → IPv6) signal compatibility even when protocols differ materially.
- **Break names** (IRC → Matrix; Jabber → XMPP) signal new architecture or resolve trademark / legitimacy problems.
- **Replacement names** (SMS → RCS) combine with gatekeeper capitulation.

Marque's situation structurally resembles Jabber → XMPP — a neutral name that can live at IETF / ETSI without being owned by a single vendor.

## 2. What was rejected and why

| Candidate          | Conflict                                                                                                                                              |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Postmark**       | Wildbit / ActiveCampaign's transactional email product since 2010. Direct conflict with revenue stream #3 ([`context/02`](./02-commercial-model.md)). |
| **Courier**        | `courier.com` is a developer notification API (explicit competitor); Courier-MTA OSS.                                                                 |
| **Missive**        | `missiveapp.com` is a registered product in the messaging space. Trademark risk.                                                                      |
| **Parcel**         | Parcel.js is a mindshare-heavy bundler in our target developer audience.                                                                              |
| **Pigeon**         | Multiple conflicts; skews toybox for eIDAS gravitas.                                                                                                  |
| **Envelope**       | Generic and crowded.                                                                                                                                  |
| **Letterbox**      | Multiple prior uses (LETTERBOXX newsletter software, FU Berlin iOS app).                                                                              |
| **Mail/2, SMTP/2** | Lies about backward compatibility in the shape of the IPv6 naming mistake.                                                                            |

## 3. Why _Marque_

_Marque_ is French for _mark_ / _stamp_ / _brand_. It is:

- **Short** and internationally readable.
- **Evocative of authenticity** — _a mark of_, _a marque of quality_, _trademark_ — central to the Registered-tier positioning.
- **Empirically low in trademark conflict** in the messaging class.
- **Pairs cleanly** with UI language: _marked delivery_, _marked sender_, _mark received_.

The name pair tests well: the protocol is **Marque**; a message on Marque is a **message** (not a "marque"). This follows the Signal / WhatsApp pattern — the brand names the platform, not the unit.

Why not coin a unit noun? Because English has an unavoidable homophone collision between _marque_ and the email flag-verb _mark as …_ (_mark as read_, _mark as unread_, _mark as spam_). Keeping _Marque_ strictly as a platform noun and using plain English verbs for actions eliminates the collision for screen-reader and voice-assistant users. This is specified normatively in [`protocol/01 §2`](../protocol/01-terminology.md#2-naming-principle).

## 4. The tagline

**"A protocol for human correspondence in the age of AI."**

Three layers:

1. **Protocol** — Marque is a wire-format specification on the IETF Standards Track, not a product, a brand, or a vendor. The word signals neutrality and the expected destination (an RFC, not a SaaS).
2. **Human correspondence** — the wedge. Marque is for people writing to people, not for AI-to-AI inbox flooding, AI-mediated platform conversations, or LLM-generated phishing. Verified identity, MLS encryption, and tiered legal proof all serve that one definition.
3. **In the age of AI** — names the historical moment that makes the patch path no longer credible: zero-click prompt-injection ([CVE-2025-32711][echoleak]), LLM-generated phishing past the content-filter ceiling, and AI-assistant inboxes that cannot structurally distinguish instructions from data. SMTP has no answer to any of these.

Localizes cleanly:

- French: _Un protocole pour la correspondance humaine à l'ère de l'IA._
- Italian: _Un protocollo per la corrispondenza umana nell'era dell'IA._
- German: _Ein Protokoll für menschliche Korrespondenz im Zeitalter der KI._
- Spanish: _Un protocolo para la correspondencia humana en la era de la IA._

[echoleak]: https://nvd.nist.gov/vuln/detail/CVE-2025-32711

## 5. Protecting the mark

All internal documents, the Internet Drafts, and the first interoperable implementations adopt **Marque** from day one. "Postmark" as an earlier working codename is retired.

Before finalizing the trademark, the editor team will run:

- USPTO TESS search.
- EUIPO eSearch.
- WIPO Global Brand Database.
- Classes 9, 38, 41, and 42.

Professional trademark opinion will be obtained in US, EU, UK, and Switzerland. The mark is to be held in trust by the editor team pending transfer to the Internet Mail Identity Foundation chartered under IETF / ISOC.

---

Continue to [**Risks and open questions**](./05-risks.md).
