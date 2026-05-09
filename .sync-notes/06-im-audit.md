# Instant Messaging Documentation Audit Report
**Date:** 2026-05-09  
**Auditor:** Claude Code  
**Status:** CRITICAL ISSUES FOUND

---

## EXECUTIVE SUMMARY

The instant-messaging documentation is **SUBSTANTIALLY MISALIGNED** with the actual SDK. The docs describe only the legacy `MessagingPeer` class (port 3005) but **completely omit** the newer `L2PSMessagingPeer` class (port 3006) which is the primary implementation. Critical issues:

1. **Missing L2PSMessagingPeer entirely** — The new L2PS-backed messaging system is completely undocumented
2. **Inconsistent export paths** — FAQ imports from `@kynesyslabs/demosdk/instant_messaging` (not in package.json) while quickstart uses `instantMessaging` namespace
3. **Legacy-only documentation** — All 8 docs focus exclusively on the deprecated port 3005 signaling server
4. **No mention of L2PS network isolation, offline delivery, or history API** — Major features missing from docs

**Verdict: 6 docs FAIL, 2 docs PASS with caveats**

---

## CROSS-CUTTING ISSUES

### Issue 1: Undocumented L2PSMessagingPeer Class
**Severity:** CRITICAL  
**Impact:** Users cannot discover or use the primary implementation

The SDK exports **two completely different messaging systems**:
- **MessagingPeer** (legacy, port 3005) — Simple peer-to-peer via signaling server
- **L2PSMessagingPeer** (production, port 3006) — L2PS rollup-backed, with history, offline delivery, network isolation

**Evidence:**
- `/Users/tcsenpai/kynesys/sdks/src/instant_messaging/index.ts:4` exports `L2PSMessagingPeer`
- `/Users/tcsenpai/kynesys/sdks/src/instant_messaging/L2PSMessagingPeer.ts` — 595-line implementation with types
- Documentation makes **zero mention** of L2PSMessagingPeer or L2PS network features

