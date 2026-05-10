# L2PS Backend Docs Audit

2 NEEDS-REVIEW, 1 FAIL. Multiple cross-file inconsistencies on encryption mode and IV length.

| File | Verdict |
|------|---------|
| `backend/l2ps-subnet-framework/overview.mdx` | ⚠ NEEDS-REVIEW |
| `backend/l2ps-subnet-framework/how-are-l2ps-transactions-handled.mdx` | ✗ FAIL |
| `backend/l2ps-subnet-framework/quickstart.mdx` | ⚠ NEEDS-REVIEW |

## overview.mdx — NEEDS-REVIEW

| Line | Claim | Reality |
|------|-------|---------|
| 15, 34 | "AES-256" (mode unspecified) | Impl uses **AES-GCM** (`sdks/src/l2ps/l2ps.ts:200`). Should clarify mode. |
| 92 | mentions "AES-256 key and IV" without specifics | quickstart claims 16-byte IV; impl uses **12 bytes** (`l2ps.ts:129`) |
| 40 | Batch size up to 10 tx | ✓ verified `ZK_CIRCUIT_MAX_BATCH_SIZE = 10` (`constants.ts:12`) |
| 21 | PLONK ZK proofs | ✓ verified (snarkjs PLONK, `zk/L2PSBatchProver.ts`) |
| 75 | 1 DEM per tx | ✓ verified `L2PS_TX_FEE = 1` (`L2PSTransactionExecutor.ts:33`) |

## how-are-l2ps-transactions-handled.mdx — FAIL

| Line | Issue | Severity |
|------|-------|----------|
| 13 | "**AES-256-CBC**" | 🔴 **WRONG** — impl uses AES-GCM (authenticated). CBC is not authenticated. Fundamental security claim error. |
| 17 | `import { L2PSEncryption } from '@kynesyslabs/demosdk/l2ps'` | 🔴 `L2PSEncryption` does not exist; SDK exports `L2PS` class |
| 28 | `L2PSEncryption.encrypt(innerTx, aesKey, iv)` | 🔴 fictional. Real API: `await L2PS.create()` then `instance.encryptTx()` |
| 49 | references `handleL2PS.ts` | file exists but pseudocode signatures don't match real method shape |
| 55, 61 | `network.decrypt()` and `L2PSTransactionExecutor.execute(decryptedTx)` 1-arg form | 🔴 real: `execute(l2psUid, tx, l1BatchHash, simulate?)` (4 params). Examples won't compile. |

## quickstart.mdx — NEEDS-REVIEW

| Line | Issue |
|------|-------|
| 31–32 | `openssl rand -hex 16` for IV (16 bytes = 32 hex). **Impl wants 12 bytes.** Following this guide produces wrong IV size. |
| 196 | "32 hex characters" claim mirrors the wrong 16-byte assumption |
| 74–76 | PLONK key dirs `keys/batch_5/`, `keys/batch_10/` ✓ verified |
| 178 | "50 DEM burned for 50 tx" ✓ verified (1 DEM/tx) |
| 178 | "~5 proofs (1 per batch of 10)" ✓ verified (MAX_BATCH_SIZE=10) |

## Top 3 issues

1. **AES-256-CBC vs AES-GCM** (how-are-l2ps-transactions-handled.mdx:13). Security claim materially wrong.
2. **IV length 16 vs 12 bytes**. Cross-file inconsistency. Quickstart users generate wrong key material → all L2PS ops fail.
3. **`L2PSEncryption` class fictional** (how-are-l2ps-transactions-handled.mdx:17, 28, 61). Code examples not executable. Real API is `L2PS.create()` + `instance.encryptTx()` (already documented correctly in `sdk/websdk/l2ps/overview.mdx` from earlier session).

## Cross-link

`sdk/websdk/l2ps/overview.mdx` (rewritten earlier this sync) already documents the correct SDK API. Backend docs should align.
