# SDK Public API Surface (v3.1.0, May 2026)

Source: `/Users/tcsenpai/kynesys/sdks` @ release `v3.1.0` (commit `5c5cfca`).

## Version Timeline

| Version | Date | Highlights |
|---------|------|-----------|
| **v3.1.0** | 2026-05 | Serializer key-order fixes, fork-status cache per RPC URL, JSDoc/types polish |
| **v3.0.0-rc.1** | 2026-05-01 | P4 decimals: dual-format wire serializer, OS bigint, fork detection, sub-DEM guard |
| **v2.12.2** | ~2026-04 | Remove invalid `override` modifier on Error.cause |
| **v2.12.1** | ~2026-03 | Network parameter validation hardening; replace `Math.random()` with `crypto.getRandomValues` |
| **v2.12.0** | ~2026-02 | `broadcastAndWait`, Retry-After header, `MAX_BACKOFF_MS=8000` cap |
| **v2.11.x** | 2025-11..2026-01 | Network upgrade/staking, Ethos, Human Passport, L2PS messaging |

## Decimal / Sub-DEM Amounts

**Conversion:** `1 DEM = 10^9 OS` via constants `OS_DECIMALS = 9`, `OS_PER_DEM = 10n^9n`.

**Denomination module:** `src/denomination/index.ts` exports:
- `demToOs(dem: number | string): bigint`
- `osToDem(os: bigint): string`
- `parseOsString(osString: string): bigint`
- `toOsString(os: bigint): string`
- `formatDem(os: bigint): string` — display with "DEM" suffix
- Constants: `OS_DECIMALS`, `OS_PER_DEM`, `MIN_AMOUNT_OS`, `ZERO_OS`
- `serializeTransactionContent(content, isPostFork)` — internal dual-format wire serializer

**Public API impact:**
- `Demos.transfer(to: string, amount: number | bigint): Promise<Transaction>` — `bigint` (OS) preferred; `number` (DEM) still accepted
- `Demos.pay(to: string, amount: number | bigint): Promise<Transaction>` — same
- `Demos.getAddressInfo(address).balance` — returns `bigint` (OS) on post-fork nodes

**Wire format:** `TransactionContent.amount`, `TxFee.*`, `RawTransaction.{amount,networkFee,rpcFee,additionalFee}`, `StatusNative.balance` widened to `number | string`. SDK normalizes internally to `bigint`.

**Fork-aware routing:**
- `Demos.getNetworkInfo(): Promise<NetworkInfo | null>` — calls node RPC, caches per RPC URL with TTL recovery on transient failure (`_NETWORK_INFO_FAILURE_TTL_MS = 30_000`)
- Returns `{ forks: { osDenomination: { activationHeight, activated, currentHeight } } }` or `null` on persistent failure (assumes pre-fork)
- `Demos.sign()` calls `serializeTransactionContent()` with the cached `isPostFork` flag

**Errors:**
- `SubDemPrecisionError` — thrown by `transfer()`/`pay()` if caller passes sub-DEM precision (e.g., `1_234_567n` OS) against a pre-fork node. Public properties: `subDemRemainderOs`, `amountOs`. Fails fast before signing.

**Bug fix in v3.0.0-rc.1:** `STORAGE_PROGRAM_CONSTANTS.FEE_PER_CHUNK` corrected from `1n` to `OS_PER_DEM` (1 DEM), fixing pre-existing 10^9× under-billing.

---

## Demos Class — Core Methods (`@kynesyslabs/demosdk/websdk`)

`src/websdk/demosclass.ts` (1766 LOC).

**Connection / wallet:**
- `connect(rpc_url: string)`
- `connectWallet(mnemonic: string, ...)`
- `getAddress(): string`
- `getEd25519Address(): string`
- `keypair` (getter)
- `walletConnected` (getter)
- `newMnemonic(strength?: 128 | 256): string`

**Transactions:**
- `transfer(to, amount: number | bigint): Promise<Transaction>`
- `pay(to, amount: number | bigint): Promise<Transaction>`
- `sign(raw_tx: Transaction): Promise<Transaction>`
- `confirm(tx: Transaction): Promise<RPCResponseWithValidityData>`
- `broadcast(validationData): Promise<RPCResponse>`
- `broadcastAndWait(validationData, opts?: { timeoutMs?: number, pollIntervalMs?: number, failFastOnBroadcastError?: boolean }): Promise<{broadcast, status}>` — **NEW v2.12.0**
- `signMessage(address, options): Promise<{data, publicKey}>`
- `verifyMessage(message, publicKey, signature): Promise<boolean>`

**RPC queries:**
- `getLastBlockNumber()`, `getLastBlockHash()`
- `getBlocks(start, end)`, `getBlockByNumber(n)`, `getBlockByHash(h)`
- `getTxByHash(h)`, `getTransactionHistory(address, ...)`
- `getAddressInfo(address)`, `getAddressNonce(address)`
- `getMempool()`, `getPeerlist()`, `getPeerIdentity()`

