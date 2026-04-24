# Rooms — shared mailboxes, teams, and mailing lists

> A Room is an identity that is not a natural person: a shared inbox, a team thread, a mailing list, an executive-assistant delegation, or a legally-bound corporate identity. It is a `did:mail` whose authority to act is distributed across a roster under explicit policy.

**You will learn**

- Why Rooms exist — and what the BCC-duplication model cannot do.
- The five Room kinds: `shared`, `team`, `list`, `delegation`, `org`.
- Roster and role definitions, policy flavors (any-of / threshold / owner-only).
- Send-as authorization, role rotation, and audit logs.
- Legal-person Rooms under eIDAS.

---

## 4.1 Why Rooms exist

[`protocol/05 §5.2`](./05-cryptography.md) treats CC as visible MLS group members and BCC as parallel two-member groups using the `external_senders` extension. That model is correct for natural-person messaging but fails for three specific patterns:

1. **Shared mailboxes** (`support@company.com`, `abuse@`, `legal@`) where multiple people handle the same inbox.
2. **Delegated access** (an executive assistant reads and replies on behalf of a principal).
3. **Mailing lists** and broadcast distributions where the recipient set is much larger than MLS's practical per-group envelope.

The BCC-duplication model does not scale past a few hundred recipients, and the delegation and role-rotation patterns business communication requires cannot be expressed as MLS groups of natural persons.

Rooms are the answer. A Room is a first-class identity with its own `did:mail`, its own KEL, and its own service endpoints, backed by a roster of natural-person DIDs under explicit policy.

## 4.2 The Room identity

A Room DID document carries an `x-marque.room` object:

```json
"x-marque": {
  "room": {
    "kind":       "shared" | "team" | "list" | "delegation" | "org",
    "roster":     [ /* ... */ ],
    "policy":     { /* ... */ },
    "parent":     "<principal DID>",   // delegation only
    "visibility": "public" | "unlisted" | "private"
  }
}
```

The Room's IRK is held by the organization — typically a hardware-backed KMS or a threshold group. The Room's DSKs are per-roster-member. Natural-person roster members authenticate with their own DSKs but sign Room-originated messages under the Room's identity.

## 4.3 The five kinds

| Kind             | Example                      | Who can send-as                   | Who receives on behalf         |
| ---------------- | ---------------------------- | --------------------------------- | ------------------------------ |
| **`shared`**     | `support@company.com`        | Any roster member                 | All roster members             |
| **`team`**       | `launch-q3@company.com`      | Any roster member                 | All roster members             |
| **`list`**       | `announce@foundation.org`    | Owner, or threshold of moderators | All subscribers (broadcast)    |
| **`delegation`** | `alice-asst@alice.example`   | Designated assistants             | Forwarded to the principal DID |
| **`org`**        | `legal@firm.com` (Qualified) | Threshold of partners (m-of-n)    | Roster per engagement          |

## 4.4 Roster and roles

Each entry lists a natural-person DID (or a nested Room) with:

```json
{
  "did": "did:mail:marque.id:alice",
  "role": "member",
  "weight": 1,
  "scope": { "send_tiers": ["SIGNED"], "bond_budget_usd": 100 },
  "added": "2026-04-23T10:00:00Z",
  "added_by": "did:mail:company.com:ceo"
}
```

Roles:

| Role            | Capabilities                                                                                        |
| --------------- | --------------------------------------------------------------------------------------------------- |
| **`owner`**     | Modify policy, rotate the roster, revoke delegates. One or a threshold of owners REQUIRED per Room. |
| **`moderator`** | Add/remove non-owner members, set non-authority policy flags.                                       |
| **`member`**    | Send-as and receive per policy.                                                                     |
| **`read-only`** | Receive only. Cannot send-as.                                                                       |
| **`assistant`** | For `delegation`: send-as the principal within a bounded scope.                                     |

`weight` and `scope` are used for threshold policies and bounded delegation respectively.

## 4.5 Send-as authorization

An outer envelope originating from a Room carries the Room's DID as `sender_did`. A new envelope field `sender_acting` names the natural-person DSK that actually signed, plus a **Room authority signature** proving the acting DSK was authorized at send time:

- **Any-of policy** (`shared`, `team`, `delegation`): a signature by the Room's KMS that the acting DSK is currently in the roster and has the required role.
- **Threshold policy** (`org`, threshold `list`): a FROST ([RFC 9591](https://datatracker.ietf.org/doc/html/rfc9591)) signature from m of n owners or moderators.
- **Owner-only policy** (strict `list`): the owner's DSK signature.

Recipients verify both the acting DSK signature and the Room authority signature. A failure of either invalidates the message as a Room-originated send.

## 4.6 Receiving for a Room

Inbound messages to a Room are deposited at the Room's DID's service endpoints and fanned out to the roster per policy. Fan-out is cryptographic, not a re-encryption at the provider: the Room's MLS state treats the roster as group members; each member's client fetches with its own DSK.

For high-cardinality Rooms (mailing lists >1 000 subscribers), TreeKEM's O(log n) per-epoch cost becomes the binding constraint. The `list` kind MAY therefore relax forward secrecy to a **rolling epoch** (weekly advance regardless of membership changes) to bound operational cost.

## 4.7 Mailing lists (`list` kind)

Broadcast semantics for large recipient sets.

| Feature         | Specification                                                                                                                                                                          |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Authority       | Owner or moderator-threshold sends; subscribers read-only by default.                                                                                                                  |
| Subscribe       | Prospective subscriber sends a `subscribe` control frame to the list endpoint. Owner policy controls approval.                                                                         |
| Unsubscribe     | One-click via `unsubscribe` control frame. Reflected in the roster within 24 hours. Equivalent to [RFC 8058](https://datatracker.ietf.org/doc/html/rfc8058) `List-Unsubscribe`.        |
| Double-opt-in   | Policy flag; subscription is pending until the prospective subscriber confirms via a signed challenge.                                                                                 |
| Reply semantics | Policy flag: `reply-to-sender`, `reply-to-list`, or `reply-to-both`.                                                                                                                   |
| Archive         | Lists MAY publish a public archive at a stable URL. Public archive content is `public_untrusted` at the MBS layer for AI-assistant safety (see [protocol/08 §8.5](./08-anti-spam.md)). |

Very large lists (>50 000 subscribers) SHOULD use rolling-epoch policy and MAY be federated across multiple Rooms to stay within MLS's per-group envelope.

## 4.8 Shared mailboxes (`shared` kind)

The `support@company.com` pattern.

- Any roster member can send-as the Room.
- A new inbound message is **claimed** via a `claim` control frame — a client-side soft lock; other members see the claim and can assist but typically do not double-respond.
- Threads preserve Room identity across handoffs — if Alice starts a thread and Bob takes over, `sender_did` remains the Room; only `sender_acting.acting_dsk` changes.
- Claims and handoffs are retained in the Room's audit log (§4.11).

## 4.9 Delegation (`delegation` kind)

Executive-assistant access. Unusual in that it does **not** have its own public DID — it sits in front of a principal DID.

- The delegation Room's DID is private; correspondents address the principal directly.
- The principal's DID document lists the delegation Room as a `LegacyDelegation` service endpoint.
- Inbound to the principal fans out to the principal's own DSKs **and** to the delegation roster.
- Delegation scope limits the assistant's send-as:

  | Scope       | Capabilities                                           |
  | ----------- | ------------------------------------------------------ |
  | `read_only` | Cannot send.                                           |
  | `triage`    | Reply with templated responses; no free-form compose.  |
  | `full`      | Compose freely; **Qualified tier is still forbidden**. |

- The assistant's acting signature is always surfaced in UI — recipients see _"Alice (via assistant Bob)"_. No hidden delegation.
- Revoke is one-click in the principal's client; takes effect across all providers within 15 minutes.

**Delegation MUST NOT permit Qualified-tier send-as.** Legal-identity operations cannot be delegated cryptographically.

## 4.10 Organizations (`org` kind)

Corporate identity under partner-threshold authority.

- Every outbound message requires m-of-n owner signatures via FROST.
- Roster rotation itself requires threshold consent.
- Qualified-tier support: an `org` Room MAY send at Qualified tier if its QTSP qualified certificate is issued to the organization as a legal person per [ETSI TS 119 461](https://www.etsi.org/deliver/etsi_ts/119400_119499/119461/).
- Corporate handover: on sale of the underlying legal entity, the Room's IRK MUST be rotated and all owners MUST sign the transition KEL event.

## 4.11 Role rotation

A member leaves; their DSK must no longer act-as the Room.

1. Owner (or moderator with add/remove role) publishes a roster-update KEL event removing the departing member.
2. The Room's KMS stops issuing Room-authority signatures for messages where the acting DSK is the departed member.
3. In-flight messages signed before the removal timestamp remain valid; messages signed after are invalid.
4. Optionally: the Room publishes a grace-period list of the departed member's recently used DSK certificate fingerprints, so recipients can reject late arrivals.

The departed member retains their own natural-person DID and its archive; they lose the Room's archive and fan-out.

## 4.12 Audit logs

Every Room MUST maintain an append-only audit log of:

- Roster additions and removals with the authorizing DID.
- Policy changes.
- Send-as events (acting DSK, timestamp, message `content_id`).
- Claim and handoff events for `shared` kinds.
- Delegation scope changes.

The audit log is a transparency log with witness-signed tree heads. Readable by any current roster owner; `read-only` access available to auditors designated in policy (external auditors, compliance officers). **Distinct from the message archive** — losing the archive does not lose the audit trail, and vice versa.

## 4.13 Family and household

Household accounts (shared `family@...`) and minor-under-guardian are modeled as Rooms of the `org` kind with a simplified policy — a parent or legal guardian is the owner; children are roster members with scope restrictions. Minors MUST NOT send at Qualified tier. Age-of-majority enforcement is a jurisdictional concern delegated to the QTSP identity-proofing step.

## 4.14 Visibility

| Level      | Meaning                                                      |
| ---------- | ------------------------------------------------------------ |
| `public`   | Listed in the federated directory; discoverable by anyone.   |
| `unlisted` | Resolvable by DID but not discoverable by directory listing. |
| `private`  | Resolvable only with an out-of-band secret access token.     |

Visibility is a property of the Room, not individual messages; message metadata follows the privacy tiers of [`protocol/05 §5.5`](./05-cryptography.md).

## 4.15 Handle conventions

User-facing Room handles follow familiar email conventions to preserve intuition:

| Pattern             | Example                    | Kind             |
| ------------------- | -------------------------- | ---------------- |
| Role                | `support@company.com`      | `shared`         |
| Team / project      | `launch-q3@company.com`    | `team`           |
| Announce            | `announce@foundation.org`  | `list`           |
| Discussion          | `users@foundation.org`     | `list` + two-way |
| Corporate role      | `legal@firm.com`           | `org`            |
| Personal delegation | `alice-asst@alice.example` | `delegation`     |

---

Continue to [**Cryptography**](./05-cryptography.md).
