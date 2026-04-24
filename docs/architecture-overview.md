# Architecture overview

> Visual and prose walkthrough of the Marque system at two levels: the **identity stack** (who is Alice) and the **message path** (how a letter travels from Alice to Bob). Non-normative; for normative detail see [`spec/protocol/03-architecture.md`](../spec/protocol/03-architecture.md) and [`spec/protocol/05-cryptography.md`](../spec/protocol/05-cryptography.md).

---

## 1. The identity stack

A Marque identity is three layers of keys plus a DID document. The DID document is resolved via DNS by default; the keys are rotated and recoverable.

```mermaid
flowchart TB
    DID["<b>DID Document</b><br/>did:mail:example.com:alice<br/><i>served at</i><br/>/.well-known/mail-did/alice/did.json"]
    IRK["<b>Identity Root Key (IRK)</b><br/>— years —<br/>hardware / cold storage<br/>answers: <i>who is Alice?</i>"]
    KEL["<b>KEL</b><br/>append-only<br/>signed rotation log<br/>(KERI-style)"]
    DSK1["<b>DSK — phone</b><br/>— months —<br/>TPM / Secure Enclave"]
    DSK2["<b>DSK — laptop</b><br/>— months —<br/>TPM / Secure Enclave"]
    EPOCH["<b>MLS ratchet keys</b><br/>— per-epoch, ephemeral —<br/>forward-secret message encryption"]

    DID -->|lists service endpoints,<br/>verificationMethods, x-marque| IRK
    IRK -->|signs rotations| KEL
    IRK -->|signs DSK certificate| DSK1
    IRK -->|signs DSK certificate| DSK2
    DSK1 -->|joins MLS group,<br/>signs per-epoch commits| EPOCH
    DSK2 -->|joins MLS group,<br/>signs per-epoch commits| EPOCH

    classDef root fill:#eef2ff,stroke:#4f46e5,stroke-width:2px;
    classDef device fill:#fef3c7,stroke:#b45309,stroke-width:1px;
    classDef epoch fill:#ecfdf5,stroke:#047857,stroke-width:1px;
    class IRK root
    class KEL root
    class DSK1 device
    class DSK2 device
    class EPOCH epoch
```

_Source: [`diagrams/identity-stack.mmd`](./diagrams/identity-stack.mmd)._

**Why three tiers:**

- The IRK is the answer to "who is Alice" — it is what her correspondents trust.
- The DSK is the answer to "which device is signing" — compromising a phone does not compromise the identity; the DSK cert is revoked by publishing a KEL rotation.
- The MLS ratchet keys are the answer to "which epoch is this message in" — they provide forward secrecy without being tied to long-term identity.

**Recovery:**

1. **Primary** — device-resident DSK.
2. **Social** — 3-of-5 guardians, 72-hour delay, primary-device notification.
3. **Optional** — 24-word phrase for power users.

---

## 2. The message path

Alice wants to send a message to Bob (who also has a `did:mail`). Three possible paths; the client picks the first that succeeds.

```mermaid
sequenceDiagram
    autonumber
    participant A as Alice client
    participant R as did:mail resolver
    participant PB1 as Bob provider A
    participant PB2 as Bob provider B
    participant NYM as Nym mixnet
    participant B as Bob client

    A->>R: Resolve did:mail:example.com:bob
    R-->>A: DID doc — service endpoints + KeyPackages

    rect rgba(79, 70, 229, 0.08)
    note over A,B: (1) Direct P2P — 500 ms budget
    A-)B: Hyperswarm UDP holepunch
    B--)A: If both online and policy permits IP exposure
    end

    rect rgba(4, 120, 87, 0.08)
    note over A,B: (2) Store-and-forward via 2-of-3 mailboxes
    A->>PB1: Deposit (ciphertext envelope)
    A->>PB2: Deposit (ciphertext envelope)
    Note over PB1,PB2: Providers store opaque blobs. No content access.
    B->>PB1: Pull when online
    PB1-->>B: Envelope
    end

    rect rgba(180, 83, 9, 0.08)
    note over A,B: (3) Metadata-sensitive path — Mixed tier
    A->>NYM: Sphinx-wrapped envelope
    NYM->>PB1: Relay (10–30 s per hop)
    B->>PB1: Pull
    PB1-->>B: Envelope
    end
```

_Source: [`diagrams/message-path.mmd`](./diagrams/message-path.mmd)._

**The envelope.**

