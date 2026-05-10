# Audit: Introduction & Sundry Pages (11 Files)

**Date:** 2026-05-10  
**Scope:** overview, what-is-demos-network, demos-network-architecture, quick-start, getting-started, demoswork, utilities, authentication/overview, fido2-passkeys/*, signing-a-transaction  
**Checks:** TPS/block-time claims, SDK code examples, FIDO2 methods, deprecated patterns, hallucinated methods

---

## Per-File Status

### 1. `/introduction/overview.mdx`
**Status:** PASS  
- Generic intro page, no code examples or network claims.
- Links to FAQ and Discord for reporting discrepancies — correct guidance.

### 2. `/introduction/what-is-demos-network.mdx`
**Status:** PASS  
- High-level overview, no TPS/block-time claims.
- Mentions "PoR-BFT", "Game Theory Incentives (GTI)", "Zero Knowledge Proofs" generically — no false claims.

### 3. `/introduction/demos-network-architecture.mdx`
**Status:** NEEDS-REVIEW  
**Issues:**
- Line 8: "Link to section" broken placeholder (not a sync issue, but UX debt)
- Line 10: Another "Link to section" broken placeholder
- No code or TPS claims, so technically accurate but incomplete.

### 4. `/introduction/quick-start.mdx`
**Status:** PASS  
**Lines checked:** 28, 37–95, 107–138  
- ✅ `new Demos()` — Correct. Demos class exists, constructor is public.
- ✅ `demos.newMnemonic()` — Exists in demosclass.ts:174.
- ✅ `demos.connectWallet(mnemonic, {algorithm: "ed25519"})` — Exists, signature matches.
- ✅ `demos.connect("https://node2.demos.sh")` — Exists, verified RPC URL is valid test endpoint.
- ✅ `demos.getLastBlockNumber()`, `demos.getLastBlockHash()`, `demos.getAddressInfo(address)` — All exist.
- ✅ `demos.signMessage(message)` → `demos.verifyMessage(message, signature.data, publicKey, {algorithm: signature.type})` — Both exist, return types match.
- ✅ `demos.disconnect()` — Exists, properly disconnects wallet and RPC.
- **No TPS/block-time claims present.** No deprecated patterns detected.
- Import path `@kynesyslabs/demosdk/websdk` is correct per websdk/index.ts exports.

### 5. `/sdk/getting-started.mdx`
**Status:** PASS  
- Installation instructions accurate: `bun add @kynesyslabs/demosdk@latest`.
- Vite Buffer polyfill example correct for browser environments.
- No code examples to verify. Links to Auth, WebSDK, Cross-Chain, Cookbook all valid.

### 6. `/sdk/demoswork.mdx`
**Status:** PASS  
**Lines checked:** 22–32 (DemosWork class), 45–58 (WorkStep), 79–134 (ConditionalOperation)  
- ✅ `DemosWork.push(operation)` — Exists (work.ts:20–26).
- ✅ `DemosWork.toJSON()` — Exists (work.ts:55–75).
- ✅ `WorkStep` properties (id, context, content, output, description, timestamp, critical, depends_on) — All documented match source.
- ✅ `DemosWorkOperation.addWork(work)` — Exists (base class abstract method).
- ✅ `ConditionalOperation.if(value_a, operator, value_b)`, `.then()`, `.else()` — Chaining methods exist.
- ✅ `BaseOperation` — Correctly described at lines 148–159.
- **No cookbook cross-references found** (search for "sdk/cookbook/demoswork" returned 0 matches) — **not a sync issue; demoswork.mdx is self-contained**.
- **No hallucinated methods detected.** All class signatures match source code exactly.

### 7. `/sdk/utilities.mdx`
**Status:** NEEDS-REVIEW  
- File is empty (only frontmatter). This page is incomplete/placeholder.
- No claims to audit, but content is missing.

### 8. `/sdk/websdk/authentication/overview.mdx`
**Status:** PASS  
**Lines checked:** 12–69  
- ✅ `demos.newMnemonic()` — Exists.
- ✅ `demos.connectWallet(mnemonic, {algorithm: "ed25519"})` — Correct signature.
- ✅ `demos.getAddress()` — Exists (demosclass.ts:242–245), returns hex string.
- ✅ `demos.getEd25519Address()` — **EXISTS** (demosclass.ts:300). Doc states "get ed25519 address of connected wallet" — correct.
- ✅ `demos.crypto.getIdentity("ed25519")` — Verified in UnifiedCrypto.
- ✅ `demos.crypto.generateIdentity("falcon")` — Verified.
- ✅ `demos.signMessage(msg)` & `demos.verifyMessage()` — Both exist with correct return types.
- ✅ `demos.connect("https://node2.demos.sh")` — Verified.
- ✅ `demos.disconnect()` — Verified.
- **iframe embeds DemosWebAuth API reference** (line 146) — No longer a global; deprecated pattern flagged in changelog, but iframe is correct as DemosWebAuth is still exported from websdk/index.ts.
- **No TPS/block-time claims.**

### 9. `/sdk/websdk/authentication/fido2-passkeys/overview.mdx`
**Status:** PASS  
**Lines checked:** 13–42  
- ✅ Module path `sdks/src/wallet/passkeys/hmywallet` — Correct.
- ✅ Wrapper path `sdks/src/wallet/passkeys/passkeys.ts` — Correct.
- ✅ `wallet.generatePasskey()` — **Exists** (Wallet.ts:132–135), returns Promise<string> (hex private key). Method is public and accessible via `import wallet from "@kynesyslabs/demosdk"`.
- ✅ Test command `tsx src/wallet/passkeys/passkeys.ts` — passkeys.ts has test block at lines 50–63.
- ✅ Returns "hexadecimal string containing the private key linked to credentials" — Exact match to implementation (passkeys.ts:37).
- **No hallucinated methods.** All claims verified.

### 10. `/sdk/websdk/authentication/fido2-passkeys/under-the-hood.mdx`
**Status:** PASS  
**Lines checked:** 10–30  
- ✅ Describes fido2 module interaction, credential.pkl generation, password-based encryption, salt derivation.
- ✅ PasskeyGenerator class & `.generate()` method exist (passkeys.ts:8–46).
- ✅ `generate.sh` script referenced — exists in hmywallet/ directory.
- No false claims about implementation. Technical depth matches source.

### 11. `/sdk/websdk/transactions/signing-a-transaction.mdx`
**Status:** PASS  
**Lines checked:** 12–23  
- ✅ `new Demos()` — Correct.
- ✅ `demos.connectWallet(mnemonic)` — Correct, algorithm defaults to "ed25519".
- ✅ `demos.sign(tx)` — Exists (demosclass.ts ~line 1067+), takes Transaction and returns signed tx.
- No TPS/block-time claims. Minimal example, no errors.

---

## Top 5 Issues Across All Files

### 1. **`/sdk/utilities.mdx` — Empty Page (Content Missing)**
   - **Severity:** MEDIUM  
   - **Impact:** User lands on blank page expecting utility function docs.  
   - **Fix:** Add utility function reference or mark as "Coming Soon".

### 2. **`/introduction/demos-network-architecture.mdx` — Broken Link Placeholders (Lines 8, 10)**
   - **Severity:** LOW  
   - **Impact:** Broken documentation navigation, unclear references.  
   - **Fix:** Replace "[Link to section]" with actual anchors (e.g., `/sdk/getting-started`, `/sdk/overview`).

### 3. **`/sdk/websdk/authentication/overview.mdx` — Deprecated DemosWebAuth iframe (Line 146)**
   - **Severity:** LOW  
   - **Impact:** References legacy API reference style; still works but might confuse developers about class availability.  
   - **Observation:** Changelog notes `DemosWebAuth` is no longer a global singleton, but it remains exported from websdk. iframe is accurate; no action needed unless deprecating class entirely.

### 4. **No Network Parameter Documentation for Consensus Time**
   - **Severity:** INFO  
   - **Finding:** Audit checked for "CONSENSUS_TIME=10s" in docs per instructions. No docs mention network consensus time, block time, or TPS. Source code confirms `CONSENSUS_TIME=10` in node test rehearsals, but **no documentation makes false claims about this**, so no sync issue found.

### 5. **`/sdk/websdk/authentication/fido2-passkeys/overview.mdx` — Import Pattern Could Be More Explicit (Line 33)**
   - **Severity:** LOW  
   - **Impact:** Docs show `import wallet from "@kynesyslabs/demosdk"` without showing the Wallet.generatePasskey() pattern fully.  
   - **Observation:** This is actually correct per Wallet.ts default export, but example could show: `const w = wallet.default.getInstance("name"); await w.generatePasskey()`. Current example works but is terse. **Not a sync error; minor UX improvement.**

---

## Summary

**Files Audited:** 11  
**PASS:** 9  
**NEEDS-REVIEW:** 2 (utils.mdx empty; architecture.mdx has placeholder links)  
**FAIL:** 0  

**SDK Code Examples:** All verified. No hallucinated methods, no broken imports, no deprecated patterns in quick-start or signing examples.

**FIDO2/Passkeys:** `wallet.generatePasskey()` exists and matches documentation exactly.

**Network Claims:** No TPS/block-time/consensus claims in any page — **no false assertions found**.

**DemosWork:** All class methods, WorkStep properties, and ConditionalOperation chaining verified against source. No hallucinated methods.

**Status:** ✅ **Documentation is in sync with source code.** Two minor UX issues noted (empty page, placeholder links) but no correctness failures.

---

**Report Generated:** 2026-05-10  
**Auditor:** Claude Code
