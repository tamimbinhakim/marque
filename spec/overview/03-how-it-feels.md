# How Marque feels

> A walkthrough from the user's perspective. No normative language, no wire formats — just what a person sees, taps, and reads.

**You will learn**

- What happens when you sign up, send, reply, forward, and register a message.
- What chips mean and when they appear.
- How SMTP bridging is surfaced.
- What the archive and multi-device experience looks like.

---

## Alice signs up

Alice downloads a Marque client. Setup prompts her for a handle:

> **Choose your handle**
> `alice@marque.id` &nbsp;*(recommended — portable across any provider)*
> or
> `alice@acme.com` *(tied to acme.com's cooperation)*
>
> Your handle is yours forever. Changing providers later doesn't change your handle.

She picks the portable option. The client generates her keys, establishes a recovery plan (her spouse, her lawyer, and her home provider as three guardians), and uploads the first device's signing key. Onboarding is under a minute.

She gets 100 **bootstrap sends** free — enough to reach her first contacts without them treating her as a stranger.

## Sending a message

Alice opens her **Inbox**, taps *Compose*.

> **To:** `bob@example.com`
> **Subject:** Quarterly budget draft
>
> *Hi Bob — draft attached. Thursday afternoon for a walk-through?*
>
> 📎 `Q2-budget.xlsx`
> 📅 *Pick a slot:* Thu 1:00 PM · Thu 3:00 PM
>
> \[ Attach · Schedule · Register · Send \]

She hits *Send*. The client resolves Bob's handle, encrypts the message to him, deposits to his two listed providers, and shows the message in her **Sent** folder.

## Bob receives

On Bob's phone:

> **Alice Chen** · `alice@marque.id` · 2 min ago
> **Quarterly budget draft**
>
> Hi Bob — draft attached. Thursday afternoon for a walk-through?
> 📎 `Q2-budget.xlsx`
> 📅 *Pick a slot*
>
> \[ Reply · Forward · Mark as unread · Archive \]

No chip. It's an ordinary message — Casual tier. Bob taps *Thu 3:00 PM*; the calendar slot flows back to Alice automatically. He replies with one line.

## Sending something that matters

Next week, Alice needs Bob to acknowledge the final contract. She composes with **Register**:

> **Register this message**
>
> Bob will see a **Registered** chip on delivery. He'll be asked to acknowledge receipt — his client produces a cryptographic delivery receipt you can present in court.
>
> *Estimated cost: €0.35* (provider €0.10 + QTSP €0.25)
>
> \[ Cancel · Register and send \]

On Bob's side:

> **Alice Chen** · `alice@marque.id` · **⚖ Registered**
> **Signed contract — please acknowledge**
>
> *You are receiving a Registered message. Opening this message will produce a cryptographic read receipt.*
>
> \[ Acknowledge and read · Decline \]

Bob acknowledges. The signed read receipt flows back to Alice; her client shows a green checkmark under the message: *Delivered · Acknowledged by Bob · 14:32 UTC*.

If Bob had declined, Alice would see *Refused to acknowledge · 14:32 UTC* — still legally significant, equivalent to a registered letter declined at the door.

## Forwarding

Bob wants to forward Alice's contract to his lawyer.

> **Forward**
> ⚪ **Quoted** — include as a quote. The lawyer won't see Alice's cryptographic signature.
> ⚪ **Attested** — carry Alice's original signature along. Requires Alice's permission; she didn't grant it for this message.
> ⚪ **Registered** — preserve the legal chain. Requires Alice's permission and a Registered-capable forwarder.

Only **Quoted** is available; the contract was marked no-further-forward. Bob forwards as quoted. His lawyer sees the content attributed to Bob's forwarding assertion, not to Alice directly.

## The Inbox

Bob's Inbox at a glance:

```
Inbox  ·  5 unread  ·  1 Registered

⚖  Alice Chen      Signed contract — please acknowledge    14:32
   Dana Patel      Fwd: RFP draft                          12:04
🔒  Éric Martin    bridged · encrypted  ·  Meeting notes   Wed
📧  newsletter     email  ·  Weekly Hacker News            Wed
⚠  suspicious     caution  ·  "Security alert"             Tue
```

Four chips tell Bob at a glance what kind of message each row is:

- **⚖ Registered** — legal-grade cryptographic proof.
- **🔒 bridged · encrypted** — came over SMTP, but the sender's key was discoverable and we wrapped it in MLS at ingress.
- **📧 email** — legacy SMTP plaintext. Same as today's email.
- **⚠ caution** — bridged with failing DKIM / DMARC. Probably phishing.

No chip on unmarked Marque messages; they're the baseline, like unlabeled email today.

## Reply-All and BCC

Alice replies to a four-person thread. The client asks once:

> **Reply All** — keep Bob, Carol, Dana on the thread
> **Reply** — just the sender (Carol)

If she removes Dana from Reply All, the client surfaces: *"Dana will no longer see messages in this thread after this one."* Dana is cryptographically removed from the MLS group; she can't see future messages even if she asks Bob.

If Alice had been BCC'd on a message and tried to Reply-All, she'd see:

> ⚠ **You were BCC'd on this message.**
> Replying to everyone would reveal that you were on the thread. Reply to the sender only?

## Bridging to Gmail

Alice writes to `olivia@gmail.com`. Olivia doesn't have a `did:mail` yet.

> **Olivia will receive this as ordinary email.**
> The content will not be end-to-end encrypted. We'll attach an Autocrypt header so Olivia can upgrade on her next reply.

Alice confirms. Olivia receives a normal email — DKIM-signed, sanitized Received chain. When Olivia eventually signs up for Marque, her next reply arrives at Alice's client with the `bridged · encrypted` chip, and the thread silently upgrades to native on future exchanges.

## Multi-device and archive

Alice adds her laptop. The existing phone shows:

> **Authorize this device?**
> This laptop is asking to join your Marque account. Approving shares your message history and lets you send from the laptop.
>
> \[ Deny · Approve \]

Alice approves on her phone. Her laptop fetches her archive key and the last 1 000 messages per thread. From now on both devices see the same Inbox, stars, reads, and drafts in real time.

Her archive lives in two places: **ciphertext at her provider** (durable; survives device loss) and **cleartext on her devices** (searchable offline). Searching is local, fast, private.

## Switching providers

Alice's provider raises prices. She signs up with a new one.

1. Her new provider is added to her DID document. Senders now deposit to both.
2. She exports a `.marquebox` from the old provider — one click, complete in a few minutes.
3. She imports to the new provider. History restored.
4. She removes the old provider from her DID. The old provider keeps her ciphertext for 30 days, then deletes.

Alice's handle `alice@marque.id` never changed. None of her contacts had to update anything.

## Losing a device

Alice's phone is lost.

1. She signs in on her laptop (or a replacement phone).
2. Three-of-five guardians — her spouse, her lawyer, and her home provider — receive recovery notifications. They approve.
3. A 72-hour nullification window begins. Her laptop shows a banner on every outbound message: *"Recovery in progress — verify with another contact method."*
4. If nothing goes wrong, the window closes. New device signs in. The phone's device key is revoked in her KEL.
5. If the lost phone was actually stolen and the thief tried the same flow, Alice's laptop would have pushed a loud alert, and a single tap would have nullified the attack.

## The overall feel

- **Familiar.** Inbox, Sent, Archive, Reply, Forward — the words you know.
- **Honest.** Every message's provenance is visible on the chip: legal, encrypted, bridged, caution. Nothing hidden.
- **Portable.** Your handle is yours. You never change it.
- **Considered.** Registered sends cost money and produce proof. Casual messages are free and cheap. The system gets out of the way until you need it.

---

Continue to [**When to adopt**](./04-when-to-adopt.md).
