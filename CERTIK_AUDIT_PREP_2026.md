# NeuralNet Coin (NNC) — Security Audit Preparation Summary  
*(Prepared for CertiK or an equivalent independent auditor)*

## 1. Overview

NeuralNet Coin (NNC) is a decentralized, Social-AI-driven token built on the **Solana Token-2022** standard.

The current canonical references are:

- **NeuralNet Coin (NNC) Whitepaper — 2026 Edition (“Social AI First”, Rev 4)**  
- **NeuralNet Coin (NNC) Roadmap — 2026–2029+**  
- **NeuralNet Coin Mint Transparency Report (Token-2022, 9 decimals, immutable)**  

The 2026 whitepaper and roadmap define a phased build:

1. **Phase 1 — Foundations (Years 1–2)**  
   - Token-2022 NNC mint (1,000,000,000 supply, 9 decimals, immutable).  
   - **Social-AI Trust Engine (alpha → beta)** for bot / scam / fake-engagement detection.  
   - **DecentraNet Marketplace (digital-only beta)** with NNC payments and optional escrow.  
   - **NNC Fee Router** — a 3% capped (max 25 NNC/tx) ecosystem fee on **defined flows** only.  
   - **Engage-to-Earn v0** — off-chain Engagement Credits → on-chain NNC redemptions.

2. **Phase 2 — Hardening & Phygital Seed (Years 2–3)**  
   - Trust Engine beta; vendor verification and reputation for DecentraNet.  
   - Selective phygital commerce pilots with NFT receipts.  
   - Engage-to-Earn v1 with stricter anti-Sybil controls.  
   - Governance v1 (Reputation DAO) for non-critical decisions.

3. **Phase 3–4 — Industry Pilots & zk-Privacy Research / Expansion (Years 3–5+)**  
   - Limited, sector-specific pilots (creator economy, DeSci, etc.).  
   - **NNC Privacy Router** + shielded pool (zk-privacy layer) on testnet → possible mainnet.  
   - Multi-industry expansion if earlier phases are stable and adopted.

This audit prep document aligns the **security scope and priorities** with that phased plan.

The immediate goal is to prepare for **Phase-appropriate audits**, not to over-specify future modules that do not exist yet.

---

## 2. Audit Goals

### 2.1 Primary Goals (Near-Term / Phase 1–2)

1. **Correctness & Safety of Core On-Chain Contracts**
   - Token-2022 mint configuration (immutability, authorities revoked).  
   - NNC Fee Router (3% capped fee on specific flows).  
   - DecentraNet marketplace contracts (escrow / order lifecycle).  
   - Engage-to-Earn redemption contract (off-chain credits → on-chain NNC).  

2. **Robust Access Control & Upgrade Safety**
   - Clear separation of authorities (treasury, router admin, DAO execution, etc.).  
   - Well-defined upgrade paths (or immutability) for each program.  
   - Emergency pause mechanisms for critical contracts where appropriate.

3. **Economic Integrity**
   - No hidden or unbounded minting paths.  
   - No unintended transfer fees beyond the defined 3% router behavior.  
   - Enforcement of redemption caps and budget limits as described in the whitepaper.

4. **Alignment with the Published Threat Model**
   - Protection against common Solana-specific issues (CPI abuse, PDA collisions, address spoofing, account re-initialization, replay, etc.).  
   - Validation that assumptions in the whitepaper/roadmap match how the contracts actually behave.

### 2.2 Secondary Goals (Later Phases)

- Formal verification or extended review of:  
  - NNC Privacy Router + shielded pool (zk-privacy layer).  
  - Advanced DAO / governance execution modules.  
  - Complex treasury automation and any future staking or yield-bearing features (if implemented).

---

## 3. In-Scope Components (Phase 1–2)

### 3.1 On-Chain Programs

**A. NNC Token-2022 Mint (Mainnet)**

- **Mint Address:** `BhwvuTEBCdYYCUVWSCmpekG42TrpNQxNUGHyR5rQtxtF`  
- **Config (per Transparency Report):**
  - Total Supply: `1,000,000,000` NNC (fixed).  
  - Decimals: `9`.  
  - Mint Authority: **Revoked**.  
  - Freeze Authority: **Revoked**.  
  - Metadata Update Authority: **Disabled** (immutable metadata).  
- **Audit tasks:**
  - Confirm the on-chain config matches the transparency report and whitepaper.  
  - Verify metadata URI and hash are consistent with public GitHub/IPFS sources.  
  - Confirm there is no remaining authority that can alter supply or core token behavior.

**B. NNC Fee Router**

- Solana program (Anchor) that applies a **3% fee capped at 25 NNC per tx** on specific ecosystem flows:  
  - DecentraNet marketplace NNC payments (buyer → seller).  
  - Escrow releases.  
  - Engage-to-Earn redemptions.  
  - B2B subscription flows that choose to route through it.
- **Audit tasks:**
  - Verify fee calculation logic (3% with 25 NNC cap) for all paths.  
  - Verify only intended flows are routed through this program; there is **no global transfer tax**.  
  - Check for rounding issues, double-charging risk, or bypass paths.  
  - Review how router fees are distributed (treasury, burn, operations) and confirm no privileged drains.

**C. DecentraNet Marketplace (Digital-Only)**

- Programs for listing, buying, and finalizing digital service orders with optional escrow.  
- **Audit tasks:**
  - Validate order lifecycle: create → fund → fulfil → release/cancel.  
  - Ensure escrowed NNC cannot be stolen or locked permanently under normal usage.  
  - Check seller/buyer authorization checks, PDA derivations, and access control.  
  - Confirm compatibility with the NNC Fee Router (no double-spend / race conditions).

