# Glossary and citation hub

> Two-purpose document. **Top half** is a quick lookup for every acronym you will meet in the spec, with a one-line definition and a link to the authoritative source. **Bottom half** is the cross-reference into the normative terminology in [`spec/protocol/01-terminology.md`](../spec/protocol/01-terminology.md), which is where these terms are defined for Marque's wire format.
>
> If you ever encounter a Marque term you do not recognise, this page should resolve it in one click.

## Cryptography and key management

| Term | One-line meaning | Learn more |
| ---- | ---------------- | ---------- |
| **MLS** | Messaging Layer Security — the IETF group-key-agreement protocol Marque uses for end-to-end encryption. | [RFC 9420][mls] · [MLS WG][mls-wg] |
| **TreeKEM** | The MLS group-key tree that gives `O(log n)` add/remove/update cost — what makes 50 000-member groups practical. | [RFC 9420 §5][mls] |
| **KeyPackage** | An MLS pre-key bundle a user publishes so others can add them to a group while they're offline. | [RFC 9420 §10][mls] |
| **HPKE** | Hybrid Public-Key Encryption — the `seal/open` primitive MLS and Marque's archive sync are built on. | [RFC 9180][hpke] |
| **X25519** | Elliptic-curve Diffie-Hellman key agreement on Curve25519. | [RFC 7748][rfc7748] |
| **Ed25519** | Elliptic-curve signature scheme on the same curve, deterministic and fast. | [RFC 8032][rfc8032] |
| **ML-KEM** (Kyber) | NIST FIPS 203 post-quantum key encapsulation. Marque uses `ML-KEM-768` in the hybrid suite. | [FIPS 203][mlkem] |
| **ML-DSA** (Dilithium) | NIST FIPS 204 post-quantum signature. Marque uses `ML-DSA-65` in the hybrid suite. | [FIPS 204][mldsa] |
| **X25519MLKEM768** | Hybrid KEM combining X25519 and ML-KEM-768 — Marque's post-quantum default for key agreement. | [TLS hybrid draft][hybrid-kem] |
| **FROST** | Flexible Round-Optimised Schnorr Threshold signatures — `t`-of-`n` signing without trusted dealer. Marque uses it for provider + QTSP co-signing at the Qualified tier. | [RFC 9591][frost] |
| **BLAKE3** | Fast Merkle-tree hash. Marque uses BLAKE3 for content addressing of attachments and message bodies. | [BLAKE3 spec][blake3] |
| **Argon2id** | Memory-hard password / proof-of-work function. Marque uses it for stranger-outreach PoW. | [RFC 9106][argon2] |

## Identity and DIDs

| Term | One-line meaning | Learn more |
| ---- | ---------------- | ---------- |
| **DID** | Decentralised Identifier — a URI scheme for cryptographic identifiers the user owns, with a resolvable document listing keys and service endpoints. | [W3C DID Core 1.0][did] |
| **`did:mail`** | The Marque DID method. Resolves over DNS-rooted `/.well-known/mail-did/` or the neutral `marque.id` portable-tier registry. | [`spec/protocol/02-identity.md`](../spec/protocol/02-identity.md) |
| **DID document** | The JSON document a DID resolves to: verification methods, service endpoints, and the `x-marque` Marque-specific block. | [W3C DID Core §3][did] |
| **KERI** | Key Event Receipt Infrastructure — the rotation-log discipline Marque borrows so a `did:mail` can rotate keys without trusting any provider. | [KERI white paper][keri] · [ToIP KSWG][toip-keri] |
| **KEL** | Key Event Log — the append-only signed log that records every rotation of an identity's keys. KERI's central data structure. | [KERI §6][keri] |
| **IRK** | Identity Root Key — the long-lived keypair that defines a Marque identity. Held in cold storage; rotates rarely. | [`protocol/02 §2.3`](../spec/protocol/02-identity.md) |
| **DSK** | Device Signing Key — a per-device working key signed by the IRK. Rotates per-device on compromise. | [`protocol/02 §2.3`](../spec/protocol/02-identity.md) |
| **`did:plc`** | Bluesky's DID method, with a 72-hour rotation nullification window. Marque's KEL borrows the nullification idea. | [did:plc spec][did-plc] |
| **`did:web`** | DNS-anchored DID method. The closest analogue to `did:mail`'s resolver path. | [did:web spec][did-web] |

## Legal proof, eIDAS and trust services

