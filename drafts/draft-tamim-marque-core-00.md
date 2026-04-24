---
title: "Marque Core Protocol"
abbrev: "marque-core"
docname: draft-tamim-marque-core-00
category: std
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - messaging
  - end-to-end encryption
  - mls
  - did
  - post-quantum
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:
  RFC3161:
  RFC7748:
  RFC8032:
  RFC8126:
  RFC9000:
  RFC9106:
  RFC9420:
  RFC9591:

informative:
  RFC5322:
  RFC8058:
  RFC8615:
  FIPS204:
    title: "Module-Lattice-Based Digital Signature Standard"
    target: "https://csrc.nist.gov/pubs/fips/204/final"
    author:
      - org: "NIST"
    date: 2024-08
  FIPS205:
    title: "Stateless Hash-Based Digital Signature Standard"
    target: "https://csrc.nist.gov/pubs/fips/205/final"
    author:
      - org: "NIST"
    date: 2024-08
  I-D.ietf-tls-ecdhe-mlkem:
  I-D.ietf-mimi-protocol:
  BLAKE3:
    title: "BLAKE3: one function, fast everywhere"
    target: "https://github.com/BLAKE3-team/BLAKE3-specs/blob/master/blake3.pdf"
    author:
      - name: Jack O'Connor
      - name: Jean-Philippe Aumasson
      - name: Samuel Neves
      - name: Zooko Wilcox-O'Hearn
    date: 2020
  OPENTIMESTAMPS:
    title: "OpenTimestamps: Scalable, Trust-Minimized, Distributed Timestamping with Bitcoin"
    target: "https://opentimestamps.org"
  SEALED-SENDER:
    title: "Technology preview: Sealed sender for Signal"
    target: "https://signal.org/blog/sealed-sender/"
    date: 2018-10
  MARQUE-FOUNDING:
    title: "Marque — a protocol proposal to replace email (founding specification)"
    target: "https://marqueproto.org/spec/"

--- abstract

This document specifies the core wire protocol for Marque, a cryptographically
signed, federated-and-peer-to-peer messaging protocol proposed as a successor
to SMTP {{RFC5322}} and IMAP.

Marque defines: a canonical CBOR envelope; a mandatory profile of Messaging
Layer Security {{RFC9420}} for asynchronous group-addressed mail; a QUIC
{{RFC9000}} transport binding with Decentralized Identifier (DID)-derived
authentication; a three-tier metadata-privacy model; and the registries
required for algorithm, envelope-field, and block-type codepoints.

This document is the core protocol. Companion documents (cited in the
*Informative References*) specify the `did:mail` identity method, the Marque
Block Spec (MBS) content layer, the tiered non-repudiation envelope, and the
bidirectional SMTP bridge used during the 10–20 year transition period.

--- middle

# Introduction

The Simple Mail Transfer Protocol {{RFC5322}} has served as the internet's
primary asynchronous messaging substrate since 1982. Over four decades, a
succession of bolt-on mechanisms (SPF, DKIM, DMARC, ARC, BIMI, MTA-STS,
TLS-RPT, one-click List-Unsubscribe {{RFC8058}}, Autocrypt, S/MIME, OpenPGP)
have attempted to retrofit sender authentication, end-to-end confidentiality,
and abuse resistance onto a protocol that has none by design. Each bolt-on has
demonstrated partial success and structural limitation.

Three independent pressures make incremental patching inadequate for the next
decade:

1. The concentration of reputation gating in two large operators has made
   independent mail-transfer-agent operation infeasible for most legitimate
   senders, even those correctly implementing every published extension.
2. Large-language-model text generation has collapsed the cost of crafting
   novel, personalized, context-aware abuse content, driving the
   content-classification approach to its asymptotic ceiling.
3. The widening deployment of automated mail-reading assistants has
   introduced prompt-injection as a new weaponized vulnerability class in a
   format that provides no structural separation between untrusted data and
   trusted instructions.

Marque responds to these pressures with a protocol that treats cryptographic
identity as portable across providers, encrypts content end-to-end by
default, separates instruction from data at the content layer, and shifts
abuse mitigation from content classification toward cryptographic
verification of sender identity and recipient-controlled economic policy.