**D. Engage-to-Earn Redemption Contract**

- Converts off-chain **Engagement Credits** into on-chain NNC during **redemption epochs**.  
- **Audit tasks:**
  - Verify that only authorized “redemption operators / oracles” can initiate redemptions.  
  - Confirm that **epoch-level caps** exist and enforce the budget limits defined by governance/treasury.  
  - Ensure that credits cannot be redeemed twice (anti-replay / nullifier design if applicable).  
  - Validate that router fees are properly applied to redemption payouts.

**E. Governance / Treasury Multisig**

- Programs / accounts for:  
  - Treasury control (NNC + SOL/stables).  
  - Upgrade authority for router, marketplace, and redemption contracts.  
- **Audit tasks:**
  - Confirm multisig thresholds and signer sets match documentation.  
  - Verify there are no single-signer “back doors” or escape hatches.  
  - Review any DAO execution bridges (if present) for re-entrancy and abuse risk.

### 3.2 Out-of-Scope for Initial Audits (Documented Only)

The following are **design-level only** in the current roadmap and should be treated as informational, not yet audited code:

- NNC Privacy Router + shielded pool (zk-privacy layer).  
- Advanced DAO execution modules (on-chain proposal execution beyond spending / parameter updates).  
- Any future staking / yield products that are not yet specified or deployed.  
- Game / metaverse integrations (NeuralNet Universe / City).

---

## 4. Off-Chain Components (Context Only)

The following systems are **not** smart contracts but are security-critical:

1. **Social-AI Trust Engine**  
   - Off-chain pipeline that scores accounts/content for bots, scams, fake engagement.  
   - Feeds into Engage-to-Earn and marketplace trust decisions.  

2. **Engagement Credits Service**  
   - Stores per-user off-chain credits before redemption.  
   - Applies rate limits, anomaly detection, and manual review flags.  

3. **Back-Office Tools / Admin Panels**  
   - UIs for treasury management, redemption ops, vendor review, etc.  

For a full security picture, we will provide:

- Architecture diagrams (data flows, admin access, logging).  
- Role / permission matrix for admins, operators, and oracles.  
- Incident response procedures and audit-log retention policy.

---

## 5. Threat Model Summary

Key risks we expect an auditor to consider:

1. **On-Chain Financial Risk**
   - Unauthorized minting or draining of NNC.  
   - Escrow funds being stuck or stolen.  
   - Router mis-configuration leading to over- or under-charging fees.  

2. **Access Control & Governance Risk**
   - Single point of failure in multisig or upgrade authorities.  
   - Malicious or compromised admin keys.  
   - Unsafe DAO execution paths.

3. **Redemption & Reward Abuse**
   - Bypassing Engage-to-Earn caps and budgets.  
   - Double redemption of off-chain credits.  
   - Manipulation of oracle / operator roles.

4. **Integration Risk**
   - Incorrect use of PDAs or CPIs in router / marketplace / redemption flows.  
   - Re-entrancy or race conditions when multiple programs interact.

5. **Future zk-Privacy Risk (Design-Level)**
   - Double-spend in shielded pool.  
   - Leaking privacy through metadata or usage patterns.  
   - Compliance conflicts if privacy is deployed without proper controls.

---

## 6. Documentation & Artifacts to Provide to Auditor

Before formal engagement, we will assemble:

1. **Canonical Documents**
   - NNC Whitepaper — 2026 Edition (“Social AI First”, Rev 4).  
   - NNC Roadmap 2026–2029+.  
   - Mint Transparency Report with addresses and config.

2. **Technical Specs**
   - Program-by-program specs for:  
     - NNC Fee Router.  
     - DecentraNet Marketplace.  
     - Engage-to-Earn Redemption Contract.  
     - Governance / multisig layout.  
   - Interface definitions and invariants for each program.

3. **Codebase**
   - Repos for on-chain programs (Anchor).  
   - Repos for off-chain services (for context).  
   - Deployment scripts and configuration files.

4. **Address & Role Map**
   - All relevant program IDs and accounts on devnet/mainnet.  
   - List of signer keys (multisig, DAO, ops) with their responsibilities.  
   - Network configuration (devnet / mainnet, clusters used, etc.).

5. **Test Suites & Scenarios**
   - Unit and integration tests for each program.  
   - Edge-case tests covering fee caps, cancellations, disputes, and redemption caps.

---

## 7. Phased Audit Plan (Aligned with Roadmap)

**Phase 1 Audit (Foundations)**  
Scope: Token-2022 mint, NNC Fee Router, DecentraNet digital marketplace (core flows), Engage-to-Earn redemption contract, governance / multisig.  

**Phase 2 Audit (Hardening & Phygital Seed)**  
Scope: Expanded marketplace features (disputes, reputation-linked features), updated router parameters if any, upgraded redemption logic, and integration glue between programs.  

**Phase 3 Audit (zk-Privacy & Advanced DAO, When Ready)**  
Scope: NNC Privacy Router + shielded pool, advanced DAO execution paths, and any new treasury automation or staking logic.

Vendor choice (CertiK or equivalent) and exact timelines will be made once each phase’s code reaches a stable pre-audit state.

---

## 8. Next Steps

1. Finalize and tag the code versions intended for **Phase 1** audit.  
2. Produce minimal but precise specs for each in-scope contract.  
3. Compile the address / role map and update it alongside the Mint Transparency Report.  
4. Share this document, the whitepaper, the roadmap, and repos with the chosen auditor.  
5. Iterate on findings and update documentation to keep it aligned with deployed code.

---

**Prepared by:**  
**Hallbjorn (“The Architect”)**  
Founder & Developer — NeuralNet Coin (NNC)  
📧 neuralnetcoin@gmail.com