```
┌────────────────────────── CBOR envelope ──────────────────────────┐
│ recipient_pseudonym: HMAC(bob_key, epoch)                         │
│ sender_cert:         DSK cert chained to Alice's IRK              │
│ auth_mode:           CASUAL | SIGNED | QUALIFIED                  │
│ timestamp:           RFC 3161 + OpenTimestamps (Attested+)        │
│ ciphertext:                                                       │
│   ┌───────────── MLS AppMessage (encrypted) ─────────────┐       │
│   │ alt_plaintext:  "Bob, budget attached..."            │       │
│   │ locales:        ["en", "fr"]                         │       │
│   │ blocks:                                              │       │
│   │   - core.text:      "..."                            │       │
│   │   - core.attachment: BLAKE3 CID, source list         │       │
│   │   - core.schedule:   proposed slots                  │       │
│   └──────────────────────────────────────────────────────┘       │
│ pad_to:              4KB | 16KB | 64KB | 256KB | 1MB | 4MB       │
└───────────────────────────────────────────────────────────────────┘
```

**What the provider sees:**

| Metadata tier | Provider sees |
|---|---|
| Casual | Sender DID, recipient DID, timestamps. |
| **Sealed (default)** | Recipient pseudonym, delivery token, timestamps. |
| Mixed | Only the pseudonym; routing via Nym mix hops. |

---

## 3. The provider's job

A Marque provider is a **dumb encrypted mailbox**. It is NOT a conversation state machine.

```mermaid
flowchart LR
    S([Sender client]) -->|QUIC deposit| IN[Ingress]
    IN --> V{Validate<br/><br/>• DID lists this<br/>provider<br/>• anti-spam policy<br/>• size / rate limits}
    V -->|accept| STORE[(Ciphertext store<br/><br/>opaque blobs<br/>≥ 30 day retention)]
    V -->|reject| ERR[Structured<br/>policy error]
    ERR -.->|back to sender| S

    STORE --> FETCH[Pull / long-poll API<br/>authenticated by DSK cert]
    FETCH -->|envelope| C([Recipient client])

    STORE --> TLOG[Transparency log<br/>Sigstore-style + OTS]
    NOTARY([Notary /<br/>QTSP FROST signer]) -. co-sign .-> TLOG
    STORE -.->|reputation<br/>bulletins| G[Federated<br/>pull-gossip]
    G -.- STORE

    subgraph NEVER [" NEVER does "]
        direction TB
        N1[Read content]
        N2[Own conversation state]
        N3[See social graph<br/>beyond immediate deposit]
    end
    STORE -. red lines .- NEVER

    classDef never fill:#fef2f2,stroke:#b91c1c,stroke-width:1px,color:#7f1d1d;
    classDef store fill:#eef2ff,stroke:#4f46e5,stroke-width:2px;
    class NEVER,N1,N2,N3 never
    class STORE store
```

_Source: [`diagrams/provider-role.mmd`](./diagrams/provider-role.mmd)._

**What this gives providers:**

- Storage scales as `Σ (envelopes × ciphertext_size)`. Not as `Σ (users × rooms × membership × history)`.
- Compliance, storage, AI inference, transactional APIs are their revenue streams (Part 9).
- Portable identity means they compete on service, not lock-in.

---

## 4. The legal-proof layer

For the Attested and Certified tiers, the envelope carries a `ProofEnvelope`:

```mermaid
flowchart TB
    subgraph CASUAL ["CASUAL auth_mode — ordinary email"]
        M[Message content]
    end

    subgraph SIGNED ["SIGNED auth_mode — Attested (ERDS, Art. 43)"]
        H["Canonical SHA-256 hash"]
        S1["Sender signature<br/>Ed25519 + ML-DSA-65"]
        CC["DSK certificate chain<br/>DSK → IRK (KEL-anchored)"]
        TSA["RFC 3161 timestamp<br/>+ OpenTimestamps proof"]
        TL["Transparency log entry<br/>(Rekor-style)"]
    end

    subgraph QUALIFIED ["QUALIFIED auth_mode — Certified (QERDS, Art. 44)"]
        QC["Qualified certificate<br/>from eIDAS-listed QTSP<br/>(CAdES-B-B / B-T)"]
        FR["FROST threshold signatures<br/>provider + QTSP co-sign"]
        AT["Archive timestamps<br/>re-anchored every 2 yr<br/>(CAdES-B-LTA)"]
    end

    M --> H --> S1 --> CC
    CC --> TSA
    TSA --> TL
    TL --> QC
    QC --> FR
    FR --> AT

    classDef casual fill:#f5f5f5,stroke:#737373;
    classDef signed fill:#eef2ff,stroke:#4f46e5;
    classDef qualified fill:#ecfdf5,stroke:#047857,stroke-width:2px;
    class M casual
    class H,S1,CC,TSA,TL signed
    class QC,FR,AT qualified
```

