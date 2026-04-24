# SMTP interop — the bidirectional bridge

> Marque must coexist with SMTP for the entire 10–20 year transition. The bridge is how. It is bidirectional, opportunistic, labeled honestly at every chip — and tier-zero to operate correctly.

**You will learn**

- How inbound SMTP is discovered for sender keys and wrapped in MLS at ingress.
- How outbound Marque downgrades cleanly to SMTP with authenticated download links and Autocrypt.
- The round-trip upgrade state machine from `UNKNOWN` to `NATIVE`.
- DNS records for coexistence.
- UX chip labels and the state-to-glyph mapping.
- Bridge-specific spam and phishing mitigation.
- Why SMTP-bridged QERDS doesn't work today and what happens in the meantime.

---

## 9.1 Design inputs

Two patterns inform the bridge.

**Matrix bridges** to IRC / Telegram / WhatsApp use *portal rooms* (bridge auto-creates a room per remote channel) and *puppeting* (bridge impersonates the native user on the remote network). **E2EE across bridges is unavoidably lossy** — the bridge decrypts on one side and re-encrypts on the other.

**Delta Chat** chat-over-SMTP with Autocrypt demonstrates opportunistic encryption via in-band key exchange. Delta Chat v2 (July 2025) marks non-E2EE messages as a degraded state via an envelope icon.

The takeaway: **bridges weaken cryptographic guarantees, so the goal is to label honestly and upgrade opportunistically.**

## 9.2 Inbound SMTP → Marque

The bridge:

1. Receives SMTP at the recipient's MX.
2. Parses RFC 5322 and verifies SPF / DKIM / DMARC.
3. Looks up the recipient's DID.
4. Checks the sender's Marque capability via **four hybrid discovery methods**:
   - **DID resolver** (primary).
   - DNS TXT `_marque.legacy.com`.
   - `/.well-known/marque-did`.
   - Autocrypt header on prior messages.
5. Either:
   - **Wraps** the message in an MLS envelope encrypted to the recipient's device tree if a sender key is discoverable; renders `bridged · encrypted` (green up-arrow chip).
   - **Delivers as `PLAINTEXT-BRIDGED`** if no key is discoverable; renders `email` (gray envelope chip), with a clear "sender will not see replies encrypted" warning.

### Legacy header preservation and sanitization

Legacy headers are preserved in a `legacy-headers` field inside the new envelope, with the following **sanitization rule** to prevent internal-infrastructure leakage:

- **Always preserved verbatim:** `Message-ID`, `References`, `In-Reply-To`, `Date`, `Subject`, `From`, `To`, `Cc`, `List-Id`, `List-Unsubscribe`.
- **Received-chain truncation:** the `Received:` chain is truncated to the **outermost 2 hops** (the sending domain's outbound MTA and the bridge's inbound MTA). Internal MTA hops, internal hostnames, internal IP addresses, and internal routing headers (`X-Originating-IP`, `X-Forwarded-*`, `X-MS-Exchange-*`, etc.) MUST be stripped.
- **PII-sensitive vendor headers stripped** by default: `X-Mailer`, `User-Agent`, `X-Originating-IP`, any header matching `X-Google-*`, `X-Microsoft-*`, `X-Yahoo-*`.
- **Bridge operator** MAY configure a more permissive policy for internal-only deployments; the default is strict.

