# Node Features Audit (Oct 2025 → May 2026)

Source: /Users/tcsenpai/kynesys/node @ HEAD `af3b7a4b` (P4 / decimals branch tip).

## 1. osDenomination Fork (Sub-DEM amounts)

**What it does:** Atomic, idempotent state migration multiplying all balances/stakes by 10^9 to transition from DEM to OS denomination at a configured fork height.

**Conversion:** `1 DEM = 10^9 OS`.

**Key files:**
- `src/forks/migrations/osDenomination.ts` — migration logic
- `src/forks/forkConfig.ts` — registry, `ForkName` type, `DEFAULT_FORK_CONFIG`
- `src/forks/forkGates.ts` — `isForkActive(forkName, blockHeight)`
- `src/forks/serializerGate.ts` — serializer wrappers (gates wire format)
- `src/model/entities/ForkState.ts` — persistent ledger
- `src/migrations/CreateForkStateTable.ts` — TypeORM migration
- `src/libs/network/handlers/forkHandlers.ts` — `getNetworkInfo` RPC

**Three data sources migrated:**
1. GCRv2 `gcr_main.balance` (bigint column) — bulk UPDATE × 10^9
2. Legacy GCR `details.content.balance` (JSONB/JS number) — row-by-row with CAP at `Math.floor(Number.MAX_SAFE_INTEGER * 0.9)` = 8,106,479,329,266,892 OS
3. Validators `staked_amount` (text/bigint-as-string) — bulk UPDATE × 10^9

**CAP policy:** Accounts exceeding legacy precision are capped; lost OS recorded in `fork_state.total_value_lost_os`.

**Sum invariant:** `postSumOs == preSumDem * 10^9 - totalValueLostOs` validated before commit.

**RPC:** `getNetworkInfo()` returns:
```json
{ "forks": { "osDenomination": { "activationHeight": <number|null>, "activated": <bool>, "currentHeight": <number> } } }
```

**Status:** Currently inactive (`activationHeight: null` in default registry). Activation is per-genesis.

**Commits:** `c6b91c57` (entity), `16732481` (migration), `f21e59e7` (hook), `e142bcce` (RPC), `af3b7a4b` (integration tests).

---

## 2. Network Governance / Network Upgrade

**What it does:** On-chain governance allowing validators to propose, vote on, and apply network parameter changes.

**Governable parameters:**
- `minValidatorStake` — minimum stake for validators
- `networkFee` — per-tx network fee
- `rpcFee` — per-tx RPC fee
- `featureFlags` — binary feature toggles
- (Phase 2 planned: `blockTimeMs`, `shardSize`)

**Lifecycle:** propose (snapshot taken) → voting window (100 blocks) → tally + grace period (50 blocks) → activation at `effectiveAtBlock`.

**Tally:** 2/3 supermajority ceiling-rounded of snapshot weight; per-proposal change cap of 50% (configurable safety bound).

**Key files:**
- `src/features/networkUpgrade/types.ts`
- `src/features/networkUpgrade/constants.ts`
- `src/features/networkUpgrade/governanceWeight.ts` — `computeSnapshotWeight()`
- `src/features/networkUpgrade/safetyBounds.ts` — `withinPercentCap`, etc.
- `src/model/entities/NetworkUpgrade.ts`
- `src/model/entities/NetworkUpgradeVote.ts`
- `src/libs/blockchain/routines/tallyUpgradeVotes.ts`
- `src/libs/blockchain/routines/applyNetworkUpgrade.ts`
- `src/libs/blockchain/routines/loadNetworkParameters.ts`
- `src/libs/network/handlers/governanceHandlers.ts`
- `src/libs/network/routines/transactions/handleGovernanceTx.ts`

**RPC endpoints:**
- `getNetworkParameters()` — current active parameters
- `getActiveProposals()` — pending/approved/activating
- `getProposalVotes(proposalId)` — live tally
- `getUpgradeHistory()` — historical record

