# Setup Documentation Audit — vs. Current Node State (2026-05-09)

## Executive Summary

The setup docs are **significantly outdated** post-PR #782 (dockerization). All macOS/Ubuntu/Windows guides reference the legacy **`./install` + `./run`** pattern and manual env setup. Current canonical path is **`docker compose up`**. Critical gaps: no docker-compose steps, misleading fee structure (RPC_FEE=5 → RPC_FEE=1 + NETWORK_FEE=1 + BURN_FEE=1 = 3 total per commit 447afd9e), and missing TLSNotary profile gating.

## Per-File Audit

### 1. overview.mdx
**Status:** PASS (placeholder only)
- File contains only title/description metadata; no substantive content to audit.

### 2. run-the-project-macos.mdx
**Status:** FAIL — Entire workflow obsolete
- **Lines 7–33:** References manual Node.js version check, Docker Engine requirement, then `.env` creation with `CONSENSUS_TIME=10`, `RPC_FEE=5`, `SERVER_PORT=53550`, `EXPOSED_URL=http://127.0.0.1:53550` hardcoded values.
  - ❌ No mention of `NETWORK_FEE=1`, `BURN_FEE=1` (added in PR #778 / commit 447afd9e, May 6, 2026)
  - ❌ `SERVER_PORT` is legacy; current is `RPC_PORT=53550`
  - ❌ `EXPOSED_URL=http://127.0.0.1:53550` (hardcoded localhost) — should warn it's only for dev
- **Lines 21–25:** References "Make the following changes to the **install file**" — directs user to manually edit `/install` script (comment apt-get lines, change .bashrc→.zshrc).
  - ❌ This is the **bare-metal `./run` path**, not the current recommended docker-compose path.
- **Lines 33:** References running `./run` directly
  - ❌ No docker-compose mentioned.

### 3. run-the-project-ubuntu.mdx
**Status:** FAIL — Entire workflow obsolete
- **Lines 7–23:** Identical obsolete setup as macOS (manual Node.js, Docker Desktop, hardcoded `.env` with `RPC_FEE=5`, `SERVER_PORT`).
  - ❌ Same fee structure gap (missing `NETWORK_FEE=1`, `BURN_FEE=1`)
  - ❌ `SERVER_PORT` is wrong
  - ❌ No docker-compose
- **Line 22:** References `./install` script again.

### 4. run-the-project-windows/overview.mdx
**Status:** FAIL — Partially updated but still wrong
- **Lines 9–10:** Prerequisites list `Node version: 20.15.1` and Docker Desktop — correct for **bare-metal path**.
  - ❌ Should mention **either** docker-compose path **or** bare-metal with clear distinction.
- **Lines 19–26:** `.env` creation hardcoded with legacy values:
  - `CONSENSUS_TIME=10` ✓ correct
  - `RPC_FEE=5` ❌ should be `RPC_FEE=1`
  - `SERVER_PORT=53550` ❌ should be `RPC_PORT=53550`
  - Missing `NETWORK_FEE=1`, `BURN_FEE=1`
- **Lines 28–36:** Key generation steps reference "It will crash, this is expected" — suggests running bare-metal binary, not docker.
- **Lines 45–65:** "Install Dependencies" section uses Ubuntu terminal to run `.install` script.
  - ❌ This is bare-metal flow; no docker-compose path documented.
- **Lines 94–123:** "Run the Project" explicitly uses `./run` and references `postgres_5332/` directory structure (bare-metal postgres sidecar).

### 5. run-the-project-windows/wsl-2-setup.mdx
**Status:** PASS (infrastructure setup, still valid)
- Generic WSL2 setup; no node-specific versioning issues. Prerequisite only.

### 6. run-the-project-windows/issue-troubleshooting.mdx
**Status:** NEEDS-REVIEW
- **Lines 56–94:** "Issue 2: Yarn Command Failed"
  - ❌ References `yarn cache clean` and `yarn` — but codebase uses **Bun**, not Yarn.
  - ❌ Line 93: Suggests `bun install` as recovery, which is correct, but the issue title/context is backwards (references Yarn as primary tool).
- **General:** Troubleshooting steps are all bare-metal focused (`node_modules`, build tools, eiows) — no docker-compose path.

---

## What's Stale — Specifics

| Issue | Current Docs | Reality (INSTALL.md + .env.example) | Impact |
|-------|--------------|-------------------------------------|--------|
| **Primary install path** | `./install` + `./run` (bare-metal) | `docker compose up` (recommended); bare-metal is Track 2 | 🔴 CRITICAL: Users following docs will miss the simpler path |
| **Fee structure** | `RPC_FEE=5` hardcoded | `RPC_FEE=1` + `NETWORK_FEE=1` + `BURN_FEE=1` (total=3) per commit 447afd9e (May 6) | 🔴 CRITICAL: Wrong defaults; SDK/wallet assumptions break |
| **Env var naming** | `SERVER_PORT` | `RPC_PORT` | 🟡 MEDIUM: Non-existent var silently ignored; node uses fallback |
| **EXPOSED_URL default** | `http://127.0.0.1:53550` hardcoded as example | Docs warn "loopback default useless for peers"; should be user-set | 🟡 MEDIUM: May mislead solo-dev users |
| **TLSNotary profile** | No mention of profiles/gating | `COMPOSE_PROFILES=monitoring,tlsnotary` controls optional services; PR #782 gates TLSNotary profile | 🟡 MEDIUM: Users unaware they can skip it |
| **Postgres setup** | Manual `/install` script edits | Docker compose handles it; bare-metal uses sidecar via `postgres_5332/` | 🟡 MEDIUM: Bare-metal docs outdated (uses old flow) |
| **Node vs Yarn** | Issue troubleshooting references Yarn | Project uses Bun exclusively | 🔴 HIGH: Wrong package manager mentioned |
| **Hardware reqs** | Node version 20.15.1 (redundantly listed 3x) | INSTALL.md only mentions 4GB RAM min, no Node version requirement for docker path | 🟢 LOW: Overkill but not wrong |

---

## What's New in Node Setup That Should Be Documented

From INSTALL.md (Track 1: Docker Compose) + .env.example + PR #782 changes:

1. **Unified docker-compose stack** (PR #782, commit 4b1ed99b)
   - `docker compose up` one-command bring-up
   - Postgres + node + TLSNotary + monitoring all in one compose file
   - Profiles for optional services (`tlsnotary`, `monitoring`, `full`, `neo4j`)
   - Named volumes with `demos_` prefix (persistent across restarts)

2. **Three-component fee model** (PR #778, commit 447afd9e, May 6, 2026)
   - `RPC_FEE=1`, `NETWORK_FEE=1`, `BURN_FEE=1` (total per-tx cost = 3)
   - Once decimals land (Mycelium E#3), three components must total 1 DEM
   - Replaces legacy two-component model (rpcFee + adaptedGas)

3. **TLSNotary profile gating** (PR #782, commit 40630456)
   - TLSNotary is optional, controlled by `COMPOSE_PROFILES=...,tlsnotary`
   - Pair with `TLSNOTARY_ENABLED=true/false` in `.env`
   - Docker mode (default): sidecar container manages key
   - FFI mode (alternate): in-process Rust binding; requires `TLSNOTARY_SIGNING_KEY` or auto-gen

4. **Monitoring stack (Prometheus + Grafana)**
   - Profile-gated (`monitoring` in `COMPOSE_PROFILES`)
   - Default Grafana login: `admin` / `demos`
   - Metrics endpoint: `http://localhost:9090/metrics` (node-internal)
   - Grafana dashboard: `http://localhost:3000`

5. **OmniProtocol binary RPC** (new in docker era)
   - `OMNI_ENABLED=true` / `OMNI_PORT=53551` (auto-derived from `RPC_PORT+1`)
   - Modes: `HTTP_ONLY` | `OMNI_PREFERRED` | `OMNI_ONLY`
   - Complements HTTP RPC at port 53550

6. **MCP (Model Context Protocol) server**
   - `RPC_MCP_PORT=3001` (for AI agent integrations)

7. **Identity & peer persistence**
   - `.demos_identity` (private key) persists in `demos_node_state` volume
   - `demos_peerlist.json` cached and merged with learned peers
   - Both auto-symlinked into `/app/state` by entrypoint

8. **Postgres moved to compose** (was bare-metal sidecar in old flow)
   - Docker compose service: `postgres:16-alpine`
   - Credentials: `PG_USER=demosuser`, `PG_PASSWORD=demospassword`, `PG_DATABASE=demos`
   - Host mapping: docker-compose uses `PG_HOST=postgres` (service name), bare-metal uses `localhost:5332`

9. **TUI (Terminal User Interface)**
   - Default behavior; disable with `./run -t` or `./run --no-tui` (bare-metal only)
   - Tabs: Core, Network, Chain, Consensus, etc.
   - Controls: number keys to switch, arrow keys to scroll, H for help, Q to quit

10. **Production vs. dev settings**
    - `PROD=false` (default, dev mode)
    - `PROD=true` enables DTR production relay + prod-only checks
    - `LOG_LEVEL=info` (default; debug produces wall of consensus JSON)

11. **L2PS ZK experimental feature**
    - `L2PS_ZK_ENABLED=false` (default, off; experimental)

12. **Network ports clarity** (from README)
    - Required: 53550 (RPC), 53551 (OmniProtocol), 7047 (TLSNotary), 55000-60000 (WS proxy for TLSNotary FFI)
    - Optional: 9090 (metrics), 9091 (Prometheus), 3000 (Grafana), 5332 (Postgres on bare-metal only)

13. **Devnet (4-node local network)**
    - `devnet/` directory with docker-compose for testing
    - `scripts/setup.sh` generates identities + peerlist
    - RPC ports: 53551–53554; OmniProtocol: 53561–53564

---

## Recommended Actions

1. **Rewrite all platform guides** (macOS, Ubuntu, Windows) to default to **Track 1: Docker Compose**
   - Move bare-metal (`./run`) to secondary/advanced section
   - Link to INSTALL.md for detailed bare-metal walkthrough

2. **Update all `.env` examples** in the docs
   - Change `RPC_FEE=5` → `RPC_FEE=1`
   - Change `SERVER_PORT` → `RPC_PORT`
   - Add `NETWORK_FEE=1` and `BURN_FEE=1` with comments

3. **Document TLSNotary profile gating**
   - Show how to skip it: `COMPOSE_PROFILES=monitoring` (without `tlsnotary`) + `TLSNOTARY_ENABLED=false`

4. **Add quick-start docker-compose section**
   - 3-step: clone, cp .env.example .env, docker compose up

5. **Fix Windows troubleshooting**
   - Correct "Yarn" references to "Bun"
   - Clarify that bare-metal path (./install + ./run) is still documented in INSTALL.md

6. **Add notes on new features**
   - OmniProtocol (binary RPC)
   - MCP server port
   - TUI controls
   - 3-component fee model

7. **Hardware/versions clarity**
   - Docker compose path: no Node.js required (runs in container)
   - Bare-metal path: Node.js 20.15.1 recommended (or use Bun installer)
   - Clarify this in the platform-specific guides

---

## Files Needing Updates

- `/Users/tcsenpai/kynesys/documentation-mintlify/cookbook/project-setup/run-the-project-macos.mdx` — REWRITE
- `/Users/tcsenpai/kynesys/documentation-mintlify/cookbook/project-setup/run-the-project-ubuntu.mdx` — REWRITE
- `/Users/tcsenpai/kynesys/documentation-mintlify/cookbook/project-setup/run-the-project-windows/overview.mdx` — REWRITE (consolidate docker + bare-metal)
- `/Users/tcsenpai/kynesys/documentation-mintlify/cookbook/project-setup/run-the-project-windows/issue-troubleshooting.mdx` — REVIEW (Yarn → Bun)
- Consider: Add `/Users/tcsenpai/kynesys/documentation-mintlify/cookbook/project-setup/run-the-project-docker-compose.mdx` — NEW (unified quick-start)