| Term | One-line meaning | Learn more |
| ---- | ---------------- | ---------- |
| **eIDAS 2.0** | Regulation (EU) 2024/1183 on electronic identification and trust services. The legal frame Marque's Qualified tier targets. Entered into force **20 May 2024**. | [Regulation (EU) 2024/1183][eidas] |
| **ERDS** | Electronic Registered Delivery Service — eIDAS Article 43 baseline tier. Marque's **Signed** chip maps here. | [eIDAS Art. 43][eidas] · [ETSI EN 319 521][etsi-en319521] |
| **QERDS** | **Q**ualified ERDS — eIDAS Article 44 high-assurance tier requiring a QTSP. Marque's **Registered** chip maps here. | [eIDAS Art. 44][eidas] |
| **QTSP** | Qualified Trust Service Provider — an entity accredited under eIDAS to issue qualified certificates and run qualified services. | [eIDAS Title III, Ch. 3][eidas] · [ETSI EN 319 411][etsi-en319411] |
| **TSA** | Time-Stamping Authority — issues RFC 3161 timestamps. Qualified TSAs additionally satisfy eIDAS Art. 42. | [ETSI EN 319 421][etsi-en319421] |
| **RFC 3161** | Internet X.509 Time-Stamp Protocol. The wire format Marque uses for TSA timestamps. | [RFC 3161][rfc3161] |
| **CAdES** | CMS Advanced Electronic Signatures — eIDAS-recognised signature container family. Marque's Qualified tier uses CAdES-B-B (baseline) and CAdES-B-LTA (long-term archive). | [ETSI EN 319 122][etsi-en319122] |
| **XAdES / PAdES** | XML / PDF analogues of CAdES. Used at the bridge edge when Marque output needs to land in legacy systems. | [ETSI EN 319 132][etsi-en319132] · [EN 319 142][etsi-en319142] |
| **OpenTimestamps (OTS)** | Aggregating timestamping protocol that anchors a Merkle root of submitted hashes into Bitcoin. Zero marginal cost per timestamp. Marque uses OTS for cheap long-term proof-of-existence. | [opentimestamps.org][ots] |
| **Sigstore / Rekor** | Public transparency log for code signatures. Marque's transparency log borrows the Rekor design pattern. | [Sigstore docs][rekor] |
| **PEC** | Posta Elettronica Certificata — Italy's regulated registered-electronic-mail system. ~16 M accounts, ~2.5 B messages/year, Court of Cassation recognition. The empirical existence proof for QERDS-grade mail at scale. | [AgID supervised services][agid] · [Wikipedia (it)][pec] |
| **AgID** | Agenzia per l'Italia Digitale — Italy's digital-government agency, the supervisory body for PEC. | [agid.gov.it][agid] |

## Transport and networking

| Term | One-line meaning | Learn more |
| ---- | ---------------- | ---------- |
| **QUIC** | Modern UDP-based transport with TLS 1.3 baked in. Marque's deposit and pull APIs run over QUIC. | [RFC 9000][quic] |
| **Hyperswarm** | UDP-holepunching peer discovery used by Holepunch / Pear runtime. Marque borrows it for the optional direct-P2P delivery path. | [holepunch.to/hyperswarm][hyperswarm] |
| **Nym** | Sphinx-based mixnet. Marque's optional Mixed privacy tier rides Nym for sender-anonymity. | [nymtech.net][nym] |
| **Sphinx packet** | Onion-encryption packet format Nym is built on. | [Sphinx paper (Danezis & Goldberg, 2009)][sphinx] |
| **SimpleX** | Metadata-private message-queue protocol with no long-lived user identifier on the wire. Inspired Marque's recipient-pseudonym design. | [simplex.chat][simplex] |
| **Autocrypt** | Opportunistic key-discovery header for legacy email. Marque's SMTP bridge consumes Autocrypt headers as one of the four hybrid discovery methods. | [autocrypt.org][autocrypt] |

## Standards bodies and process

| Term | One-line meaning | Learn more |
| ---- | ---------------- | ---------- |
| **IETF** | Internet Engineering Task Force — the standards body Marque targets for its normative drafts. | [ietf.org][ietf] |
| **MIMI WG** | More Instant Messaging Interop Working Group — the IETF WG defining MLS-over-async-transport. Marque uses MIMI's transport profile. | [MIMI WG][mimi] |
| **W3C DID WG** | The W3C Decentralized Identifier Working Group, custodian of the DID Core spec. | [W3C DID WG][did-wg] |
| **ETSI ESI** | European Telecommunications Standards Institute, Electronic Signatures and Infrastructures — the body that produces the EN 319 / TS 119 standards Marque maps against. | [ETSI ESI][etsi-esi] |
| **ToIP** | Trust over IP Foundation — hosts the KERI Suite Working Group (KSWG). | [trustoverip.org][toip] |
| **Conventional Commits** | The structured commit-message convention this repository enforces in CI. | [conventionalcommits.org][cc] |
| **kramdown-rfc** | The Markdown-to-XML toolchain used to compile `drafts/*.md` into IETF-submittable XML. | [kramdown-rfc][kramdown-rfc] |
| **MBS** | Marque Block Spec — the typed, signed, schema-validated content format that replaces HTML in Marque messages. | [`spec/protocol/06-content.md`](../spec/protocol/06-content.md) |
| **MIP** | Marque Improvement Proposal — the unit of normative change in this repository. | [MIP-0001](../mips/mip-0001-process.md) |
| **`.marquebox`** | Portable archive-export ZIP container; satisfies GDPR data-portability obligations. | [`protocol/03 §3.10`](../spec/protocol/03-architecture.md) |

