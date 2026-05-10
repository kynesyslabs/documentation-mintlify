# GCR Documentation Audit Report
**Date:** 2026-05-10  
**Auditor:** Claude Code Search Agent  
**Scope:** GCR schema & sync mechanism verification against source code

---

## Files Audited

1. `/backend/global-change-registry/overview.mdx` — PASS
2. `/backend/global-change-registry/gcr-structure.mdx` — NEEDS-REVIEW  
3. `/backend/global-change-registry/how-is-gcr-synced.mdx` — FAIL

---

## 1. overview.mdx — PASS

**Status:** Accurate  
**Findings:** High-level description correct. GCR mechanism and role as trusted reference point match implementation (see `gcr.ts:14-24`, `handleGCR.ts:157-160`).

---

## 2. gcr-structure.mdx — NEEDS-REVIEW

### Schema Claims Analysis

**Claim 1: Three tables + one metatable**  
Lines 10-32 list:
- `global_change_registry` (primary table)
- `global_change_registry_subnets_txs` (subnet txs)
- `gcr_tracker` (tracking hashes)
- `gcr_hashes` (metadata/combined hashes)

**Finding:** ✅ ACCURATE but INCOMPLETE
- Implementation uses **GCRv2 architecture** with additional tables: `gcr_main` (mapped to `@Entity("gcr_main")` in `GCR_Main.ts:13`)
- Entity `GCRSubnetsTxs:9` maps to `@Entity("global_change_registry_subnets_txs")`
- Entity `GCRHashes:6` maps to `@Entity("gcr_hashes")`
- Old `GlobalChangeRegistry` entity still used for legacy balance operations (see `gcr.ts:176-198`)
- **Recommendation:** Document GCRv2 alongside legacy tables for clarity

**Claim 2: Balance column type**  
Section does not specify data type.

**Finding:** ⚠️ CRITICAL OMISSION
- Source code (`GCR_Main.ts:36-44`): Balance is **`numeric(38, 0)`** (arbitrary-precision integer, NOT `bigint` or `numeric`)
- This deviates from older references to "JS number" storage
- Configured with `transformer: bigintNumericTransformer` → ORM returns `bigint` at application boundary
- Raw SQL callers must explicitly coerce via `BigInt(row.balance)` (see `GCR_Main.ts:33-34`)
- **Recommendation:** Add explicit section: "Balance is stored as PostgreSQL `numeric(38, 0)` for precision; ORM surfaces as `bigint`"

**Claim 3: Extended JSONB field**  
Lines 20: "extended property simplifies tracking tokens, NFTs, and cross-context properties"

**Finding:** ✅ ACCURATE (legacy support)
- Still used in `GlobalChangeRegistry` entity (`gcr.ts:187`, `handleGCR.ts:204`)
- GCRv2 replaces this with typed fields in `GCRMain.identities` (`GCR_Main.ts:45`)

---

## 3. how-is-gcr-synced.mdx — FAIL

**Status:** Unimplemented  
**Finding:** Content is `TODO` placeholder (line 9).

**Source Analysis:**
- Sync logic exists in `Sync.ts:14-146` (peer gossip, block discovery)
- GCR-specific sync handled via `HandleGCR.applyTransactions()` (`handleGCR.ts:783-899`)
- Peer batch sync mechanism:
  - `HandleGCR.partitionIndependentTxs()` (line 632) groups txs by affected entities (union-find algorithm)
  - Concurrent execution across groups with `CONCURRENCY=8` (line 835)
  - Sequential within groups to preserve ordering (nonce increments, etc.)
  - Bulk updates via `bulkUpdateAssignedTxs()` (line 318-351) using raw SQL

**Recommendation:** Implement section covering:
1. Peer discovery via `getHigestBlockPeerData()` (deprecated but active)
2. Batch partition strategy (union-find, independent groups)
3. Concurrent + sequential hybrid execution model
4. assignedTxs tracking per account

---

## Identity Routine Integration Verification

**Requirement:** All identity systems (nomis, ethos, humanpassport, tlsn, ud, web2, xm, zk, pqc) should write to GCR.

**Verification:** ✅ CONFIRMED
- `GCRIdentityRoutines.ts:28` imports all identity apply functions from `./routines`
- Routines directory contains (`gcr_routines/routines/`):
  - ✅ `nomisRoutines.ts`
  - ✅ `ethosRoutines.ts`
  - ✅ `humanpassportRoutines.ts`
  - ✅ `tlsnRoutines.ts`
  - ✅ `udRoutines.ts`
  - ✅ `web2Routines.ts`
  - ✅ `xmRoutines.ts`
  - ✅ `zkRoutines.ts`
  - ✅ `pqcRoutines.ts`
- All routed through `handleGCR.applyGCREdit()` switch statement (lines 1040-1145)

---

## Methods & RPCs: Fictional Method Detection

**Checked against `gcr.ts` and `handleGCR.ts`:**

- ✅ `getGCRNativeBalance(address)` — Line 222 (real, returns `bigint`)
- ✅ `setGCRNativeBalance(address, balance, txHash)` — Line 528 (real)
- ✅ `getGCRTokenBalance(address, tokenAddress)` — Line 241 (real)
- ✅ `getGCRNFTBalance(address, nftAddress)` — Line 260 (real)
- ✅ `applyTransactions(txs[], isRollback)` — Line 783 (real, batched)
- ✅ `applyTransaction(entities, tx, isRollback, simulate)` — Line 488 (real, single-tx)
- ✅ `getAccountByTwitterUsername(username)` — Line 606 (real, GCRMain-based)
- ✅ `getAccountByIdentity(identity)` — Line 638 (real, web2 + xm support)
- ✅ `awardPoints(accounts[])` — Line 1254 (real, creates identity transaction)
- ⚠️ `getGCRLastBlockBaseGas()` — Line 279 (returns hardcoded `1`, marked TODO)
- ⚠️ `getGCRChainProperties()` — Line 289 (queries special "DEMOS Network" entry, untested)

**Finding:** No fictional RPCs detected. All documented methods have implementations.

---

## Summary

| File | Status | Key Issues |
|------|--------|-----------|
| overview.mdx | PASS | Accurate high-level description |
| gcr-structure.mdx | NEEDS-REVIEW | Missing: balance column type spec; incomplete GCRv2 docs |
| how-is-gcr-synced.mdx | FAIL | Unimplemented (TODO placeholder) |
| Identity routines | PASS | All 9 systems confirmed + routed through GCRIdentityRoutines |
| Methods/RPCs | PASS | No fictional methods; all implementations found |

---

## Action Items

1. **IMMEDIATE:** Implement `how-is-gcr-synced.mdx` with peer batch sync details
2. **HIGH:** Add balance column type documentation (`numeric(38,0)` + transformer)
3. **MEDIUM:** Document GCRv2 architecture & migration from legacy GlobalChangeRegistry
4. **LOW:** Clarify `getGCRLastBlockBaseGas()` and `getGCRChainProperties()` usage (marked TODO in code)
