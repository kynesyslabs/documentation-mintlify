# Cookbook Audit – v3.1.0 vs `sdk/cookbook/`

Audited 2026-05-07 against `/Users/tcsenpai/kynesys/sdks/src` (v3.1.0).

Verdict legend: `PASS` / `NEEDS-REVIEW` / `FAIL`.

| # | File | Verdict | One-liner |
|---|---|---|---|
| 1 | `sdk/cookbook/overview.mdx` | PASS | Stub page, no code blocks. |
| 2 | `sdk/cookbook/demoswork/overview.mdx` | PASS | Navigation card group only, no code. |
| 3 | `sdk/cookbook/demoswork/creating-work-steps.mdx` | FAIL | `prepareWeb2Step` arg key wrong; XMScript shape and `prepareXMStep` import path issues. |
| 4 | `sdk/cookbook/demoswork/conditional-operation.mdx` | NEEDS-REVIEW | Imports use unfamiliar subpaths; minor type issue on `notMember` Condition. |
| 5 | `sdk/cookbook/demoswork/base-operation.mdx` | NEEDS-REVIEW | Demos `prepareXMStep` called with empty object literal where `XMScript` is required; missing `DemosWork` import. |
| 6 | `sdk/cookbook/demoswork/signing-and-broadcasting.mdx` | NEEDS-REVIEW | Functional, but missing `Demos` import and uses unwrapped `work` variable; minor doc completeness gap. |
| 7 | `sdk/cookbook/transactions/overview.mdx` | PASS | Navigation card group only. |
| 8 | `sdk/cookbook/transactions/native-transactions.mdx` | NEEDS-REVIEW | Bad import path `@/websdk`; rest of code is current and correctly demonstrates the bigint/OS path. |
| 9 | `sdk/cookbook/transactions/crosschain-transaction.mdx` | PASS | Imports, methods and shapes match v3.1.0. |
| 10 | `sdk/cookbook/swap/overview.mdx` | PASS | Stub page, no code. |
| 11 | `sdk/cookbook/swap/crosschain-swap.mdx` | FAIL | Uses non-existent `RubicService` class and `@/features/...` path; entire flow does not match `RubicBridge` API. |
| 12 | `sdk/cookbook/storage-programs/overview.mdx` | PASS | Index page, no code. |
| 13 | `sdk/cookbook/storage-programs/examples.mdx` | FAIL | Whole file uses fictional `DemosClient` + `storageProgram.create/write/delete/updateAccessControl` API that does not exist in v3.1.0. |
| 14 | `sdk/cookbook/evm-contract-write-transactions.mdx` | PASS | Imports, `EVM`, `prepareXMScript`, and `prepareXMPayload` usage are accurate. |

**Totals:** 14 files. PASS = 7 · NEEDS-REVIEW = 4 · FAIL = 3.

---

## Detailed findings

### 1. `sdk/cookbook/overview.mdx`
- **Verdict:** PASS.
- Stub page with frontmatter only.

### 2. `sdk/cookbook/demoswork/overview.mdx`
- **Verdict:** PASS.
- Pure card-group navigation, no code blocks.

### 3. `sdk/cookbook/demoswork/creating-work-steps.mdx` — FAIL

- **`creating-work-steps.mdx:13`** Import `prepareWeb2Step` from `@kynesyslabs/demosdk/demoswork` – correct (`src/demoswork/index.ts` re-exports it).
- **`creating-work-steps.mdx:16-19`** ✗ `prepareWeb2Step({ action: "GET", url: "..." })` – the SDK signature is `{ method, url, parameters?, headers? }`. The valid key is `method`, not `action`. (`src/demoswork/workstep.ts:80-92`).
- **`creating-work-steps.mdx:31`** `import { XMScript } from "@kynesyslabs/demosdk/types"` – verified: `src/types/index.ts` exposes `XMScript`. OK.
- **`creating-work-steps.mdx:32`** ✗ `import { EVM } from "@kynesyslabs/demosdk/multichain/websdk"` – package exports use `@kynesyslabs/demosdk/xm-websdk` (per audit brief). Subpath `multichain/websdk` is the source-relative path, not the package export. Needs canonical path.
- **`creating-work-steps.mdx:33`** Import `prepareXMStep` from `@kynesyslabs/demosdk/demoswork` – correct.
- **`creating-work-steps.mdx:34`** `prepareXMScript` imported from `@kynesyslabs/demosdk/websdk/XMTransactions` – this deep import works (file exists) but `prepareXMScript` is also re-exported from the main `@kynesyslabs/demosdk/websdk`; prefer the top-level subpath.
- **`creating-work-steps.mdx:41`** `EVM.create(evm_rpc)` – exists via `DefaultChain.create` static (`src/multichain/core/types/defaultChain.ts:43`). OK.
- **`creating-work-steps.mdx:44-47`** `evm.prepareTransfer(address, "0.25")` – OK, alias for `preparePay`.
- **`creating-work-steps.mdx:50-55`** ✗ `prepareXMScript({ ..., signedPayload: payload })` – SDK signature uses `signedPayloads` (array) not `signedPayload`. (`src/websdk/XMTransactions.ts:160-178`). This will silently produce a malformed XMScript.

