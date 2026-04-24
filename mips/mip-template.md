---
mip: XXXX
title: "<short imperative title>"
author: "<Name> <email-or-did:mail>"
discussions-to: "<URL to the primary discussion thread>"
status: Draft
category: Core | Content | Bridge | Interface | Process | Informational
created: YYYY-MM-DD
requires: []            # list of MIP numbers this depends on
replaces: []            # list of MIP numbers this supersedes
superseded-by: []       # populated by editors on supersession
---

## Abstract

_A one-paragraph plain-language summary of the proposal. A reader skimming the index should be able to understand what this MIP does from the abstract alone._

## Motivation

_What problem does this MIP solve? Cite the observed failure, measured cost, or specification gap. If this is an incremental improvement, explain the delta precisely._

_MIPs that propose changes without identifying a concrete problem will be rejected._

## Specification

_The normative text. This section will be merged into the spec on Final, or republished as-is for Informational MIPs._

_Use RFC 2119 keywords (MUST, SHOULD, MAY) precisely. Every MUST and MUST NOT requires a justification in the Rationale section._

_Wire-format changes: include the exact CDDL or JSON Schema diff._
_Algorithm changes: include the exact IANA codepoint assignment and a PR number in any implementation. No implementation in the Marque ecosystem is privileged; any conformance-passing implementation qualifies._
_Block-type additions: include the full schema, a rendered example, and the fallback text behavior._

### X.1 <Section>

### X.2 <Section>

## Rationale

_Why this design and not the alternatives? Enumerate alternatives you considered. For each, explain why it was rejected._

_If the MIP has no alternatives worth enumerating, the design is probably not considered enough to merge._

## Backwards compatibility

_Does this MIP break existing implementations? Existing messages? Existing DIDs?_

_If yes: describe the transition path. Include a deprecation window if applicable._

_If no: state the invariants that are preserved._

## Security considerations

_What attacker model applies? What new attack surface does this introduce? What existing guarantees does it preserve, and which does it alter?_

_MIPs that change cryptographic primitives, trust assumptions, or metadata exposure MUST include this section in full. MIPs that do not may note "no security impact" with a one-sentence justification._

## Privacy considerations

_What new metadata does this MIP cause to be emitted, logged, or transmitted? Who observes it? Does any of it leave the sender's or recipient's provider domain?_

_If this MIP affects the Sealed or Mixed metadata tiers (Part 5.5), explicitly state the effect._

## Legal-proof considerations

_If this MIP affects the Attested or Certified tiers (Part 6), explain the impact on QERDS conformance. If it does not, state "no legal-proof impact."_

## Test vectors

_At minimum, include one worked example of every new message, block, or envelope type. Test vectors migrate to `spec/test-vectors/` on Final._

## Implementation status

_Per [RFC 7942](https://www.rfc-editor.org/rfc/rfc7942), list the implementations — branches, PRs, or repositories — that realize this MIP. At minimum, `Draft` status requires a sketch; `Review` status requires a working branch; `Final` status requires the MIP to be implemented in at least **two independent, unaffiliated codebases** that both pass the conformance suite. No implementation in the Marque ecosystem is privileged; every implementation that passes the conformance suite is a peer, and implementers are unrestrained in how they differentiate above the wire._

## Open questions

_Anything the author has not resolved and expects reviewers to opine on. Remove this section before Last Call._

## Copyright

This MIP is licensed under [CC-BY 4.0](../LICENSE-SPEC). Implementation code contributed alongside this MIP, if any, is licensed under [Apache-2.0](../LICENSE).
