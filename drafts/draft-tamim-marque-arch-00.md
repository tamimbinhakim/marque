---
title: "Marque Architecture"
abbrev: "marque-arch"
docname: draft-tamim-marque-arch-00
category: info
ipr: trust200902
submissiontype: independent
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - email
  - messaging
  - architecture
  - did
  - mls
  - eidas
author:
  -
    ins: Tamim
    name: Tamim Bin Hakim
    email: tamimbinhakim.work@gmail.com

normative:
  RFC2119:
  RFC8174:

informative:
  RFC5321:
  RFC5322:
  RFC6376:
  RFC7208:
  RFC7489:
  RFC8058:
  RFC8617:
  RFC9420:
  RFC9180:
  RFC9591:
  W3C-DID:
    title: "Decentralized Identifiers (DIDs) v1.0"
    target: "https://www.w3.org/TR/did-core/"
    author:
      - org: "W3C"
    date: 2022-07
  EIDAS2:
    title: "Regulation (EU) 2024/1183 — European Digital Identity Framework"
    target: "https://eur-lex.europa.eu/eli/reg/2024/1183/oj"
    date: 2024-04
  ETSI-EN-319-521:
    title: "ETSI EN 319 521 — Electronic Registered Delivery Services provider policy and security requirements"
    target: "https://www.etsi.org/deliver/etsi_en/319500_319599/319521/"
  ETSI-EN-319-522:
    title: "ETSI EN 319 522 — Electronic Registered Delivery Services (framework, semantic contents, formats, bindings)"
    target: "https://www.etsi.org/deliver/etsi_en/319500_319599/31952201/"
  MARQUE-CORE:
    title: "Marque Core Protocol"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-core/"
  MARQUE-DID-MAIL:
    title: "The did:mail DID Method"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-did-mail/"
  MARQUE-PROOF:
    title: "Marque ProofEnvelope"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-proof/"
  MARQUE-MBS:
    title: "Marque Block Spec (MBS)"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-mbs/"
  MARQUE-BRIDGE:
    title: "Marque Bidirectional SMTP Bridge"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-bridge/"
  MARQUE-ANTISPAM:
    title: "Marque Economic and Cryptographic Anti-Spam"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-antispam/"
  MARQUE-ROOMS:
    title: "Marque Rooms — Shared Mailboxes, Teams, Lists, Delegation, Organizations"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-rooms/"
  MARQUE-STORAGE:
    title: "Marque Storage, Archive, and Cross-Device Sync"
    target: "https://datatracker.ietf.org/doc/draft-tamim-marque-storage/"
  MARQUE-FOUNDING:
    title: "Marque — founding specification"
    target: "https://marqueproto.org/spec/"
  OPENTIMESTAMPS:
    title: "OpenTimestamps"
    target: "https://opentimestamps.org"

--- abstract

This document describes the architecture of Marque, a cryptographically
signed, federated-and-peer-to-peer messaging protocol proposed as a
successor to the Simple Mail Transfer Protocol {{RFC5321}} and the
Internet Message Format {{RFC5322}}.

It provides the architectural frame for the companion documents that
specify the normative wire format, identity method, content layer,
non-repudiation envelope, anti-spam model, SMTP bridge, Rooms, and
storage model. It does not itself specify any wire format; its purpose
is to let reviewers, implementers, and deployment planners understand
how the component specifications fit together before reading them in
depth.

The target deployment horizon is ten to twenty years. Marque is
explicitly designed to coexist with and bridge to SMTP throughout
that period, and only at equilibrium to become the majority substrate
for human correspondence.

--- middle

# Introduction

Since 1982, SMTP {{RFC5321}} has served as the internet's primary
asynchronous messaging substrate. The protocol as specified assumes a
cooperative operator community, trusted transport between operators,
and good-faith use by end-users. None of those assumptions hold in
2026.

