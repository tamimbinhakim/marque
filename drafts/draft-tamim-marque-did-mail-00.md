---
title: "The did:mail Decentralized Identifier Method"
abbrev: "did-mail"
docname: draft-tamim-marque-did-mail-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - did
  - identity
  - portability
  - email
  - keri
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC8615:
  RFC8032:
  RFC7748:
  W3C-DID:
    title: "Decentralized Identifiers (DIDs) v1.0"
    target: "https://www.w3.org/TR/did-core/"
    author:
      - org: "W3C"
    date: 2022-07

informative:
  RFC5322:
  RFC6376:       # DKIM
  RFC9000:
  RFC9420:
  DID-WEB:
    title: "The did:web Method Specification (W3C CCG Draft; not a W3C Recommendation)"
    target: "https://w3c-ccg.github.io/did-method-web/"
    author:
      - org: "W3C Credentials Community Group"
  DID-PLC:
    title: "The did:plc Method Specification (not W3C-standardized; maintained by Bluesky and the did-method-plc maintainers)"
    target: "https://github.com/did-method-plc/did-method-plc"
    author:
      - org: "did-method-plc maintainers"
  KERI:
    title: "Key Event Receipt Infrastructure (KERI) — ToIP KERI Suite Working Group (KSWG)"
    target: "https://trustoverip.github.io/kswg-keri-specification/"
    author:
      - name: Samuel Smith
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-FOUNDING:
    title: "Marque — a protocol proposal to replace email (founding specification)"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies `did:mail`, a Decentralized Identifier (DID)
method {{W3C-DID}} for mail-capable identities. The method is a profile
of `did:web` {{DID-WEB}} augmented with a mandatory KERI-style {{KERI}}
signed Key Event Log (KEL) for key rotation, a three-tier key hierarchy
separating identity from device signing from per-epoch ratchet keys, and
a set of service-endpoint types ("mailbox providers") that a `did:mail`
identity uses to receive messages under the Marque protocol
{{MARQUE-CORE}}.

The design enables users to migrate between mailbox providers without
losing their address, analogously to how E.164 number portability allows
users to change mobile carriers. Resolution is DNS-rooted by default so
that no new infrastructure is required to adopt the method; the founding
specification {{MARQUE-FOUNDING}} describes optional federated and
chain-anchored fallbacks for users who require DNS-independence.

--- middle

# Introduction

The dominant internet identifier for asynchronous human correspondence,
`<local-part>@<domain>` per {{RFC5322}}, conflates identifier and locator:
the identifier is also the routing address, and both are controlled by the
domain operator. A user cannot migrate away from a hostile or failing
operator without changing the identifier itself. Every existing mitigation
(mail forwarding, custom-domain hosting, alias services, Mastodon-style
`Move` activities) is a workaround around this conflation rather than a
solution to it.

This document specifies `did:mail`, a DID method that decouples the two.
The identifier is a long-lived cryptographic entity under the user's
control; mailbox providers are listed as service endpoints in the DID
document and MAY be rotated at will without changing the identifier.

The method deliberately avoids inventing infrastructure where DNS suffices:
default resolution walks an HTTPS path at the user's chosen domain,
identical in pattern to `did:web`. Where DNS is insufficient — key
rotation, cryptographic recovery, cross-provider portability — the method
adds a minimal, well-specified overlay (the KEL).

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

DID:
: A Decentralized Identifier as defined by {{W3C-DID}}.

IRK:
: Identity Root Key. A long-lived keypair, held in hardware or cold
  storage, that signs rotation events and Device Signing Key certificates.

DSK:
: Device Signing Key. A medium-lived keypair, one per client device, that
  carries an IRK-signed certificate and performs per-message signing.

KEL:
: Key Event Log. An append-only signed log of IRK rotation events,
  modeled on {{KERI}}.

