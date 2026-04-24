# Content — the Marque Block Spec and message lifecycle

> The content layer and the operations that act on it. Typed JSON blocks replace HTML's frozen 2005 subset. Reply, Reply-All, Forward, BCC, and thread operations are specified concretely so implementers don't have to guess.

**You will learn**

- The MBS design axioms and the core block vocabulary.
- The WASM widget sandbox and why it succeeds where AMP for Email failed.
- Content addressing for attachments via BLAKE3 CIDs.
- Reply vs Reply-All semantics over MLS groups.
- Forward in three flavors — Quoted, Attested, Certified — and the `forwardable` and `quotable` permission flags.
- BCC reply-exposure prevention and scale limits.
- Thread membership authorization, split, fork, merge.
- Scheduled send, undo send, mute, snooze.

---

## 6.1 Why HTML email is stuck

HTML email is frozen in 2005 for three structural reasons:

1. **No JavaScript, ever** — correctly — because scripts in mailboxes would be a drive-by XSS nightmare.
2. **Each client renders a different CSS subset** per [caniemail.com](https://www.caniemail.com)'s maintained matrix.
3. **Outlook on Windows still uses Microsoft Word's rendering engine in 2025**. Grid, flex, `background-image`, `border-radius`, and essentially all CSS since 2010 simply do not exist there.

MJML, Foundation for Emails, and HEML exist to transpile cleaner syntax down to the nested-table soup every client accepts. AMP for Email failed in 2019–2024 because it required Google-gated sender whitelisting, Microsoft never shipped it in Outlook, and developer apathy sealed the coffin.

**The AMP for Email post-mortem is the central cautionary tale:** a rich-content model gated by a single vendor will not survive.

## 6.2 The Marque Block Spec (MBS)

MBS is JSON-based, schema-validated, and extensible via a reverse-DNS namespace registry (`core.text`, `core.poll`, `com.example.widget.chart`).

### Design axioms

1. **Deterministic rendering.** A conformance test suite pins every core block's rendering to byte-stable output per viewport.
2. **Accessibility is not optional.** Media blocks require `a11y` metadata; schema validation rejects messages missing required fields.
3. **Dark mode is native** via semantic color roles (`surface`, `accent`, `danger`) resolved by client theme.
4. **Localization is native** — `locales` at the envelope level, per-locale content in text blocks.
5. **Render-capability negotiation** — recipients advertise supported block types; unsupported blocks render their mandatory `fallback` field (typically a `core.text`), with the envelope's `alt_plaintext` as the ultimate fallback.

### Core block types

| Block | Purpose |
|---|---|
| `core.text` | CommonMark + GFM tables, task lists, footnotes. Math as **MathML** (not KaTeX markup) for screen-reader accessibility. |
| `core.image`, `core.video`, `core.audio` | Content-addressed via BLAKE3 CIDs. **No external URLs, no tracking pixels by design.** `a11y` mandatory. |
| `core.attachment` | First-class content-addressed blobs with BLAKE3 Merkle delta sync. |
| `core.diagram` | Mermaid or Graphviz source, rendered client-side. |
| `core.poll` | Votes flow back as signed structured replies. |
| `core.schedule` | **Kills Calendly.** Recipients tap a proposed slot; `.ics` flows back automatically. |
| `core.form` | Signed submissions. |
| `core.payment_request` | PSP-neutral: ISO 20022, SEPA Request-to-Pay, FedNow, Stripe/Adyen tokens, or optional Lightning. **Crypto is one backend among many, not a requirement.** |
| `core.reaction` | Lightweight, delta-referencing message hashes. |
| `core.reply` | Structured reply with verifiable quote ranges — see §6.5. |
| `core.forward` | Three flavors with permission enforcement — see §6.6. |
| `core.retract` | Signed amendment disavowing a prior message — see [`protocol/07 §7.8`](./07-legal-proof.md). |
| `core.signed_action` | Non-repudiable recipient acknowledgment — the DocuSign replacement primitive. |
| `core.embed` | Consent-fetched rich embeds: one click to preview, no automatic correlation pings. |
| `core.edit` | Append-only amendments — distinct from `core.retract`. |
| `core.wasm_widget` | Optional sandboxed interactive component — see §6.3. |

## 6.3 WASM widgets

Interactive blocks need a sandbox. AMP for Email tried social gating; Marque uses technical gating.

**Capabilities are default-deny:**

- No network.
- No persistent storage.
- No DOM.
- No JavaScript interop.
- No wall-clock.
- Only host-supplied randomness.
- CPU and memory caps enforced by the host runtime (e.g. Wasmtime).
- Only user-visible interactions drive events.

The widget exports `render(ctx)` returning an **MBS block tree** — it cannot produce arbitrary HTML, only more MBS blocks.

**Unlike AMP for Email, the sandbox is technical, not social.** Anyone can author a widget; the capability surface is so minimal that malicious widgets cannot do more than display ugly content or spin capped CPU.

Clients render a visible *"interactive widget by sender"* chrome. Clients preferring conservatism simply decline the optional type; the block falls back to its `fallback` field.

## 6.4 Attachments and content addressing

Attachments are **BLAKE3-256 CIDs over canonical bytes**. Storage is out-of-band; a `sources[]` list specifies where to fetch:

- Sender's provider.
- Recipient's provider.
- Public IPFS-like DHT.

Attachment ciphertext is encrypted to the MLS group key. **No protocol-level size limit**; providers advertise their own in capability records.

**Delta sync** uses the BLAKE3 Merkle diff against a prior CID — design files and datasets evolve efficiently without retransmitting unchanged chunks.

**The `marque://` URI scheme** is registered for `sources[]` entries: `marque://<provider-domain>/blob/<cid>`.

## 6.5 Reply and Reply-All

A **Reply** continues an existing thread:

- A Reply is an MLS `Application` message in the current epoch of the thread's MLS group.
- The envelope's `thread_id` is the existing group's `group_id`.
- The payload typically starts with a `core.reply` block carrying the quoted range.

**Reply does not change thread membership.** All existing members remain members and receive the reply. The originating member of the quoted message may or may not be the only `To:`-line recipient in display, but cryptographically every group member receives the ciphertext.

A Reply MAY transition the thread's `auth_mode` — for example, replying at `SIGNED` to a `CASUAL` thread. Clients render the per-message chip; the thread as a whole does not have a single tier.

**Reply-All** has identical MLS semantics to Reply — the message is sent to the existing MLS group. The user-facing distinction is primarily a **display choice**: *"also CC these people who are currently on the thread?"*

If the user **removes** an existing group member in the Reply-All compose window, the client MUST initiate an MLS `Remove` proposal + `Commit`, producing a new epoch where the removed member no longer has access. This is a **thread split** (§6.9).

BCC recipients are not MLS members of the primary group and do not receive Reply-All by default; they are cryptographically invisible to the Reply-All sender.

## 6.6 Forward — three flavors

Forward has no single correct semantic. Marque defines three distinct flavors; the sender picks at compose time.

### Quoted forward (default, always permitted)

A Quoted forward is a **new message** embedding prior content as a quoted block:

- The new message is composed by the forwarder; the forwarder signs under their own DSK.
- **The original sender's signature is NOT carried.**
- **The original ProofEnvelope is NOT carried.**
- The forwarded content appears in a `core.quote` block referencing the original `content_id`.
- Recipients see *"Bob forwarded: [quoted text from Alice]"* — attributed to Bob's forwarding assertion, not cryptographically to Alice.

Quoted forward is always permitted. It is the moral equivalent of copy-pasting.

### Attested forward (requires sender consent)

An Attested forward preserves the original sender's signature and ProofEnvelope:

- The original envelope is carried intact inside a `core.attested_forward` block.
- The block references the original `ProofEnvelope` (inline or by URL).
- The forwarder signs the outer envelope; the inner ProofEnvelope proves the original send.
- **Requires `forwardable: attested` in the original message.** Clients MUST refuse the operation otherwise.
- The chain is bounded: each forward adds to `forward_chain[]`; clients MAY refuse chains longer than 8 hops.

### Certified forward (Qualified tier only)

A Certified forward preserves the Qualified tier through the forward — recipients see an unbroken chain of Qualified signatures from original sender through each forwarder:

- Requires the original was `QUALIFIED` AND had `forwardable: qualified_only`.
- Requires the forwarder is legal-person (or legal-person-authorized Room) capable of Qualified signing.
- The forward itself is `QUALIFIED`.
- The chain-of-custody is legally admissible in eIDAS jurisdictions for the duration of its archive-timestamp lifetime.

Certified forward is how legal-workflow tools (evidence handoff, contract routing) operate over Marque.

## 6.7 `forwardable` and `quotable` permission flags

Every message carries per-message permission flags at the envelope level:

```
forwardable = "no" | "quoted_only" | "attested" | "qualified_only"
quotable    = "no" | "yes"
```

**Defaults:**

| Tier | `forwardable` | `quotable` |
|---|---|---|
| `CASUAL` | `attested` | `yes` |
| `SIGNED` | `attested` | `yes` |
| `QUALIFIED` | `qualified_only` | `no` (explicit opt-in required) |

**Enforcement:**

- `forwardable: no` — clients MUST NOT perform attested or certified forward. Quoted forward (copy-paste) cannot be technically prevented; recipients' clients SHOULD surface *"sender requested no forward"* when quoted forward is attempted.
- `forwardable: quoted_only` — permits quoted; forbids attested and certified.
- `forwardable: attested` — permits attested and quoted; forbids certified unless the content itself is Qualified.
- `forwardable: qualified_only` — permits certified forward only between Qualified-capable identities.
- `quotable: no` — clients SHOULD refuse to expand the quote range in `core.reply`.

These are the same class of signal as Signal's screenshot-warning or disappearing-messages: **trust-the-recipient's-client** signals, enforced by clients, not by cryptography. A determined recipient can always screenshot. The spec is honest about this.

## 6.8 BCC

### Cryptographic mechanism

Per [`protocol/05 §5.2.2`](./05-cryptography.md#522-cc-bcc-threads), BCC is implemented as parallel two-member groups per BCC recipient using the `external_senders` extension.

### Reply-exposure prevention

A BCC recipient's envelope MUST carry a protected header `x-marque.was_bcc: true` inside the MLS-encrypted payload. Clients that honor this header:

- MUST display a *"You were BCC'd"* banner on the received message.
- MUST disable Reply-All by default. Reply-All from a BCC recipient exposes the fact that they were on the thread, defeating the sender's intent.
- MUST offer Reply (sender-only) as the obvious path.
- If the user explicitly overrides, the client surfaces a confirmation modal spelling out *"this will reveal that you were BCC'd."*

### Scale limit

BCC to more than 50 recipients SHOULD be refused by clients with a suggestion to use a `list`-kind Room ([`protocol/04 §4.7`](./04-rooms.md#47-mailing-lists-list-kind)). The cryptographic duplication cost grows linearly and becomes a provider-side storage burden at scale.

Clients SHOULD offer a one-click conversion from a large-BCC compose to a `list` Room.

## 6.9 Thread operations

### Membership authorization

For personal (non-Room) threads, **any existing member MAY** initiate Add or Remove proposals, subject to the current epoch's policy extension. The MLS `Commit` requires one member's signature.

Additional Marque rules beyond baseline MLS:

- **Rate limit** — a single member cannot add more than 5 new participants in a 24-hour rolling window without an explicit consensus step.
- **30-second undo window** — a proposed Add is visible to all existing members; during the window, any existing member can veto. Prevents an attacker who has compromised one member from sweeping the entire conversation.

For Room-backed threads, authorization follows the Room's roster policy ([`protocol/04 §4.4`](./04-rooms.md#44-roster-and-roles)).

### Split, fork, merge

**Thread split** — a subset of members continues in a new thread without others. The subset forms a new MLS group with a fresh `group_id`. The new thread's first message MAY include a `core.thread_split` informational block referencing the parent thread's `group_id` and `content_id` for UI lineage display.

**Thread fork** — one member starts a sub-topic with a different (possibly smaller) set. Same mechanic as split; UI distinguishes by the lineage metadata.

**Thread merge** — membership of the combined group is the union; all members are re-added. First message in the merged group MAY reference both parent threads. No cryptographic merge primitive exists.

All three are informational at the protocol layer. Authority is the same as Thread Membership Authorization above.

## 6.10 Scheduled send

Two implementations:

**Client-local.** Device holds the composed envelope and deposits at send-time. Fails if the device is offline at send-time.

**Provider-hosted.** Client deposits a *scheduled envelope* to the home provider with a `release_time` timestamp. The provider holds the ciphertext and releases it at the scheduled time. A scheduling receipt is signed by the provider.

**Cancel-before-send** is a control frame the client may submit until `release_time − 60 s`. After that, cancel fails; the message sends.

**Qualified-tier scheduled send** is permitted, but the ProofEnvelope timestamp reflects the actual send-to-recipient time, not the compose time — preserving legal-tier semantics.

## 6.11 Undo send

Client-side only — the client holds the composed envelope for a configurable 5–30 second delay before deposit. During that window the user can cancel. No provider-side mechanism: once deposited, an envelope cannot be un-deposited.

This is the same cryptographic honesty as [`protocol/03 §3.13`](./03-architecture.md#313-gdpr-and-append-only-log-semantics): a sender cannot force a recipient to un-read a message.

## 6.12 Mute, snooze, bump

**Mute** is client-local. The user's client filters out push notifications and inbox-new-marker increments for muted threads; MLS state is unchanged. Mute MAY be synced across devices via the archive-key-encrypted cache-delta channel ([`protocol/03 §3.9`](./03-architecture.md#39-archive-of-record-and-cross-device-sync)).

**Snooze** is client-local. The message disappears from the inbox until a user-specified time, then returns. Synced via cache-delta.

**Bump / resurrect** has no protocol primitive. A user wanting to bring a dormant thread to the top of the inbox sends a new Reply. A deliberate choice against a common email antipattern (*"++"* bump replies).

## 6.13 Registry

Block type names follow **reverse-DNS**:

- `core.*` — reserved for the Marque core spec.
- Third-party namespaces — `com.example.*`, `io.producthunt.*`.
- IANA registration required for names in any IANA-allocated root namespace.

Unknown block types MUST render their `fallback` field (a `core.text` block) or the envelope's `alt_plaintext`.

---

Continue to [**Legal proof**](./07-legal-proof.md).