A forty-year succession of extensions — SPF {{RFC7208}}, DKIM
{{RFC6376}}, DMARC {{RFC7489}}, ARC {{RFC8617}}, BIMI, MTA-STS, TLS-RPT,
List-Unsubscribe {{RFC8058}}, Autocrypt, S/MIME, OpenPGP — has
retrofitted sender authentication, transport confidentiality,
end-to-end content confidentiality, and abuse resistance onto the
substrate piecemeal. Each extension has demonstrated partial success
and structural limitation. The cumulative result is that legitimate
mail operation is now gated on compliance with roughly a dozen
overlapping mechanisms, none of which was present in the original
design, and only two operators — Google and Microsoft — possess the
reputation data to authoritatively decide whether a given sender is
legitimate.

Three independent pressures make further incremental patching
inadequate for the coming decade:

1. The concentration of reputation gating in two large operators has
   made independent mail-transfer-agent operation infeasible for most
   legitimate senders, even those correctly implementing every
   published extension.
2. Large-language-model text generation has collapsed the cost of
   crafting novel, personalized, context-aware abuse content, driving
   the content-classification approach to its asymptotic ceiling.
3. The widening deployment of automated mail-reading assistants has
   introduced prompt-injection as a new weaponized vulnerability
   class, for which the legacy message format provides no structural
   separation between untrusted data and trusted instructions.

Marque responds to these pressures with a clean-sheet protocol that:

* treats cryptographic identity as portable across providers
  ({{identity}}),
* encrypts content end-to-end by default ({{transport-and-crypto}}),
* separates data from instructions at the content layer
  ({{content}}),
* produces cryptographic non-repudiation as a tiered opt-in mapped
  to {{EIDAS2}} Electronic Registered Delivery Services ({{proof}}),
* shifts abuse mitigation from content classification to cryptographic
  sender verification and recipient-controlled economic policy
  ({{antispam}}), and
* interoperates bidirectionally with the existing SMTP ecosystem
  throughout the transition ({{bridge}}).

## Purpose of this document

This document is the architectural companion to a set of normative
drafts that collectively specify Marque. It is Informational. It does
not itself impose conformance obligations; the companion documents do.

The intended audience is:

* IETF reviewers and Working Group participants evaluating whether
  Marque warrants chartering as a new WG or adoption inside an
  existing one.
* Implementers planning a Marque client, provider, or bridge and
  wanting to know which normative drafts apply to which component.
* Deployment planners (regulators, enterprise architects, QTSPs)
  evaluating whether Marque meets their policy requirements before
  reading the normative text.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}}
{{RFC8174}} when, and only when, they appear in all capitals, as
shown here. This document is Informational and uses BCP 14 terms only
when recalling requirements imposed by the companion normative drafts.

# Terminology

Terms used throughout this document. Normative definitions live in
the companion drafts; this section summarizes them for readability.

DID:
: A W3C Decentralized Identifier {{W3C-DID}}.

`did:mail`:
: The DID method specified in {{MARQUE-DID-MAIL}}. A thin profile over
  `did:web` extended with a signed Key Event Log (KEL) inspired by
  KERI.

Identity Root Key (IRK):
: The long-lived (multi-year) keypair that roots a `did:mail` identity.
  Held in cold storage. Used only to sign KEL events.

Device Signing Key (DSK):
: A medium-lived (nominal 6 months) hardware-backed per-device key,
  authorized by the IRK via a KEL event. Used for per-message
  signatures.

MLS:
: Messaging Layer Security {{RFC9420}}. Marque's end-to-end encryption
  substrate.

Envelope:
: The canonical CBOR message unit specified in {{MARQUE-CORE}}.

ProofEnvelope:
: The out-of-band non-repudiation artifact specified in
  {{MARQUE-PROOF}}. Carries sender signature, RFC 3161 timestamps,
  optional FROST {{RFC9591}} threshold signatures, optional
  {{OPENTIMESTAMPS}} Bitcoin anchor, and (for Qualified tier) a QTSP
  co-signature.

Marque Block Spec (MBS):
: The content layer specified in {{MARQUE-MBS}}. Replaces HTML mail's
  frozen 2005 subset with a typed, schema-validated, reverse-DNS
  namespaced block vocabulary.

Auth mode:
: `CASUAL` (MLS MAC, deniable), `SIGNED` (sender signature,
  non-repudiable, ERDS), or `QUALIFIED` (QTSP-cosigned, QERDS).

