# Marque Improvement Proposals (MIPs)

A **Marque Improvement Proposal** is the normative unit of change for the Marque specification. Every substantive modification to the spec — a new block type, a new ciphersuite codepoint, a change to envelope semantics, a new DNS record, a modified bridge rule — arrives as a MIP.

## Quick links

- **[MIP-0001: MIP process](./mip-0001-process.md)** — how the MIP process itself works. The governing document.
- **[MIP template](./mip-template.md)** — copy this to `mip-XXXX-short-title.md` for a new proposal.

## Index

| MIP  | Title       | Status |
| ---- | ----------- | ------ |
| 0001 | MIP process | Active |

_The index is regenerated from MIP front-matter on merge to `main`._

## MIP categories

| Category          | Scope                                                                                        |
| ----------------- | -------------------------------------------------------------------------------------------- |
| **Core**          | Changes to wire format, envelope, identity, legal-proof, or transport. Highest bar.          |
| **Content**       | Changes to the Marque Block Spec (MBS) — new block types, schema changes, registry rules.    |
| **Bridge**        | Changes to the SMTP bridge, downgrade rules, or legacy-interop semantics.                    |
| **Interface**     | Changes to provider or client APIs that affect interop but not wire format.                  |
| **Process**       | Changes to governance, this MIP process, licensing, liaison.                                 |
| **Informational** | Design rationale, best-practice notes, deprecated-feature documentation. No normative force. |

## Lifecycle

```
Idea → Draft → Review → Last Call → Final
            ↓          ↓           ↑
        Withdrawn   Superseded   Deferred
```

The full transition rules — duration minimums, who advances each stage, what counts as a substantive objection — are normatively documented in [**MIP-0001 §3**](./mip-0001-process.md#3-lifecycle), which renders the same diagram with annotated transitions. Diagram source: [`../docs/diagrams/mip-lifecycle.mmd`](../docs/diagrams/mip-lifecycle.mmd).

| Stage | What it means |
| ----- | ------------- |
| **Idea** | Discussion on an issue, mailing list, or community call. Not in this directory. |
| **Draft** | First merged version. Substantive revisions land as PRs amending the MIP file. |
| **Review** | The MIP author declares the proposal feature-complete; editors solicit implementer feedback. |
| **Last Call** | 14-day (minimum) final comment period. Editors publish a call on the announcement list. |
| **Final** | Merged as normative. Spec updates cross-reference the MIP number. |
| **Withdrawn** | Author-withdrawn before Final. |
| **Superseded** | Replaced by a later MIP. |
| **Deferred** | Parked by editors awaiting a dependency. |

Status transitions are PRs. The editors perform the transition; the author may request a transition at any time.

## Numbering

MIPs are numbered in the order they are **opened as PRs** (not merged), to avoid the coordination cost of reserving numbers. If two PRs race, the lower PR number wins the MIP number; the loser rebases to the next free number.

Numbers are never reused.

## Writing a MIP

1. Search [open](https://github.com/marque-protocol/marque/pulls?q=is%3Apr+label%3Amip) and [merged](https://github.com/marque-protocol/marque/pulls?q=is%3Amerged+label%3Amip) MIP PRs to confirm your idea is not already in flight.
2. Open a discussion or issue to socialize the idea. This step is RECOMMENDED but not required for MIPs in the _Content_ and _Informational_ categories.
3. Copy [`mip-template.md`](./mip-template.md) to `mip-XXXX-short-title.md` using the next unallocated number.
4. Fill in every section. `Not applicable` is a valid answer but must be explicit.
5. Open a PR with title `MIP-XXXX: <short title>` and label `mip`.
6. Work with reviewers. Respond to every substantive comment; link resolutions to the PR thread.
7. On consensus, editors merge in **Draft** status. Subsequent transitions happen via process-PRs.

## When not to write a MIP

You don't need a MIP for:

- Typos, broken links, grammar. Open a regular PR.
- Spec clarifications that do not change intended behavior. Use the `spec-clarification` issue template.
- Questions about how the spec applies in your context. Use GitHub Discussions.
- Implementation bugs. Report to the implementation's own tracker.

You do need a MIP for:

- Any change to normative wire format, envelope, or identity semantics.
- Any new block type intended for the `core.*` namespace.
- Any new DNS record, capability flag, or discovery method.
- Any change to the tiered legal-proof semantics or bridge downgrade rules.
- Any change to governance, licensing, or this process.
