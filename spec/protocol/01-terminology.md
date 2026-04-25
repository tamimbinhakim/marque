# Terminology

> Normative reference for every term used in the Marque specification. Two registers: **user-facing** terms for UI and product copy, and **technical** terms for wire format, schemas, and drafts.

**You will learn**

- The user-facing lexicon and the single rule that makes it collision-free.
- The technical vocabulary used throughout the protocol sections.
- The RFC 2119 convention.

---

## 1. RFC 2119 language

Normative keywords — **MUST**, **MUST NOT**, **SHALL**, **SHOULD**, **SHOULD NOT**, **MAY**, **RECOMMENDED**, **OPTIONAL** — appear in uppercase and carry the meanings of [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174). Lowercase uses of these words carry their ordinary English meaning.

## 2. Naming principle

Marque follows the Signal / WhatsApp pattern: the brand names the platform, not the unit. Users send **messages** on Marque, not "marques."

The rule exists to eliminate a concrete UX collision. The email flag-verb family — **mark as read**, **mark as unread**, **mark as spam** — is homophonic with *marque*. If Marque were also a verb, every screen-reader user and every voice-assistant user would face ambiguity. By keeping *Marque* strictly as a platform noun and using plain English verbs for actions, every UI surface is unambiguous.

Users will naturally say "I'll Marque you" the way they say "I'll Slack you" or "I'll Venmo you" in speech. The protocol, specification, and UI never require it.

## 3. User-facing terms

These terms are normative for UI copy and product documentation.

| Concept | Word | Example |
|---|---|---|
| The platform / protocol | **Marque** *(capitalized, proper noun)* | "Sign in to Marque." |
| The message unit | **message** *(lowercase)* | "Alice sent you a message." · "3 unread messages." |
| The verb for sending | **send** | "Send Bob a message." Never "marque Bob." |
| The verb for replying | **reply** · **reply all** | Unchanged from email convention. |
| The verb for forwarding | **forward** | Unchanged from email convention. |
| Flag a message | **mark as …** | "Mark as unread." · Unchanged from email convention. |
| Inbox / Sent / Archive / Drafts / Spam / Trash | unchanged | Preserve user intuition. |
| Attachment | **attachment** | Familiar email term wins over "enclosure." |
| Thread | **thread** | A persistent back-and-forth. Backed by an MLS group. |
| Address — casual | **handle** | "What's your Marque handle?" |
| Address — technical | `did:mail:` identifier | Used in drafts, schemas, resolvers. Never user-visible by default. |
| Shared / team mailbox | **Room** | `support@company` is a Room. See [protocol/04](./04-rooms.md). |

## 4. Tier chips

Every message carries an **auth-mode** choice. The auth-mode determines which chip renders in the UI.

| Internal auth-mode | User-facing chip | Glyph | Legal status (eIDAS) |
|---|---|---|---|
| `CASUAL` | *(no chip)* | — | Ordinary message. |
| `SIGNED` | **Signed** | pen / signature | ERDS (Art. 43). |
| `QUALIFIED` | **Registered** | blue scales | QERDS (Art. 44). |

**Registered** is the user-visible label for the qualified tier. Registered-grade proof is available to any user as a per-message option; not every message is Qualified.

## 5. Bridge and interop chips

The SMTP bridge renders one of these chips on every bridged row:

| State | Chip | Glyph |
|---|---|---|
| Received via SMTP bridge, encrypted on ingress | **bridged · encrypted** | green up-arrow |
| Received via SMTP bridge, delivered plaintext | **email** | gray envelope |
| Outbound downgrade, first time per recipient | *(pre-send modal)* | amber caution |
| Bridged sender with failing DKIM / DMARC | **caution** | red triangle |

Native Marque messages at `CASUAL` auth-mode render no chip.

## 6. Technical terms

These appear throughout the `protocol/` sections and the Internet-Drafts. They are not intended for end-user surfaces.

