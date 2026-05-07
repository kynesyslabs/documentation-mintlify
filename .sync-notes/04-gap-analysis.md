# Gap Analysis & Task Map

This file maps source-of-truth → docs gaps to mycelium tasks. Use it to recover state after compaction.

## Mycelium Mapping

Run `myc task list` for current state. Epic #1 = entire sync.

| Mycelium Task | Title | Priority | Files Touched | Verify By |
|---------------|-------|----------|---------------|-----------|
| #1 | Audit research: capture findings | high | `.sync-notes/00-04*` | Files exist + non-empty |
| #2 | Verify Sui chain claim | high | `sdk/cross-chain/overview.mdx`, `sdk/changelog.mdx`, `docs.json` | grep -i sui returns no false claim |
| #3 | Document osDenomination / sub-DEM | high | `sdk/websdk/transactions/*`, `sdk/websdk/overview.mdx` | mentions OS, DEM, BigInt, getNetworkInfo |
| #4 | broadcastAndWait + error types | medium | `sdk/websdk/transactions/broadcasting-a-transaction.mdx` | method documented + 4 errors named |
| #5 | Nomis/Ethos/HumanPassport/UD identity | high | `sdk/web2/identities/{nomis,ethos,human-passport,unstoppable-domains}.mdx` + nav | 4 new pages registered in docs.json |
| #6 | Un-flag Storage Programs | medium | `backend/storage-programs/overview.mdx`, `sdk/storage-programs/overview.mdx` | "preview/in development" warnings removed |
| #7 | Un-flag DemosWork | medium | `sdk/websdk/transactions/creating-a-transaction.mdx` | "not available at the moment" replaced |
| #8 | Refresh changelog | medium | `sdk/changelog.mdx` | mentions v3.1.0 and v2.11+ entries |
| #9 | L2PS SDK methods | low | new `sdk/websdk/l2ps/*.mdx` or section | encryptTx/decryptTx documented |
| #10 | Fork system + getNetworkInfo backend | medium | `backend/internal-mechanisms/*` or new `backend/forks/*` | getNetworkInfo + osDenomination explained |
| #11 | Cookbook code-sample audit | low | reports back, no docs change required initially | spreadsheet/list of pass/fail |

## Out-of-scope (NOT touching this round)

- Storage Programs deep API docs — only un-flag the warning. Full API refresh would be a separate epic.
- DemosWork detailed cookbook update — only un-flag. Full refresh = separate epic.
- Generic post-quantum crypto SDK section — exists; no known gaps to verify in scope.
- Backend governance + validator-lifecycle — already up to date (commit `2915fb44`).
- TLSNotary docs — already up to date (commit Jan 2026 + recent iframe-integration commit).

## Recovery Procedure (after compaction or rate-limit)

1. Verify TEAM_MODE marker in `AGENTS.md` (should be present).
2. Read `TEAM.md` to refresh operating model.
3. Read `.sync-notes/00-README.md` then `01-04` in order.
4. `myc task list` — see what's open.
5. `myc task show <id>` for the next pending high-priority task.
6. Pick lowest-ID open high-priority task; mark `in_progress`.
7. Source-of-truth refs: `/Users/tcsenpai/kynesys/node` (HEAD `af3b7a4b`), `/Users/tcsenpai/kynesys/sdks` (v3.1.0 = `5c5cfca`).

## Risk / Blast Radius

- **High blast radius (Tech Lead does directly):** Task #2 (Sui claim), #3 (osDenomination — touches transaction docs broadly).
- **Medium (Senior delegation OK):** Tasks #4, #5, #6, #7, #8, #10.
- **Low (Junior or batch):** Task #9 (small new file), #11 (audit only).

Tasks #5, #6, #7, #8 are independent — can run in parallel via Senior agent dispatch when ready.
