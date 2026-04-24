# Identity

> A Marque identity is a keypair the user owns, not an address the provider rents. This section specifies how identities are named, resolved, rotated, recovered, and migrated.

**You will learn**

- Why email's identifier-as-locator is the core defect, and how `did:mail` resolves it.
- The three-tier key hierarchy (IRK / DSK / MLS ratchet) and the Key Event Log.
- DNS-rooted resolution with a portable-tier fallback at the neutral `marque.id` registry.
- The three-layer recovery model, new-user cold-start, multi-device onboarding, and disposable handles.
- The honest limits of the portability guarantee.

---

## 2.1 The core defect and its fix

Email's core defect is a single conflation: the identifier `alice@gmail.com` is also the routing address, and both are owned by the provider. A user cannot migrate away from a hostile or failing operator without changing the identifier itself. Every existing mitigation — mail forwarding, BYOD domains, alias services, Mastodon's `Move` activity — is a workaround around the conflation, not a solution to it.

Marque separates the two concerns. An identity is a long-lived **Decentralized Identifier (DID)** under the user's control. Mailbox providers are listed as **service endpoints** inside the DID document and MAY be rotated at will without changing the identifier. The identifier itself never changes — correspondents update nothing.

This is the phone-number-portability analogy, applied correctly. Under the US 1996 Telecommunications Act, E.164 numbers are pure identifiers resolved via the Number Portability Administration Center to current carriers. Under [Regulation (EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj) and its national equivalents, the same principle applies to telephony across the EU. Email never got this treatment. Marque fixes the oversight.

## 2.2 The `did:mail` method

Marque defines `did:mail`, a DID method designed as a thin profile over `did:web` with a mandatory KERI-style signed key rotation log.

An identifier takes the form:

```
did:mail:<domain>:<local-part>
```

where `<domain>` follows the [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035) preferred name syntax and `<local-part>` is a U-label per IDNA 2008, 1–32 characters of ALPHA / DIGIT / `-` / `_` / `.`, NFC-normalized, case-insensitive for equality, and rejected if the character set falls in Unicode TR 39 Highly-Restrictive or stricter homograph categories.

The full method is specified in [`draft-tamim-marque-did-mail`](../../drafts/draft-tamim-marque-did-mail-00.md). This section summarizes the essentials.

## 2.3 The three-tier key hierarchy

Separating "who is Alice," "which device is signing," and "which epoch is this message in" cleanly avoids the confusion Signal and OpenPGP both suffer from.

| Tier | Lifetime | Storage | Purpose |
|---|---|---|---|
| **Identity Root Key (IRK)** | Years | Hardware token or cold storage | Signs KEL events and DSK certificates. Never signs messages directly. |
| **Device Signing Key (DSK)** | Months | Per-device TPM / Secure Enclave / Android Keystore | Carries an IRK-issued certificate. Signs messages and authenticates TLS handshakes. |
| **Per-epoch MLS ratchet keys** | Ephemeral | Volatile | Forward-secret message encryption per [RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420). |

The IRK answers *"who is Alice?"* — it is what her correspondents trust. The DSK answers *"which device is signing?"* — compromising a phone does not compromise the identity; the DSK certificate is revoked by publishing a KEL rotation event. The ratchet keys answer *"which epoch is this message in?"* — they provide forward secrecy without tying to long-term identity.

## 2.4 Resolution

Resolvers attempt three paths in order; any path producing a valid DID document wins.

### Primary: DNS-rooted

For an identifier `did:mail:<domain>:<local-part>`, the resolver fetches:

```
https://<domain>/.well-known/mail-did/<percent-encoded-local-part>/did.json
```

per [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615). The response MUST be a DID document (JSON-LD) with content-type `application/did+ld+json`. The resolver MUST verify that the returned document's `id` field equals the requested DID. HTTP redirects to a different host MUST be treated as errors.

### Fallback: the portable-tier registry

A `did:mail:marque.id:<local-part>` identifier resolves against the **Internet Mail Identity Foundation (IMIF)** registry — a federated-neutral multi-operator directory (three to seven replicas operating under IMIF charter with Byzantine-fault-tolerant gossip).

The portable-tier is the **recommended default for new users**. It is the only class of handle whose portability guarantee holds regardless of any single provider's cooperation.

### Fallback: chain-anchored

If DNS and the portable-tier registry both fail, a resolver MAY consult an ENS record under `did.mail.<sld>.eth`. This is disaster recovery for nation-state-compelled DNS revocation, not a primary path. It is OPTIONAL; conformant implementations need not support it.

### Caching

DID documents MAY be cached per their `Cache-Control: max-age` header, or 300 seconds absent a header. Cached documents MUST be re-verified against the KEL before being used to authenticate a message more than one hour old.

## 2.5 The DID document

A conformant `did:mail` document contains the DID-Core fields plus a required `x-marque` object:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://marqueproto.org/context/did-mail/v1"
  ],
  "id": "did:mail:marque.id:alice",
  "verificationMethod": [ /* IRK, DSKs */ ],
  "authentication":     [ /* ... */ ],
  "keyAgreement":       [ /* ... */ ],
  "service":            [ /* MarqueMailbox endpoints */ ],
  "x-marque": {
    "kel":                      { "endpoint": "...", "mirrors": [ "..." ], "latestEvent": { /* ... */ } },
    "guardians":                [ /* ... */ ],
    "guardianThreshold":        3,
    "nullificationWindowHours": 72,
    "capabilities":             { /* ... */ }
  }
}
```

Service endpoints are listed with `priority` (lower preferred) and `retentionDays`. A document MUST list at least one `MarqueMailbox`; SHOULD list at least two from distinct operators. Attested- or Registered-capable identities MUST list at least two. Full schema: [`did-mail.schema.json`](../../schemas/did-mail.schema.json).

## 2.6 The Key Event Log

The KEL is an append-only signed log of IRK lifecycle events, modeled on [KERI](https://trustoverip.github.io/kswg-keri-specification/) (developed by the ToIP KERI Suite Working Group) with simplifications appropriate for mail.

Each line of the `kel.jsonl` is a self-signed JSON object:

```json
{
  "seq":       42,
  "prev_hash": "blake3:…",
  "event":     "rotation",
  "timestamp": "2026-04-23T14:32:07Z",
  "payload":   { "new_irk": "…" },
  "signatures": [ "…" ]
}
```

### Event types

| Event | Signed by | Payload |
|---|---|---|
| `inception` | Self | Initial IRK, initial guardians, initial nullification window |
| `rotation` | Outgoing IRK | New IRK (including PQ component) |
| `dsk-issuance` | IRK | DSK public key, device label, expiry |
| `dsk-revocation` | IRK | DSK id |
| `guardian-update` | IRK | New guardians and threshold |
| `recovery-initiated` | Guardians (threshold) | Candidate new IRK |
| `recovery-committed` | Guardians (threshold) | Confirmation after nullification window |
| `nullification` | Outgoing IRK | Reference to rotation or recovery event being nullified |
| `deactivate` | IRK | Terminal event; all-zero new-IRK |

### Verification

A resolver verifying a KEL MUST: (a) fetch the full log and compute per-event hashes; (b) check each `prev_hash` equals the hash of the previous event; (c) check `seq` increases monotonically by 1; (d) verify each event's signatures against the keyset authorized at that `seq`; (e) reject the log on any failure.

The IRK appearing in the most recent **committed** event is authoritative, subject to the nullification window.

### Mirrors

The DID document MAY list `x-marque.kel.mirrors[]` — additional URLs serving the byte-identical KEL. Resolvers MUST fetch from the primary and fall through to mirrors on timeout or 404. For portable-tier identities, the IMIF registry MUST mirror every KEL event.

Mirrors provide **availability**, not trust; events are self-signed by the IRK, so a malicious mirror cannot forge events — only refuse to serve current ones.

### Nullification window

`x-marque.nullificationWindowHours` defaults to 72. A rotation event published by a compromised device's IRK MAY be nullified within this window by the legitimate original IRK. A resolver seeing both MUST prefer the nullification.

## 2.7 Recovery

Three layers, applied in order:

### Primary: device-resident DSK

The DSK lives in hardware-backed storage (TPM, Secure Enclave, Android Keystore, YubiKey). Losing the device does not compromise the IRK; the DSK certificate is revoked via a `dsk-revocation` event.

### Social: guardians

The user's DID document lists guardians — other DIDs, or commitments to offline mechanisms — with `weight` values. Combined weight meeting or exceeding `guardianThreshold` (default 3) authorizes a `recovery-initiated` event. After the 72-hour nullification window, `recovery-committed` finalizes the rotation.

**Provider-as-guardian safeguard.** The home provider MAY act as one guardian by default. Provider-alone recovery is explicitly **not valid**, regardless of the provider's guardian `weight`. Clients MUST reject DID documents allowing provider-alone recovery. Every `recovery-initiated` event MUST produce a push notification to the user's primary device via channels outside Marque (OS push, SMS, email fallback) so a compromised provider cannot suppress it.

### Passphrase: 24-word mnemonic

A paper-backed commit MAY be enrolled as one guardian weight. Implementations SHOULD warn that paper loss or accidental disclosure is the dominant failure mode.

**Key-loss identity death is real.** A user who loses all devices, forgets the passphrase, and cannot reach guardians loses the identity permanently. This is structurally worse than Gmail's "call support." The mitigation is to make provider-as-one-guardian the default for normal users; power users who opt out see a visible "you have chosen to opt out of provider recovery" warning.

## 2.8 New-user cold-start

A fresh identity with no reputation and no contacts faces a structural barrier: recipient anti-spam policies demand bonds or PoW from strangers. Left unaddressed, this would be a Week-1 churn driver worse than anything email has.

**The normative cold-start mechanism:**

1. **Bootstrap quota.** On DID publication, the home provider issues a Privacy Pass VOPRF ([RFC 9578](https://datatracker.ietf.org/doc/html/rfc9578)) token pool sufficient for 100 unique recipient first-contacts in the first 30 days. Recipient providers SHOULD accept a valid bootstrap token in lieu of bond or PoW.
2. **Abuse detection.** If a bootstrap-token-using sender is spam-reported at a rate above 5% of delivered, the issuing provider MUST withdraw further tokens, and the provider's bootstrap trust on the federation-gossip channel decreases proportionally.
3. **Graduation.** A new user graduates to reputation-based when any of: (a) 10 recipients engage (reply or mark-as-not-spam); (b) 30 days elapse with spam rate below 1%; (c) an existing contact signs a cross-attestation.

This MUST be implemented by any provider claiming conformance.

## 2.9 Multi-device onboarding

A new device must generate a DSK, obtain an IRK-signed certificate, publish a `dsk-issuance` KEL event, upload KeyPackages, and fetch current MLS state for every active thread.

The last step is the non-obvious one. An existing authorized device MUST be able to transfer a per-thread **state snapshot** — an HPKE-encrypted bundle containing the MLS `GroupInfo` for each thread, the device-resident cleartext cache, and client-local state (linked DIDs, stars, filters). The transfer MAY happen over a local-network QR-code + PAKE handshake, or over the home provider's capability-`device-sync` endpoint if both devices are authenticated.

The existing device MUST present an explicit confirmation UI (*"Authorize this device?"*) and MUST record the event in the local audit log.

A new device with no reachable peer MAY use social recovery to bootstrap — with the same 72-hour nullification window as any IRK-level recovery.

## 2.10 The portability guarantee — and its limit

### What portability guarantees

A portable-tier identity (`did:mail:marque.id:alice`) survives any provider change. The handle never changes; no correspondent updates anything. The IRK remains under the user's control regardless of which mailbox providers are currently listed.

### What portability does not guarantee

A provider-hosted identity (`did:mail:acme.com:alice`) survives only while `acme.com` cooperates. If `acme.com` expires, is sold, or refuses to serve `.well-known/mail-did/alice/did.json`, the identity becomes unresolvable. The KEL is preserved cryptographically, but no resolver can find it.

**Clients MUST surface this distinction at signup.** The default for new consumer accounts SHOULD be portable-tier.

### The "bring your DID to Gmail" honesty note

A user cannot have `did:mail:gmail.com:alice` unless Google cooperates by serving `/.well-known/mail-did/alice/did.json`. They will not, at least not in Phase 1–2. The headline portability claim therefore holds between **cooperating providers**. This is exactly analogous to phone-number portability: E.164 numbers are portable among carriers that participate; a carrier that refuses is not forced to.

The mitigation is to make the portable-tier the default. Users who want DNS-native handles under their own domains can have them; users who want true portability take `marque.id`.

## 2.11 Disposable and linked handles

**Disposable handles** are short-lived DIDs issued by a provider for a specific purpose (sign up to a newsletter, receive a receipt, one-shot contact form). The DID document carries an `x-marque.disposable` object with `issued_by` (parent DID), `expires`, and `purpose`. Messages to a disposable handle are forwarded by the provider to the parent DID's primary inbox. Disposable handles MUST NOT be usable at Qualified tier.

**Linked identities** — multiple DIDs under one person's control — are permitted. A client MAY unify them into a single inbox view. The linking is **local to the client** and MUST NOT appear on the wire. Linked-identity export in `.marquebox` preserves the linking for device restoration.

## 2.12 Legal identity binding

A `did:mail` proves cryptographic continuity, not legal personhood. Three layers of identity apply:

| Layer | What it proves | Where it applies |
|---|---|---|
| `did:mail` | Same controller over time. | All tiers. |
| Verifiable Credential bound to the DID, issued by a Qualified TSP or government IdP | Legal identity: Alice Chen, born YYYY, resident of X. | Qualified tier (QERDS). |
| Web-of-trust cross-signatures from existing contacts | Social identity: "other people I know say this is Alice." | Informational; client-rendered. |

Clients MUST distinguish the layers in UI. A `SIGNED` message is cryptographically from a DID that claims to be Alice, not from Alice-the-legal-person. Spoofing a known contact by registering a lookalike DID (`did:mail:acme.com:a1ice`) remains possible; clients MUST flag inbound messages whose display name matches a known contact but whose DID does not (Unicode TR 39 similarity, Levenshtein distance on the local-part).

A Verifiable Credentials profile for sub-Qualified identity proofing is reserved for a companion document (`draft-tamim-marque-vc-identity`).

## 2.13 KEL inception and trust-on-first-use

The `inception` KEL event is self-signed by the IRK it bootstraps. The trust anchor at inception time is whoever controls the publication path — DNS for provider-hosted handles, the IMIF registry for portable-tier.

A resolver MUST treat the first resolution of a DID as **trust-on-first-use**: persist the inception hash locally, and fail loudly if a subsequent resolution returns a different inception hash for the same DID.

For portable-tier DIDs, the IMIF registry publishes each inception event to a Sigstore-style transparency log. A resolver MAY require transparency-log inclusion before accepting an inception event as authoritative. This closes the first-publish DNS-hijack attack against portable-tier identities. Provider-hosted DIDs remain materially weaker at inception time; clients SHOULD surface the asymmetry at signup.

---

Continue to [**Architecture**](./03-architecture.md).
