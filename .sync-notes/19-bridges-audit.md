# Bridges Docs Audit

2 PASS, 1 NEEDS-REVIEW, 1 FAIL.

| File | Verdict |
|------|---------|
| `backend/bridges/overview.mdx` | ✓ PASS (generic intro) |
| `backend/bridges/rubic-bridge.mdx` | ✗ FAIL |
| `sdk/bridges/overview.mdx` | ✓ PASS (generic intro) |
| `sdk/bridges/rubic-bridge-test.mdx` | ⚠ NEEDS-REVIEW |

## backend/bridges/rubic-bridge.mdx — FAIL

- Line 6: Typo "Brdige" → "Bridge"
- Line 3: file path `src/features/bridges/rubic.ts` not verifiable in node repo as referenced
- Line 36: references `src/libs/network/manageBridges.ts` not found in current source
- Mixes backend and SDK class names without clarity. Backend doc shouldn't reference an SDK file path.

## sdk/bridges/rubic-bridge-test.mdx — NEEDS-REVIEW

- Line 6: references **"RubicService"** — same fictional class name we removed from `sdk/cookbook/swap/crosschain-swap.mdx` earlier this sync. Real class is `RubicBridge` (`sdks/src/websdk/bridge.ts` and `sdks/src/bridge/rubicBridge.ts`).
- Line 3: references `rubic.test.ts` — file path verified at `/Users/tcsenpai/kynesys/sdks/src/tests/bridge/rubic.test.ts` ✓
- Line 20: references `yarn test:rubic-service` — script likely stale (project uses Bun). Verify package.json scripts.
- No imports shown — should align with the cookbook fix that imports `RubicBridge` from `@kynesyslabs/demosdk/websdk`.

## Cross-cutting

- `RubicBridge` is the real class. `RubicService` is fictional (same hallucination class we hit twice now).
- `BridgeTradePayload` accepts only `NATIVE | USDC | USDT` (`bridgeTradePayload.ts:13–14`) ✓
- `NativeBridgeMethods` exists (`sdks/src/bridge/index.ts:16–18`) but is undocumented. Possible future doc gap.

## Recommended actions

1. backend/bridges/rubic-bridge.mdx: rewrite or remove. Either describe the node-side bridge handlers (verify they exist) or shrink to a pointer at the SDK doc.
2. sdk/bridges/rubic-bridge-test.mdx: rename RubicService → RubicBridge; use Bun (`bun test`) not Yarn.
3. Optional: document `NativeBridgeMethods` if intended for public use.
