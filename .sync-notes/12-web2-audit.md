# Web2 SDK Docs Audit (DAHR, keyserver-oauth, making-requests, quick-start)

**Verdict: 4 PASS, 1 FAIL** — DAHR field naming is wrong throughout.

Source-of-truth: `/Users/tcsenpai/kynesys/sdks/src/websdk/Web2Calls.ts`, `/Users/tcsenpai/kynesys/sdks/src/keyserver/`, `/Users/tcsenpai/kynesys/sdks/src/tlsnotary/`.

## Per-file verdicts

| File | Verdict | Notes |
|------|---------|-------|
| `sdk/web2/quick-start.mdx` | ✓ PASS | `createDahr()` and `startProxy()` signatures correct |
| `sdk/web2/making-requests.mdx` | ✓ PASS | All request patterns match `IStartProxyParams` |
| `sdk/web2/keyserver-oauth.mdx` | ✓ PASS | `KeyServerClient` and OAuth methods all verified against source |
| `sdk/web2/identities/overview.mdx` | ✓ PASS | Accurate structural overview (already touched earlier in sync) |
| `sdk/web2/dahr-api-reference/overview.mdx` | ✗ FAIL | Lines 57–58: type shows `dataHash`/`headersHash` but real fields are `responseHash`/`responseHeadersHash` |
| `sdk/web2/dahr-api-reference/types.mdx` | ✗ FAIL | Line 35: narrative claims `dataHash`/`headersHash` stored on-chain (inconsistent with type definition below); line 68: example mixes old (`res.dataHash`) and new (`res.responseHeadersHash`) |

## DAHR field-name drift

| Doc field | Actual field |
|-----------|--------------|
| `dataHash` | `responseHash` |
| `headersHash` | `responseHeadersHash` |

## Root cause

A September 2025 merge conflict (visible in `sync-conflict-*` branches in the SDK repo) was resolved in favor of the `responseHash`/`responseHeadersHash` naming. The DAHR docs were never synced to that resolution.

## Recommended actions

1. In `sdk/web2/dahr-api-reference/overview.mdx`: replace `dataHash` → `responseHash`, `headersHash` → `responseHeadersHash` (lines 57–58).
2. In `sdk/web2/dahr-api-reference/types.mdx`:
   - Line 35: rewrite the narrative paragraph to use the current names.
   - Line 68: replace `res.dataHash` with `res.responseHash`.
   - Audit the rest of the file for any other lingering old-name references.
3. Confirm against `/Users/tcsenpai/kynesys/sdks/src/websdk/Web2Calls.ts` once more before publishing — the merge resolution is what the source carries today.

## Cross-cutting

- No references to deprecated global `DemosWebAuth`. ✓
- Import paths look correct (no `@/` aliases). ✓
- No fictional classes detected outside the DAHR field-naming issue.