**Validator / network:**
- `getValidatorInfo(address)`, `getValidators(blockNumber?)`, `getStakedAmount(address)`
- `getNetworkParameters()` — cached per RPC, 30-second TTL
- `getNetworkInfo()` — fork detection, **NEW v3.0.0-rc.1**
- `getActiveProposals()`, `getProposalVotes(proposalId)`, `getUpgradeHistory()`

**Storage programs:**
- `storagePrograms.sign(payload): Promise<Transaction>`
- `storagePrograms.read(address): Promise<StorageProgramResponse>`

**TLSNotary:**
- `tlsnotary(config?: TLSNotaryConfig): Promise<TLSNotary>` — lazy init

**Low-level:**
- `rpcCall(method, params, ...)`, `call(message, args, ...)`, `nodeCall(message, args, ...)`

---

## Validator Staking & Governance API

`DemosTransactions` (used internally by `Demos`).

**Staking:**
- `stake(amount: string, connectionUrl: string, demos: Demos): Promise<Transaction>`
  - `amount` is bigint-encoded string in OS
  - `connectionUrl` required on first stake; optional for top-ups
- `unstake(demos: Demos): Promise<Transaction>`
- `claimUnstakedAmount(demos: Demos): Promise<Transaction>` (per source ~line 662) — claim after cooldown
- `validatorExit(demos: Demos): Promise<Transaction>` (alternate: per docs)

**Governance:**
- `proposeNetworkUpgrade(params: { proposalId: string, proposedParameters: Partial<NetworkParameters>, rationale: string, effectiveAtBlock: number }, demos: Demos): Promise<Transaction>`
  - `rationale` ≤ 1024 bytes
  - `effectiveAtBlock` ≥ `tallyBlock + gracePeriodBlocks`
  - Only active validators may propose
- `voteOnUpgrade(proposalId: string, approve: boolean, demos: Demos): Promise<Transaction>`
  - One vote per proposal, final, must be in snapshot

**Proposal lifecycle:** `pending` → `approved` | `rejected` → `activating` → `active`.

---

## Identities Class (`@kynesyslabs/demosdk/abstraction`)

`src/abstraction/Identities.ts`.

### Web2 (social proof)

- `addGithubIdentity(username, userId?, proof?, demos: Demos)`
- `addTwitterIdentity(handle, proof, demos)`
- `addDiscordIdentity(username, proof, demos)`
- `addTelegramIdentity(username, proof: TelegramSignedAttestation, demos)`
- `addWeb2IdentityViaTLSN(context, url, proof, demos)` — generic
- `removeWeb2Identity(context, demos)`
- `removeWeb2IdentityViaTLSN(context, demos)`

### Cross-chain (XM)

- `inferXmIdentity(payload: XMCoreTargetIdentityPayload, demos)`
- `removeXmIdentity(payload, demos)`

### Post-Quantum (PQC)

- `bindPqcIdentity(algorithm: PQCAlgorithm, publicKey, signature, demos)`
- `removePqcIdentity(algorithm, demos)`

### Unstoppable Domains (UD)

- `addUnstoppableDomainIdentity(domain, paymentMethod?, demos)`
- `removeUnstoppableDomainIdentity(domain, demos)`
- `resolveUDDomain(domain): Promise<UnifiedDomainResolution>`
- `getUDIdentities(demos?, address?): Promise<UDIdentityPayload[]>`

### Nomis

- `getNomisScore(wallet): Promise<NomisWalletIdentity>`
- `addNomisIdentity(nomisData: NomisWalletIdentity, demos)`
- `removeNomisIdentity(wallet, demos)`

### Human Passport

- `getHumanPassportScore(address): Promise<HumanPassportScore>`
- `getHumanPassportIdentities(demos?, address?): Promise<SavedHumanPassportIdentity[]>`
- `addHumanPassportIdentity(data: HumanPassportIdentityData, demos)`
- `removeHumanPassportIdentity(demos)`

### Ethos

- `getEthosScore(address): Promise<EthosWalletIdentity>`
- `getEthosIdentities(demos?, address?): Promise<EthosWalletIdentity[]>`
- `addEthosIdentity(data: EthosWalletIdentity, demos)`
- `removeEthosIdentity(payload: EthosIdentityRemoveData, demos)`

### Query / reverse lookup

- `getIdentities(address?, context?: IdentityContext)`
- `getXmIdentities(demos, address?)`, `getWeb2Identities(demos, address?)`
- `getDemosIdsByIdentity(identity)`
- `getDemosIdsByWeb2Identity(context, proof)`
- `getDemosIdsByTwitter(handle)`, `getDemosIdsByGithub(username, userId?)`, `getDemosIdsByDiscord(username)`, `getDemosIdsByTelegram(username)`

### Referrals & points

- `getUserPoints(address): Promise<number>`
- `validateReferralCode(demos, referralCode)`
- `getReferralInfo(demos?, address?): Promise<ReferralInfo>`