_Source: [`diagrams/proof-envelope.mmd`](./diagrams/proof-envelope.mmd)._

This is the structure a court sees. The verifier:

1. Checks the sender signature against the DSK cert chain.
2. Verifies the TSA timestamp against a trusted trust list.
3. Verifies the FROST threshold — one QTSP-listed signer is required at Certified tier.
4. Verifies the OpenTimestamps proof against any Bitcoin full node.
5. Confirms the archive timestamp chain is unbroken to the present.

**No trusted third party is required** for step 4 — OpenTimestamps is verifiable against public Bitcoin state alone. This is the property RFC 3161 TSAs cannot provide.

---

## 5. The SMTP bridge

Bridges are tier-zero operations during the 10–20 year transition. Their job is to preserve reply continuity with `@gmail.com` and label honestly.

```mermaid
flowchart LR
    GM[["Legacy sender<br/>alice@gmail.com"]] -->|SMTP| BR[Bridge MTA]

    BR --> D1{Sender key<br/>discoverable?}

    subgraph DISCOVERY ["Four hybrid discovery methods"]
        direction TB
        M1[DID resolver<br/><i>primary</i>]
        M2[DNS TXT<br/>_marque.legacy.com]
        M3[/.well-known/<br/>marque-did/]
        M4[Autocrypt header<br/>on prior mail]
    end
    D1 -.-> DISCOVERY

    D1 -->|yes| ENC[Wrap in MLS<br/><b>ENCRYPTED-BRIDGED</b><br/>green upgrade arrow chip]
    D1 -->|no| PT[PLAINTEXT-BRIDGED<br/>gray envelope chip<br/>+ warning]

    ENC --> MB[(Marque mailbox)]
    PT --> MB
    MB --> RCPT[["Marque recipient<br/>bob · did:mail:example.com:bob"]]

    classDef legacy fill:#fafaf9,stroke:#78716c;
    classDef bridge fill:#eef2ff,stroke:#4f46e5,stroke-width:2px;
    classDef enc fill:#ecfdf5,stroke:#047857;
    classDef plain fill:#fef3c7,stroke:#b45309;
    class GM legacy
    class BR bridge
    class ENC enc
    class PT plain
```

_Source: [`diagrams/smtp-bridge.mmd`](./diagrams/smtp-bridge.mmd)._

UX chips (full vocabulary in [`lexicon.md`](./lexicon.md)):

- *(no chip)* — native Marque message, `CASUAL` auth-mode.
- **Signed** (pen glyph) — native, `SIGNED` auth-mode (Attested tier, eIDAS ERDS).
- **Registered** (blue scales) — native, `QUALIFIED` auth-mode (Certified tier, eIDAS QERDS).
- **bridged · encrypted** (green up-arrow) — inbound SMTP, MLS-wrapped on ingress.
- **email** (gray envelope) — inbound SMTP, plaintext.
- *Pre-send amber modal* — first outbound downgrade to a given SMTP recipient.
- **caution** (red triangle) — bridged sender with failing DKIM / DMARC.

---

## 6. Reading order

If you are implementing, read the specification in the order under [`spec/protocol/`](../spec/protocol/):

1. [Terminology](../spec/protocol/01-terminology.md) — lexicon and technical vocabulary.
2. [Identity](../spec/protocol/02-identity.md) — `did:mail`, keys, KEL, recovery, onboarding.
3. [Architecture](../spec/protocol/03-architecture.md) — envelope, delivery, providers, archive, sync.
4. [Rooms](../spec/protocol/04-rooms.md) — shared mailboxes, teams, mailing lists.
5. [Cryptography](../spec/protocol/05-cryptography.md) — primitives, MLS profile, privacy tiers.
6. [Content](../spec/protocol/06-content.md) — MBS blocks, lifecycle, reply/forward/CC/BCC.
7. [Legal proof](../spec/protocol/07-legal-proof.md) — ProofEnvelope, tiers, non-delivery states.
8. [Anti-spam](../spec/protocol/08-anti-spam.md) — economic + cryptographic policy.
9. [Interop](../spec/protocol/09-interop.md) — SMTP bridge.
10. [Conformance](../spec/protocol/10-conformance.md) — MUST/SHOULD matrix and self-test.

For motivation, commerce, and strategy see [`spec/context/`](../spec/context/).