Privacy tier:
: `CASUAL` (DIDs in cleartext behind TLS), `SEALED` (recipient DID
  concealed by a per-epoch pseudonym; RECOMMENDED default), or
  `MIXED` (mix-network delivery).

Room:
: A first-class shared identity kind specified in {{MARQUE-ROOMS}}.
  One of `shared`, `team`, `list`, `delegation`, or `org`.

# Architectural commitments

Marque is defined by seven commitments. Each commitment removes a
class of SMTP failure.

1. **Identity is cryptographic and portable.** A user's identity is
   a `did:mail` (see {{identity}}). Changing providers is an entry in
   the DID document; correspondents update nothing.

2. **Content is end-to-end encrypted by default.** Messages are
   carried inside MLS {{RFC9420}} groups — one group per thread.
   Providers never read content.

3. **Legal proof is native and tiered.** The `ProofEnvelope` produces
   ERDS-grade (Attested) or QERDS-grade (Qualified) non-repudiation
   without leaving the protocol.

4. **Providers are commodity encrypted mailboxes.** Providers store
   envelopes, not conversations. Storage scales as
   `envelopes × ciphertext_size`, not as
   `users × rooms × membership × history`.

5. **Content is a typed signed block log.** MBS replaces HTML tables
   with JSON-based, schema-validated, reverse-DNS-namespaced blocks.
   Rendering is deterministic. Accessibility is mandatory.

6. **Anti-spam is economic and cryptographic.** Verified sender
   identity, proof-of-work, refundable bonds, reputation gossip, and
   recipient-controlled policy — not AI-beats-AI content
   classification.

7. **SMTP interop is bidirectional and honestly labeled.** A Marque
   provider bridges to SMTP from day one. No silent downgrade at
   legal tiers.

# Roles

Marque defines three roles. Any deployed entity plays at least one
and MAY play several.

**Clients** act on behalf of one or more `did:mail`-identified users.
Clients construct envelopes, deposit them at provider endpoints,
authenticate to providers for fetch, hold MLS group state, render MBS
content, and produce `ProofEnvelope` signatures at the Signed and
Qualified auth modes.

**Providers** accept envelopes addressed to recipients for whom they
are a listed service endpoint, persist envelopes for their advertised
retention period, serve them to authenticated clients, and
participate in federation gossip. Providers MUST NOT read envelope
content and MUST NOT own conversation state.

**Notaries** co-sign delivery receipts for the Attested and Qualified
non-repudiation tiers. A notary is frequently also a provider;
separating the roles is architecturally required but deployment-wise
optional. A Qualified-tier notary is a QTSP in the sense of
{{EIDAS2}}.

The three-role model is deliberately narrower than SMTP's. There is
no submission MTA distinct from a receiving MTA; there is no
open-relay role; there is no boundary MTA. Providers are a single
homogeneous role along the delivery path.

# Identity {#identity}

Specified normatively in {{MARQUE-DID-MAIL}}.

Marque binds user identity to a W3C DID {{W3C-DID}} rather than to an
SMTP address. The DID method is `did:mail`. A `did:mail` identifier
has two forms:

* **Provider-hosted**, e.g. `did:mail:example.com:alice`. The
  authoritative DID document is served at a well-known URL under
  `example.com`. Familiar for users migrating from SMTP.
* **Portable-tier**, e.g. `did:mail:marque.id:alice`. The
  authoritative DID document is served by a registry (IMIF, the
  Internet Mail Identity Foundation, proposed in {{MARQUE-FOUNDING}})
  such that the identity survives the loss of any particular DNS
  domain.

A `did:mail` identity is rooted in an Identity Root Key (IRK) held in
cold storage. Device Signing Keys (DSKs) are authorized by the IRK
via signed events appended to a Key Event Log (KEL) inspired by KERI.
Clients MUST verify the KEL of a correspondent on first contact (a
Trust-On-First-Use anchor), and MUST re-verify on any signature
produced under a DSK whose authorization event the client has not
previously observed.

Recovery uses a guardian threshold. A user designates N guardians
(human or institutional) and a threshold M-of-N; M guardians
co-signing a recovery event can attach a fresh DSK to the KEL in the
event the user's last DSK is lost. The home provider MAY be one
guardian but MUST NOT be sole guardian.