---

## L2PS (`@kynesyslabs/demosdk/l2ps`)

**Factory & registry:**
- `L2PS.create(privateKey?: string, iv?: string): Promise<L2PS>`
- `L2PS.getInstance(id: string): L2PS | undefined`
- `L2PS.getInstances(): L2PS[]`
- `L2PS.hasInstance(id: string): boolean`
- `L2PS.removeInstance(id: string): boolean`

**Instance methods:**
- `encryptTx(tx: Transaction, senderIdentity?: any): Promise<L2PSTransaction>` — wraps tx in `L2PSEncryptedPayload` `{ l2ps_uid, encrypted_data, tag, original_hash }`; tx type becomes `l2psEncryptedTx`
- `decryptTx(encryptedTx: L2PSTransaction): Promise<Transaction>`

**Config interface:**
```typescript
interface L2PSConfig {
  uid: string
  config: { created_at_block: number, known_rpcs: string[] }
}
```

---

## DemosWork (`@kynesyslabs/demosdk/demoswork`)

`src/demoswork/index.ts`.

**Class:** `DemosWork`
- `push(operation: DemosWorkOperation)`
- `validate(script: DemoScript)`
- `fromJSON(script: DemoScript)`
- `toJSON()`

**Operations / steps:**
- `DemosWorkOperation` (interface)
- `BaseOperation` (abstract)
- `ConditionalOperation`
- `Condition`
- `WorkStep`
- `NativeWorkStep`, `Web2WorkStep`, `XmWorkStep`

**Helpers:**
- `prepareDemosWorkPayload(work: DemosWork, demos: Demos): Promise<Transaction>`
- `runSanityChecks(script: DemoScript)`

---

## Cross-Chain (XM SDK) — Verified Chains

`src/multichain/core/index.ts` exports:

```typescript
export { EVM } from './evm'
export { IBC } from './ibc'
export { MULTIVERSX } from './multiversx'
export { SOLANA } from './solana'
export { TON } from './ton'
export { BTC } from './btc'
export { APTOS } from './aptos'
export { TRON } from './tron'
export { XRPL, xrplGetLastSequence } from './xrp'
export { NEAR } from './near'
```

**Confirmed chains: EVM, IBC, MultiversX, Solana, TON, BTC, Aptos, TRON, XRPL, NEAR.**

**TEN.xyz:** documented in docs but NOT explicitly in the multichain barrel above — needs verification (may be EVM-compatible and routed through EVM).

**Sui:** **NOT exported** from `src/multichain/core/index.ts`. Docs claim Sui — needs removal.

**Entry points:** `@kynesyslabs/demosdk/xmcore` (barrel), `@kynesyslabs/demosdk/xmcore/{evm,solana,btc,ibc,...}` (per-chain).

---

## RPC / Transport Layer

**Transport (`_doPost`) features:**
- HTTP `Retry-After` header support (seconds or HTTP-date)
- `MAX_BACKOFF_MS = 8000` cap
- 5xx automatic retry
- Accurate attempt counting

**broadcastAndWait (v2.12.0):**
- Polls `getTransactionStatus` until terminal state
- `failFastOnBroadcastError: true` opt-in for unreachable nodes
- Defaults: `timeoutMs = 30_000`, `pollIntervalMs = 500`
- Throws `BroadcastTimeoutError` on timeout, `BroadcastFailedError` on fail-fast

**rpcCall vs _doPost:**
- `_doPost` retries network-level failures (5xx, no contact)
- `rpcCall` retries RPC-level errors (separate `retries` parameter)
- Avoids multiplying retry attempts across layers

---

## Error Types (new in v2.12+, v3.0+)

- `TransportError` — network-level failure
- `BroadcastTimeoutError` — tx inclusion timeout
- `BroadcastFailedError` — fail-fast on unreachable node (opt-in)
- `SubDemPrecisionError` — sub-DEM amount on pre-fork node (v3.0.0-rc.1)

---

## Storage Programs SDK

`src/storage/StorageProgram.ts`.

- Deterministic address derivation: `stor-{sha256(ownerAddress + programName)}`
- Dual encoding: JSON key-value or binary blob
- ACL: owner, allowed, blacklisted, public, groups

**Operations (write):** `SET_FIELD`, `SET_ITEM`, `APPEND_ITEM`, `DELETE_FIELD`, `DELETE_ITEM`.

**Read methods (via Demos.storagePrograms):** `read(address)`, `getFields()`, `getValue(field)`, `getItem(field, index)`, `hasField(field)`, `getFieldType(field)`.

---

## Notable Singletons / Caching

- `Demos.instance` — static singleton (instances can also be created freely)
- `_cryptoInstanceId` — UUID per instance, fixes multi-instance identity bleed (v2.11.3+)
- `_cachedNetworkInfoRpcUrl` — per-RPC caching key (v3.0.0-rc.1, myc#18)
- `_NETWORK_INFO_FAILURE_TTL_MS = 30_000` — transient outage recovery