This document specifies the core wire format, transport binding, and MLS
profile. It does **not** specify:

* The `did:mail` identity method (see
  `draft-tamim-marque-did-mail`).
* The Marque Block Spec content model (future companion draft).
* The tiered non-repudiation `ProofEnvelope` (future companion draft).
* The SMTP bridge (future companion draft).
* The economic anti-spam policy (future companion draft).

The founding specification {{MARQUE-FOUNDING}} provides narrative context
for the design choices made here.

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

The following terms are used normatively in this document. Terms defined
in {{RFC9420}} (MLS) retain their original meaning unless explicitly
overridden.

DID:
: A Decentralized Identifier as defined by the W3C DID Core
  Recommendation.

Envelope:
: The outermost Marque message unit defined in {{envelope}}.

Ciphertext payload:
: The MLS `PublicMessage` or `PrivateMessage` carried as the opaque content
  of an envelope.

Home provider:
: A service endpoint of type `MarqueMailbox` listed in a DID document with
  the lowest `priority` value, used as the default deposit and retention
  target for the DID subject.

Recipient pseudonym:
: A per-epoch opaque identifier derived from the recipient's MLS KeyPackage
  that addresses an envelope without revealing the recipient's DID to
  intermediate providers. See {{recipient-pseudonym}}.

Pad bucket:
: One of six fixed size classes to which every on-wire envelope MUST be
  padded. See {{padding}}.

Cipher suite:
: A named combination of signature, key-agreement, and hash primitives
  registered in the IANA *Marque Algorithms* registry established by this
  document.

# Architecture overview

Marque is a federation of mailbox providers that store and forward opaque
encrypted envelopes, with an optional peer-to-peer fast path for messages
whose senders and recipients are both online. Providers do not maintain
conversation state; all group and thread state lives in MLS epochs held by
clients.

The architecture imposes two hard invariants:

1. **Providers MUST NOT read envelope content.** They MAY read addressing
   and policy metadata as required for routing and anti-spam enforcement.
   A compliant implementation MUST NOT retain metadata beyond the retention
   policy advertised in its capability record.

2. **Providers MUST NOT own conversation state.** Group membership, thread
   continuity, and cryptographic epoch progression are client state. A
   provider that persists such state beyond the envelopes it is asked to
   retain for delivery does not conform to this specification.

These invariants distinguish Marque from server-stateful messaging systems
and bound its storage cost to the function
`(envelopes × mean_ciphertext_size)`.

## Roles

Marque defines three roles:

**Clients** act on behalf of one or more DID-identified users. Clients
construct envelopes, deposit them at provider endpoints, authenticate to
providers for fetch, and maintain local MLS group state.

**Providers** accept envelopes addressed to pseudonyms that resolve to DIDs
for which they are a listed service endpoint, persist envelopes for at
least their advertised retention period, serve them to authenticated
clients, and participate in federation gossip.

**Notaries** co-sign delivery receipts for the Attested and Certified
non-repudiation tiers. A notary MAY also act as a provider; the roles are
distinct but compatible.

# Envelope {#envelope}

The Marque envelope is a canonical CBOR {{!RFC8949}} map. Every envelope
MUST be serialized in CBOR Canonical Encoding.

The normative CDDL {{!RFC8610}}:

~~~cddl
; Marque envelope v1
envelope = {
  ? 0 : "marque/1",              ; version marker, REQUIRED
    1 : bstr .size 32,           ; recipient_pseudonym (see §5)
    2 : tstr,                    ; sender_did
    3 : tstr,                    ; sender_cert_ref (DID URL fragment)
    4 : auth_mode,
    5 : privacy_tier,
    6 : tstr,                    ; cipher suite identifier
    7 : bstr .size 32,           ; content_id (BLAKE3-256 of ciphertext)
    8 : uint,                    ; timestamp (seconds since 1970-01-01 UTC)
    9 : pad_bucket,
  ? 10: legacy-headers,          ; only present for bridged messages
    11: bstr,                    ; ciphertext
    12: signature,
  ? 13: proof-envelope-ref       ; REQUIRED iff auth_mode = QUALIFIED
}

