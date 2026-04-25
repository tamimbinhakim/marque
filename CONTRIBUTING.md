# Contributing to Marque

Thank you for considering a contribution. Marque is a pre-RFC protocol specification; contributions happen primarily through **issues**, **pull requests against the spec**, and **Marque Improvement Proposals (MIPs)**.

## Ground rules

> [!WARNING]
> **No scope creep.** Marque is the wire protocol, identity model, envelope, block spec, legal-proof semantics, and SMTP bridge. Client UX, provider business logic, AI features, and MTA internals belong downstream — to [Mailroom](https://github.com/marque-protocol/mailroom) or to any other implementation. PRs that pull product surface into the spec will be closed.

1. **Read [`GOVERNANCE.md`](./GOVERNANCE.md) first.** It explains who decides what and how.
2. **Be specific.** "I don't like this" is not a contribution. "Section 5.3 mandates MLS but does not specify a post-quantum ciphersuite codepoint; proposal: allocate `0xMarque0001` for `X25519MLKEM768+Ed25519+ML-DSA-65`" is.
3. **Show your work.** If you claim a design is insecure, expensive, or slow, cite the attack, the cost model, or the benchmark.
4. **Assume good faith, state disagreement explicitly.** This repository governs a protocol that could outlive all of us; polite evasion wastes everyone's time.
5. **No scope creep.** (See callout above.)

## How to contribute

### 1. Open an issue

Use one of the templates under [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE):

- **Spec clarification** — the spec is ambiguous, contradictory, or unclear.
- **Bug report** — the spec is internally inconsistent or contradicts a cited reference.
- **MIP stub** — you want to propose a substantive change and are looking for co-authors before writing the full MIP.

For **security issues** involving cryptography, metadata leakage, or economic attacks, do **not** open a public issue. See [`SECURITY.md`](./SECURITY.md).

### 2. Open a pull request

Pull requests are welcome for:

- **Editorial fixes** — typos, broken links, grammar, clarity without semantic change. Merge with one maintainer approval.
- **Normative clarifications** — resolving an existing ambiguity without changing intended behavior. Needs a linked issue and two maintainer approvals.
- **Examples, schemas, test vectors** — additions under `examples/`, `schemas/`, or `spec/test-vectors/`. One maintainer approval; editorial review of prose.
- **MIP drafts** — a new file under `mips/` following [`mips/mip-template.md`](./mips/mip-template.md). See the MIP process below.

> [!IMPORTANT]
> Pull requests **that change normative behavior** (MUST/SHOULD/MAY language, wire formats, cryptographic suites, trust model) **require a merged MIP**. Do not send such PRs without a MIP number.

### 3. Propose a change via MIP

The Marque Improvement Proposal process is defined in [MIP-0001](./mips/mip-0001-process.md) and lives under [`mips/`](./mips/).

**TL;DR of the MIP lifecycle:**

```
Idea  →  Draft  →  Review  →  Last Call  →  Final
                      ↓          ↓           ↑
                  Withdrawn   Superseded   Deferred
```

- Copy [`mips/mip-template.md`](./mips/mip-template.md) to `mips/mip-XXXX-short-title.md` using the next unallocated number.
- Fill in every section. "Not applicable" is a valid answer but must be explicit.
- Open a PR titled `MIP-XXXX: <short title>`. The PR discussion is the review venue.
- Once merged in `Draft` status, substantive changes happen via amendment PRs tracking the MIP number.

## Style

- **Markdown**, wrapped at 100 columns except inside tables and code fences.
- **American English** for consistency with RFCs.
- **RFC 2119 keywords** (MUST, SHOULD, MAY, …) appear in UPPERCASE and only inside normative sections. Editorial prose uses ordinary English.
- **Citations** — link inline to the primary source (RFC, W3C REC, ETSI standard, academic paper, CVE). Do not cite blog posts for normative claims; cite them for motivation and context only.
- **No emoji** in normative text. MIPs and docs may use them sparingly.

## Commit and PR hygiene

- One logical change per commit; one logical change per PR when possible.
- PR descriptions explain **why**, link the issue or MIP, and list any consumers who need to be notified.
- Do not force-push once reviewers have engaged. Use `git commit --fixup` or `git merge main`.

### Conventional Commits

This repository follows the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification. The `CHANGELOG.md` is generated from the git log by [`git-cliff`](https://git-cliff.org/) using [`cliff.toml`](./cliff.toml), and commit messages are validated in CI by [commitlint](https://commitlint.js.org/) using [`.commitlintrc.yaml`](./.commitlintrc.yaml).

**Format:**

```
<type>(<scope>): <imperative subject — 72 chars max, lowercase, no full stop>

<optional body — explains the WHY, wrapped at ~100 cols>

<optional footer — BREAKING CHANGE: …, Refs #123, Co-Authored-By: …>
```

**Allowed `<type>` values** (full list in [`.commitlintrc.yaml`](./.commitlintrc.yaml)):

| Type           | Use when                                                                       |
| -------------- | ------------------------------------------------------------------------------ |
| `feat`         | New normative addition (requires a merged MIP for protocol-affecting changes). |
| `add`          | New non-normative addition (doc, example, schema).                             |
| `change`       | Substantive non-normative change.                                              |
| `update`       | Refresh existing content (numbers, citations, dates).                          |
| `fix`          | Correct an error, typo, broken link, or inconsistency.                         |
| `deprecate`    | Mark content for later removal.                                                |
| `remove`       | Delete content.                                                                |
| `security`     | Security-relevant change.                                                      |
| `perf`         | Performance-relevant change.                                                   |
| `ref` / `cite` | Add, correct, or update a citation.                                            |
| `docs`         | Non-normative documentation change.                                            |
| `spec`         | Change under `spec/`.                                                          |
| `draft`        | Change under `drafts/`.                                                        |
| `schema`       | Change under `schemas/`.                                                       |
| `mip`          | Change under `mips/`.                                                          |
| `governance`   | Change to `GOVERNANCE.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`.                |
| `refactor`     | Non-behavioral restructure.                                                    |
| `test`         | Conformance test vectors.                                                      |
| `ci`           | CI configuration.                                                              |
| `build`        | Build tooling (`cliff.toml`, `kramdown-rfc`, `xml2rfc`).                       |
| `chore`        | Misc repo maintenance.                                                         |
| `style`        | Formatting only (Prettier, whitespace).                                        |
| `revert`       | Revert a prior commit.                                                         |

**Allowed `<scope>` values** reflect the repo layout — the full list is in [`.commitlintrc.yaml`](./.commitlintrc.yaml). Common scopes: `protocol`, `overview`, `context`, `identity`, `architecture`, `rooms`, `cryptography`, `content`, `proof`, `antispam`, `interop`, `conformance`, `core`, `arch`, `mbs`, `bridge`, `storage`, `did-mail`, `whitepaper`, `eidas`, `roadmap`, `faq`, `readme`, `governance`, `ci`.

**Breaking changes.** A commit that changes normative MUSTs, wire format, cryptographic suites, or the trust model MUST include a `BREAKING CHANGE:` footer. Such commits MUST also reference the merged MIP number.

**Examples:**

```
fix(protocol/05): correct FROST non-determinism framing for Ed25519 ciphersuite

FROST signatures under the (Ed25519, SHA-512) ciphersuite are Ed25519-verifier
compatible but not deterministic. The prior wording implied determinism and
could mislead implementers relying on RFC 8032 semantics.

Refs: MIP-0007
```

```
ref(whitepaper): add AgID report citation for Italian PEC scale

Replaces the unsourced "~13M users" figure with the AgID supervised-services
report's ~16M accounts / ~2.5B messages/year.
```

```
feat(protocol/07)!: introduce QERDS-tier read-receipt obligation

BREAKING CHANGE: Qualified-tier messages now REQUIRE a read-receipt path.
The recipient's decline path is the new REFUSED_TO_ACKNOWLEDGE non-delivery
state. Implementations claiming Certified-tier conformance must ship this
before the next release tag.

Refs: MIP-0012
```

### Regenerating the CHANGELOG

```sh
# Preview the next release section without touching CHANGELOG.md
git-cliff --unreleased

# Prepend a new release section (typical flow, one command per release)
git-cliff --tag v2026.05.01 --prepend CHANGELOG.md

# Full regeneration from scratch (rare; use with --context to inspect)
git-cliff --tag v2026.05.01 --output CHANGELOG.md
```

Install `git-cliff` once per developer with `brew install git-cliff` (macOS), `cargo install git-cliff` (any platform with Rust), or download a release binary from [git-cliff.org](https://git-cliff.org/docs/installation).

## Code of conduct

All participation is governed by [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md). Violations should be reported to `conduct@marqueproto.org`.

## Attribution

All contributions are licensed under the repository's license terms (see [`LICENSE`](./LICENSE) and [`LICENSE-SPEC`](./LICENSE-SPEC)). By opening a PR or MIP you affirm you have the right to contribute under those terms.

Significant contributors are credited in the relevant MIP and, at the editors' discretion, in the authors' list of the subsequent Internet-Draft.
