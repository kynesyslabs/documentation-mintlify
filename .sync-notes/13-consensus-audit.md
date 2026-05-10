# Consensus Mechanism Documentation Audit vs Node Implementation

**Date:** 2026-05-10  
**Scope:** 7 files from `/backend/consensus-mechanism/` vs `/node/src/libs/consensus/v2/` and related configs  
**Hard Numbers Checked:** TPS, block times, validator counts, shard sizes  
**Protocol Claims Verified:** PoR-BFT, slashing, finality, sybil resistance  

---

## Per-File Assessment

### 1. overview.mdx — PASS
- **Claim:** "PoR-BFT framework aims to address [scalability, security, efficiency]"
- **Status:** Generic introduction, no hard numbers to verify ✓
- **Code Reference:** `/node/src/libs/consensus/v2/PoRBFT.ts` implements consensus mechanism

---

### 2. unparalleled-scalability.mdx — NEEDS-REVIEW (Lines 5–15)
- **Claim:** "PoR-BFT's parallel block validation mechanism allows for concurrent processing of multiple transactions within a shard"
- **Finding:** **Code does NOT implement true parallel/concurrent multi-shard validation.** Single consensus routine runs sequentially per shard:
  - `consensusRoutine()` forks one block at a time per shard (line 50, PoRBFT.ts)
  - No multi-shard concurrent processing evidence in codebase
  - getShard() returns sequential shard selection, not parallel shards
- **Documentation claim:** Technically misleading; should say "serial per-shard" not "parallel"
- **Severity:** Medium — implies higher throughput than actually delivered

---

### 3. enhanced-security.mdx — NEEDS-REVIEW (Lines 8–15)
- **Claim:** "Forced exclusion of malicious nodes by continuously evaluating blockchain status mathematically through pseudorandom seeds"
- **Finding:** 
  - **Actual:** Nodes with divergent seeds are simply out-of-sync and won't participate — not actively "forced excluded"
  - getCommonValidatorSeed() comment (line 9) says: "unsynced/malicious nodes will produce different seeds, enabling automatic network coordination"
  - **No active slashing/exclusion logic found** — nodes are excluded via sync status check only (getShard.ts line 16: `peer.sync.status && Math.abs(peer.sync.block - lastBlockNumber) <= 1`)
  - Not a security mechanism, just a synced-node filter
- **Severity:** Medium — misleading framing of passive sync-check as active malicious node mitigation

---

### 4. decentralization-in-por-bft.mdx — NEEDS-REVIEW (Lines 12–20)
- **Claim:** "PoR-BFT implements weighted voting system based on mathematical proofs, ensuring more democratic decision-making"
- **Finding:**
  - **Voting system verified:** 2/3 supermajority (SUPERMAJORITY_NUMERATOR=2, DENOMINATOR=3) ✓
  - **Stake-weighted voting confirmed:** Network Governance docs state "vote weight = staked_amount @ snapshotBlock" ✓
  - **"Mathematical proofs" claim:** Vague and unsupported; no novel proof mechanism found
  - CVSA generates deterministic seed (SHA-256) but is not a "proof" in formal sense
- **Severity:** Low — broad marketing language, core mechanics correct

---

### 5. comparative-advantage.mdx — PASS (with caveats)
- **Claim:** Comparisons vs PoW, PoS, PoA energy/scalability
- **Status:** Relative comparisons are defensible but no concrete numbers given ✓
- **Note:** Claims "high throughput" and "efficient resource distribution" but provides no TPS benchmarks
- **Verification:** Cannot verify without published benchmarks

---

### 6. addressing-potential-criticisms.mdx — NEEDS-REVIEW (Line 8–10)
- **Claim:** "Continuous evaluation of nodes based on pseudorandom seed verification ensures compromised nodes are identified and excluded"
- **Finding:** Same as #3 — passive sync filtering, not active exclusion
  - No slashing mechanism exists (grep for "slash" returns 0 results)
  - Validator Lifecycle docs explicitly state: "slashing is on the Phase-2 roadmap"
  - Current Phase 1 only has UNSTAKE_LOCK_BLOCKS=1000 as economic deterrent
- **Severity:** High — directly contradicts documented roadmap

---

### 7. conclusion.mdx — PASS
- Generic forward-looking statement, no testable claims

---

## Cross-File Consistency Check vs Backend Docs

### Network Governance (overview.mdx) — VERIFIED ✓
| Parameter | Claimed | Actual (constants.ts) |
|-----------|---------|----------------------|
| `CONSENSUS_TIME` | 1 (=1000ms) | ✓ consensusTime env var → blockTimeMs*1000 |
| `VOTING_WINDOW_BLOCKS` | 100 | ✓ (constants.ts:16) |
| `GRACE_PERIOD_BLOCKS` | 50 | ✓ (constants.ts:19) |
| `SUPERMAJORITY` | 2/3 | ✓ (2n/3n ceiling-rounded, line 27) |
| `SHARD_SIZE` (default) | 4 | ✓ (defaults.ts, networkUpgrade/constants.ts) |

