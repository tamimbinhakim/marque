# What Marque replaces

> Email's pain is measurable, documented, and has inflected sharply in 2024–2026. A protocol replacement is only worth its adoption cost if the problems it fixes are real, specific, and getting worse.

**You will learn**

- The five defining failures of email today.
- The structural reasons each one resists fixing.
- What email gets right — what a successor must preserve.

---

## The five defining failures

### 1. Self-hosting is dead

The Gmail–Microsoft oligopoly has effectively killed legitimate independent mail operation. Carlos Fenollosa's 2022 essay *"After self-hosting my email for twenty-three years I have thrown in the towel"* is the canonical statement. 2025 threads on `r/selfhosted` confirm the situation has worsened.

- Correctly-configured SPF, DKIM, and DMARC do **not** overcome Gmail's silent spam-foldering of residential and low-reputation IP ranges.
- Port 25 is blocked on most consumer ISPs — a de facto protocol amendment outside any RFC.
- Running a personal MTA now requires paying a reputation-laundering relay (Mailgun, SES, Postmark), which reimposes the very centralization self-hosting was meant to escape.

The protocol layer has not failed; the reputation gatekeeping layered on top has. There is no path to fix this without replacing the reputation model — and the reputation model is structurally tied to who owns the sending IP, not to who the sender actually is.

### 2. Zero-click AI prompt injection is now a weapon

[CVE-2025-32711](https://nvd.nist.gov/vuln/detail/CVE-2025-32711) — *EchoLeak*, disclosed by Aim Security / Aim Labs, CVSS 9.3 — was the first documented zero-click prompt-injection exploit in a production LLM system. A single crafted email caused Microsoft 365 Copilot to exfiltrate internal data with no user interaction, chaining:

- an XPIA classifier bypass,
- a Markdown reference-link redaction bypass,
- a Teams-proxy CSP hole.

WithSecure Labs documented an analogous attack against Gemini email-summary, and Microsoft has shipped follow-on prompt-injection mitigations through 2025–2026. OpenAI's public stance is blunt: *"Prompt injection, much like scams and social engineering on the web, is unlikely to ever be fully solved."*

SMTP delivers arbitrary text into a context where LLMs cannot structurally distinguish instructions from data. **RFC 5322 has no untrusted-data tagging.** This is a protocol-level defect that cannot be patched at the model layer.

### 3. Account suspension is a catastrophic single point of failure

*Wired* and *Texas Standard* have documented that contacting a human at Google is generally impossible for users outside YouTube Partner or a paid tier. Small businesses lose years of customer correspondence; families lose photo archives; freelancers lose their professional identity — because the address **is** the provider.

This is the user-observable form of the structural defect Marque exists to fix: email conflates *identifier* and *locator* at the protocol layer, so no remediation short of ownership reallocation can work.

### 4. Deliverability is opaque and punitive

[Google and Yahoo's February 2024 bulk-sender rules](https://support.google.com/a/answer/14229414) — tightened to SMTP-level rejection in late 2025 — impose SPF/DKIM/DMARC alignment, [RFC 8058](https://datatracker.ietf.org/doc/html/rfc8058) one-click `List-Unsubscribe`, and a 0.3% spam-complaint ceiling. [Microsoft followed with enforcement on 5 May 2025](https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730). A church secretary running a 300-person newsletter is held to the same bar as SendGrid.

The resulting whack-a-mole is documented on Shopify, Intercom, and Postmark community forums through 2025:

- The SPF 10-DNS-lookup limit forces SPF flattening as a hack.
- DKIM key rotation breaks silently.
- DMARC alignment with third-party ESPs is a pit of edge cases.

None of these are user-visible; legitimate senders discover them only when replies stop arriving.

### 5. The inbox has no priority, no commitments, no structured actions

Industry surveys consistently report employees spend multiple hours per day in the inbox; Microsoft's [Work Trend Index](https://www.microsoft.com/en-us/worklab/work-trend-index) documents email and chat interrupting knowledge-workers at cadences near one every two minutes during core hours; and recurring burnout-and-email surveys attribute a majority share of workplace burnout to email overload. The specific figures vary by methodology, but the direction is consistent.

This is not a protocol defect per se, but it is a symptom. **Every message is a peer.** No structural distinction between "please acknowledge receipt," "let me know when you're free Thursday," "here's a document to sign," "FYI," or "spam." The inbox cannot help the user because it cannot tell these apart.

## The remaining pains

The five above define the stakes. Research across Reddit (r/gmail, r/selfhosted, r/sysadmin, r/privacy, r/smallbusiness), Hacker News, deliverability forums, and academic literature yields a stable ranking of 28 distinct email pains. The remaining 23 cluster into four themes:

- **Identity lock-in** — the forwarding-forever trap, the [Skiff sunset (acquired by Notion Feb 2024; shutdown Aug 2024; forwarding ended 9 Feb 2025)](https://techcrunch.com/2024/02/09/notion-acquires-privacy-focused-productivity-platform-skiff/), loss of decades-old addresses.
- **Cryptographic failure** — PGP dead per [Moxie Marlinspike's 2015 post](https://moxie.org/2015/02/24/gpg-and-me.html), S/MIME enterprise-only, metadata leakage even with body encryption.
- **UX brokenness** — HTML stuck in 2005, no reliable recall, reply-all storms at scale (Microsoft's own 1997 internal incident is a canonical case study), scheduling back-and-forth reflected in the multi-billion-dollar valuations of Calendly and peers, threading inconsistency, top-posting chaos, attachment size limits.
- **AI-era emergent pains** — the LLM-generated share of phishing in recent industry reports is ~80%+, voice-clone CFO-fraud incidents (e.g., the [Hong Kong deepfake-video heist reported in 2024](https://www.scmp.com/news/hong-kong/law-and-crime/article/3250851/everyone-looked-real-multinational-firms-hong-kong-office-loses-hk200-million-after-scammers-stage)), and prompt injection into inbox-reading assistants (see §2).

## What email gets right

Any replacement must meet or beat each of these, or it fails adoption regardless of technical superiority:

- **Federation and the right to own your address** — email is the last mass-scale federated identity system.
- **Asynchronous, durable, offline-friendly composition.**
- **Universal reach** — every adult globally has an email address.
- **Simple composition** — subject, body, attachments.
- **Long-lived searchable archive** as personal knowledge base.
- **Gmail's 99.9% spam filter genuinely works** at 333 billion messages per day.
- **Human-readable addresses** that fit on a business card.

---

Continue to [**What Marque is**](./02-what-marque-is.md).
