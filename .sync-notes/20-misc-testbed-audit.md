# Audit Report: Miscellaneous & Developers Testbed Docs
**Date:** 2026-05-10  
**Baseline:** INSTALL.md (Track 1 Docker Compose, Track 2 bare metal) + .env.example  
**Scope:** 10 files in `/backend/miscellaneous/` and `/backend/developers-testbed/`

---

## File-by-File Assessment

### 1. **backend/miscellaneous/overview.mdx** — PASS
- **Status:** No technical content (front matter only)
- **Issues:** None

---

### 2. **backend/miscellaneous/browsing-the-postgres-db-via-psql.mdx** — NEEDS-REVIEW
- **Status:** Mixed accuracy
- **Issues:**
  1. **Line 23:** `psql -p 5332` — correct for bare metal (Track 2), but misleading without context
     - Docker Compose uses `PG_PORT=5432` (internal container port), not 5332
     - Bare metal uses `PG_PORT=5332` (host-mapped sidecar)
     - **Recommendation:** Add Docker vs. bare-metal instructions
  2. **Line 31:** Reference to `postgres/docker-compose.yml` exists but doc should clarify it's Track 2 legacy
  3. **No mention of docker exec approach** for Docker Compose users (correct method: `docker exec demos-postgres psql -U demosuser -d demos`)

---

### 3. **backend/miscellaneous/backing-up-and-restoring-a-node.mdx** — NEEDS-REVIEW
- **Status:** Relies on obsolete `./run` script workflow
- **Issues:**
  1. **Line 10:** `./run -b true` — this is **bare-metal only (Track 2)**, not relevant to Docker Compose (Track 1)
  2. **Line 18:** References `-c true` flag — valid in bare metal, but Docker Compose doesn't use `./run`
  3. **No Docker Compose backup method** — INSTALL.md documents correct method (lines 221–225):
     ```bash
     docker run --rm -v demos_node_state:/src -v "$PWD":/dst alpine \
       tar czf /dst/node-state-backup.tar.gz -C /src .
     ```
  4. **Missing reference** to Docker Compose restore procedure

---

### 4. **backend/miscellaneous/joining-the-testnet-using-a-custom-genesis.mdx** — NEEDS-REVIEW
- **Status:** Partially outdated; assumes bare-metal setup
- **Issues:**
  1. **Line 8 & 13:** References `/genesis` endpoint — valid but no mention of how genesis works in current fork system
  2. **Line 14:** `data/genesis.json` — correct path but only relevant for bare metal (Track 2)
  3. **Line 28:** `./run -c true` — bare-metal only; Docker Compose users must use `docker compose down -v` + volume recreation
  4. **No mention of forkConfig** — recent system uses `genesis.json` + `forkConfig` (per project context); doc should clarify modern approach
  5. **Line 34:** Claim about genesis hash check — true but no guidance on what to do if hashes mismatch

---

### 5. **backend/developers-testbed/overview.mdx** — PASS
- **Status:** Introductory; no technical claims
- **Issues:** None (accurate context setting)

---

### 6. **backend/developers-testbed/setting-up-the-environment.mdx** — PASS
- **Status:** Hardware/Bun requirements accurate
- **Issues:** None (aligns with INSTALL.md system requirements)

---

### 7. **backend/developers-testbed/setting-up-the-repository.mdx** — PASS
- **Status:** Clone instructions correct
- **Issues:** None

---

### 8. **backend/developers-testbed/installing-dependencies.mdx** — PASS
- **Status:** `bun install` instructions accurate
- **Issues:** None (matches INSTALL.md Track 2 steps)

---

### 9. **backend/developers-testbed/node-configuration.mdx** — NEEDS-REVIEW
- **Status:** Conflates Track 1 (Docker) and Track 2 (bare metal)
- **Issues:**
  1. **Line 9:** Claims `.env` is precompiled — should specify this is copied from `.env.example`
  2. **Line 12:** States Postgres listens on port **5332** — **ONLY true for bare metal**
     - Docker Compose: Postgres listens on `5432` (container-internal), accessible via `PG_HOST=postgres`
     - **Major clarity issue:** This doc appears to be Track 2 (bare metal) but is presented as generic
  3. **Line 14:** Note about Twitter/GitHub config is vague; should clarify these are optional API keys (see .env.example lines 177–199)
  4. **Line 22:** `./run` mentioned but no Docker Compose equivalent (should say "if using Track 2" or split into two paths)
  5. **Line 34:** `demos_peerlist.example.json` — correct for Track 2, but Docker Compose guide should document bootstrap peerlist approach (INSTALL.md lines 113–130)