Identity and legal identity are deliberately decoupled. A
`did:mail` identifies a cryptographic subject. Qualified tier binds
that subject to a legal person via a qualified certificate issued by a
QTSP; the binding is expressed in the `ProofEnvelope`, not in the
envelope, so a user can hold one `did:mail` that routes both Casual
personal correspondence and Qualified legal correspondence.

# Transport and cryptography {#transport-and-crypto}

Specified normatively in {{MARQUE-CORE}}.

Marque runs over QUIC with TLS 1.3. Client TLS certificates carry the
client's `did:mail` in `subjectAltName`, and the server verifies by
DID resolution rather than by CA trust anchors. This ties TLS
authentication to the same cryptographic identity that signs
envelopes, eliminating the SMTP-era split between
reputation-authenticated transport and content-authenticated signing.

The end-to-end encryption substrate is MLS {{RFC9420}} with two
asynchronous-mail extensions specified in {{MARQUE-CORE}}:

* **KeyPackage pool.** Each identity maintains ≥32 fresh KeyPackages
  at each listed provider, with at least one flagged as `last_resort`,
  so a sender can create a group with an offline recipient without
  coordination.
* **Epoch checkpoints.** A sender MAY request its provider to persist
  an HPKE-encrypted {{RFC9180}} `GroupContext` snapshot for a member
  who has been offline for extended periods; the member leaps to the
  current epoch on return without processing intervening commits.

The mandatory MLS cipher suite is
`MLS_256_DHKEMX25519_AES256GCM_SHA512_Ed25519`. A post-quantum hybrid
suite combining X25519MLKEM768 for key agreement and Ed25519 + ML-DSA-65
for signatures is registered for optional use and will become
mandatory when NIST deprecates the classical baseline.

Three metadata-privacy tiers are selectable per message: Casual
(DIDs in cleartext behind TLS), Sealed (recipient concealed by a
per-epoch pseudonym; RECOMMENDED default), and Mixed (mix-net
delivery). Sealed is the default because it provides the strongest
metadata reduction compatible with a 2026-realistic mobile-client
power budget.

# Content {#content}

Specified normatively in {{MARQUE-MBS}}.

The Marque Block Spec (MBS) replaces HTML mail's frozen 2005 subset
with JSON-based, schema-validated, reverse-DNS-namespaced blocks.
Every payload is a sequence of blocks in a typed document. Rendering
is deterministic: given the same blocks, any conformant client produces
the same user-visible output.

Core block types defined in {{MARQUE-MBS}} include:

* `core.text` — CommonMark + GFM text with strict sanitization.
* `core.image`, `core.attachment` — content-addressed media via
  BLAKE3 content identifiers, with mandatory accessibility metadata
  (alt text, descriptions, captions).
* `core.reply`, `core.forward` — thread and forwarding primitives
  with three forward flavors (Quoted, Attested, Certified) and
  explicit `forwardable` / `quotable` per-message flags.
* `core.poll`, `core.schedule`, `core.form` — structured
  interaction.
* `core.payment` — structured payment requests (no protocol token;
  integrates with existing rails).
* `core.signed_action` — the Marque-native DocuSign replacement.
* `core.retract` — sender-initiated retraction within an undo window.

Unknown blocks are rendered by their `fallback` field. The envelope
additionally carries an `alt_plaintext` so a message is never
unreadable on any conformant client. Accessibility metadata is
mandatory at the block schema level, not recommended in prose.