Mailbox provider:
: An operator of a Marque service endpoint of type `MarqueMailbox`
  {{MARQUE-CORE}}. A DID document MAY list one or more mailbox providers;
  senders deposit envelopes at any listed provider.

Nullification window:
: A configurable interval, nominally 72 hours, during which a rotation
  event MAY be nullified by the rotating IRK itself, protecting against
  hostile provider-driven rotations.

# Method name

The DID method name of this specification is `mail`. A `did:mail`
identifier SHALL begin with the literal prefix `did:mail:`.

# Method-specific identifier

The method-specific identifier follows the form:

~~~
method-specific-id = domain ":" local-part
domain             = <preferred name syntax, RFC 1035>
local-part         = 1*32 ( ALPHA / DIGIT / "-" / "_" / "." )
~~~

Additional constraints:

* The `local-part` MUST be treated as a U-label per IDNA 2008 and
  compared in NFC-normalized form.
* The `local-part` MUST NOT begin or end with a `.`.
* The `local-part` MUST NOT contain two consecutive `.` characters.
* Implementations MUST reject identifiers whose `local-part` falls in the
  character set known to enable homograph attacks (Unicode TR 39 Highly
  Restrictive or stricter).
* Case in the `local-part` is insignificant for equality comparison and
  MUST be canonicalized to lowercase before hashing or signing.

Example:

~~~
did:mail:example.com:alice
~~~

# Resolution

Resolution produces a DID document conforming to the schema in
{{did-document}}. Three resolution mechanisms are defined; resolvers MUST
attempt them in the order given.

## Primary: DNS-rooted

For an identifier `did:mail:<domain>:<local-part>`, the resolver fetches:

~~~
https://<domain>/.well-known/mail-did/<percent-encoded-local-part>/did.json
~~~

per {{RFC8615}}. The response MUST be a DID document (JSON-LD) and MUST
have content-type `application/did+ld+json`. The resolver MUST verify
that the `id` field of the returned document equals the requested DID.

The resolver MUST NOT follow HTTP redirects to a different host than the
one named in the DID. A redirect is an error.

## Fallback: federated directory

If DNS resolution fails (NXDOMAIN, connection refused, 404), the resolver
MAY consult the federated directory specified in a future companion draft.
This fallback is OPTIONAL and non-conformant implementations need not
implement it.

## Fallback: chain-anchored

If both primary and federated resolution fail, a resolver MAY consult an
ENS (Ethereum Name Service) record under the `did.mail.<sld>.eth`
delegation pattern. This fallback is OPTIONAL and is intended as disaster
recovery in the case of nation-state-compelled DNS revocation.

## Caching

DID documents MAY be cached for up to the time indicated by their
`Cache-Control: max-age` HTTP header, or for 300 seconds if no header is
present. Cached documents MUST be re-verified against the KEL (see
{{key-event-log}}) before being used to authenticate a message more than
one hour old.

# DID document structure {#did-document}

A conformant `did:mail` DID document contains the following members in
addition to those required by {{W3C-DID}}:

~~~json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://marqueproto.org/context/did-mail/v1"
  ],
  "id":                "did:mail:example.com:alice",
  "verificationMethod": [ ... ],
  "authentication":     [ ... ],
  "keyAgreement":       [ ... ],
  "service":            [ ... ],
  "x-marque": {
    "kel":                       { ... },
    "guardians":                 [ ... ],
    "nullificationWindowHours":  72,
    "capabilities":              { ... }
  }
}
~~~

The `x-marque` object is REQUIRED for `did:mail` documents.

## Verification methods

Every DID document MUST contain at least one `verificationMethod` of
relationship `authentication`. The DID Controller MUST publish exactly
one IRK at any time and MAY publish additional DSK entries per device.

The IRK verification method MUST have the form:

~~~json
{
  "id":               "did:mail:example.com:alice#irk-<date>",
  "type":             "Ed25519VerificationKey2020",
  "controller":       "did:mail:example.com:alice",
  "publicKeyMultibase": "z6Mk..."
}
~~~