**Tx types:** `proposeNetworkUpgrade`, `networkUpgradeVote`.

**Commits:** PR #778 (`9d28097d`).

---

## 3. Validator Staking & Lifecycle

**State machine:** ACTIVE → UNSTAKING → EXITED. Driven by reflexive tx types: `validatorStake`, `validatorUnstake`, `validatorExit`.

**Two-phase unstake:**
- `validatorUnstake()` sets `unstake_requested_at` block
- After 1000-block lockup (non-governable), `validatorExit()` finalizes
- Restaking via `validatorStake()` clears unstake timestamps

**Validator entity fields:**
- `status` — text status
- `staked_amount` — bigint-as-string (× 10^9 in OS post-fork)
- `unstake_requested_at` — block (null when staking)
- `unstake_available_at` — block (indexed)

**Fees (per-tx flat fee):**
- `networkFee = 1`
- `rpcFee = 1`
- `burnFee = 1`
- Total = 3 units per tx
- TODO in source: post-OS-merge, sum should equal 1 DEM (≈ 333_333_333 OS each)

**Validator addresses:** canonicalized to lowercase end-to-end.

**RPC:** `getValidators()`, `getValidator(address)`.

**Files:**
- `src/model/entities/Validators.ts`
- `src/features/staking/constants.ts`
- `src/features/staking/types.ts`
- `src/libs/blockchain/gcr/gcr_routines/GCRValidatorStakeRoutines.ts`
- `src/libs/blockchain/routines/validatorsManagement.ts`
- `src/libs/network/handlers/validatorHandlers.ts`
- `src/libs/network/routines/transactions/handleStakingTx.ts`

**Commits:** `da3eb5f2`, `abbf3b24`, `447afd9e`, `c029f4c5`, `32ec1945`.

---

## 4. Storage Programs (storage_v2)

**What it does:** Mutable, owner-controlled, ACL-gated key-value containers stored in the Global Change Registry. Field-level read/write/invoke permissions.

**Specs:** `/Users/tcsenpai/kynesys/node/specs/storageprogram/01-overview.mdx` … `06-examples.mdx`.

**RPC endpoints (9):**
1. `getStorageProgram(programId)` — full metadata + data
2. `getStorageProgramAll()` — all programs (paginated)
3. `getStorageProgramFields(programId)` — field defs
4. `getStorageProgramFieldType(programId, fieldName)`
5. `getStorageProgramItem(programId, itemId)`
6. `getStorageProgramValue(programId, fieldPath)`
7. `getStorageProgramsByOwner(ownerAddress)` — pagination pushed into SQL
8. `hasStorageProgramField(programId, fieldName)`
9. `searchStoragePrograms(query)` — text search with ACL filter

**ACL:** owner-defined groups with read/write/invoke; predicates for conditional access; anonymous limited to public; ACL applied at SQL layer.

**Files:**
- `src/features/storageprogram/index.ts`
- `src/features/storageprogram/routes.ts`
- `src/libs/network/handlers/storageProgramHandlers.ts`
- `src/libs/network/routines/nodecalls/getStorageProgram*.ts` (multiple)
- `src/libs/network/routines/nodecalls/storageProgramShared.ts` — `checkReadPermission`, `withFieldRead`, `readReachablePredicate`

**Status:** Production. Health endpoint `/health` added; rate-limit headers; identity-header parsing consolidated.

**Commits:** PR #681 (`50ab5469`), PR #797 (`6bf37074`), `e28dd8c8`, `f9c1b2b2`.

---

## 5. Identity Systems (Multi-Provider)

