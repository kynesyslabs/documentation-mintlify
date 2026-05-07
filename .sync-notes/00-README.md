# Documentation Sync Notes

This directory captures the audit findings used to drive a sync of `documentation-mintlify` against the source-of-truth repos:

- `/Users/tcsenpai/kynesys/node` — node implementation (current HEAD: `af3b7a4b`, branch `decimals` or similar P4 area)
- `/Users/tcsenpai/kynesys/sdks` — SDK packages (current release: **v3.1.0**, commit `5c5cfca`)

Audit run date: 2026-05-07.

## Files

- `01-node-features.md` — features added to the node since ~Oct 2025 (governance, staking, fork system, storage v2, identity providers, decimals)
- `02-sdks-api-surface.md` — SDK public API surface as of v3.1.0 (Demos class, Identities class, L2PS, DemosWork, cross-chain chains, broadcast helpers, error types)
- `03-docs-current-state.md` — content-level inventory of `documentation-mintlify/` with last-touched dates and stale-area flags
- `04-gap-analysis.md` — synthesis: what features exist in code but are missing/stale in docs, mapped to mycelium task IDs

## Working method

Tasks tracked in mycelium (`myc task list`). Epic #1 covers the whole sync. Work progresses in priority order (high → medium → low). Each task closes with verifiable artifacts.

If returning to this work after compaction or rate-limit pause:

1. Read `AGENTS.md` (TEAM_MODE marker still present means Tech Lead mode is on)
2. Read `TEAM.md` for the operating model
3. Read this directory in order (00 → 04)
4. Run `myc task list` to see what's left
5. Resume from the next open task in priority order