A central property of the block layer is the `public_untrusted`
semantic: quoted content, forwarded content from uncertified origins,
and public list-archive content are flagged so an AI assistant
operating on the inbox treats them as untrusted data rather than as
instructions. This structural data-versus-instruction separation is
the principal mitigation for the prompt-injection class introduced in
({{RFC5322}}-based inboxes and represented by CVE-2025-32711.

# Legal proof {#proof}

Specified normatively in {{MARQUE-PROOF}}.

Non-repudiation is selected per message via `auth_mode`:

* **`CASUAL`** — MLS-level MAC authentication. Deniable. No
  `ProofEnvelope`.
* **`SIGNED`** (Attested tier) — the sender's DSK signs a canonical
  hash of the envelope; an RFC 3161 timestamp is attached by the
  sender's provider; optionally an {{OPENTIMESTAMPS}} Bitcoin anchor
  is attached for long-term verifiability without a trusted third
  party. Maps to the Electronic Registered Delivery Service (ERDS)
  semantics of {{EIDAS2}} Article 43.
* **`QUALIFIED`** (Certified tier) — adds a qualified certificate
  from a QTSP binding the `did:mail` to a legal person, a qualified
  timestamp from the same or a peer QTSP, optional FROST {{RFC9591}}
  threshold signature from an M-of-N notary quorum, and archival
  long-term validation via CAdES-B-LTA. Maps to the Qualified
  Electronic Registered Delivery Service (QERDS) semantics of
  {{EIDAS2}} Article 44, profiled against {{ETSI-EN-319-522}}
  (semantic contents + formats + bindings) and {{ETSI-EN-319-521}}
  (provider policy and security requirements).

The `ProofEnvelope` is retrieved out-of-band, referenced by URL or
CID from the core envelope. This keeps the common-case envelope small
while making proof artifacts independently archivable and verifiable.

Qualified tier is mutually exclusive with the Mixed privacy tier,
because a QTSP co-signature presupposes a named counterparty.

# Providers, storage, and archival {#providers}

Specified normatively in {{MARQUE-CORE}} §Architecture and
{{MARQUE-STORAGE}}.

Providers are commodity encrypted mailboxes. They MUST NOT read
envelope content, MUST NOT own conversation state, and MUST NOT see
the social graph beyond immediate deposit metadata.

Marque adopts a hybrid archive-of-record model:

* **Provider side** — ciphertext is the durable archive. Retention
  is per-envelope-tier and advertised in the provider's capability
  record.
* **Device side** — cleartext lives in an encrypted, hardware-backed
  cache on authorized devices. Cross-device synchronization is by
  HPKE-encrypted {{RFC9180}} cache deltas under a per-identity
  archive key.

Senders' Sent-copies are preserved via a self-addressed single-member
MLS group, so the sender's archive uses the same cryptographic
substrate as incoming mail.

Provider migration uses a standardized export format, `.marquebox`.
The export carries the user's cleartext archive, ProofEnvelopes at
their retention-appropriate depth, the KEL, and capability records.
It is portable across providers and import-idempotent.

The GDPR right-to-erasure interacts with append-only signed logs.
Marque reconciles by distinguishing: a recipient may delete local
copies or request provider deletion; a sender cannot force deletion
of recipient copies; legal-tier archival copies are retained in
ciphertext at the provider level per jurisdictional retention rules
and are discoverable only by key-holders.

# Rooms {#rooms}

Specified normatively in {{MARQUE-ROOMS}}.

Rooms are first-class Marque identities of five kinds:

* **`shared`** — a shared mailbox (a "household inbox" or
  `support@example.com`) backed by an MLS group of authorized senders.
* **`team`** — a team handle (`security-team@example.com`) with
  roster, roles, and send-as authorization.
* **`list`** — a mailing list, with rolling-epoch relaxation for
  lists above 50 000 subscribers.
* **`delegation`** — an assistant delegation scope, cryptographically
  scope-limited; qualified send-as is explicitly forbidden.
* **`org`** — an organization handle bound to a legal-person
  qualified certificate per {{EIDAS2}}-aligned ETSI TS 119 461.

Send-as authorization is any-of (any roster member), threshold (M-of-N
via FROST {{RFC9591}} producing an Ed25519-compatible output), or
owner-only. Role rotation, audit logs, visibility tiers, and handle
conventions are specified in {{MARQUE-ROOMS}}.

# Anti-spam {#antispam}

Specified normatively in {{MARQUE-ANTISPAM}}.

Marque shifts the anti-spam battleground from content classification
to an AND-of-ORs of cryptographic and economic gates, any one of which
suffices for a given recipient policy:

* **Verified sender identity** — spoofing is structurally impossible,
  making SPF / DKIM / DMARC / ARC obsolete for the Marque-to-Marque
  path.
* **Argon2id proof-of-work**, tuned asymmetrically: fast for known
  contacts, slow for strangers.
* **Refundable bonds** for stranger outreach, burned on spam report,
  with a 14-day appeal window and serial-false-positive-reporter
  weighting.
* **Reputation gossip** — provider-agnostic hashes of
  high-confidence abuse envelopes propagate via pull-gossip.
* **Recipient policy**, enforced at the recipient provider, surfaced
  as machine-readable rejection reasons so silent drop-to-spam is
  prohibited.
* **AI-assistant allowlist** — per-user, client-maintained, opt-in;
  providers MAY seed but MUST NOT force-add.
* **Structural data-vs-instruction separation** at the content layer
  (see {{content}}).

Rejection reasons are a machine-readable enum: `POLICY_REJECTED_BOND`,
`POLICY_REJECTED_POW`, `POLICY_REJECTED_TIER`,
`REFUSED_TO_ACKNOWLEDGE`, and so on. A sender receiving a rejection
knows why, and an end-user receiving a suppressed message sees a
machine-readable cause, not a silent disappearance.

# SMTP bridge {#bridge}

Specified normatively in {{MARQUE-BRIDGE}}.

Every Marque provider MUST ship a bidirectional SMTP bridge from day
one. The bridge is not an afterthought; it is tier-zero for the
migration. A broken bridge in the first two years churns
early-adopter users back to SMTP within a week.

Inbound (SMTP → Marque) the bridge:

* resolves the sender's `did:mail` if one is discoverable (via four
  mechanisms enumerated in {{MARQUE-BRIDGE}});