**Providers integrated (8):**
1. **Nomis** — wallet score (types: `NomisWalletScorePayload`, `NomisScoreRequestOptions`, `NomisApiResponse`)
2. **Ethos** — profile + score (`EthosScorePayload`, `EthosProfileResponse`)
3. **Human Passport** — humanity proof with expiration (`HumanPassportVerification`, `RawScoreResponse`, `CachedScore`; `passingScore`, `threshold`, `stamps`, `expirationTimestamp`)
4. **TLSNotary** — verified web proofs (`applyTLSNIdentityAdd`, `applyTLSNIdentityRemove`)
5. **UD (Unstoppable Domains)** — domain-based identity
6. **Web2** — Discord, Twitter/X (OAuth/proof verification)
7. **XM (Cross-Chain)** — multi-chain wallet addresses (Ethereum, Solana, Aptos, etc.)
8. **ZK** — zero-knowledge commitments + attestations (Groth16 proofs, Poseidon hashes, Merkle tree state)

**Tools:** `src/libs/identity/tools/{nomis,ethos,humanpassport,twitter,discord,crosschain}.ts`
**Providers:** `src/libs/identity/providers/{nomisIdentityProvider,ethosIdentityProvider}.ts`
**GCR routines:** `src/libs/blockchain/gcr/gcr_routines/routines/{nomis,ethos,humanpassport,tlsn,ud,web2,xm,zk,pqc}Routines.ts`
**Tx handler:** `src/libs/network/routines/transactions/handleIdentityRequest.ts`
**RPC handlers:** `src/libs/network/handlers/identityHandlers.ts`

**ZK Identity:** Two-phase flow:
1. Link: client generates secret → commitment `Poseidon(provider_id, secret)` → `identity_commitment` tx → validators add to Merkle tree
2. Prove: ZK proof using secret + Merkle path → nullifier `Poseidon(provider_id, context)` → `identity_attestation` tx → uniqueness check

DB: `identity_commitments`, `used_nullifiers`, `merkle_tree_state`.

---

## 6. Fork System (general machinery)

**Pluggable hard-fork infrastructure** for atomic state migrations at block heights. P2 introduced registry/gates without rule changes; P3b activates osDenomination.

**Key files:**
- `src/forks/forkConfig.ts` — `ForkConfig`, `DEFAULT_FORK_CONFIG`, `ForkName` union
- `src/forks/loadForkConfig.ts` — load from genesis.json into SharedState
- `src/forks/forkGates.ts` — `isForkActive`
- `src/forks/serializerGate.ts` — conditional serialization wrappers
- `src/forks/migrations/` — migration functions

**Integration:**
- `src/libs/blockchain/chainBlocks.ts` — block-insert hook checks fork state
- Tx hashing routes through serializer gate
- Block hashing routes through serializer gate

---

## 7. Other Major Changes

**Docker compose unification:** `/devnet/docker-compose.yml` with profiles for tlsnotary, monitoring; multi-arch builds (linux/amd64, linux/arm64). PR #782.

**Health & monitoring:** `/health` endpoint; rate-limit headers; Prometheus metrics; Grafana dashboards.

**Fee column widening:** `WidenFeeColumnsToBigint.ts` widens `transactions.networkFee`/`rpcFee` to `bigint`.

**P2P hardening:** Loopback warning, socket-close handling, peer fallback in `verifyLastBlockIntegrity`, mempool filtering.

---

## Summary Table

| Area | Status | RPC | User-Visible |
|------|--------|-----|--------------|
| osDenomination fork | P3b ready (inactive by default) | `getNetworkInfo` | 1 DEM = 10^9 OS post-fork |
| Governance | Live | `getNetworkParameters`, `getActiveProposals`, `getProposalVotes`, `getUpgradeHistory` | 2/3 supermajority voting |
| Staking | Live | `getValidators`, `getValidator` | Two-phase unstake; flat 3-unit fee |
| Storage v2 | Live | 9 endpoints | ACL-controlled mutable storage |
| Identity (8 providers) | Live | `getIdentity`, `applyIdentity` | Multi-provider verification |
| Fork machinery | Live (extensible) | shared with osDenomination | Hard-fork infrastructure |
| Docker stack | Live | `/health` | One-command bring-up |