The client renders the message with a *"Bridged from SMTP"* chip per [`protocol/01 §5`](./01-terminology.md#5-bridge-and-interop-chips).

## 9.3 Outbound Marque → SMTP

The client:

1. Detects no Marque record for the recipient.
2. Downgrades to RFC 5322 generated from `alt_plaintext` and `alt_html`.
3. **Strips MLS wrapping and legal-proof metadata.**
4. Replaces E2EE-only attachments with **authenticated download links** (§9.3.1).
5. DKIM-signs with the provider domain, aligned per Yahoo / Google / Microsoft 2024–2025 rules.
6. Inserts an **Autocrypt header** (§9.3.2) unless the sender has opted out.

**Clients MUST warn before the first outbound downgrade** to a given recipient and cache consent for 90 days.

**Legal-proof-tier messages MUST NOT auto-downgrade.** They either fail closed or invoke a QERDS→legacy gateway. Note that the QERDS→SMTP gateway exists in production today only in Italy (PEC); senders elsewhere targeting a Qualified-tier send to a legacy recipient see `UNDELIVERABLE / NO_QERDS_GATEWAY` until non-PEC gateways exist.

### 9.3.1 Authenticated download links

When E2EE-only attachments are replaced with download links on outbound downgrade, the links MUST:

- Be **time-limited.** Default lifetime **7 days**; configurable down to 1 hour or up to 30 days by sender preference. **Links MUST NOT be open-ended.**
- Require **one-time-use tokens or per-recipient scoped tokens** — a shared link forwarded to a third party is not a valid access path. The token MUST bind to the recipient's email address (verified via a lightweight click-through-to-email-verification or a signed cookie at the gateway).
- Log each access (time, source IP, authentication outcome) to a delivery-audit record visible to the sender.
- Be **revocable** at any time before expiry by the sender via a `revoke-attachment-link` control frame.
- Not expose the original `content_id` in the URL — use a random token instead, so the CID does not leak to HTTP logs.

### 9.3.2 Autocrypt opt-out

The Autocrypt header advertises the sender's long-lived key. For users who do not want their correspondents' email infrastructure to retain a copy, the client MUST offer:

- **Per-recipient opt-out** — one-click to suppress Autocrypt for a given SMTP correspondent.
- **Global opt-out** — client-level setting suppressing Autocrypt for all SMTP downgrades.
- **Per-send explicit toggle** — compose-window checkbox.

Opt-out prevents future opportunistic upgrade from that correspondent; clients make the trade-off explicit in the opt-out UI.

### 9.3.3 Round-trip upgrade state machine

Autocrypt-based opportunistic upgrade follows a state machine per-correspondent:

| State | Meaning | Transition |
|---|---|---|
| `UNKNOWN` | No prior contact or no Autocrypt data | Send includes Autocrypt header. |
| `AVAILABLE` | Peer's Autocrypt key cached from a prior message | Send continues SMTP downgrade; recipient MAY upgrade on next round-trip. |
| `MUTUAL` | Both sides have published Autocrypt; encryption is possible | Send uses opportunistic encryption; still SMTP transport. |
| `NATIVE` | Peer has published a `did:mail` DID | Send natively over Marque, no bridge downgrade. |

Clients surface the state per-correspondent and MAY proactively prompt *"upgrade to encrypted"* when `MUTUAL` is achieved.

## 9.4 DNS records for coexistence

A domain publishes:

| Record | Purpose |
|---|---|
| `_marque._tcp.example.com SRV` | Service discovery. |
| `_marque.example.com TXT` | Capability and policy flags: `bridge=accept\|reject\|strict`, `legal-proof=qerds\|erds\|none`, `downgrade=allow\|warn\|deny`. |
| `_marque._keys.example.com TXT` | Optional. HPKP-analog key pins. |
| `MX` | Retained for SMTP bridge fallback. |

**Clients prefer SRV when both records are present.**

## 9.5 UX labeling

Borrows from HTTPS lock-icon evolution and Delta Chat's retreat from binary lock-vs-unlock toward neutral state icons. User-facing labels follow the normative [Terminology](./01-terminology.md).

| State | Chip | Glyph |
|---|---|---|
| Native Marque, `CASUAL` auth-mode | *(no chip)* | — |
| Native Marque, `SIGNED` auth-mode | **Signed** | pen / signature |
| Native Marque, `QUALIFIED` auth-mode | **Registered** | blue scales |
| Bridged-inbound, encrypted on ingress | **bridged · encrypted** | green up-arrow |
| Bridged-inbound, delivered plaintext | **email** | gray envelope |
| Outbound downgrade, first time per recipient | *(pre-send modal)* | amber caution |
| Bridged sender with failing DKIM / DMARC | **caution** | red triangle |

When a bridged correspondent starts publishing a DID, the thread chip animates **gray → green** with a one-line banner (*"alice@example.com now supports Marque natively"*), mirroring Autocrypt Level 1's opportunistic-upgrade pattern.

## 9.6 Bridge spam and phishing mitigation

Stricter than native:

- **New bridged senders** are rate-limited for 72 hours.
- **DMARC alignment** is enforced at `p=quarantine` minimum for any domain sending >100 messages/day across the bridge.
- Bridged senders **never earn the "Signed" or "Registered" chip** regardless of DKIM alignment — tier is a property of the original trust envelope, not the transport.
- **Brand-impersonation detection** pins recognized brands (PayPal, banks, government) to their Marque DIDs and warns when SMTP-bridged mail claims their identity.

## 9.7 Bridge as tier-zero operation

Bridges are load-bearing for Phase 1–4 adoption. Buggy bridges churn early adopters back to Gmail within a week. Any provider shipping a bridge must:

- Treat the bridge MX as a production service with SRE discipline, not a hobby project.
- Publish uptime SLOs in its capability record.
- Participate in federation gossip for bridge-specific abuse signals.
- Run bridge infrastructure on hardware distinct from the Marque-native provider infrastructure, to limit correlated failure.

---

Continue to [**Conformance**](./10-conformance.md).