### Validator Lifecycle — CRITICAL MISMATCH (validator-lifecycle/overview.mdx)
- **Claim:** "slashing is on the Phase-2 roadmap; the lock alone already keeps the malicious stake at risk"
- **Consensus docs claim:** "Continuous evaluation...ensures...malicious nodes are excluded"
- **Contradiction:** Consensus docs overstate current phase-1 capabilities
- **Slashing status:** NOT implemented; only UNSTAKE_LOCK_BLOCKS=1000 exists

---

## Top 3 Issues

### 🔴 ISSUE 1: Slashing Claims Not Rooted in Phase 1
**Files:** `enhanced-security.mdx` (lines 8–10), `addressing-potential-criticisms.mdx` (lines 8–10)  
**Claim:** "Forced exclusion of malicious nodes...actively mitigates potential threats"  
**Reality:** 
- Phase 1 has only passive sync-based filtering (getShard.ts)
- True slashing promised for Phase 2 (validator-lifecycle/overview.mdx confirmed)
- Current mitigation is economic lock (1000 blocks) on voluntary unstake, not active exclusion
- **Fix:** Clarify that dynamic shard rotation + sync checks are passive, not "forced exclusion"

### 🔴 ISSUE 2: Parallel Block Validation Claim Overstates Throughput
**File:** `unparalleled-scalability.mdx` (lines 10–15)  
**Claim:** "Parallel block validation mechanism allows concurrent processing of multiple transactions within a shard"  
**Reality:**
- consensusRoutine() is strictly sequential per block
- One shard validates one block at a time
- No inter-shard parallelism visible (would require multiple consensusRoutine() instances, not found)
- **Fix:** Rephrase to "shard-based sequential validation" or document multi-shard parallelism if future

### 🟡 ISSUE 3: "Mathematical Proofs" / Pseudorandom Seed Verification Conflates Terms
**Files:** Multiple (enhanced-security, decentralization, addressing-potential-criticisms)  
**Pattern:** "pseudorandom seed verification," "mathematical proofs," "continuous evaluation"  
**Reality:**
- CVSA is deterministic SHA-256 hash of last 3 block hashes + genesis (getCommonValidatorSeed.ts)
- Not a cryptographic "proof" (no ZK, no formal verification)
- Seed is used for deterministic shard selection, not node evaluation
- sync status check (peer.sync.status) is the actual eligibility gate, not seed verification
- **Fix:** Separate concerns: (a) deterministic shard selection via CVSA seed, (b) sync-based eligibility, (c) economic lock on exit

---

## Severity Summary

| File | Status | Severity |
|------|--------|----------|
| overview.mdx | PASS | — |
| unparalleled-scalability.mdx | NEEDS-REVIEW | Medium |
| enhanced-security.mdx | NEEDS-REVIEW | High |
| decentralization-in-por-bft.mdx | NEEDS-REVIEW | Low |
| comparative-advantage.mdx | PASS | — |
| addressing-potential-criticisms.mdx | NEEDS-REVIEW | High |
| conclusion.mdx | PASS | — |

---

## Verified Hard Numbers (Phase 1)

✓ **Block Time:** 1000 ms default (CONSENSUS_TIME env var, blockTimeMs in constants)  
✓ **Shard Size:** 4 default (SHARD_SIZE env, Phase 2 = governable; Phase 1 = fixed)  
✓ **Min Validator Stake:** 10^18 (1 DEM, raw bigint)  
✓ **BFT Threshold:** 2/3 supermajority (ceiling-rounded), threshold = floor(2×votes/3) + 1  
✓ **Governance Voting Window:** 100 blocks (~100 seconds @ 1s blocks)  
✓ **Governance Grace Period:** 50 blocks  
✓ **Unstake Lock:** 1000 blocks (~1000 seconds)  
⚠️ **TPS:** No benchmarked throughput found in code or testing docs  

---

## Recommendations

1. **Rephrase slashing references** as "Phase 2 feature" with Phase 1 alternatives (economic locks, sync filtering)
2. **Clarify "parallel" claim** — change to "shard-sequential" or document multi-shard concurrency if planned
3. **Separate CVSA (deterministic seed)** from "malicious node evaluation" — they're different mechanisms
4. **Add disclaimer** that "unparalleled scalability" claims lack TPS benchmarks in Phase 1 testnet
5. **Align consensus & validator-lifecycle docs** on slashing status and Phase 2 roadmap visibility

---

**Audit Completed:** Hard numbers verified against `/node/src/config/defaults.ts`, `/node/src/features/networkUpgrade/constants.ts`, `/node/src/libs/consensus/v2/PoRBFT.ts`, `/node/src/libs/consensus/v2/routines/getShard.ts`, and governance handlers.