## Marque-internal terminology

The user-facing lexicon (chips, verbs, naming rules) and the wire-format terminology (envelope, `auth_mode`, `privacy_tier`, `ProofEnvelope`, capability record, self-group, archive key) are defined normatively in [`spec/protocol/01-terminology.md`](../spec/protocol/01-terminology.md). Quick jumps:

- [User-facing terms](../spec/protocol/01-terminology.md#3-user-facing-terms) — message, send, reply, forward, mark as, handle, Room.
- [Tier chips](../spec/protocol/01-terminology.md#4-tier-chips) — *(none)*, **Signed**, **Registered**.
- [Bridge and interop chips](../spec/protocol/01-terminology.md#5-bridge-and-interop-chips) — `bridged · encrypted`, `email`, `caution`.
- [Technical / wire-format terms](../spec/protocol/01-terminology.md#6-technical-terms) — envelope, `auth_mode`, `privacy_tier`, cipher suite, recipient pseudonym, pad bucket, home provider, ProofEnvelope, capability record, self-group, archive key, MBS, Room, `.marquebox`, portable-tier handle.
- [Grammar rules for specification authors](../spec/protocol/01-terminology.md#7-grammar-rules-for-specification-authors).

---

<!-- Authoritative external sources, reused as link references throughout the repo. -->

[mls]: https://datatracker.ietf.org/doc/html/rfc9420
[mls-wg]: https://datatracker.ietf.org/wg/mls/about/
[hpke]: https://datatracker.ietf.org/doc/html/rfc9180
[rfc7748]: https://datatracker.ietf.org/doc/html/rfc7748
[rfc8032]: https://datatracker.ietf.org/doc/html/rfc8032
[mlkem]: https://csrc.nist.gov/pubs/fips/203/final
[mldsa]: https://csrc.nist.gov/pubs/fips/204/final
[hybrid-kem]: https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/
[frost]: https://datatracker.ietf.org/doc/rfc9591/
[blake3]: https://github.com/BLAKE3-team/BLAKE3
[argon2]: https://datatracker.ietf.org/doc/html/rfc9106

[did]: https://www.w3.org/TR/did-core/
[did-wg]: https://www.w3.org/2019/did-wg/
[did-plc]: https://github.com/did-method-plc/did-method-plc
[did-web]: https://w3c-ccg.github.io/did-method-web/
[keri]: https://arxiv.org/abs/1907.02143
[toip-keri]: https://trustoverip.org/working-groups/keri/

[eidas]: https://eur-lex.europa.eu/eli/reg/2024/1183/oj
[etsi-en319521]: https://www.etsi.org/deliver/etsi_en/319500_319599/319521/
[etsi-en319411]: https://www.etsi.org/deliver/etsi_en/319400_319499/31941101/
[etsi-en319421]: https://www.etsi.org/deliver/etsi_en/319400_319499/319421/
[etsi-en319122]: https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/
[etsi-en319132]: https://www.etsi.org/deliver/etsi_en/319100_319199/31913201/
[etsi-en319142]: https://www.etsi.org/deliver/etsi_en/319100_319199/31914201/
[etsi-esi]: https://www.etsi.org/committee/esi
[rfc3161]: https://datatracker.ietf.org/doc/html/rfc3161
[ots]: https://opentimestamps.org
[rekor]: https://docs.sigstore.dev/logging/overview/
[pec]: https://it.wikipedia.org/wiki/Posta_elettronica_certificata
[agid]: https://www.agid.gov.it/en/news/digital-identity-pec-qualified-electronic-signature-online-report-on-agid-supervised-services

[quic]: https://datatracker.ietf.org/doc/html/rfc9000
[hyperswarm]: https://github.com/holepunchto/hyperswarm
[nym]: https://nymtech.net/
[sphinx]: https://cypherpunks.ca/~iang/pubs/Sphinx_Oakland09.pdf
[simplex]: https://simplex.chat
[autocrypt]: https://autocrypt.org/

[ietf]: https://www.ietf.org/
[mimi]: https://datatracker.ietf.org/wg/mimi/about/
[toip]: https://trustoverip.org/
[cc]: https://www.conventionalcommits.org/en/v1.0.0/
[kramdown-rfc]: https://github.com/cabo/kramdown-rfc
