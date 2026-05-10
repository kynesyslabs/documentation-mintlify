# Internal Mechanisms Audit

| File | Verdict |
|------|---------|
| `backend/internal-mechanisms/overview.mdx` | ✓ PASS (placeholder) |
| `backend/internal-mechanisms/network-time-synchronization.mdx` | ✓ PASS |
| `backend/internal-mechanisms/cross-context-identities.mdx` | ⚠ NEEDS-REVIEW |

## network-time-synchronization.mdx — PASS

Verified against `/Users/tcsenpai/kynesys/node/src/libs/utils/calibrateTime.ts`:

| Function | Source line | Status |
|----------|-------------|--------|
| `getTimestampCorrection()` | 12 | ✓ exported async |
| `getNetworkTimestamp()` | 18 | ✓ exported (sync) |
| `getMeasuredTimeDelta()` | 26 | private async |
| `getNtpTime()` | 49 | private async |
| `getFallbackNtpTime()` | 67 | private async |

Config: primary `pool.ntp.org` (line 5), fallbacks `time.google.com`, `time.windows.com`, `time.apple.com` (lines 6–10). Doc accurately treats only public functions as exposed.

## cross-context-identities.mdx — NEEDS-REVIEW

3 issues:

1. **StoredIdentities oversimplified** (lines 28–34): doc shows `Map<string, Identity>` with context "xm" | "web2". Real impl supports 8 contexts: `xm, web2, pqc, ud, nomis, humanpassport, ethos, tlsn` per `/Users/tcsenpai/kynesys/node/src/libs/blockchain/gcr/gcr_routines/GCRIdentityRoutines.ts` (lines 79–188).

2. **DB entity terminology stale**: doc references "StatusNative table" with "identities" column. Current impl uses GCRMain entity (GCRv2). Conceptually accurate but naming wrong.

3. **GCR routine references vague** (lines 39–51): vague reference to "cross-context system." Real flow: `Identities.add*Identity()` SDK methods → `inferIdentity()` (lines 86–135) → `gcr_routine` RPC with context-specific payloads (line 518–524). All SDK methods exist; no fictional methods.

## Recommended actions

1. cross-context-identities.mdx: expand StoredIdentities type to document all 8 contexts.
2. cross-context-identities.mdx: rename "StatusNative" → "GCRMain" (GCRv2).
3. network-time-synchronization.mdx: add note that helper functions are private/internal (low priority).
