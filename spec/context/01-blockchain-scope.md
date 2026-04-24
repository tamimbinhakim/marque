# Blockchain scope — where chain earns its place, and where not

> The empirical record on web3 email is unambiguous: on-chain message content does not work. Bitcoin anchoring, DID sovereignty, and transparency logs do work — and Marque uses each of them in a narrow, defensible role.

**You will learn**

- The failure patterns of pure-web3 email projects.
- The four narrow roles blockchain primitives earn in Marque.
- Marque's firm red lines against on-chain content, per-message gas, and mandatory wallets.

---

## 1. The empirical record

Blockchain's role in messaging has been the subject of more marketing than engineering. The empirical record through 2026:

- **Mailchain**, the best-architected pure-web3 email project, is shutting down 5 May 2026 per its own front-page banner.
- **Skiff** — the best-UX privacy email, ~1M+ users — was acquired by Notion on 9 February 2024 (Sequoia had invested in both); services shut down ~9 August 2024 and email forwarding terminated on 9 February 2025.
- **LedgerMail**'s on-chain-per-message design would exhaust XDPoS capacity at 0.06% of global email throughput.
- **Dmail**'s 50-million-user claim is an airdrop-farming artifact.
- **EtherMail** has correctly pivoted to wallet-native marketing CRM; "read-to-earn" is a Sybil incentive, not a correspondence model.

The intersection of cost, GDPR right-to-erasure incompatibility, wallet-UX key-loss catastrophe, and sub-1 000-TPS throughput makes **on-chain message content a non-starter** for human correspondence.

## 2. The four narrow roles blockchain earns

Yet four narrow roles do earn a place for cryptographic-chain primitives — and Marque uses each precisely.

### 2.1 OpenTimestamps on Bitcoin for legal-tier timestamps

[OpenTimestamps (OTS)](https://opentimestamps.org) aggregates arbitrary numbers of hashes into a Merkle tree and commits only the root to one Bitcoin transaction per calendar cycle. Users pay nothing per timestamp; calendar servers absorb Bitcoin fees; verification requires no trusted third party; proof survives any vendor's collapse.

For the Certified tier of non-repudiation, OTS is **strictly superior to RFC 3161 TSAs** on vendor-independence and cost, while complementary on immediate legal weight. Marque's Certified-tier envelopes carry both:

- **RFC 3161 qualified timestamps** for immediate eIDAS admissibility.
- **OTS proofs** for long-term archival.

### 2.2 Optional ENS-based naming fallback

Marque's default resolution is DNS-rooted. For users who want DNS-independence and squat-resistance, ENS provides a proven ~2.8-million-name precedent with working wallet integration.

The 2026 ENS Labs decision to keep ENSv2 on Ethereum L1 (after Ethereum's gas-limit doubling and L1-fees collapse in 2025) makes this more accessible than it was. **No user is forced to touch a wallet**; those who want chain-anchored naming can have it.

### 2.3 Sigstore-style transparency log for delivery receipts

Not blockchain — an **append-only Merkle log with witness-signed tree heads**, proven at internet scale by Google's and Let's Encrypt's Certificate Transparency and Sigstore's Rekor.

Tree heads are optionally checkpointed into Bitcoin weekly via OTS for belt-and-suspenders verifiability. Gives global append-only guarantees without running a cryptocurrency.

### 2.4 Censorship-resistant root-naming fallback

If a nation-state compels DNS revocation of a provider's domain, the user's DID remains resolvable via DHT and optional blockchain anchor. **This is disaster recovery, not a primary path.**

## 3. Red lines

**Marque's blockchain red lines are firm:**

- **No on-chain message content.**
- **No per-message gas.**
- **No mandatory wallet.**
- **No protocol token.**

Any critique that these exclusions are insufficiently decentralized is answered by: Bitcoin provides the anchoring, DIDs provide the sovereignty, and the provider tier is commodity-competitive. Every non-chain alternative — DHTs for discovery, MLS for group keying, HPKE for transport, content-addressed storage via BLAKE3, federated reputation gossip — is demonstrably cheaper, faster, and more GDPR-compatible than its chain equivalent.

---

Continue to [**Commercial model**](./02-commercial-model.md).
