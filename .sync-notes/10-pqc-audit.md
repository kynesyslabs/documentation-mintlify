# PQC Documentation Audit Report

**Date:** 2026-05-09  
**File Audited:** `/sdk/post-quantum-cryptography.mdx`  
**Sources Checked:**
- `/sdks/src/encryption/PQC/` (algorithms)
- `/sdks/src/encryption/unifiedCrypto.ts` (UnifiedCrypto class)
- `/sdks/src/abstraction/Identities.ts` (bindPqcIdentity/removePqcIdentity)
- `/sdks/src/types/cryptography.ts` (PQCAlgorithm type)

---

## Executive Verdict

**PASS with GAPS**

The doc is largely accurate regarding algorithms (Falcon, ML-DSA) and includes key methods (`bindPqcIdentity`, `removePqcIdentity`). Code samples are correct. However:
- **No code sample for `removePqcIdentity` method** (major gap)
- **No mention of `removePqcIdentity` method at all** in the doc
- Missing documentation of `generateAllIdentities()` helper
- The pattern `identities.bindPqcIdentity(demos, "all")` is correct but no dual_sign=false guidance

---

## Detailed Findings

### 1. Algorithms Verification

| Claimed in Doc | Verified in Source | Status |
|---|---|---|
| Falcon | ✓ `/sdks/src/encryption/PQC/falconts/` | OK |
| ML-DSA | ✓ `UnifiedCrypto.supportedPQCAlgorithms = ["falcon", "ml-dsa"]` | OK |

**Source:** `/sdks/src/types/cryptography.ts:1` defines `type PQCAlgorithm = "falcon" | "ml-dsa"`

No other algorithms are implemented or exposed. Doc correctly lists only 2.

---

### 2. Code Sample Verification

#### Sample 1: Wallet Connection with Falcon (Lines 15-27)
```typescript
import { Demos } from "@kynesyslabs/demosdk/websdk"
const demos = new Demos()
const mnemonic = demos.newMnemonic()
await demos.connectWallet(mnemonic, {
    algorithm: "falcon",
    dual_sign: true
})
const tx = demos.sign(tx)
```

**Verification:**
- ✓ `connectWallet()` exists at `/sdks/src/websdk/demosclass.ts:218`
- ✓ `algorithm` option accepted (line 224: `algorithm?: SigningAlgorithm`)
- ✓ `dual_sign` option accepted (line 226: `dual_sign?: boolean`)
- ✓ `sign()` method exists (line 485)
- ✓ Import path `@kynesyslabs/demosdk/websdk` is correct

**Minor issue:** Doc uses `demos.sign(tx)` but sample shows `tx = demos.sign(tx)` - signature implies it modifies and returns the tx. Per source line 485, this is correct.

#### Sample 2: PQC Identity Binding (Lines 34-43)
```typescript
import { Identities } from "@kynesyslabs/demosdk/abstraction"
const identities = new Identities()
const validityData = await identities.bindPqcIdentity(demos, "all")
const res = await demos.broadcast(validityData)
```

**Verification:**
- ✓ `Identities` class exists at `/sdks/src/abstraction/Identities.ts:43`
- ✓ `bindPqcIdentity()` method exists at line 430:
  ```typescript
  async bindPqcIdentity(demos: Demos, algorithms: "all" | PQCAlgorithm[] = "all")
  ```
- ✓ Accepts `"all"` string parameter (line 432: `if (algorithms === "all")`)
- ✓ Returns result suitable for broadcast (line 466: `return await this.inferIdentity(...)` → `RPCResponseWithValidityData`)
- ✓ `broadcast()` method exists on Demos class

**Good documentation for this section.**

---

### 3. PQC Identity Methods - Gap Analysis

**Methods in Identities class (lines 430-498):**

1. **`bindPqcIdentity(demos, algorithms?)`** (line 430)
   - ✓ Documented with code sample (lines 34-43)
   - ✓ Signature supports `"all"` or `PQCAlgorithm[]`
   - ✓ Creates ed25519 signatures for each algorithm and calls `inferIdentity()`