auth_mode = &(
  casual:    0,
  signed:    1,
  qualified: 2
)

privacy_tier = &(
  casual: 0,
  sealed: 1,
  mixed:  2
)

pad_bucket = &(
  b4k:   0,
  b16k:  1,
  b64k:  2,
  b256k: 3,
  b1m:   4,
  b4m:   5
)

signature = {
    alg:          tstr,
  ? classical:    bstr,
  ? pq:           bstr
}

legacy-headers = { * tstr => tstr }

proof-envelope-ref = tstr .regexp "https?://.*" / bstr
~~~

Fields are semantically equivalent to the JSON projection in the founding
specification's `examples/envelope.json`. The canonical form for signing
and hashing purposes is the CBOR encoding above; the JSON projection is
non-normative and SHOULD NOT be used for wire interop.

## Recipient pseudonym {#recipient-pseudonym}

The recipient pseudonym is computed as:

~~~
recipient_pseudonym = HKDF-Expand(
  HKDF-Extract(salt = "marque:pseudonym/v1",
               ikm  = recipient_epoch_secret),
  info = sender_cert_ref || content_id,
  len  = 32
)
~~~

where `recipient_epoch_secret` is the MLS `epoch_authenticator` of the
current epoch for the group addressed by the envelope.

At the Casual privacy tier the sender MAY additionally transmit the
recipient DID in cleartext; the `recipient_pseudonym` field remains present
and MUST be computed as above for integrity purposes but is not relied on
for confidentiality.

The pseudonym construction binds a given ciphertext to a specific
recipient, sender, and content, preventing a cross-recipient correlator
from identifying identical-content envelopes across mailboxes.

## Padding {#padding}

Every envelope MUST be padded to one of six fixed sizes before
transmission:

| `pad_bucket` | Size on wire |
|---|---|
| `b4k`   | 4 KiB    |
| `b16k`  | 16 KiB   |
| `b64k`  | 64 KiB   |
| `b256k` | 256 KiB  |
| `b1m`   | 1 MiB    |
| `b4m`   | 4 MiB    |

A sender MUST select the smallest bucket into which the fully encoded
envelope fits. A receiver MUST reject envelopes whose on-wire size does not
exactly equal the nominal bucket size.

Padding bytes MUST be cryptographically random and MUST be placed in a
trailing `bstr` field outside the signed envelope map, so that padding does
not interact with the envelope signature.

Envelopes that would exceed the `b4m` bucket MUST split attachments into
content-addressed blobs referenced by the companion Marque Block Spec and
carried out-of-band; core envelopes that cannot fit in 4 MiB after such
splitting MUST be rejected by conformant senders.

## Timestamp

The `timestamp` field carries a UTC Unix-epoch seconds value. At the Casual
auth mode it is advisory. At the Signed and Qualified auth modes the
companion `ProofEnvelope` carries a cryptographically verifiable timestamp
that takes precedence; receivers MUST reject a message whose envelope
`timestamp` diverges by more than 60 seconds from the `ProofEnvelope`
timestamp.

# Transport binding

Marque runs over QUIC {{RFC9000}} with TLS 1.3 {{!RFC8446}} as the security
layer. Clients establish long-lived connections to each of their home and
alternate providers and reuse them for both deposit and fetch operations.

## TLS handshake

A client's TLS certificate is self-signed with `SubjectPublicKeyInfo` equal
to the client's current Device Signing Key as published in the client's
DID document. The server verifies the certificate by:

1. Extracting the DID from the certificate's `subjectAltName` of type
   `URI` (form: `did:mail:<domain>:<local-part>`).
2. Resolving the DID per the `did:mail` method specification.
3. Confirming that the certificate's public key appears under
   `verificationMethod` in the resolved DID document.
4. Confirming that the Device Signing Key certificate chains to an Identity
   Root Key whose most-recent KEL event is at least one hour old (to bound
   race conditions with rotation).

The server's certificate is constructed identically with the server's
provider-identity DID.

