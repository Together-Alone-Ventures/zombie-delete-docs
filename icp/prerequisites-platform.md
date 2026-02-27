<!-- shared:icp/prerequisites-platform.md -->
<!-- Composable section: platform-level prerequisites for any ICP-based Zombie Delete product. -->
<!-- Product guides inject product-specific prerequisite rows alongside these. -->

## Platform Prerequisites (ICP)

The following prerequisites apply to all Zombie Delete products deployed on the Internet Computer.

| Prerequisite | Detail | Responsibility |
|---|---|---|
| **Rust toolchain** | Stable Rust (edition 2021+) with `wasm32-unknown-unknown` target installed. | Enterprise dev team |
| **dfx SDK** | DFINITY SDK for local canister development, testing, and deployment. | Enterprise dev team |
| **ic-cdk version** | ic-cdk 0.17 or later. All current Zombie Delete ICP libraries depend on ic-cdk 0.17; canisters using earlier versions must upgrade. | Enterprise dev team |
| **Canister architecture** | A deployed (or deployable) Rust canister using StableCell, StableBTreeMap, or equivalent stable memory structures for PII storage. | Enterprise dev team |
| **Canister upgrade mechanism** | Ability to push new WASM to deployed canisters (e.g., factory `install_code` with Upgrade mode, governance proposal, or direct `dfx deploy`). The Zombie Delete integration is deployed as a standard canister upgrade — the enterprise must already have this infrastructure. | Enterprise dev team |
| **ICP cycles** | Sufficient cycles balance for canister operations (deletion processing, certified variable updates, receipt storage). | Enterprise ops |
| **Subnet ID** | The Principal of the ICP subnet hosting the canister. ICP canisters cannot discover their subnet at runtime. Obtain from the [ICP dashboard](https://dashboard.internetcomputer.org/) or NNS registry. Hardcoded in config for single-subnet deployments; passed as an init/upgrade argument for multi-subnet architectures. | Enterprise ops |
| **PII field inventory** | A complete enumeration of which fields constitute PII and must be included in the state hash and tombstoned (or otherwise erased) on deletion. | Compliance + dev team |
| **Access control design** | A decision on which principals are authorised to trigger deletion (controller, specific principals, DAO governance canister, etc.). | Enterprise governance |
| **Module hash pipeline** | A mechanism to compute SHA-256 of the final deployed WASM bytes and pass the hash to the canister at init or upgrade time. See [Module Hash: Deployment Patterns](module-hash-pipeline.md). | Enterprise dev team |

### Compile-time dependency model

All Zombie Delete ICP libraries are compile-time Rust crate dependencies. The library code runs inside the canister's own execution context — there is no external service, no inter-canister call, and no network dependency at runtime. ICP guarantees single-threaded message execution per canister, so all library operations within a single update call are atomic.

### Candid maintenance

After adding library endpoints, regenerate and commit the canister's `.did` file. The Candid interface must match the compiled canister exactly — deployment will fail if they diverge. Include `.did` regeneration in the build pipeline or CI checks.