2. **`removePqcIdentity(demos, algorithms?)`** (line 469)
   - ✗ **NOT DOCUMENTED** in mdx file
   - ✗ **NO CODE SAMPLE** for removal workflow
   - Signature identical to `bindPqcIdentity`:
     ```typescript
     async removePqcIdentity(demos: Demos, algorithms: "all" | PQCAlgorithm[] = "all")
     ```
   - Calls `removeIdentity()` instead of `inferIdentity()` (line 497)

**Gap Finding:** Lines 29-48 of mdx mention binding but omit the removal operation entirely. Users have no guidance on how to unlink PQC identities.

---

### 4. UnifiedCrypto Helper Methods

**Available but not documented:**

- `generateAllIdentities()` (line 236) - called internally by `bindPqcIdentity` when `algorithms="all"`
  - Doc does not explain this; it just happens
  - Users cannot call this directly if they need to generate identities separately

- `getIdentity(algorithm)` (line 282) - used to retrieve keypairs
  - Already called in code sample indirectly via `identities.bindPqcIdentity`
  - No direct mention in doc

**Impact:** Low - these are abstracted away by Identities methods. Acceptable omission.

---

### 5. Dual-Sign Guidance

**Current doc (line 47):**
> "For future wallet connections, you can connect your wallet without the `dual_sign` option. Transactions will not have the ed25519 signature and your ownership of the ed25519 address will be validated on the network."

**Verification:**
- ✓ Accurate: `connectWallet` defaults `dual_sign: false` (line 237 of demosclass.ts: `this.dual_sign = options?.dual_sign || false`)
- ✓ Correct behavior: when `dual_sign=false`, only PQC signature is used (line 616)

**No issues here.**

---

## Gap List: Features in SDK but Not in Doc

1. **`removePqcIdentity()` method**
   - Complete absence in documentation
   - No usage example
   - Severity: HIGH (users cannot properly clean up identities)

2. **Algorithm parameter flexibility**
   - Doc only shows `"all"`
   - SDK supports passing individual algorithm array: `bindPqcIdentity(demos, ["falcon"])` or `["ml-dsa"]`
   - Severity: MEDIUM (nice-to-have, not critical)

3. **`generateAllIdentities()` helper**
   - Not documented
   - Pre-generated identities option not explained
   - Severity: LOW (automatically called, not user-facing)

4. **Error handling / validation**
   - No mention of what happens if already bound
   - No guidance on payload structure
   - Severity: LOW (RPC handles validation)

---

## Source-of-Truth References

| Finding | Line Reference |
|---------|-----------------|
| Falcon algorithm | `/sdks/src/encryption/PQC/falconts/` (exists) |
| ML-DSA algorithm | `unifiedCrypto.ts:123` |
| Supported algorithms type | `/sdks/src/types/cryptography.ts:1` |
| `bindPqcIdentity` definition | `Identities.ts:430` |
| `removePqcIdentity` definition | `Identities.ts:469` |
| `connectWallet` signature | `demosclass.ts:218` |
| `sign` method | `demosclass.ts:485` |
| `dual_sign` default | `demosclass.ts:237` |

---

## Recommendations

### Priority 1 (Implement)
- [ ] Add section "### Removing PQC Identities" with code sample:
  ```typescript
  const validityData = await identities.removePqcIdentity(demos, "all")
  const res = await demos.broadcast(validityData)
  ```
- [ ] Document optional parameter: `removePqcIdentity(demos, ["falcon"])` for selective removal

### Priority 2 (Nice-to-have)
- [ ] Mention `generateAllIdentities()` if users need pre-generation before binding
- [ ] Add note about algorithm-specific binding: `bindPqcIdentity(demos, ["ml-dsa"])` vs `"all"`

### Priority 3 (Polish)
- [ ] Add warning/note about binding both algorithms vs selective binding

---

## Conclusion

The doc is **functionally sound** but has a **critical documentation gap** around the removal workflow. The `removePqcIdentity` method exists in the SDK but is completely absent from user-facing documentation. Adding a removal section with a code sample would resolve this.

**Estimated effort to fix:** 15 minutes (add ~15 lines of markdown + code sample)
