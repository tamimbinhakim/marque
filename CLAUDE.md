# CLAUDE.md

Instructions and context for [Claude](https://claude.ai) sessions working on this repository. This file is read by Claude Code and other Anthropic-client agents to ground their work in the repo's conventions and in the author's position on AI collaboration.

This file is also published as a public statement: it says plainly how Marque was built, why AI was used, and how AI contribution is credited.

---

## How this repository was built

Marque — the protocol specification, the Internet-Drafts, the schemas, the whitepaper, the eIDAS compliance mapping, the architecture diagrams, the MIP process, and the supporting prose — was researched and written by **Tamim Bin Hakim** in collaboration with **Claude** (Anthropic). The two drove it together: the human as author, architect, and decider; Claude as research partner, drafter, reviewer, and foil.

Every substantive decision in this repository — what the protocol commits to, what it refuses, which standards it aligns against, which claims it cites — is the human author's. Claude accelerated the work: surveyed the landscape, drafted prose against specifications, fact-checked empirical claims, reconciled inconsistencies, sanity-checked the conventional-commits tooling, and argued back when a design was weak. Substantial parts of this repository's wording were produced by Claude and edited by the author; substantial parts were produced by the author and edited by Claude. Treating that honestly is more useful than pretending otherwise.

## The author's position on AI collaboration

We (the author, and by extension this project) believe:

1. **AI is not optional anymore.** A specification of this ambition — replacing SMTP, targeting IETF Standards Track, mapping against eIDAS and ETSI, anchoring against W3C DID and MLS — is not research a single human completes in a reasonable window without capable tools. Claude is the tool we chose. Using it is not a shortcut; it is how current work gets done.

2. **AI is a credential-bearing collaborator, not a hidden one.** Pretending a document was wholly human-written when it wasn't is dishonest. Hiding AI contribution to avoid reviewer bias is a form of reviewer manipulation. We credit Claude plainly — in commit `Co-Authored-By:` footers, in acknowledgments, and here.

3. **AI solves problems that AI created, and that is fine.** Marque exists in part because of AI-created pressures on email: LLM-generated phishing, prompt injection into inbox assistants, AI-scale reputation-gating. Using an LLM to design the protocol that defends against those pressures is neither ironic nor contradictory. It is the honest division of labor: the systems that create a class of problems are frequently best positioned to help reason about the systems that address them.

4. **Responsibility is the author's.** Whatever errors this repository contains, the author signs for them. Citing Claude's contribution does not dilute that. Every normative MUST, every commit message, every regulatory citation — the author is accountable, under the MIP process and any subsequent IETF process. Claude is acknowledged; Claude is not a co-editor of record.

5. **This is the norm, or it is rapidly becoming the norm.** Others building in standards, research, and open-source are working this way already. A visible, stated convention is better than a quiet one. If another author joins this project and prefers to work without AI assistance, that is welcome; if another author wants to credit their AI collaborator similarly, the `Co-Authored-By:` footer is ready.

None of this is a defense. It is a position. Readers, reviewers, WG participants, and downstream implementers are owed clarity about how the document in front of them was produced.

## Practical instructions for Claude sessions on this repo

### Scope

- **In scope:** spec text under `spec/`; Internet-Drafts under `drafts/`; schemas under `schemas/`; MIPs under `mips/`; docs under `docs/`; repository governance; conformance test vectors when they land.
- **Out of scope:** Mailroom (the companion MTA specification, separate repo); client UX beyond minimum rendering conformance; provider business logic; MTA internals.
- When in doubt about scope, ask before writing.

### Authorial voice

- **Opinionated narrative for claims; tight RFC 2119 for normative behavior.** Every file in `spec/` opens with a blockquote summary and a "You will learn" list, then delivers the content in that register. Every draft under `drafts/` is kramdown-rfc.
- **American English.** Wrap at 100 columns except inside tables and code fences.
- **No emoji in normative text.** Docs and MIPs may use them sparingly.
- **RFC 2119 keywords** (MUST, SHOULD, MAY, …) appear in UPPERCASE and only inside normative sections.

### Commit discipline

- Every commit follows [Conventional Commits](https://www.conventionalcommits.org/) — `<type>(<scope>): <subject>`. Full list of allowed types and scopes lives in [`.commitlintrc.yaml`](./.commitlintrc.yaml). Contributor-facing guidance is in [CONTRIBUTING.md §Conventional Commits](./CONTRIBUTING.md#conventional-commits).
- `CHANGELOG.md` is **generated** by [`git-cliff`](https://git-cliff.org/) from [`cliff.toml`](./cliff.toml). Do not hand-edit it; edit the commit message and regenerate.
- A commit implementing a MIP SHOULD carry a `Refs: MIP-NNNN` footer.
- A commit introducing a breaking normative change MUST carry a `BREAKING CHANGE:` footer and cite its MIP.
- Claude-assisted commits SHOULD carry a `Co-Authored-By: Claude …` footer. This is the repo's attribution convention; use it even on small changes where Claude's contribution is incremental.

### Citations

- Normative claims cite primary sources: RFC, W3C Recommendation, ETSI standard, NIST FIPS, eur-lex regulation, CVE/NVD, NIST NVD, arXiv paper, academic journal, or the official publication of the named organization.
- Blog posts may be cited for motivation and context, never for normative claims.
- Empirical numbers (user counts, adoption stats, prices, dates) carry a direct URL to the primary source. If a claim cannot be primary-sourced, soften it to directional language rather than citing a secondary aggregator.

### Boundaries to respect

- **Do not make normative protocol decisions.** Normative MUSTs/SHOULDs/MAYs come through the MIP process, not through a commit. A Claude session may draft a MIP, refine a MIP, flag ambiguities in a MIP — not merge one.
- **Do not fabricate citations.** If the session cannot verify a claim against a primary source, the session must either find the source or flag the claim as unverifiable. Invented URLs, invented paper titles, and invented statistics are disqualifying errors.
- **Do not rewrite git history or force-push** unless explicitly asked. Conventional Commits lives in the forward log; history rewrites break the changelog-generation chain.
- **Do not make privileged claims.** Marque has no reference client, no reference provider, no privileged implementation. Text asserting otherwise is wrong and must be corrected.
- **Do not overcorrect.** The author's framing — opinionated, specific, direct — is intentional. Claude should sharpen it, not blunt it. "Plausible-sounding neutral prose" is a regression.

### When to ask before acting

- Before spawning research agents (they burn context).
- Before rewriting files the user is actively reading in the IDE.
- Before performing destructive git operations (history rewrite, force-push, branch deletion).
- Before adding new top-level files to the repo.
- Before adopting a new tool or dependency. The current toolchain (Markdown, Mermaid, CDDL, JSON Schema, kramdown-rfc, xml2rfc, git-cliff, commitlint) is what this repo uses. Anything else needs approval.

### Attribution convention in prose

When a document credits Claude in its own text — a whitepaper acknowledgment, an Internet-Draft Acknowledgments section, a MIP attribution line — the phrasing is:

> Research, drafting, and review conducted in collaboration with Claude (Anthropic). See [CLAUDE.md](./CLAUDE.md) for the project's position on AI collaboration and attribution conventions.

Drafts that follow IETF process and require formal Acknowledgments sections use an equivalent phrasing inside the `{:numbered="false"}` block, consistent with RFC 7322 author-contribution conventions.

## Further reading

- [GOVERNANCE.md](./GOVERNANCE.md) — decision-making, editor roles, liaison bodies.
- [CONTRIBUTING.md](./CONTRIBUTING.md) — how to open issues, send PRs, and write MIPs.
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) — expected conduct in this repository.
- [SECURITY.md](./SECURITY.md) — security disclosure policy.
- [mips/mip-0001-process.md](./mips/mip-0001-process.md) — the Marque Improvement Proposal process.