Clients and servers MUST support the `X25519MLKEM768` key-share group
{{I-D.ietf-tls-ecdhe-mlkem}}. Clients SHOULD prefer it. `X25519` alone
remains available for deployments that cannot yet ship the hybrid KEM.

## Streams

Each QUIC connection SHALL carry:

* **Control stream** (QUIC stream 0): bidirectional, length-prefixed CBOR
  control frames for capability discovery, KeyPackage publication, and
  presence.
* **Deposit streams**: client-initiated unidirectional streams, one per
  envelope, closed after the ack.
* **Fetch streams**: client-initiated bidirectional streams, one per
  long-poll, closed by the client when it wishes to terminate the poll.

Control-frame types are allocated in {{iana-envelope-fields}}.

## Connection migration

Providers MUST support QUIC connection migration so mobile clients can
preserve long-poll continuity across network changes.

# MLS profile for asynchronous mail {#mls-profile}

Marque uses MLS {{RFC9420}} as its end-to-end encryption foundation, with
two asynchronous-delivery extensions specified here.

## Cipher suites

Conformant implementations MUST support MLS cipher suite
`MLS_256_DHKEMX25519_AES256GCM_SHA512_Ed25519`. They MAY additionally
support the hybrid post-quantum suite registered by this document (see
{{iana-algorithms}}).

## KeyPackage pool

Each identity MUST maintain a pool of at least 32 fresh KeyPackages at each
listed provider, with at least one flagged as `last_resort`. A client MUST
upload replacement KeyPackages whenever the pool drops below 16. Providers
MUST NOT distribute `last_resort` KeyPackages unless the pool is exhausted.

## Epoch checkpoints

To support correspondents who remain offline for extended periods, a
sending client MAY request its provider to persist an HPKE
{{!RFC9180}}-encrypted `GroupContext` snapshot against a specified offline
member's latest KeyPackage. On return, the offline member fetches the
snapshot and uses it to leap to the current epoch without processing
intervening commits.

Epoch checkpoint frames are listed in {{iana-envelope-fields}}. Providers
SHOULD retain at most the two most-recent checkpoints per offline member
per group.

## Threads

A Marque thread corresponds to a persistent MLS group. Replies are MLS
`Application` messages in the current epoch. The MLS group's `group_id`
field SHALL be the SHA-256 hash of the concatenation of all founding
members' DIDs in canonical byte order.

## CC and BCC semantics

Visible recipients (`To`, `CC`) are MLS group members. BCC recipients
MUST be implemented as parallel two-member groups (sender plus each BCC
recipient) created with the `external_senders` extension; the content of
these groups is independent of the primary group's state.

A sending client MUST inform BCC recipients, via a protected header, that
they are receiving a BCC copy, so replies are not misdirected to the full
primary group.

# Metadata privacy tiers

A client selects, per message, one of three metadata-privacy tiers via
the `privacy_tier` envelope field.

**Casual.** The sender and recipient DIDs are visible to both providers
behind TLS 1.3. Sealed Sender-style pseudonymization is not applied.

**Sealed.** The recipient DID is hidden from the recipient's provider
behind the `recipient_pseudonym` construction in {{recipient-pseudonym}}.
The sender authenticates by co-delivering a Privacy Pass {{!RFC9578}} token
from the sender's provider, which does not reveal the sender DID to the
recipient's provider. This is the RECOMMENDED default tier.

**Mixed.** The envelope is wrapped in Sphinx packets and delivered via a
mix network or onion routing service. Latency expands by 10–30 seconds per
hop. Mixed tier is mutually exclusive with auth modes that depend on
provider co-signatures.

Implementations MUST NOT silently downgrade the tier selected by the
sender. If a provider cannot honor the requested tier (for example, because
it lacks a mixnet peering), it MUST reject the deposit with error code
`tier-unsupported`.

# Federation

Providers federate via a pull-gossip protocol. A provider periodically
polls its peer set for:

* Transparency-log tree heads and consistency proofs.
* Anti-spam fingerprint bulletins (provider-agnostic hashes of
  high-confidence abuse envelopes).
* Reputation deltas.