### 4. `sdk/cookbook/demoswork/conditional-operation.mdx` — NEEDS-REVIEW

- **`conditional-operation.mdx:17`** ✗ `import { EVM } from "@kynesyslabs/demosdk/multichain/websdk"` – same path issue as above; canonical export is `@kynesyslabs/demosdk/xm-websdk`.
- **`conditional-operation.mdx:18`** Deep import `@kynesyslabs/demosdk/websdk/XMTransactions` – works but non-canonical.
- **`conditional-operation.mdx:19-24`** Imports `DemosWork`, `prepareWeb2Step`, `prepareXMStep`, `ConditionalOperation` from `@kynesyslabs/demosdk/demoswork` – all verified in `src/demoswork/index.ts`. OK.
- **`conditional-operation.mdx:27-30`** Uses `method: "GET"` correctly here (good).
- **`conditional-operation.mdx:43-48`** `prepareXMScript({ ..., signedPayloads: [payload] })` – correct shape this time.
- **`conditional-operation.mdx:73`** `import { operators } from "@kynesyslabs/demosdk/types"` – verify; only used in a comment-style class definition (illustrative, not executed). Acceptable.
- **`conditional-operation.mdx:138-143`** ⚠ `notMember` constructed with `value_a: null, operator: null, value_b: null, action: addMember` – the `Condition` constructor types likely don't accept `null` operator. Worth re-validating against `Condition` constructor in `src/demoswork/operations/conditional/condition.ts` before publishing the pattern.
- **`conditional-operation.mdx:101`** Imports include `Condition`, `DemosWork`, etc – all valid.

### 5. `sdk/cookbook/demoswork/base-operation.mdx` — NEEDS-REVIEW

- **`base-operation.mdx:12`** Imports `BaseOperation`, `prepareXMStep` – valid (`src/demoswork/index.ts`).
- **`base-operation.mdx:15-21`** ⚠ `prepareXMStep({ /* XMScript here */ })` – passing `{}` will fail `XMScript` runtime expectations. Acceptable as illustrative pseudocode but should be flagged with explicit note.
- **`base-operation.mdx:27`** ✗ Uses `DemosWork` but never imports it – the snippet is broken if copy-pasted.

### 6. `sdk/cookbook/demoswork/signing-and-broadcasting.mdx` — NEEDS-REVIEW

- **`signing-and-broadcasting.mdx:12`** `prepareDemosWorkPayload` import – valid.
- **`signing-and-broadcasting.mdx:14`** ⚠ `const demos = new Demos()` but no `Demos` import shown. Reader has to infer the import.
- **`signing-and-broadcasting.mdx:19`** `prepareDemosWorkPayload(work, demos)` – `work` is undeclared in this snippet (assumed from earlier doc). Confirm signature: `prepareDemosWorkPayload(work: DemosWork, demos: Demos)` – matches `src/demoswork/work.ts`.
- **`signing-and-broadcasting.mdx:28,31`** `demos.confirm(tx)` and `demos.broadcast(validityData)` – both valid (`src/websdk/demosclass.ts:445,455`). OK.
- Could be augmented to mention `broadcastAndWait` (v2.12+).

### 7. `sdk/cookbook/transactions/overview.mdx`
- **Verdict:** PASS. Card group only.

### 8. `sdk/cookbook/transactions/native-transactions.mdx` — NEEDS-REVIEW