**API Differences (docs don't acknowledge):**
- L2PSMessagingPeer requires `l2psUid` and `signFn` (config-level proof signing)
- L2PSMessagingPeer has `history()` API with pagination (MessagingPeer doesn't)
- L2PSMessagingPeer has offline message delivery (`messageHash`, `status: "queued"`)
- L2PSMessagingPeer uses `requestId` for request-response correlation (different pattern)

### Issue 2: Broken Import Paths
**Severity:** HIGH  
Two different import patterns in documentation, only one registered in package.json:

**quickstart.mdx:9** (uses namespace):
```typescript
import { instantMessaging } from "@kynesyslabs/demosdk"
```
✓ VERIFIED — works via `/Users/tcsenpai/kynesys/sdks/src/index.ts:28`

**faq.mdx:71** (uses subpath):
```typescript
import { MessagingPeer } from "@kynesyslabs/demosdk/instant_messaging"
```
✗ NOT REGISTERED in package.json — likely fails at runtime
- package.json exports list shows no `./instant_messaging` entry
- Only registered exports: `./websdk`, `./l2ps`, `./encryption`, etc.

### Issue 3: Port Confusion
**Severity:** MEDIUM

Documentation references port **3005 throughout** but the SDK comment indicates:
- Port **3005** = legacy MessagingPeer (undocumented in SDK, but docs use it)
- Port **3006** = L2PSMessagingPeer (completely undocumented in docs)

No docs clarify when to use which port.

---

## PER-FILE AUDIT DETAILS

### 1. **overview.mdx** — PASS with caveats
**Lines audited:** All 16 lines  
**Verdict:** PASS (but incomplete)

Issues:
- Line 15: Claims "we will learn how to use the `MessagingPeer` class" — true but misleading, doesn't mention L2PSMessagingPeer exists
- Line 11: "signalingServer" runs "automatically on every Demos node" — ✓ verified in code comments
- Describes port 3005 implicitly but doesn't specify

No factual errors, but context is incomplete.

---

### 2. **what-is-the-instant-messaging-protocol.mdx** — FAIL
**Lines audited:** All 19 lines  
**Verdict:** FAIL — Oversimplified, ignores L2PS features

**Claim:** "The Instant Messaging Protocol is a robust, secure, and flexible framework for building peer-to-peer messaging applications."
- ✓ True for MessagingPeer
- ✗ Incomplete — L2PSMessagingPeer adds L2PS rollup backing, offline queuing, history API

**Claim (line 12):** "End-to-end encryption using ml-kem-aes"
- ✓ Verified both classes use this (index.ts:471, L2PSMessagingPeer.ts doesn't override)

**Missing from doc:**
- L2PS network isolation (`l2psUid` parameter)
- Message history with pagination
- Offline delivery and message queuing
- L2PS mempool status tracking

---

### 3. **architecture-overview.mdx** — FAIL
**Lines audited:** All 11 lines  
**Verdict:** FAIL — Architecture is incomplete without L2PSMessagingPeer

**Claim (line 8):** "Signaling Server: A central hub that facilitates peer discovery and message routing. It runs automatically on every Demos node..."
- ✓ True for port 3005 (legacy)
- ✗ Incomplete — doesn't mention port 3006 (L2PS) which also provides message persistence

**Missing:**
- L2PS batch → proof → L1 pipeline (mentioned in L2PSMessagingPeer.ts:4-6 but not in architecture doc)
- Distinction between transient signaling (port 3005) vs. persistent L2PS (port 3006)
- How messages "are attested onto the Demos Network" (line 12 of overview.mdx) is never explained

**What's actually in L2PSMessagingPeer (undocumented):**
```
Messages → L2PS mempool → Batch → L1 proof → L1 finality
Status tracking: l2ps_pending → l2ps_batched → l2ps_confirmed
```

---

### 4. **encryption.mdx** — PASS
**Lines audited:** All 15 lines  
**Verdict:** PASS (technically accurate for MessagingPeer)

**Claim (line 6):** "The Instant Messaging Protocol uses ml-kem-aes"
- ✓ Verified in index.ts:471, L2PSMessagingPeer.ts uses same encryption

**Claims on key flow (lines 8-12):**
- Line 8: "Key Generation: Each peer generates a public/private key pair" — ✓ verified
- Line 10: "Signatures: peers sign their ml-kem public keys with a ml-dsa signing key" — ✓ verified in index.ts:411-420
- Line 11: "Messages are encrypted using the recipient's public key through encapsulation" — ✓ verified index.ts:470-474
- Line 12: "Recipients decrypt messages using their private key to decapsulate" — ✓ verified index.ts:749-751

**Minor note:** Doc doesn't distinguish between MessagingPeer flow and L2PSMessagingPeer flow (which also uses same encryption), so while accurate for what it covers, it's incomplete in scope.

---

### 5. **quickstart.mdx** — FAIL
**Lines audited:** All 45 lines  
**Verdict:** FAIL — Wrong import pattern and outdated class focus

**Line 9-10: Import statements**
```typescript
import { instantMessaging } from "@kynesyslabs/demosdk"
import { encryption } from "@kynesyslabs/demosdk"
```
- ✓ `instantMessaging` — verified in index.ts:28
- ✓ `encryption` — verified in index.ts:4
- But FAQ imports differently (line 71: `from "@kynesyslabs/demosdk/instant_messaging"`) — inconsistent!

**Line 23-27: Constructor**
```typescript
const peer = new instantMessaging.MessagingPeer({
    serverUrl: "ws://your-signaling-server:3005",
    clientId: "user-" + Date.now(),
    publicKey: mlKemAes.publicKey,
})
```
- ✓ MessagingPeerConfig interface verified (index.ts:160-164)
- ✓ Constructor exists (index.ts:204)
- ⚠ **Missing L2PSMessagingPeer example** — user would not know it exists

**Line 24: Port 3005 hardcoded**
- Not wrong, but undocumented trade-off vs. port 3006
- No guidance on choosing between MessagingPeer vs. L2PSMessagingPeer

**Line 37: discoverPeers() call**
- ✓ Method exists (index.ts:444-455)
- ✓ Returns string[] (verified)

**Missing context:**
- When to use MessagingPeer (transient) vs. L2PSMessagingPeer (persistent)
- No mention of online peer discovery differences

---

### 6. **message-types.mdx** — FAIL
**Lines audited:** All 20 lines  
**Verdict:** FAIL — Incomplete message type coverage

Lists message types for **MessagingPeer only**:
```
"register", "discover", "message", "peer_disconnected",
"request_public_key", "public_key_response", "server_question",
"peer_response", "debug_question", "error"
```

**Verified in index.ts:167-177** ✓ (for MessagingPeer)

**But L2PSMessagingPeer uses DIFFERENT message types** (l2ps_types.ts:43-63):
```
Client → Server:
  "register", "send", "history", "discover", "request_public_key", "ack"

Server → Client:
  "registered", "message", "message_sent", "message_queued",
  "history_response", "discover_response", "public_key_response",
  "peer_joined", "peer_left", "error"
```

**Completely different protocol!** L2PSMessagingPeer has:
- `message_sent` / `message_queued` (delivery status)
- `history` request type (not in MessagingPeer)
- `peer_joined` / `peer_left` (different from `peer_disconnected`)
- `ack` message type (not documented anywhere)

**Doc lists only one protocol, doesn't acknowledge two coexist.**

---

### 7. **api-reference.mdx** — FAIL
**Lines audited:** All 406 lines  
**Verdict:** FAIL — Missing L2PSMessagingPeer entirely, incomplete method signatures

**Constructor documentation (lines 12-35):**
```typescript
constructor(config: MessagingPeerConfig)
```
- ✓ Signature matches index.ts:204
- ✗ **Completely omits L2PSMessagingPeer constructor** (L2PSMessagingPeer.ts:112-114)
  ```typescript
  constructor(config: L2PSMessagingConfig)
  // with different required fields: serverUrl, publicKey, l2psUid, signFn
  ```

**Methods documented for MessagingPeer:**
- ✓ `connect()` — exists (index.ts:212)
- ✓ `disconnect()` — exists (index.ts:621)
- ✓ `register()` — exists (index.ts:394)
- ✓ `registerAndWait()` — exists (index.ts:409)
- ✓ `discoverPeers()` — exists (index.ts:444)
- ✓ `sendMessage(targetId, message)` — exists (index.ts:462)
- ✓ `requestPublicKey(peerId)` — exists (index.ts:502)
- ✓ `respondToServer(questionId, response)` — exists (index.ts:527)
- ✓ `sendToServerAndWait<T>(message, expectedResponseType, options)` — exists (index.ts:666)

**Methods MISSING for L2PSMessagingPeer:**
- `send(to, encrypted, messageHash)` — L2PSMessagingPeer.ts:205 **NOT DOCUMENTED**
- `history(peerKey, options)` — L2PSMessagingPeer.ts:234 **NOT DOCUMENTED**
- `discover()` — L2PSMessagingPeer.ts:268 (different signature from MessagingPeer!)
- `requestPublicKey(targetId)` — L2PSMessagingPeer.ts:294 (returns `Promise<string|null>` not `Uint8Array`)

**Event handlers (lines 255-361):**
Documented for MessagingPeer:
- ✓ `onMessage()` — exists (index.ts:556)
- ✓ `onError()` — exists (index.ts:564)
- ✓ `onPeerDisconnected()` — exists (index.ts:572)
- ✓ `onConnectionStateChange()` — exists (index.ts:580)
- ✓ `onServerQuestion()` — exists (index.ts:541)

**Missing L2PSMessagingPeer event handlers:**
- `onPeerJoined()` — L2PSMessagingPeer.ts:324 **NOT DOCUMENTED**
- `onPeerLeft()` — L2PSMessagingPeer.ts:328 **NOT DOCUMENTED**
(Different from `onPeerDisconnected`)

**Port 3005 hardcoded in examples:**
- Lines 31, 34, 82, 152 — all reference port 3005
- No mention of port 3006 or when to use it

**Message Types section (lines 363-395):**
Lists MessagingPeer types only, doesn't mention L2PSMessagingPeer uses completely different types.

---

### 8. **faq.mdx** — FAIL
**Lines audited:** All 218 lines  
**Verdict:** FAIL — Wrong import path, misleading on L2PS claims

**Line 71: Broken import**
```typescript
import { MessagingPeer } from "@kynesyslabs/demosdk/instant_messaging"
```
✗ **NOT REGISTERED in package.json**
- Works via TypeScript path mapping at build time but not guaranteed to work at runtime
- Inconsistent with quickstart.mdx which uses `instantMessaging` namespace

**Line 11-12 (overview.mdx referenced):**
"Communications are attested onto the Demos Network at each block while preserving confidentiality"
- ✓ True for L2PSMessagingPeer (l2ps_types.ts:224: `l2psTxHash`)
- ✗ Misleading for MessagingPeer — it has NO blockchain attestation
- **Doc doesn't distinguish which class this applies to**

**Line 38-44: Encryption questions**
All answers are accurate for MessagingPeer:
- ✓ "Uses public-key cryptography" (index.ts:470)
- ✓ "ML-KEM-AES" (index.ts:471)
- ✓ "Perfect forward secrecy" (line 143) — ✓ verified via fresh encapsulation per message

But these also apply to L2PSMessagingPeer, just not documented as part of L2PS section.

**Line 52: Offline delivery claim**
"Messages are queued if the recipient is offline and delivered when they reconnect."
- ✗ **INCORRECT for MessagingPeer** — it has no offline queue (no persistence)
- ✓ **CORRECT for L2PSMessagingPeer** (l2ps_types.ts:230-236 shows message statuses including "queued")
- **Doc doesn't distinguish!**

**Line 53: Acknowledgment mechanisms**
"The protocol also includes acknowledgment mechanisms to confirm message receipt."
- ✗ **INCORRECT for MessagingPeer** — no ack mechanism (index.ts has no "ack" handler)
- ✓ **CORRECT for L2PSMessagingPeer** (has `message_sent` / `message_queued` responses)

**Line 70-86: Code example**
Uses correct import pattern from websdk (✓) but references:
- Port 3005 (legacy)
- `MessagingPeer` class only
- No mention of L2PSMessagingPeer alternative

---

## SUMMARY TABLE

| File | Verdict | Critical Issues | Notes |
|------|---------|-----------------|-------|
| overview.mdx | PASS* | 0 | Accurate but incomplete context |
| what-is-the-instant-messaging-protocol.mdx | FAIL | 1 | Omits L2PS features entirely |
| architecture-overview.mdx | FAIL | 2 | No mention of L2PS pipeline; incomplete |
| encryption.mdx | PASS | 0 | Accurate for both classes (same algo) |
| quickstart.mdx | FAIL | 1 | Missing L2PSMessagingPeer example |
| message-types.mdx | FAIL | 2 | Only documents MessagingPeer types; L2PSMessagingPeer types are completely different |
| api-reference.mdx | FAIL | 4 | L2PSMessagingPeer API completely missing; method signatures differ |
| faq.mdx | FAIL | 3 | Wrong import path; conflates MessagingPeer/L2PSMessagingPeer capabilities |

**TOTAL: 2 PASS, 6 FAIL**

---

## TOP 3 MOST CRITICAL ISSUES

### 1. **L2PSMessagingPeer Completely Undocumented**
- SDK exports L2PSMessagingPeer as primary implementation (595 lines, full-featured)
- Zero documentation for this class
- Users cannot discover it exists
- Different API, types, capabilities than MessagingPeer
- **Impact:** Users may build on deprecated MessagingPeer instead of production L2PSMessagingPeer

### 2. **Inconsistent Import Paths & Missing package.json Export**
- quickstart.mdx: `import { instantMessaging } from "@kynesyslabs/demosdk"` ✓
- faq.mdx: `import { MessagingPeer } from "@kynesyslabs/demosdk/instant_messaging"` ✗ not registered
- package.json has no `./instant_messaging` export entry
- **Impact:** FAQexample fails at runtime; no subpath export

### 3. **Conflated L2PS Features (Offline Delivery, Ack, History)**
- faq.mdx line 52 claims "Messages are queued if the recipient is offline"
  - True for L2PSMessagingPeer only (has `messageHash`, `status: "queued"`)
  - False for MessagingPeer (no persistence)
- faq.mdx line 53 claims "acknowledgment mechanisms"
  - True for L2PSMessagingPeer (`message_sent`, `message_queued`)
  - False for MessagingPeer (no ack)
- faq.mdx line 52 claims "conveyed when they reconnect"
  - Only true for L2PSMessagingPeer; MessagingPeer has no offline queue
- **Impact:** Users expect features that don't exist in MessagingPeer

---

## RECOMMENDATIONS

1. **Create L2PSMessagingPeer documentation** (new file or section)
   - Document L2PSMessagingConfig interface
   - Document L2PS-specific methods: `send()`, `history()`, offline delivery
   - Explain l2psUid and `signFn` requirement
   - Explain L2PS batch → proof → L1 pipeline

2. **Add a "Choosing Your API" guide**
   - When to use MessagingPeer (transient, simple, port 3005)
   - When to use L2PSMessagingPeer (persistent, L2PS-backed, port 3006, history, offline)
   - Comparison table

3. **Fix import inconsistencies**
   - Update faq.mdx line 71 to use namespace import: `import { instantMessaging } from ...`
   - OR register `./instant_messaging` export in package.json
   - Ensure all examples use consistent pattern

4. **Document message-types separately for each class**
   - MessagingPeer message types (current)
   - L2PSMessagingPeer message types (new section)

5. **Clarify which features apply to which class**
   - Mark offline delivery, ack, history as "L2PSMessagingPeer only"
   - Mark blockchain attestation as "L2PSMessagingPeer only"
   - Mark port numbers explicitly

---

## AUDIT METHODOLOGY

- **Source of truth:** `/Users/tcsenpai/kynesys/sdks/src/instant_messaging/`
- **Verified against:** All exported classes, interfaces, methods, parameters
- **Import paths checked:** Against `/Users/tcsenpai/kynesys/sdks/src/index.ts` and `package.json`
- **Type signatures:** Exact match verification for public APIs
- **Feature parity:** Whether documented features exist in SDK code

