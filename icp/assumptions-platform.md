<!-- shared:icp/assumptions-platform.md -->
<!-- Composable section: platform-level assumptions shared across all ICP-based Zombie Delete products. -->
<!-- Product guides inject product-specific assumptions alongside these. -->

## Platform Assumptions (ICP)

The following assumptions apply to all Zombie Delete products deployed on the Internet Computer. Product-specific assumptions are listed in the product's own integration guide.

| # | Assumption | Implication |
|---|---|---|
| **P1** | Library executes within the host canister. | Enterprise retains full control. No external service dependency. |
| **P2** | Receipts contain only hashes, no plaintext PII. | Safe to store publicly. One-way hashes — even the data subject can't reverse them. |
| **P3** | ICP single-threaded execution guarantees atomicity. | No locking or transaction management required. All library operations within a single update call are atomic. |
| **P4** | Canister snapshots may contain pre-deletion state. | Snapshot restore changes the certified commitment, which is detectable by verifiers comparing the receipt's post-state against the canister's current state. |
| **P5** | Code attestation via ICP module hash + subnet BLS. | Replaces TEE attestation model. Trust anchor is subnet consensus, not hardware. See [Module Hash: Deployment Patterns](module-hash-pipeline.md) for the deployment pipeline. |
| **P6** | Tombstone guard is procedural enforcement. | No ICP-level triggers exist. Code change (minimal) is required. Guard removal is detectable via module hash verification (V3). |
| **P7** | Tombstoning is irreversible per principal. | The same identity cannot re-register after deletion. Re-registration requires a new identity and a new canister. Enterprise UX must not offer "rejoin with same identity" flows. See [Tombstone Patterns](tombstone-patterns.md) for anti-resurrection semantics. |