Each DSK verification method MUST additionally carry an `x-marque` object
specifying:

~~~json
"x-marque": {
  "device":    "phone|laptop|tablet|hardware-token",
  "issuedBy":  "<IRK verification method id>",
  "issuedAt":  "<RFC 3339 UTC timestamp>",
  "expires":   "<RFC 3339 UTC timestamp>",
  "pqHybrid":  {
    "algorithm": "ML-DSA-65",
    "publicKey": "..."
  }
}
~~~

`expires` MUST NOT be more than 365 days after `issuedAt`.

## Service endpoints

A `did:mail` document MUST list at least one `service` entry of type
`MarqueMailbox`:

~~~json
{
  "id":              "did:mail:example.com:alice#inbox-primary",
  "type":            "MarqueMailbox",
  "serviceEndpoint": "https://mail.example.com/marque/v1",
  "priority":        10,
  "retentionDays":   90
}
~~~

`priority` is a non-negative integer; lower values are preferred. A
sender SHOULD deposit to the two lowest-priority endpoints and MAY
additionally deposit to higher-priority ones if policy requires.

A document MAY list `LegacySMTPBridge` service endpoints for
interoperation with SMTP correspondents during the transition period.

## Key Event Log reference

The `x-marque.kel` object REQUIRES:

~~~json
"kel": {
  "endpoint":    "https://example.com/.well-known/mail-did/alice/kel.jsonl",
  "latestEvent": {
    "seq":       <non-negative integer>,
    "hash":      "blake3:<64 hex>",
    "timestamp": "<RFC 3339 UTC>"
  }
}
~~~

Resolvers MUST fetch the KEL and verify its integrity per
{{key-event-log}} before treating the DID document as authoritative for
cryptographic operations.

## Guardians

The `x-marque.guardians` array lists social-recovery guardians. Each
entry is either another DID or a commit to an out-of-band recovery
mechanism:

~~~json
[
  { "label": "spouse", "did": "did:mail:example.com:bob",     "weight": 1 },
  { "label": "lawyer", "did": "did:mail:firm.law:dan",        "weight": 1 },
  { "label": "phrase", "type": "PassphraseCommit", "commit": "blake3:..." }
]
~~~

Recovery requires guardians whose combined `weight` meets or exceeds the
DID document's configured threshold (nominally 3). Recovery is subject to
the nullification window.

## Nullification window

`x-marque.nullificationWindowHours` SHALL be a non-negative integer;
the default and RECOMMENDED value is 72. A rotation event published by a
device whose DSK has been compromised and replaced by a malicious actor
MAY be nullified by the legitimate IRK within this window, invalidating
the intervening events.

# Key hierarchy

Three tiers:

## Identity Root Key (IRK)

Long-lived (years). Held in hardware-backed storage or cold storage. Signs
only:

* KEL events.
* DSK certificates.
* Irregular operations such as guardian-list updates.

The IRK MUST NOT be used directly to sign messages.

## Device Signing Key (DSK)

Medium-lived (months). One per device. Stored in the device's secure
enclave (TPM, Apple Secure Enclave, Android Keystore, hardware token).
Signs messages and authenticates TLS handshakes.

A DSK certificate chains to the IRK. A DSK's validity is bounded by its
certificate's `issuedAt` and `expires`.

## Per-epoch MLS ratchet keys

Ephemeral. Derived by {{RFC9420}} TreeKEM per epoch. Not visible in the
DID document.

# Key Event Log (KEL) {#key-event-log}

The KEL is an append-only log of signed events describing the lifecycle
of the IRK and DSKs. It is modeled on {{KERI}} with simplifications
appropriate for the mail use case.

## Format

The KEL is JSON Lines (`application/jsonl`). Each line is a self-signed
JSON object:

~~~json
{ "seq":           <non-negative integer, monotonically increasing>,
  "prev_hash":     "blake3:<64 hex>",      // hash of previous event, or null for inception
  "event":         "<event type>",
  "timestamp":     "<RFC 3339 UTC>",
  "payload":       { ... },                // event-specific
  "signatures":    [ ... ]
}
~~~

## Event types

| Event type              | Signed by | Payload |
|---|---|---|
| `inception`             | Self (bootstrap) | Initial IRK public key, initial guardians, initial nullification window |
| `rotation`              | Outgoing IRK | New IRK public key, optional PQ component |
| `dsk-issuance`          | IRK       | DSK public key, device label, expires |
| `dsk-revocation`        | IRK       | DSK id being revoked |
| `guardian-update`       | IRK       | New guardians list |
| `recovery-initiated`    | Guardians (threshold) | Proposed new IRK public key |
| `recovery-committed`    | Guardians (threshold) | Confirmation after nullification window expires |

## Verification rules

A resolver verifying a KEL MUST:

1. Fetch the full log and compute per-event hashes.
2. Check that each `prev_hash` equals the hash of the previous event.
3. Check that `seq` increases monotonically by 1.
4. Verify each event's `signatures` against the keyset authorized at that
   `seq` per the preceding events.
5. Reject the log if any check fails.

A resolver treats the IRK appearing in the most recent committed event as
authoritative, subject to the nullification window.

## Nullification

If a `rotation` event appears, the IRK published *before* that event MAY
publish a `nullification` event within the nullification window. A
resolver seeing both MUST prefer the nullification and treat the
originally published keyset as still authoritative.

# Operations

The four standard DID operations are defined as follows.

## Create

1. Generate an IRK in hardware-backed storage.
2. Generate at least one DSK; issue a DSK certificate signed by the IRK.
3. Choose one or more mailbox providers; upload KeyPackages and register
   service endpoints.
4. Publish an `inception` KEL event.
5. Publish the DID document at
   `https://<domain>/.well-known/mail-did/<local-part>/did.json` with a
   content-type of `application/did+ld+json`.

## Read

Resolution is specified in {{resolution}}.

## Update

All updates are expressed as KEL events signed by the IRK. Non-KEL
updates (service endpoints, priorities, retention days) are expressed by
re-publishing the DID document; the DID document's `updated` field MUST
increase monotonically and its current state MUST match the guarantees of
the most recent KEL event.

## Deactivate

An IRK publishes a `rotation` event whose new IRK public key is the
all-zero bytes. The DID is thereby irrevocably deactivated; resolvers
MUST reject all subsequent events under the DID.

# Recovery

Three layers, applied in order:

## Primary

Device-resident DSK, backed by device-local secure-enclave hardware.

## Social

Guardians per {{guardians}} initiate recovery by co-signing a
`recovery-initiated` event. The event names a candidate new IRK public
key. The nullification window begins. If not nullified within the window,
a `recovery-committed` event finalizes the rotation.

During the nullification window, the DID document MUST carry a banner
field indicating recovery in progress:

~~~json
"x-marque": {
  "recoveryInProgress": {
    "initiatedAt":    "<RFC 3339 UTC>",
    "nullifiableUntil": "<RFC 3339 UTC>",
    "candidateIRK":   "z6Mk..."
  }
}
~~~

The DID's home provider SHOULD send a push notification to the primary
device on `recovery-initiated`.

## Passphrase

A 24-word mnemonic bound to a paper-backed commit may be enrolled as one
guardian weight. Implementations SHOULD warn users that paper loss or
accidental disclosure is the dominant failure mode.

# Migration

## Between Marque providers

1. User creates a new `MarqueMailbox` service endpoint entry in the DID
   document pointing to the new provider.
2. User uploads KeyPackages to the new provider.
3. Senders deposit envelopes to both providers during a transition
   window (default 30 days).
4. User removes the old provider entry from the DID document.