The federation substrate does not require global consensus; eventual
consistency is sufficient. Federation frame definitions are deferred to a
companion draft.

# IANA Considerations {#iana}

This document requests the creation of three registries. The registration
policy for each is "Specification Required" {{RFC8126}}.

## Marque Algorithms registry {#iana-algorithms}

Registers named cipher suites usable in the envelope `cipher_suite` field
and the TLS / MLS handshakes. Each registration provides:

* A short string identifier (IANA-assigned prefix `marque-`).
* The signature algorithm, key-agreement algorithm, and hash algorithm.
* A reference document.

Initial registrations:

| Identifier | Signature | KEM | Hash | Reference |
|---|---|---|---|---|
| `marque-classical-v1` | Ed25519 {{RFC8032}} | X25519 {{RFC7748}} | SHA-256 | this document |
| `marque-pq-v1` | Ed25519 + ML-DSA-65 {{FIPS204}} | X25519MLKEM768 {{I-D.ietf-tls-ecdhe-mlkem}} | SHA-256 | this document |
| `marque-archive-v1` | SLH-DSA-SHA2-128f {{FIPS205}} | X25519MLKEM768 | SHA-256 | this document |

## Marque Envelope Fields registry {#iana-envelope-fields}

Registers integer keys in the envelope CBOR map and the control-frame CBOR
map. Keys 0–15 are reserved for this specification; keys 16–127 are
available for companion drafts under Specification Required; keys ≥128 are
available for private-use extensions.

Control-frame type assignments defined by this document:

| `type` | Name | Direction |
|---|---|---|
| 1 | `capability-request`       | Client → Provider |
| 2 | `capability-response`      | Provider → Client |
| 3 | `keypackage-upload`        | Client → Provider |
| 4 | `keypackage-withdraw`      | Client → Provider |
| 5 | `epoch-checkpoint-deposit` | Client → Provider |
| 6 | `epoch-checkpoint-fetch`   | Client → Provider |
| 7 | `long-poll-subscribe`      | Client → Provider |
| 8 | `long-poll-event`          | Provider → Client |
| 9 | `ack`                      | Either            |
| 10 | `error`                   | Either            |

## Marque Block Types registry

Registers reverse-DNS-namespaced identifiers used in Marque Block Spec
payloads. Names under `core.*` are reserved for specifications produced by
an IETF WG that may form around Marque. All other namespaces are available
under Expert Review.

Initial registrations (from companion drafts, not this document) are
expected to include `core.text`, `core.attachment`, `core.schedule`,
`core.reply`, `core.reaction`, and `core.signed_action`.

# Security Considerations {#security}

## Endpoint compromise

A compromised endpoint reveals all envelopes the endpoint can decrypt.
This is inherent to any end-to-end encrypted system. Marque mitigates
lateral damage by MLS epoch forward-secrecy and by scope-limiting Device
Signing Keys: compromising a single device does not compromise the
Identity Root Key and does not grant signing authority beyond the device's
certificate validity window (nominally six months).

## Key compromise impersonation

At the Casual auth mode, MLS provides MAC-based authentication, which does
not distinguish between legitimate members. An adversary who compromises
any member's device can forge transcripts purporting to originate from any
other member. Applications requiring non-repudiation MUST use the Signed
or Qualified auth mode, in which case forgery requires compromise of the
specific sender's Device Signing Key.

## Cryptographic agility

The `cipher_suite` field is a negotiated identifier; it is not protected
by the signature on the envelope. An active adversary MAY attempt a
downgrade attack by rewriting the field in transit. Marque defends against
this by (a) binding the `cipher_suite` into the MLS `GroupContext`
extension field, and (b) requiring receivers to reject envelopes whose
advertised suite does not match the MLS-level context. Specification of
the binding mechanism is deferred to the MLS cipher-suite codepoint
registered in {{iana-algorithms}}.

## Pseudonym linkability

The pseudonym construction in {{recipient-pseudonym}} changes per-epoch.
An adversary who observes a single provider over a long window MAY link
pseudonyms belonging to the same recipient across epochs by access-timing
correlation. Implementations SHOULD randomize fetch schedules and SHOULD
implement batched cover fetches per {{cover-traffic}} when the Sealed tier
is selected.

