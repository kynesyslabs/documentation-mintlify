# Documentation Current State (May 2026)

Source: `/Users/tcsenpai/kynesys/documentation-mintlify` @ HEAD `2915fb44`.

## Section Inventory

| Section | Last Touched | Status | Notes |
|---------|--------------|--------|-------|
| Validator Lifecycle | 2026-05-06 | ✓ Current | Complete state machine; commit `2915fb44` |
| Network Governance | 2026-05-06 | ✓ Current | Stackable Genesis system documented |
| Validator Staking SDK | 2026-05-06 | ✓ Current | bigint string amounts |
| Governance SDK | 2026-05-06 | ✓ Current | propose/vote/query methods |
| WebSDK Overview | 2025-12-24 | △ No osDenomination mention | High-level only |
| Transactions (sdk/websdk/transactions/) | 2025-12-08 | △ No osDenomination/BigInt | DemosWork flagged unavailable |
| Cross-Chain | 2025-12-08 | △ Lists Sui (not in SDK) | 11 chains listed |
| Storage Programs | 2025-12-24 | ⚠ "preview / still in development" | Code is shipped |
| ZK Identity | 2026-01-29 | ✓ Current | Backend doc; SDK side missing providers |
| TLSNotary | 2026-01-12 | ✓ Current | Clear |
| L2PS | 2026-03-04 | △ SDK side unclear | Architecture doc only |
| MCP Server | 2025-12-24 | ✓ Current | |
| Changelog | 2025-12-24 | ⚠ Stale (Dec 2024) | Missing all 2025–2026 releases |

## Identity Providers Documented vs in SDK

| Provider | In SDK? | In Docs? | Gap |
|----------|---------|----------|-----|
| Twitter | ✓ | ✓ `sdk/web2/identities/twitter.mdx` | None |
| GitHub | ✓ | ✓ `sdk/web2/identities/github.mdx` | None |
| Discord | ✓ | ✓ `sdk/web2/identities/discord.mdx` | None |
| Telegram | ✓ | ✓ `sdk/web2/identities/telegram.mdx` | None |
| Nomis | ✓ | ❌ | **MISSING** |
| Ethos | ✓ | ❌ | **MISSING** |
| Human Passport | ✓ | ❌ | **MISSING** |
| Unstoppable Domains (UD) | ✓ | △ (in cross-chain only) | Identity-side not documented |
| TLSNotary (generic Web2) | ✓ | △ (TLSNotary section, not identity) | Cross-link missing |
| XM cross-chain | ✓ | △ (covered via xmscript) | OK |
| PQC (Post-Quantum) | ✓ | △ (advanced section) | bind/remove methods not documented |
| ZK Commitments | ✓ | ✓ `backend/zk-identity/*` | Backend yes, SDK side missing |

## Cross-Chain Claims vs SDK Reality

Docs `sdk/cross-chain/overview.mdx` lists 11 chains. SDK `src/multichain/core/index.ts` exports 10:

| Chain | Docs | SDK | Status |
|-------|------|-----|--------|
| EVM | ✓ | ✓ | OK |
| MultiversX (EGLD) | ✓ | ✓ | OK |
| Solana | ✓ | ✓ | OK |
| IBC | ✓ | ✓ | OK |
| BTC | ✓ | ✓ | OK |
| TEN.xyz | ✓ | △ (likely EVM-routed) | Verify |
| TON | ✓ | ✓ | OK |
| XRPL | ✓ | ✓ | OK |
| NEAR | ✓ | ✓ | OK |
| Sui | ✓ | **❌** | Remove from docs |
| Aptos | ✓ | ✓ | OK |
| TRON | ❌ | ✓ | **Add to docs** |

## Stale Warnings to Address

1. **Storage Programs:** "Storage Programs are still being developed; this documentation is a preview and might not work correctly." (sdk and backend overview)
2. **DemosWork:** "The execution of a demoscript is not available at the moment as the Demoswork spec is still in development." (sdk/websdk/transactions/creating-a-transaction.mdx)
3. **Changelog:** Last entry December 2024; no SemVer; missing v2.11.x, v2.12.x, v3.0.x, v3.1.0.

## Missing Topics / Pages

- No mention of `getNetworkInfo()` RPC anywhere in docs
- No mention of `osDenomination`, `OS_PER_DEM`, sub-DEM precision in docs
- No mention of `broadcastAndWait()` or new error types (`SubDemPrecisionError`, `BroadcastTimeoutError`, `BroadcastFailedError`, `TransportError`)
- No L2PS SDK usage docs (architecture is documented; client API is not)
- No fork-system general documentation (only osDenomination would be referenced once we add it)
