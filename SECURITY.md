# Security policy

Marque is a security-critical specification. If you believe you have found a vulnerability — in the spec itself, or in an interoperable implementation or deployment you can cite — please follow this policy.

> [!CAUTION]
> **Do not open public issues or PRs for vulnerabilities.** Public disclosure of an unpatched protocol-level flaw forfeits the [safe-harbor](#safe-harbor) protections below and may put deployed users at risk. Report privately via the [encrypted email channel](#how-to-report).

## Scope

This repository coordinates disclosure for:

- **The Marque specification** (`spec/`, `drafts/`, `mips/`, `schemas/`).
- **Conformance test vectors** distributed in this repository.
- **Any Marque implementation or deployment** that can be cited. Example implementations exist (Mailroom on the provider side, and others forthcoming), but none is privileged — every implementation that passes the conformance suite is a peer.

Out of scope (report directly to the vendor):

- Bugs in third-party MTAs, clients, or QTSPs that are conformant with the spec. Report to the vendor; CC the editors only if the vulnerability stems from a spec ambiguity.
- Bugs in underlying primitives (MLS, X25519, Ed25519, ML-KEM, ML-DSA, BLAKE3). Report to the relevant upstream; CC editors if the spec misuses the primitive.
- Operational issues at specific providers.

## Classes of vulnerability we care about

**Critical** — protocol-level breaks.

- Cryptographic weakness in a mandatory suite (key recovery, signature forgery, ciphertext malleability).
- Identity spoofing (impersonating another `did:mail` to one or more recipients).
- Legal-proof forgery (producing a valid QERDS `DeliveryReceipt` without actual delivery).
- Transparency-log split-view or equivocation attacks.
- Metadata deanonymization at the Mixed tier against the stated threat model.
- AI-assistant prompt-injection classes the spec was designed to prevent (Part 8).
- SMTP bridge credential exposure or cross-recipient leakage.

**High** — significant weaknesses under specific conditions.

- Denial of service affordances (asymmetric cost: attacker cheap, defender expensive) beyond documented anti-spam economics.
- Replay that violates documented freshness guarantees.
- Key-compromise impersonation beyond what the tier states.
- Capability leakage (sender learning recipient's capabilities or vice versa against policy).

**Medium** — weaknesses limited in impact or requiring unrealistic conditions.

**Low / informational** — spec clarity that could mislead implementers into insecure defaults.

## How to report

**Use email, encrypted.**

```
To:          security@marqueproto.org
PGP key:     https://marqueproto.org/.well-known/pgp-key.asc
DID:         did:mail:marqueproto.org:security   (once Marque is self-hosting)
Fingerprint: (published at marqueproto.org/security)
```

Include:

1. A clear description of the vulnerability and its impact.
2. A concrete reproduction or proof of concept (the minimum needed to verify the claim).
3. The spec section, MIP, or implementation version affected.
4. Your preferred attribution and whether you want public credit.
5. Any disclosure deadline you plan to observe.

## Our commitments

- **Acknowledge receipt within 72 hours.**
- **Initial triage within 7 days**, with a severity assessment and a tentative timeline.
- **Coordinate with implementers** if the vulnerability affects deployed code. We will share the report only with affected implementers under NDA until disclosure.
- **Credit the reporter** in the published advisory unless the reporter requests anonymity.
- **Publish a CVE** for implementation vulnerabilities and a Marque Security Advisory (MSA) number for spec-level issues.

## Disclosure timeline

- **Standard:** 90 days from report to public disclosure, or as soon as a mitigation ships to known implementers, whichever is sooner.
- **Extensions** may be negotiated up to 180 days for cryptographic or coordinated-disclosure cases.
- **Immediate disclosure** will happen if we have evidence the vulnerability is being actively exploited.

## Safe harbor

We will not pursue legal action against reporters who:

- Follow this policy in good faith.
- Do not exfiltrate data beyond what is minimally necessary to demonstrate the vulnerability.
- Do not impact live users beyond the reporter's own account or accounts under the reporter's authorized control.
- Give us a reasonable opportunity to address the issue before any public disclosure.

## Out-of-scope methods

The following are **not** safe-harbored and **will** be reported:

- Social engineering of editors, contributors, or provider staff.
- Denial-of-service attacks against live services.
- Intercepting or tampering with other users' traffic.
- Accessing systems or data you are not authorized to access.

## Questions

Non-sensitive policy questions may go to the public issue tracker with the `security-policy` label. Sensitive or vulnerability-related questions go to the email address above.