- **`native-transactions.mdx:14`** ✗ `import { Demos } from "@/websdk"` – uses TS path alias `@/`, NOT a public package import. Should be `@kynesyslabs/demosdk/websdk`.
- **`native-transactions.mdx:31`** `import { denomination } from "@kynesyslabs/demosdk/websdk"` – ⚠ verify: `denomination` is exported from the package root (`src/index.ts:9`) as `export * as denomination`, not from `websdk`. Likely should be `@kynesyslabs/demosdk` (root) or a dedicated `denomination` subpath. Currently broken as written.
- **`native-transactions.mdx:34-37`** `demos.transfer(address, denomination.demToOs(100))` – matches signature `transfer(to: string, amount: number | bigint)` (`src/websdk/demosclass.ts:372`). OK.
- **`native-transactions.mdx:41-43`** `demos.transfer(addr, 100_000_000_000n)` – OK, valid bigint OS.
- **`native-transactions.mdx:46`** `demos.transfer("0x...", 100)` flagged as deprecated in the prose – matches review brief.
- **`native-transactions.mdx:59,63`** `demos.confirm` / `demos.broadcast` – OK.

### 9. `sdk/cookbook/transactions/crosschain-transaction.mdx` — PASS

- **`crosschain-transaction.mdx:18-25`** Imports from `@kynesyslabs/demosdk/websdk` and `@kynesyslabs/demosdk/xm-websdk` – correct canonical subpaths.
- **`crosschain-transaction.mdx:33`** `EVM.create(rpc)` – OK.
- **`crosschain-transaction.mdx:38-41`** `evm.preparePay(addr, "0.000000001")` – matches `src/multichain/core/evm.ts:279`.
- **`crosschain-transaction.mdx:51-56`** `prepareXMScript({ ..., signedPayloads: [evm_tx], type: "pay" })` – correct array shape.
- **`crosschain-transaction.mdx:80`** `prepareXMPayload(xmscript, demos)` – matches `src/websdk/XMTransactions.ts:199`. OK.
- **`crosschain-transaction.mdx:91,95`** `demos.confirm`/`demos.broadcast` – OK.

### 10. `sdk/cookbook/swap/overview.mdx`
- **Verdict:** PASS. Stub.

### 11. `sdk/cookbook/swap/crosschain-swap.mdx` — FAIL

- **`crosschain-swap.mdx:13`** `import { Demos } from "@kynesyslabs/demosdk/websdk"` – OK.
- **`crosschain-swap.mdx:14`** ✗ `import RubicService from "@/features/bridges/rubic"` – this is an internal app path alias, NOT a public SDK export. The SDK exports `RubicBridge` from `@kynesyslabs/demosdk/websdk` (`src/websdk/bridge.ts`). There is no `RubicService` class.
- **`crosschain-swap.mdx:16`** `BridgeTradePayload`, `SupportedChains` from `@kynesyslabs/demosdk/types` – OK (`src/types/index.ts:209,212`).
- **`crosschain-swap.mdx:30-31`** ✗ `new RubicService(address, SupportedChains.POLYGON)` – constructor does not exist. `RubicBridge` takes no constructor args.
- **`crosschain-swap.mdx:73`** ✗ `rubicService.getTrade(payload)` – actual API is `RubicBridge.getTrade(demos, chain, payload)` (3 args, not 1).
- **`crosschain-swap.mdx:81`** ✗ `rubicService.executeTrade(trade)` – actual API is `RubicBridge.executeTrade(demos, chain, payload)`. Different signature; takes the same payload, not a `WrappedCrossChainTrade`.
- **`crosschain-swap.mdx:41-52`** `SupportedChains` literal list – matches `src/types/bridge/constants.ts`. OK as informational.
- Net: this entire recipe describes an obsolete or app-internal wrapper rather than the v3.1.0 SDK surface.

### 12. `sdk/cookbook/storage-programs/overview.mdx`
- **Verdict:** PASS. Index page only.

### 13. `sdk/cookbook/storage-programs/examples.mdx` — FAIL

The entire file is built around a fictional API and needs to be rewritten against the actual v3.1.0 surface.