| Term | Meaning | First defined in |
|---|---|---|
| **envelope** | Outer CBOR wire unit carrying a ciphertext payload, routing metadata, and signature. | [protocol/03 §3.2](./03-architecture.md), [`schemas/cddl/envelope.cddl`](../../schemas/cddl/envelope.cddl) |
| **auth_mode** | Per-message tier enum: `CASUAL`, `SIGNED`, `QUALIFIED`. | [protocol/05 §5.3](./05-cryptography.md) |
| **privacy_tier** | Per-message metadata-privacy selection: `CASUAL`, `SEALED`, `MIXED`. | [protocol/05 §5.5](./05-cryptography.md) |
| **cipher suite** | Registered (signature, KEM, hash) tuple. | [protocol/05 §5.1](./05-cryptography.md) |
| **did:mail** | The Marque DID method. | [protocol/02 §2.2](./02-identity.md) |
| **IRK** / **DSK** / **KEL** | Identity Root Key / Device Signing Key / Key Event Log. | [protocol/02 §2.3](./02-identity.md) |
| **KeyPackage** | MLS pre-key per [RFC 9420](https://datatracker.ietf.org/doc/html/rfc9420). | [protocol/05 §5.2](./05-cryptography.md) |
| **recipient pseudonym** | Per-epoch HMAC identifier in place of plaintext recipient. | [protocol/03 §3.3](./03-architecture.md) |
| **pad bucket** | One of six fixed on-wire sizes: `4KB`, `16KB`, `64KB`, `256KB`, `1MB`, `4MB`. | [protocol/05 §5.6](./05-cryptography.md) |
| **home provider** | The lowest-priority `MarqueMailbox` service endpoint in a user's DID document. | [protocol/03 §3.1](./03-architecture.md) |
| **ProofEnvelope** | Non-repudiation wrapper for `SIGNED` and `QUALIFIED` messages. | [protocol/07](./07-legal-proof.md) |
| **capability record** | Provider manifest at `/.well-known/marque-provider.json`. | [protocol/03 §3.6](./03-architecture.md) |
| **self-group** | Sender's single-member MLS group used to encrypt the Sent-copy. | [protocol/03 §3.8](./03-architecture.md) |
| **archive key** | Per-identity key used to encrypt cross-device cache deltas. Distinct from MLS group keys. | [protocol/03 §3.9](./03-architecture.md) |
| **MBS** | Marque Block Spec — typed, signed, schema-validated content format. | [protocol/06](./06-content.md) |
| **Room** | First-class identity for non-natural-person mailboxes (shared, team, list, delegation, org). | [protocol/04](./04-rooms.md) |
| **`.marquebox`** | Portable archive-export format. | [protocol/03 §3.10](./03-architecture.md) |
| **portable-tier handle** | A `did:mail` under a neutral registry (`marque.id`) that survives any provider change. | [protocol/02 §2.4](./02-identity.md) |

## 7. Grammar rules for specification authors

1. **"Marque"** is a proper noun. Never a verb in UI copy or normative language. Never pluralized as "marques" in UI copy.
2. **"message"** is the unit noun, always lowercase.
3. **"send"** is the verb for originating a message. Never coin "to marque."
4. **"mark as …"** is the flag-verb family from email. Always remains unambiguous.
5. **"Registered"** is the QUALIFIED-tier adjective. Applied to messages ("a Registered message"), to the act of sending ("send Registered"), and to the UI chip.
6. **"Signed"** is the SIGNED-tier adjective. Distinguishable from Registered by both glyph and label.
7. **Localizations of "Registered"** follow each jurisdiction's registered-mail term: *raccomandata*, *Einschreiben*, *recommandée*, *заказное*, *書留*.

## 8. Example copy

> **Inbox** · 12 unread · 1 **Registered**
>
> ---
>
> Alice Chen · `alice@marque.id` · **Registered**
> *Q2 budget — please review*
> Registered · 23 Apr 2026 · 2 attachments
>
> \[ Reply \] \[ Reply all \] \[ Forward \] \[ Mark as unread \] \[ Archive \]

Every action on this screen reads unambiguously — the flag verb "Mark as" and the platform noun "Marque" never occupy the same grammatical slot.

---

Continue to [**Identity**](./02-identity.md).