* wraps a Marque-discoverable inbound message as a Marque envelope
  with a `bridged · encrypted` UI chip;
* surfaces a non-Marque inbound message with a plain `email` chip,
  never claiming more provenance than was proven.

Outbound (Marque → SMTP) the bridge:

* downgrades cleartext content with an Autocrypt header for
  opportunistic upgrade on the next round-trip;
* replaces content attachments with authenticated download links
  (default 7-day expiry, per-recipient tokens, revocable, access-logged);
* truncates the Received chain to the two outermost hops to reduce
  vendor-internal PII leakage;
* refuses to downgrade legal-tier messages (Attested and Qualified
  fail closed rather than silently losing their proof chain; a
  QERDS-to-legacy gateway is the supported escape hatch for Qualified
  tier).

The bridge implements an explicit upgrade state machine per
correspondent:
`UNKNOWN → AVAILABLE → MUTUAL → NATIVE`. A correspondent moves through
the state machine as their client proves Marque capability, and the
upgrade is observable to the user.

# Conformance and interop tiers

Specified normatively in {{MARQUE-CORE}} and the conformance section
of the founding specification {{MARQUE-FOUNDING}}.

Marque defines four interop tiers, each a superset of the previous:

* **Casual** — envelope, `did:mail`, MLS, bridge.
* **Attested** — adds `ProofEnvelope` at the Signed auth mode, ERDS
  semantics, OpenTimestamps anchor.
* **Certified** — adds Qualified auth mode, QTSP co-signature,
  CAdES-B-LTA archival, QERDS semantics.
* **Bridge** — a provider operating only the SMTP bridge role without
  native Marque storage.

An implementation advertises its tier in its capability record. A
sender MUST NOT select a tier the recipient's provider does not
advertise; if it does, the deposit MUST fail with a machine-readable
error.

# Deployment and adoption model

Marque is designed for a 10–20 year transition. The founding
specification {{MARQUE-FOUNDING}} describes a five-phase adoption
model:

1. **Foundation** (Year 0–2) — 5 000–10 000 active handles; at least
   two independent interoperable implementations from unaffiliated
   groups (example implementations welcome, none privileged); bridge
   load-bearing.
2. **First commercial providers** (Year 2–3) — 500 000–2 000 000
   active handles; first QTSP running QERDS-mode Marque.
3. **Regulatory recognition and enterprise** (Year 3–5) — eIDAS
   QERDS listings; Fortune 500 compliance adoption.
4. **Majority of new signups** (Year 5–10) — major clients ship
   native support; default-on for new accounts at top-10 providers.
5. **Equilibrium** (Year 10–20) — major providers announce SMTP
   receive-only; Marque becomes the default for human correspondence.

A minimum-two-unaffiliated-providers rule is applied at every
milestone to prevent launch capture.

# Relationship to other protocols

