# nodecalls.mdx Audit (vs Demos class @ v3.1.0)

**Verdict: FAIL — CRITICAL GAPS** (~40-50% complete)

Source-of-truth: `/Users/tcsenpai/kynesys/sdks/src/websdk/demosclass.ts`.
Audited file: `/Users/tcsenpai/kynesys/documentation-mintlify/sdk/websdk/nodecalls.mdx`.

## Documented methods — status check

| Method | Doc Line | SDK Line | Status |
|--------|----------|----------|--------|
| `getLastBlockNumber()` | 13–20 | 1108 | ✓ matches |
| `getLastBlockHash()` | 22–29 | 1115 | ✓ matches |
| `getBlockByNumber(blockNumber)` | 31–39 | 1135 | ✓ matches |
| `getBlockByHash(blockHash)` | 41–49 | 1146 | ✓ matches |
| `getTxByHash(txHash)` | 55–63 | 1157 | ✓ matches |
| `getAllTxs()` | 65–72 | 1172 | ⚠ **DEPRECATED** in SDK; redirects to `getTransactions()` |
| `getAddressInfo(address)` | 78–86 | 1247 | ⚠ **SIGNATURE MISMATCH** — returns `AddressInfo \| null` with `balance: bigint` (P4); doc has no type annotation, no denomination guidance |
| `getPeerIdentity()` | 92–99 | 1226 | ✓ matches |
| `getPeerlist()` | 101–108 | 1211 | ✓ (note: title says "Get Peer List" but method is `getPeerlist`, casing) |
| `getMempool()` | 110–117 | 1219 | ✓ matches |

## Methods on the SDK but undocumented (gaps)

| Category | Method | SDK Line | Priority | Notes |
|----------|--------|----------|----------|-------|
| **Transactions** | `broadcastAndWait(validationData, opts)` | 471 | 🔴 HIGH | v2.12.0; deterministic tx confirmation |
| **Blocks** | `getBlocks(start, limit)` | 1123 | 🔴 HIGH | Multi-block query; accepts `start: number \| "latest"` |
| **Transactions** | `getTransactions(start, limit)` | 1201 | 🔴 HIGH | Replaces deprecated `getAllTxs()` |
| **Transactions** | `getTransactionHistory(address, type, options)` | 1186 | 🔴 HIGH | Per-address tx history, type filter, pagination |
| **Validators** | `getValidators(blockNumber?)` | 1307 | 🔴 HIGH | List active validators |
| **Validators** | `getValidatorInfo(address)` | 1296 | 🔴 HIGH | Single-validator query |
| **Validators** | `getStakedAmount(address)` | 1317 | 🔴 HIGH | Staked amount as bigint-string |
| **Governance** | `getNetworkParameters()` | 1328 | 🔴 HIGH | Active governance-driven params |
| **Governance** | `getNetworkInfo()` | 1397 | 🔴 HIGH | **v3.0** fork detection |
| **Governance** | `getActiveProposals()` | 1520 | 🔴 HIGH | Open proposals |
| **Governance** | `getProposalVotes(proposalId)` | 1530 | 🔴 HIGH | Live vote tally |
| **Governance** | `getUpgradeHistory()` | 1540 | 🔴 HIGH | Activated proposal history |
| **Storage** | `storagePrograms.read(address)` | 417 | 🔴 HIGH | Returns `StorageProgramResponse` |
| **Storage** | `storagePrograms.sign(payload)` | 397 | 🟡 MED | Sign storage program tx |
| **Web2** | `web2.getTweet(tweetUrl)` | 1652 | 🟡 MED | Tweet via RPC |
| **Web2** | `web2.getDiscordMessage(discordUrl)` | 1663 | 🟡 MED | Discord message via RPC |
| **Web2** | `web2.createDahr()` | 1649 | 🟡 MED | Digital Address History Record |
| **IPFS** | `ipfs.quote(fileSizeBytes, operation, durationBlocks)` | 1705 | 🟡 MED | Cost estimation; pre/post-fork aware |
| **TLSNotary** | `tlsnotary(config?)` | 1590 | 🟡 MED | HTTPS attestation; lazy init |
| **Utility** | `signMessage(message, options)` | 644 | 🔴 HIGH | Arbitrary message signing |
| **Utility** | `verifyMessage(message, sig, pubKey, options)` | 679 | 🔴 HIGH | Verify arbitrary message sig |
| **Address** | `getAddressNonce(address)` | 1271 | 🟡 MED | Nonce for tx construction |

## Top 3 most-impactful gaps

1. **`broadcastAndWait()`** — essential v2.12 feature, breaks tx-finality workflows.
2. **Governance suite** (5 methods) — Phase 1 governance is live; doc says nothing.
3. **Validator staking** (3 methods) — staking UI requires these; doc has zero coverage.

## Recommended actions

1. Add `broadcastAndWait` section (HIGH).
2. Add "Governance" section with 5 methods.
3. Add "Validators" section with 3 methods.
4. Add "Message Signing" section (signMessage/verifyMessage).
5. Update `getAddressInfo` example with bigint/denomination conversion (P4 awareness).
6. Mark `getAllTxs` as deprecated; route to `getTransactions`.
7. Expand existing sections to cover Web2/IPFS/TLSNotary nodecalls.
