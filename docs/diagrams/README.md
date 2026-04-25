# Diagrams

Source for Marque's canonical diagrams. Stored as [Mermaid](https://mermaid.js.org) (`.mmd`) because:

- GitHub renders Mermaid inline in Markdown via ` ```mermaid ` code fences — no build step.
- Source is diff-friendly; a small PR to a diagram reads as prose, not as binary churn.
- Mermaid output adapts to GitHub's light/dark theme automatically.

## Catalog

| File                                         | Subject                                                                      | Used by                                                                                                                                                                                                              |
| -------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`identity-stack.mmd`](./identity-stack.mmd) | Three-tier key hierarchy and its relationship to the DID document.           | [Architecture overview §1](../architecture-overview.md#1-the-identity-stack) · [`spec/protocol/02-identity.md`](../../spec/protocol/02-identity.md) · [`05-cryptography.md`](../../spec/protocol/05-cryptography.md) |
| [`message-path.mmd`](./message-path.mmd)     | The three delivery paths (P2P, store-and-forward, mixnet).                   | [Architecture overview §2](../architecture-overview.md#2-the-message-path) · [`spec/protocol/03-architecture.md`](../../spec/protocol/03-architecture.md)                                                            |
| [`provider-role.mmd`](./provider-role.mmd)   | What a dumb-encrypted-mailbox provider does and does not do.                 | [Architecture overview §3](../architecture-overview.md#3-the-providers-job) · [`spec/protocol/03-architecture.md`](../../spec/protocol/03-architecture.md)                                                           |
| [`proof-envelope.mmd`](./proof-envelope.mmd) | Composition of a `ProofEnvelope` across Casual → Attested → Certified tiers. | [Architecture overview §4](../architecture-overview.md#4-the-legal-proof-layer) · [`spec/protocol/07-legal-proof.md`](../../spec/protocol/07-legal-proof.md)                                                         |
| [`smtp-bridge.mmd`](./smtp-bridge.mmd)       | Bidirectional SMTP ↔ Marque bridge with discovery and downgrade.             | [Architecture overview §5](../architecture-overview.md#5-the-smtp-bridge) · [`spec/protocol/09-interop.md`](../../spec/protocol/09-interop.md)                                                                       |
| [`mip-lifecycle.mmd`](./mip-lifecycle.mmd)   | MIP state transitions from Idea to Final.                                    | [MIP-0001](../../mips/mip-0001-process.md)                                                                                                                                                                           |

## Rendering

**On GitHub:** diagrams embedded in Markdown via ` ```mermaid ` code fences render natively.

**Locally:**

```sh
npm i -g @mermaid-js/mermaid-cli
mmdc -i docs/diagrams/identity-stack.mmd -o identity-stack.svg
mmdc -i docs/diagrams/message-path.mmd   -o message-path.png -s 2
```

**In a presentation:** render to SVG via `mmdc`, drop into slides. The diagrams use neutral styling so they work on any background.

## Editing

- Keep line count short. A diagram that runs past ~30 lines of Mermaid source is trying to say too much — split it into two.
- Use `<br/>` for line breaks inside node labels, not literal newlines.
- **No hardcoded colors.** Do not use `fill:#…`, `stroke:#…`, `rgba(…)`, or `color:#…` in `classDef` or `rect` directives. Hardcoded colors fight GitHub's light/dark auto-theming and produce the low-contrast diagrams that prompted this policy. Let Mermaid's default theme render the diagram; GitHub will re-theme it automatically based on the reader's selected mode.
- **For emphasis**, use `classDef emphasized stroke-width:3px` (theme-neutral) or thicker edge syntax (`==>` for thick, `-->` for normal, `-.->` for dotted). These adapt to both modes.
- **For visual grouping**, prefer `subgraph` blocks (which Mermaid themes automatically) and full-width `Note over A,B: ━━━ Section title ━━━` dividers in sequence diagrams instead of colored `rect` wrappers.
- Normative text belongs in the spec, not in the diagram. Diagrams are orientation, not specification.