Marque adopts mechanisms from, and deliberately differs from, several
existing protocols.

* **MLS {{RFC9420}}** — Marque uses MLS as its E2EE substrate with
  async extensions (KeyPackage pools, epoch checkpoints) specified in
  {{MARQUE-CORE}}.
* **DIDs {{W3C-DID}}** — `did:mail` is a new DID method registered
  with the W3C DID WG.
* **SMTP {{RFC5321}} / IMF {{RFC5322}}** — Marque bridges
  bidirectionally throughout its 10–20 year transition. There is no
  flag-day cutover.
* **eIDAS {{EIDAS2}} / ETSI ESI** — Qualified auth mode is profiled
  against {{ETSI-EN-319-522}} and {{ETSI-EN-319-521}} so Marque
  messages can carry QERDS semantics.
* **MIMI (IETF WG)** — Marque shares MIMI's motivation (async MLS
  transport) but targets different semantic scope: MIMI is instant
  messaging interoperability; Marque is email replacement with
  legal-tier proof.
* **Matrix / Signal / SimpleX** — real-time chat substrates. Marque
  is async-first and does not compete in the chat domain.
* **PGP / S/MIME** — earlier email E2EE attempts. Marque does not
  retrofit E2EE onto RFC 5322; it replaces the message format
  entirely and bridges.

# Security Considerations

This document is Informational. Normative security considerations
appear in each companion draft. Reviewers evaluating the Marque
architecture as a whole should concentrate on:

* **Identity compromise and recovery** — {{MARQUE-DID-MAIL}} §Security
  Considerations. Guardian threshold model; IRK cold-storage
  requirements; bounded blast radius from DSK compromise.
* **Metadata exposure** — {{MARQUE-CORE}} §Security Considerations
  and §Privacy Considerations. What each provider along the path
  learns at each privacy tier; cover-traffic requirements;
  mix-network assumptions.
* **Endpoint compromise** — {{MARQUE-STORAGE}} §Security
  Considerations. Archive-key storage is the most sensitive secret
  after the IRK. Secure deletion at provider-migration endpoints.
* **Legal-process response** — what a provider can surrender under
  compulsion (pseudonyms, timestamps, envelope sizes; at the Casual
  tier, sender and recipient DIDs) and what it cannot (content).

# Privacy Considerations

Summarized in {{MARQUE-CORE}} §Privacy Considerations. The
architectural commitments relevant at this document's level:

* Providers do not learn envelope content at any tier.
* At the Sealed privacy tier (RECOMMENDED default) the recipient's
  provider does not learn the recipient DID.
* At the Mixed privacy tier no single intermediary learns both
  sender and recipient.
* A passive global adversary may observe envelope size buckets and
  traffic-analyze across TLS at the IP layer; cover traffic is
  therefore mandatory at the provider-to-provider level.

# IANA Considerations

This document makes no IANA requests. Companion documents request:

* **{{MARQUE-CORE}}** — *Marque Algorithms* registry, *Marque
  Envelope Fields* registry, *Marque Block Types* registry.
* **{{MARQUE-DID-MAIL}}** — registration of the `did:mail` DID
  method in the W3C DID Method Registry.
* **{{MARQUE-MBS}}** — `core.*` block-type codepoints.
* **{{MARQUE-PROOF}}** — proof-tier identifiers and CAdES profile
  identifiers.
* **{{MARQUE-BRIDGE}}** — `marque://` URI scheme registration.
* **{{MARQUE-STORAGE}}** — `.marquebox` media type registration.

# Acknowledgments
{:numbered="false"}

The architecture in this document composes elements from the IETF
MIMI WG, the W3C DID WG, the ETSI ESI TC, RFC 9420 MLS, RFC 9000
QUIC, RFC 9591 FROST, RFC 9106 Argon2, {{OPENTIMESTAMPS}}, Sigstore,
KERI, and Signal Sealed Sender. The founding specification
{{MARQUE-FOUNDING}} provides the narrative rationale and extensive
deployment analysis.

--- back

# Change Log
{:numbered="false"}

## draft-tamim-marque-arch-00
{:numbered="false"}

* Initial submission. Informational architectural frame for the
  companion normative drafts.