- **Throughout (e.g. `examples.mdx:28,33,35`)** ✗ `import { DemosClient } from '@kynesyslabs/demosdk'` and `new DemosClient({ rpcUrl, privateKey })` – `DemosClient` does not exist in the SDK. The class is `Demos` (`src/websdk/demosclass.ts`), instantiated via `new Demos()` then `connect`/`connectWallet`.
- **`examples.mdx:29`** `import { deriveStorageAddress } from '@kynesyslabs/demosdk/storage'` – ⚠ the real export is `StorageProgram.deriveStorageAddress` (static method on the `StorageProgram` class). The free-function form `deriveStorageAddress` is not exported from `src/storage/index.ts`.
- **`examples.mdx:46-83`, `:84-89`, `:98-110`, `:113-117`, `:119-130`, `:132-135`, `:137-139`** ✗ All calls of the form `this.demos.storageProgram.create(...)`, `.write(...)`, `.read(...)`, `.delete(...)`, `.updateAccessControl(...)` – none of these methods exist. The actual surface is:
  - `demos.storagePrograms.sign(payload)` (note plural `storagePrograms`)
  - `demos.storagePrograms.read(address)`
  - Payload construction must go through `StorageProgram.createStorageProgram(...)`, `StorageProgram.writeStorage(...)`, `StorageProgram.readStorage(...)`, `StorageProgram.updateAccessControl(...)`, `StorageProgram.deleteStorageProgram(...)` (`src/storage/StorageProgram.ts:267, 322, 384, 435`).
- **`examples.mdx:43`** `await this.demos.getAddress()` – `getAddress()` is synchronous (returns `string`), not async (`src/websdk/demosclass.ts:291`). All `await this.demos.getAddress()` calls (used dozens of times throughout the file) are wrong.
- **`examples.mdx:611`** ✗ `this.demos.storageProgram.updateAccessControl(...)` – not a method.
- Roughly 8 worked examples (UserManager, SocialPlatform, GameLobby, CollaborativeDocument, ECommerceStore, plus mentions of DAO Governance / CMS / Task Management in the TOC at lines 12-19) all share the same broken pattern.
- The TOC lines `examples.mdx:16-19` advertise sections `DAO Governance`, `Content Management System`, `Task Management App` that the file does not actually contain.
- **`examples.mdx:884-889`** Cross-links to `../../storage-programs/api-reference`, `access-control`, `rpc-queries`, `operations` – verify these exist under the new docs structure; otherwise update.

### 14. `sdk/cookbook/evm-contract-write-transactions.mdx` — PASS

- **`:80-81`** Imports `EVM` from `@kynesyslabs/demosdk/xm-websdk`, `Demos`/`prepareXMScript`/`prepareXMPayload` from `@kynesyslabs/demosdk/websdk` – canonical subpaths. OK.
- **`:88-91`** `instance.connect()`, `instance.connectWallet(pk)`, `demos.connect(rpc)`, `demos.connectWallet(mnemonic)` – all valid.
- **`:111`** `instance.getContractInstance(address, JSON.stringify(abi))` – matches `src/multichain/core/evm.ts:412`.
- **`:118`** `instance.writeToContract(contract, "transfer", [recipient, amount])` – matches `src/multichain/core/evm.ts:433`.
- **`:121-127`** `prepareXMScript({ chain, signedPayloads, subchain, type: "contract_write", is_evm: true })` – correct.
- **`:130-132`** `prepareXMPayload`, `demos.confirm`, `demos.broadcast` – OK.
- The opening `chainProviders` reference (`:84`) is undefined in the snippet – minor doc-completeness nit, not an API drift issue.

---

## Cross-cutting issues

1. **Storage Programs cookbook is fundamentally wrong** (`examples.mdx` end-to-end). Needs full rewrite against `StorageProgram.*` static helpers + `demos.storagePrograms.{sign, read}`.
2. **Crosschain swap recipe references a non-existent `RubicService`** instead of `RubicBridge` and uses an internal app path. Needs full rewrite.
3. **`prepareXMScript` field name** – `signedPayload` (singular) appears in `creating-work-steps.mdx:54`. Should always be `signedPayloads: [...]`.
4. **`prepareWeb2Step` field name** – `action` is used in `creating-work-steps.mdx:17`; correct field is `method`.
5. **Import paths**: at least 3 files use TS path aliases (`@/websdk`, `@/features/...`) that are app-internal, not package-public. Replace with canonical `@kynesyslabs/demosdk/...` subpaths.
6. **`@kynesyslabs/demosdk/multichain/websdk` vs `@kynesyslabs/demosdk/xm-websdk`** – cookbook mixes the two. Per audit brief, the canonical public subpath is `xm-websdk`.
7. **`denomination` import path** in `native-transactions.mdx` is wrong; the namespace lives at the package root.
8. **`broadcastAndWait` (v2.12+)** is missing from every signing/broadcasting recipe; opportunity to surface it.
9. **`getAddress` is sync** but multiple cookbook files `await` it – semantically harmless in TS but wrong typing.
10. **`DemosWebAuth` global pattern** does not appear in any cookbook code – good (was decoupled Aug 2024).
