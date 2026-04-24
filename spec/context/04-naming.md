# Naming

> Why _Marque_, what it means, and why the tagline is _"Registered communication, by default."_

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

**"Registered communication, by default."**

Three layers:

1. **Registered** as legally-proof — direct QERDS / PEC nod.
2. **Registered** as DID-registered identity — portable, sovereign.
3. **By default** — E2EE and cryptographic proof are baseline, not premium. The anti-Skiff positioning.

Localizes cleanly:

- French: _Communication enregistrée par défaut._
- Italian: _Comunicazione raccomandata per impostazione predefinita._
- German: _Eingeschriebene Kommunikation, standardmäßig._
- Spanish: _Comunicación certificada, por defecto._

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
