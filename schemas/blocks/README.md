# MBS block schemas

JSON Schema for each `core.*` Marque Block Spec block type.

## Current schemas

- [`core.text`](./core.text.schema.json)
- [`core.attachment`](./core.attachment.schema.json)
- [`core.schedule`](./core.schedule.schema.json)
- [`core.reply`](./core.reply.schema.json)
- [`core.forward`](./core.forward.schema.json)
- [`core.retract`](./core.retract.schema.json)
- [`core.signed_action`](./core.signed_action.schema.json)
- [`core.image`](./core.image.schema.json)

## Still to be added

Stub PRs welcome. Track in the [open MIPs](../../mips/) list.

- `core.video`, `core.audio` — content-addressed media (parallel to `core.image`).
- `core.poll` — poll with signed vote replies.
- `core.form` — signed form submissions.
- `core.payment_request` — PSP-neutral payment request (ISO 20022, SEPA, FedNow, Stripe, Adyen, Lightning).
- `core.reaction` — lightweight hash-referenced reactions.
- `core.embed` — consent-fetched rich embeds.
- `core.edit` — append-only amendment (distinct from `core.retract`).
- `core.quote` — non-reply standalone quote block (used in forward, context-inclusion).
- `core.thread_split` — informational lineage marker (§7a.7).
- `core.wasm_widget` — sandboxed interactive widget.
- `core.diagram` — Mermaid / Graphviz source.
- `core.agent_instruction` — signed instruction block for allowlisted AI-assistant integrations (§8.2.7).

## Extension blocks

Third parties register block types under reverse-DNS namespaces:

- `com.example.widget.chart`
- `io.producthunt.embed`
- `org.mozilla.observatory.report`

Their schemas are NOT hosted here. They MAY be published wherever the namespace owner chooses; the envelope carries a `$schema` URI so clients can fetch and validate.