## Padding oracles

Envelopes are padded in a tail `bstr` outside the signed map. The padding
does not feed into ciphertext construction; thus no padding oracle is
exposed. Receivers MUST still reject oversize envelopes at the QUIC stream
layer before CBOR decode to avoid amplification.

## Provider-malicious case

A malicious provider can drop envelopes, delay delivery, or refuse to
serve fetches. Marque mitigates such cases by:

* Allowing each user to list multiple service endpoints in their DID
  document and have senders deposit to 2-of-N.
* Signing delivery receipts with a transparency log entry, so a provider
  cannot plausibly deny delivery to an auditor.

A malicious provider cannot read envelope content (MLS), cannot forge
sender identity (DID-bound Device Signing Key), and cannot silently
re-order conversation state (MLS epoch numbering).

## Denial of service

Anti-spam is specified in a companion draft. The core protocol permits
but does not mandate proof-of-work, bond, or rate-proof tokens at the
control-frame level.

# Privacy Considerations

## Metadata exposure

At the Casual tier, every provider in the delivery path learns sender DID,
recipient DID, envelope size bucket, and timestamp. At the Sealed tier the
recipient's provider learns only the pseudonym and size bucket; the
sender's provider learns the sender DID only. At the Mixed tier no single
intermediary learns both sender and recipient.

A passive global adversary with visibility into the transport layer can
observe envelope size buckets even under Mixed tier. Implementations are
therefore encouraged to emit cover traffic at the provider level as
specified in {{cover-traffic}}.

## What is not achievable {#what-is-not}

This specification does not conceal:

* The existence of Marque traffic between two providers (traffic analysis
  across TLS is possible at the IP layer).
* The fact that a given DID is a Marque participant (DNS resolution of
  the `did:mail` document is observable).
* Typing-cadence leakage in long-polled connections.
* Full deanonymization if a state adversary controls more than 40% of the
  mix nodes in the selected mixnet.

## Cover traffic {#cover-traffic}

Providers MUST emit server-to-server cover traffic at a Poisson-distributed
rate calibrated to make egress rates to every peer uniform regardless of
real user activity. Clients MAY emit client-to-provider cover traffic; this
is not mandated because mobile battery impact has historically produced
user abandonment.

## Legal-process response

A provider compelled by legal process can surrender only what it observes:
pseudonyms, timestamps, envelope sizes, and (at the Casual tier) sender
and recipient DIDs. Envelope content is inaccessible to the provider and
cannot be surrendered short of compromising an endpoint.

A notary QTSP participating in the Qualified auth mode can surrender the
delivery receipts it co-signed. It cannot surrender content.

# Acknowledgments
{:numbered="false"}

The architecture in this document composes elements from RFC 9420 MLS,
RFC 9000 QUIC, RFC 9591 FROST, RFC 9106 Argon2, RFC 8032 Ed25519, RFC 7748
X25519, Sigstore, OpenTimestamps, Bluesky AT, and the IETF MIMI WG. Its
approach to metadata minimization draws from Signal Sealed Sender
{{SEALED-SENDER}}. The founding specification {{MARQUE-FOUNDING}} provides
the narrative rationale for the choices made here.

--- back

# Implementation status
{:numbered="false"}

At the time of initial submission, no interoperable implementation of
this document exists. Phase 1 of the migration roadmap
{{MARQUE-FOUNDING}} targets at least two independent interoperable
implementations (the IETF standards-track maturity bar), explicitly
developed by unaffiliated groups. Example implementations are welcome
and expected, but none is privileged — no implementation in the
Marque ecosystem is designated "reference." Implementers are
unrestrained in how they differentiate above the wire; competition
is expected at the feature layer (user experience, AI triage, search,
discovery) rather than at the protocol.

# Worked example
{:numbered="false"}

A JSON projection of a minimal envelope appears in the founding
specification under `examples/envelope.json`. The canonical CBOR encoding
for wire transmission is deterministic and is produced from the JSON by
any conformant CBOR canonicalizer.