No message addressing changes; no correspondents are notified; the
identifier is stable across the migration.

## From a legacy SMTP address

A user with `alice@example.com` may prove control of the address by
responding to a DKIM-signed challenge {{RFC6376}} and publish the result
as a `LegacySMTPBridge` service endpoint in a newly created `did:mail`
document. Inbound mail continues to flow via SMTP; outbound to other
`did:mail` recipients uses Marque natively.

# Security Considerations

## IRK compromise

If the IRK is compromised, the attacker can issue arbitrary DSKs and
forge any DSK-level signature thereafter. Mitigations:

* Hardware-backed IRK storage.
* Social recovery path (the legitimate guardians can rotate to a new IRK;
  the attacker's rotation is subject to the nullification window, during
  which the true user can observe the push notification and nullify).
* Audit monitoring of the KEL — a user's home provider SHOULD notify the
  user of any new KEL event.

## DSK compromise

Limited-scope: an attacker can sign messages only for the compromised
device's lifetime, bounded by `expires` (nominally six months). A DSK
revocation via the KEL terminates authority immediately.

## Hostile provider rotation

A hostile home provider could publish a `rotation` KEL event under a
compromised IRK. The nullification window limits damage. Any other
guardian or the user's backup device MAY publish a nullification within
72 hours.

## Key ceremony

No normative key-ceremony procedure is specified; implementations SHOULD
use a Trusted Setup ceremony with airgapped IRK generation for
high-assurance deployments.

## Downgrade

An attacker controlling the DNS path to `/.well-known/mail-did/` MAY
serve an old DID document with stale keys. The KEL's append-only
property detects this: a resolver MUST reject a DID document whose
advertised `latestEvent` is older than the KEL itself.

## Post-quantum readiness

The IRK SHOULD carry a post-quantum hybrid component (ML-DSA-65 in the
default profile). Classical-only IRKs remain valid but implementations
SHOULD warn users that their identity is not post-quantum safe.

# Privacy Considerations

## Resolution leakage

Every DID resolution is a DNS query observable by network infrastructure
and an HTTPS request observable by the serving host. Resolvers SHOULD
support DNS-over-HTTPS or DNS-over-TLS. Providers MAY co-host the
`.well-known` path on a CDN to obscure individual resolution fanout, at
the cost of sharing resolution metadata with the CDN operator.

## Enumeration

The `.well-known/mail-did/<local-part>` path is enumerable under a chosen
domain. Operators concerned about user-directory enumeration SHOULD
return a 404 for unresolved local-parts indistinguishable from the 404
for non-existent ones, rate-limit lookups, and offer CAPTCHA challenges
for suspicious query patterns.

## Pseudonymity

A natural person MAY control multiple `did:mail` identifiers under
different domains for pseudonymous correspondence. Nothing in this
method binds real-world identity to the DID absent an external credential
presented via Verifiable Credentials.

## Guardian disclosure

The `guardians` list reveals relationships that MAY be sensitive. A DID
Controller MAY publish only commitments (BLAKE3 hashes) of guardian DIDs
instead of the DIDs themselves; recovery then requires the candidate
guardian to present a pre-image alongside their signature.

# IANA Considerations

This document requests that `mail` be added to the W3C DID Method
Registry with this document as its reference. No IANA registries are
established or modified by this document.

# Acknowledgments
{:numbered="false"}

This method draws on {{DID-WEB}} for its DNS-rooted resolution,
{{DID-PLC}} for its account-migration semantics and nullification-window
design, {{KERI}} for its Key Event Log approach, and Vitalik Buterin's
social-recovery writings for the guardian model. The author thanks the
Bluesky PDS migration community and David Buchanan specifically for
public documentation of adversarial PDS migration cases.

--- back

# Worked example
{:numbered="false"}

A complete example DID document appears in the founding specification
{{MARQUE-FOUNDING}} under `examples/did-document.json`.
