<!-- shared:icp/deterministic-encoding.md -->
<!-- Composable section: deterministic CBOR encoding requirements for state hashing. -->
<!-- Applies to all Zombie Delete products that hash serialised PII state. -->

## Deterministic Encoding Requirements

### CBOR determinism checklist

`pii_field_manifest()` and `get_state_bytes()` (or their product-specific equivalents) MUST produce identical output across builds and upgrades. The encoding is deterministic under this library's encoder (ciborium + serde). This is **not** RFC 8949 canonical CBOR — determinism relies on the specific rules below.

**All of the following must be followed:**

- **Stable field order:** The manifest must be an ordered sequence with fixed `field_order` values. Never change field ordering after deployment.
- **No `HashMap` / `BTreeMap` in PII structs:** Use structs with named fields. ciborium serialises struct fields in declaration order.
- **No floats:** `encode_pii_state()` rejects any floating-point values (non-deterministic NaN representations).
- **Use `encode_pii_state()` as the only encoding path** for `get_state_bytes()`. Do not call ciborium directly.
- **Version bump rule:** Any change to the PII boundary or serialisation rules requires a manifest version bump (new `manifest_hash`). Old receipts remain verifiable under the `manifest_hash` recorded in each receipt.

### State Encoding Spec

The adapter must publish a **State Encoding Spec** documenting:

| Item | Description |
|---|---|
| **Field names** | Exact field names as they appear in the CBOR encoding. |
| **Types** | Rust types and their CBOR representations. |
| **Serialisation order** | The fixed field order used by the manifest. |
| **Tombstone values** | The hex-encoded `TOMBSTONE_CONSTANT` value written to each field on deletion. |
| **Encoding library version** | The ciborium version and any serde configuration. |

This spec is anchored by `manifest_hash` — any change to the spec changes the manifest hash, ensuring receipt–spec alignment. A verifier with the State Encoding Spec can independently reconstruct the expected CBOR encoding and verify state hashes in receipts.

### Adapter correctness invariant

The library calls `adapter.is_tombstoned()` immediately after `adapter.tombstone_state()` as a post-condition check on the PII fields. If it returns `false`, the library traps and the entire message is rolled back. This catches adapter bugs that would produce invalid receipts.

Test this invariant explicitly during adapter development. A simple test case: call `tombstone_state()`, then assert `is_tombstoned()` returns `true`.
