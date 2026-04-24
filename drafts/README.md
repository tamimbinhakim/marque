# IETF Internet-Drafts

This directory hosts the kramdown-rfc source for Marque's IETF Internet-Drafts. The drafts are filed as **individual submissions** to [datatracker.ietf.org](https://datatracker.ietf.org), which is the standard path for protocol proposals that do not yet have a chartered Working Group. Individual drafts are real, citable IETF documents; they are how MLS (originally `draft-barnes-mls-*`), ACME, OAuth 2.1, and most other protocols entered IETF process.

## Naming

- `draft-tamim-marque-<topic>-NN.md` — individual drafts. `tamim` is the author short name; `NN` is a two-digit version that increments on each submission (drafts expire 6 months after submission and must be renewed).
- After a Working Group adopts a draft, the canonical name changes to `draft-ietf-<wg>-<topic>-NN`. This rename happens at the datatracker, not in this repo.

## Catalog

| File                                                                       | Topic                                                                         | Status                            | Covers                                                                   |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------ |
| [`draft-tamim-marque-arch-00.md`](./draft-tamim-marque-arch-00.md)         | Architecture (Informational) — frames the normative companion drafts           | Draft complete, not yet submitted | `spec/` (whole specification)                                            |
| [`draft-tamim-marque-core-00.md`](./draft-tamim-marque-core-00.md)         | Core wire protocol — envelope, MLS profile, QUIC binding, IANA registries     | Draft complete, not yet submitted | `spec/protocol/03-architecture.md`, `05-cryptography.md` §5.1–§5.2       |
| [`draft-tamim-marque-did-mail-00.md`](./draft-tamim-marque-did-mail-00.md) | The `did:mail` DID method — resolution, DID document structure, KEL, recovery | Draft complete, not yet submitted | `spec/protocol/02-identity.md`                                           |
| [`draft-tamim-marque-proof-00.md`](./draft-tamim-marque-proof-00.md)       | ProofEnvelope — tiered non-repudiation, eIDAS QERDS                           | Stub — awaiting content migration | `spec/protocol/07-legal-proof.md`, `03-architecture.md` §3.11 (retention)|
| [`draft-tamim-marque-mbs-00.md`](./draft-tamim-marque-mbs-00.md)           | Marque Block Spec — content layer                                             | Stub — awaiting content migration | `spec/protocol/06-content.md`                                            |
| [`draft-tamim-marque-bridge-00.md`](./draft-tamim-marque-bridge-00.md)     | Bidirectional SMTP bridge                                                     | Stub — awaiting content migration | `spec/protocol/09-interop.md`                                            |
| [`draft-tamim-marque-antispam-00.md`](./draft-tamim-marque-antispam-00.md) | Economic + cryptographic anti-spam                                            | Stub — awaiting content migration | `spec/protocol/08-anti-spam.md`, `02-identity.md §2.8`                   |
| [`draft-tamim-marque-rooms-00.md`](./draft-tamim-marque-rooms-00.md)       | Rooms — shared mailboxes, teams, mailing lists, delegation, organizations     | Stub — awaiting content migration | `spec/protocol/04-rooms.md`                                              |
| [`draft-tamim-marque-storage-00.md`](./draft-tamim-marque-storage-00.md)   | Storage, archive, `.marquebox` export, device sync                            | Stub — awaiting content migration | `spec/protocol/03-architecture.md §3.8–§3.13`                            |

**Stub** means the draft contains frontmatter, abstract, scope, and references but the normative body has not yet been migrated from the founding specification. A stub is a valid placeholder for authoring order tracking and IANA-codepoint-allocation planning; it is **not** suitable for IETF submission until content migration is complete.

## Tooling

Drafts are authored in Markdown using [kramdown-rfc](https://github.com/cabo/kramdown-rfc) and compiled to `xml2rfc` v3 XML for submission.

Install once:

```sh
gem install kramdown-rfc2629
pip install xml2rfc
```

Render a draft to text and XML locally:

```sh
kramdown-rfc drafts/draft-tamim-marque-core-00.md > draft-tamim-marque-core-00.xml
xml2rfc --text draft-tamim-marque-core-00.xml
```

The generated `.txt` and `.xml` artifacts are ignored by `.gitignore`; only the kramdown-rfc `.md` source lives in the repo.

## Submitting

1. Render to XML as above.
2. Go to <https://datatracker.ietf.org/submit/>.
3. Create an IETF account if you do not have one (datatracker supports passkey sign-in).
4. Upload the XML. The datatracker validates it against xml2rfc schema rules and produces the canonical text output.
5. Once accepted, the draft gets a datatracker URL (`https://datatracker.ietf.org/doc/draft-tamim-marque-core/`) and enters the public IETF corpus.

**Before first submission, run** [`idnits`](https://tools.ietf.org/tools/idnits/) **on the rendered text**:

```sh
idnits --verbose draft-tamim-marque-core-00.txt
```

`idnits` checks boilerplate, reference formatting, RFC 2119 usage, and the dozens of small rules the datatracker enforces on upload. Clean up any errors before submitting; warnings are usually fine to ignore.

## After submission

An individual draft expires **6 months after submission** unless renewed. To renew, increment `NN` and re-submit. The datatracker keeps every prior version accessible by URL.

Next steps after a first draft lands:

1. Announce on `mpc@ietf.org` or the Applications Area mailing list (`apps-discuss@ietf.org`) requesting review.
2. Propose a **Birds-of-a-Feather (BoF)** session at the next IETF meeting to gauge Working Group interest.
3. If a BoF produces rough consensus, file a charter proposal for the `mpc` (Marque Protocol Core) WG.
4. On adoption, the draft is renamed `draft-ietf-mpc-core-00` and governance moves from the individual author to the WG.

## Licensing

Internet-Draft text is licensed under the [IETF Trust Legal Provisions](https://trustee.ietf.org/license-info/). By submitting a draft the author grants the IETF Trust the rights in TLP Section 4. In this repository, draft source is additionally available under [CC-BY 4.0](../LICENSE-SPEC) until first submission; after first submission, the TLP terms govern.

## Style

- One topic per draft. Splitting keeps the review surface manageable and lets the WG adopt drafts independently.
- Aim for ~15–40 pages per individual draft at `-00`. Longer is possible but reviewers may ask you to split.
- Cite the founding specification {{MARQUE-FOUNDING}} for motivation and narrative; keep the drafts themselves normative and terse.
- Use `{{RFC2119}}` keyword tagging via kramdown-rfc's `{::boilerplate bcp14-tagged}` include. Lowercase uses of must / should / may carry ordinary English meaning.
- Every MUST and MUST NOT SHOULD be matched to an item in a conformance test suite. If you can't test it, reconsider whether it should be MUST.
