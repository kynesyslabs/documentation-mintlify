# Cross-Chain Docs Audit (10 chain pages)

**Verdict: 7 PASS (with minor issues), 2 FAIL, 1 NEEDS-REVIEW.**

Source-of-truth: `/Users/tcsenpai/kynesys/sdks/src/multichain/core/{evm,btc,solana,aptos,multiversx,near,ton,xrp,ten,ibc}.ts` + `index.ts`.

## Per-chain status

| Chain | Status | Key Issues |
|-------|--------|-----------|
| EVM | ⚠ NEEDS-REVIEW | `readFromContract`/`writeToContract` doc signatures simplified; users will hit runtime errors trying simple-param calls |
| BTC | ✓ PASS | `getBalance()` returns sum of all UTXOs (main + change), not just wallet — minor doc gap |
| Solana | ✗ FAIL | Line 86: import path is placeholder `xm-<localsdk\|websdk>`; line 114 references `prepareTransfer` (real method is `preparePay`) |
| APTOS | ✓ PASS | Param name `receiver` vs doc `recipient` (cosmetic) |
| MultiversX (EGLD) | ✓ PASS | All methods + signatures match |
| NEAR | ✓ PASS | All methods + signatures match |
| TON | ✓ PASS-with-rename | Doc `prepareTransfer`/`prepareTransfers`; SDK `preparePay`/`preparePays`. Functionally identical, doc names are stale |
| XRPL | ✓ PASS | All methods correct; WebSocket-only note accurate |
| TEN | ✗ FAIL | ten.mdx is a stub ("Incoming...!"); ten.ts impl is incomplete (`signMessage` returns "Not implemented", `verifyMessage` returns false) |
| IBC | ✓ PASS | All methods match; `getBalance(address, options.denom)` shape documented correctly |

## Import path canonicalization

Per-chain docs should use one of:
- `@kynesyslabs/demosdk/xm-websdk` (the canonical barrel for browser)
- `@kynesyslabs/demosdk/xmcore/<chain>` (per-chain subpath, e.g. `xmcore/evm`)

Drift table:

| File | Used path | Should be |
|------|-----------|-----------|
| solana.mdx:86 | `@kynesyslabs/demosdk/xm-<localsdk\|websdk>` (placeholder) | `@kynesyslabs/demosdk/xm-websdk` |
| Other 9 chains | `xm-websdk` | OK |

## Method-rename drift

| Chain | Doc method | Real method |
|-------|-----------|-------------|
| Solana | `prepareTransfer(addr, amount)` | `preparePay(addr, amount)` |
| TON | `prepareTransfer(addr, amount)` | `preparePay(addr, amount)` |
| TON | `prepareTransfers([...])` | `preparePays([...])` |

## Top 3 most-concerning issues

1. **Solana import placeholder unrendered** (solana.mdx:86): `xm-<localsdk\|websdk>` is template syntax that was never resolved. Anyone copy-pasting hits an immediate import error.
2. **TEN documentation absent**: stub only ("Incoming...!"). Worse, the SDK impl itself isn't ready — `signMessage`/`verifyMessage` return placeholder values. Either the page should be hidden from nav until ready, or the stub should explain "TEN support is in development."
3. **EVM contract methods simplified**: `readFromContract(contract, funcName, args)` and `writeToContract(contract, funcName, args)` shown as simple. Real signatures require a `Contract` instance from `getContractInstance(address, abi)` plus an options object. Following the doc literally produces runtime errors.

## Methods documented in source but missing from docs

- **EVM**: `listenForEvent`, `listenForAllEvents`, `getContractInstance`, `waitForReceipt` (only mentioned briefly in Advanced section)
- **APTOS**: `getAPTBalance`, `getCoinBalance`, `getAPTBalanceDirect`, `getCoinBalanceDirect`, `readFromContractDirect` (partial coverage)
- **Solana**: `runAnchorProgram`, `runRawProgram`, `getProgramIdl` (covered in "Programs" section ✓)

## Other reference files in `sdk/cross-chain/`

- `general-layout-of-the-xm-sdks.mdx`, `the-xmscript.mdx`, `identities.mdx`, `unstoppable-domains.mdx` — not deep-audited in this pass; recommend sweep with the same lens since they likely use the same import-path pattern that some chain docs got wrong.
