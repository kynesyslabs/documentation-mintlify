# ZK Identity Backend Audit

2 PASS, 2 NEEDS-REVIEW. One **critical security-claim discrepancy**: nullifier formula is wrong in 2 places.

| File | Verdict |
|------|---------|
| `backend/zk-identity/overview.mdx` | ⚠ NEEDS-REVIEW |
| `backend/zk-identity/setup.mdx` | ✓ PASS |
| `backend/zk-identity/api-reference.mdx` | ⚠ NEEDS-REVIEW |

## overview.mdx — NEEDS-REVIEW

- ✓ Two-phase flow (commitment + attestation) correct
- ✓ Groth16 proof system (verified `ProofVerifier.ts:102–105`)
- ✓ Poseidon for commitment (verified `identity_with_merkle.circom:107`)
- ❌ **Line 24:** Nullifier shown as `Poseidon(provider_id, context)`. Real circuit (`identity_with_merkle.circom:131`): `Poseidon(provider_id, secret, context)`. **Doc omits the secret component** — without it the nullifier wouldn't actually be unlinkable across attestations.
- ✓ Merkle tree, privacy model, security guarantees accurate

## setup.mdx — PASS

- ✓ Directory `src/features/zk/` correct
- ✓ Git policy (verification_key.json committed) correct
- ✓ Powers of Tau download link valid
- ✓ snarkjs commands well-formed (lines 78–86)

## api-reference.mdx — NEEDS-REVIEW

- ✓ Tx types `identity_commitment` + `identity_attestation` verified (`GCRIdentityRoutines.ts:65, 71`)
- ✓ Payload structures match `features/zk/types/index.ts`
- ✓ DB tables `identity_commitments`, `used_nullifiers`, `merkle_tree_state` all match entity definitions
- ❌ **Line 41:** Same nullifier formula error as overview — `Poseidon(provider_id, context)` should be `Poseidon(provider_id, secret, context)`
- ✓ All API methods exist (no fictional methods)
- ✓ MerkleProofResponse matches `MerkleTreeManager.ts`

## Cross-file checks

| Claim | Status |
|-------|--------|
| Tx types (`zk_commitmentadd`, `zk_attestationadd`) | ✓ verified in `zkRoutines.ts` |
| Commitment hash `Poseidon(provider_id, secret)` | ✓ correct |
| **Nullifier hash** | ❌ doc says 2-arg, code is 3-arg (includes secret) |
| Proof system Groth16 | ✓ verified (groth16VerifyBun, snarkjs) |
| DB schema | ✓ all match |
| Fictional methods | none found |

## Recommended actions

1. Fix nullifier formula in both overview.mdx:24 and api-reference.mdx:41 to `Poseidon(provider_id, secret, context)`.
2. Add a sentence explaining why `secret` is included (cross-context unlinkability).
