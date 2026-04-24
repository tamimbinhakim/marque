# Marque governance

Marque is a protocol specification intended for IETF Standards Track adoption. Until a Working Group is chartered, governance is operated by the **Marque Editors** under the principles set out below. After charter, governance hands off to the WG; this document is retained as the pre-charter record.

## Principles

1. **Rough consensus and running code.** Decisions require substantive agreement among editors and at least one independent implementation, or a compelling argument why implementation must wait.
2. **No single-vendor capture.** No decision may be made that requires a specific provider, chain, custodian, or trust-service operator. The protocol must remain implementable by independent parties.
3. **Conformance over compatibility.** Where a design choice trades backwards compatibility for a narrower, testable spec, we choose conformance. Spec-optional extensions are refused at the interop surface.
4. **Public audit trail.** Every normative decision is traceable to a merged MIP or a recorded working-group minute. No private decisions on public spec.
5. **User sovereignty.** When user interests conflict with provider or implementer interests, user interests prevail. "User" means the natural or legal person identified by a `did:mail`, not the provider hosting their mailbox.

## Roles

### Editors

Editors are the custodians of the specification. They:

- Merge MIPs after consensus is reached.
- Maintain editorial consistency across the spec.
- Publish Internet-Draft revisions.
- Represent Marque in liaison with IETF, ETSI ESI, W3C DID WG, and the MIMI WG.

Editors are appointed by consensus of the existing editor team at the time of appointment. There MUST be at least 3 and SHOULD be no more than 7 editors at any time. Editors affiliated with a single organization MUST NOT hold a majority.

The current editors are listed in [`docs/editors.md`](./docs/editors.md).

### Contributors

Anyone who opens an issue, submits a PR, or authors/co-authors a MIP is a contributor. Contributors have no formal role but drive the substantive work; editors exist to serve the contributor community.

### Implementers

Operators of conformant Marque stacks (clients, provider MTAs, bridges, QTSPs). Implementers are consulted on any MIP that changes normative behavior. Implementer feedback is not binding on editors but is heavily weighted.

### Working Group (future)

On IETF charter, the chartered WG replaces the editor team as the decision-making body. Editors become WG editors or step down per WG process. This repository MAY continue to host pre-RFC artifacts; normative text migrates to datatracker.ietf.org.

## Decision-making

1. **Editorial changes** (typos, link fixes, grammar, clarity without semantic change): one editor merge.
2. **Clarifications** (resolve ambiguity without changing intended behavior): two editor approvals, 7-day public comment period.
3. **Normative changes** (MUST/SHOULD/MAY language, wire formats, cryptographic suites, trust model): merged MIP in `Final` status, see [MIP-0001](./mips/mip-0001-process.md).
4. **Charter-level decisions** (scope, name, liaison, license, governance itself): editor supermajority (⅔) plus 30-day public comment period; any editor may veto on grounds of Principle 2 (no single-vendor capture) or Principle 5 (user sovereignty).

Disagreements among editors are resolved by discussion, then by majority vote at a scheduled editors' call. Editors MUST record their rationale publicly for any dissenting vote.

All merges — editorial, clarifying, and normative — use [Conventional Commits](https://www.conventionalcommits.org/). CI enforces the convention via [commitlint](./.commitlintrc.yaml); `CHANGELOG.md` is generated from the git log by [`git-cliff`](./cliff.toml). A commit implementing a MIP SHOULD reference the MIP number in its footer (`Refs: MIP-NNNN`); a commit introducing a breaking normative change MUST carry a `BREAKING CHANGE:` footer and cite the governing MIP. Full commit guidance is in [CONTRIBUTING.md](./CONTRIBUTING.md#conventional-commits).

## Liaison

Marque maintains active liaison with:

- **IETF** — primary standards venue. Target WG: _mpc_ (Marque Protocol Core) or equivalent.
- **ETSI ESI** — QERDS profile conformance (EN 319 521 policy + EN 319 522 framework/semantic/formats/bindings + EN 319 532 REM + TS 119 524-1/-2 conformance test suites).
- **W3C DID WG** — `did:mail` method registration.
- **MIMI WG** — async MLS transport profile.
- **NIST / BSI / ANSSI** — post-quantum ciphersuite tracking.

Liaison statements are drafted by editors, reviewed publicly for 14 days, and sent by the named liaison editor. Inbound liaison statements are published in `docs/liaison/`.

## Trademark and name

"Marque" and the Marque word mark are held in trust by the editor team pending transfer to a neutral foundation (proposed: Internet Mail Identity Foundation, chartered under IETF/ISOC). No editor, contributor, or implementer may register conflicting marks in classes 9, 38, 41, or 42.

## Conflicts of interest

Editors MUST disclose:

- Employment or retainer from any organization that operates, plans to operate, or sells products to operators of Marque providers, clients, bridges, or QTSPs.
- Any financial interest (equity, options, token holdings) in the above.
- Any cryptographic-chain holding that could be materially affected by Marque's blockchain-scope decisions (Part 3).

Disclosures are published in [`docs/editors.md`](./docs/editors.md) and updated within 30 days of change. An editor MUST recuse from any decision where their disclosure creates material conflict.

## Amendment

This document is amended per the charter-level process above. Prior versions are preserved in git history.

---

_Version 0.1, April 2026._