---

### 10. **backend/developers-testbed/running-the-node.mdx** — NEEDS-REVIEW
- **Status:** Bare-metal focus (Track 2 only)
- **Issues:**
  1. **Line 16:** `./run` command — **Track 2 only**, not universal
  2. **Line 18:** Postgres sidecar spinning up — true for `./run`, but Docker Compose users need `docker compose up`
  3. **No mention of Docker Compose** — entire doc assumes bare-metal `./run` workflow
  4. **Line 28:** `-c true` flag behavior described correctly for Track 2 but missing Docker equivalent
  5. **Line 48:** Runtime flag `-r bun/node` states "`bun` is required" — accurate but confuses with Node.js; should emphasize Bun only, no Node.js fallback

---

## Top 5 Critical Issues Across All Files

1. **Track 1 vs. Track 2 Conflation** (files 3, 4, 9, 10)
   - Docs mix Docker Compose (Track 1: `PG_PORT=5432`, `PG_HOST=postgres`, `docker compose up`) and bare-metal `./run` (Track 2: `PG_PORT=5332`, `localhost`)
   - **Impact:** Users attempting Docker Compose follow bare-metal instructions and fail at database connection

2. **Postgres Port Misguidance** (files 2, 9)
   - Docs hardcode port 5332 without distinguishing context
   - **Correct values:**
     - Docker Compose: `5432` (internal); `PG_HOST=postgres` (service name)
     - Bare metal: `5332` (host-mapped); `PG_HOST=localhost`

3. **Missing Docker Compose Procedures** (files 3, 4, 10)
   - Backup, restore, genesis, and node startup docs omit Docker Compose entirely
   - Example: Backup docs don't mention `docker run ... tar czf` (INSTALL.md lines 221–225)
   - Example: Running node doc doesn't mention `docker compose up` (only `./run`)

4. **Obsolete `./run` Script References** (files 3, 4, 9, 10)
   - Docs describe `-b` (backup), `-c` (clean), `-d` (db port) flags as primary workflows
   - Reality: INSTALL.md Track 1 uses `docker compose`, Track 2 uses `./run`
   - Docs fail to introduce two tracks or guide users to correct path

5. **Genesis & Fork System Vagueness** (file 4)
   - "Custom genesis" doc assumes `genesis.json` is the only config
   - Recent fork system uses `genesis.json` + `forkConfig` per project context
   - Docs don't explain when/how to use forkConfig or how it differs from genesis

---

## Detailed Recommendations

| File | Action | Priority |
|------|--------|----------|
| browsing-postgres-db-via-psql.mdx | Add Docker Compose section with `docker exec` method | High |
| backing-up-and-restoring-a-node.mdx | Split into Track 1 (docker compose) and Track 2 (./run) sections | High |
| joining-the-testnet-using-a-custom-genesis.mdx | Add forkConfig guidance; clarify genesis hash behavior | Medium |
| node-configuration.mdx | Rewrite for two tracks OR redirect to INSTALL.md for setup | High |
| running-the-node.mdx | Add "Choose your track" intro; provide both `docker compose` and `./run` commands | High |

---

## Summary

**PASS:** 5 files (overview x2, environment, repository, dependencies)  
**NEEDS-REVIEW:** 5 files (psql, backup, genesis, config, running)  
**FAIL:** 0 files (but 5 are seriously misaligned)

**Root cause:** Docs were written for Track 2 (bare-metal `./run`) and never updated for Track 1 (Docker Compose). INSTALL.md clearly dual-documents both; these 5 files do not. Environment file (`.env.example`) unambiguously defaults to Docker Compose (PG_HOST=postgres, PG_PORT=5432, TLSNOTARY_HOST=tlsnotary), but procedural docs ignore this.

**Estimate to fix:** ~4 hours (rewrite 5 files to dual-track or upstream to INSTALL.md).

