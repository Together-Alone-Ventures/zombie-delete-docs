<!-- shared:icp/authentication-patterns.md -->
<!-- Composable section: ICP authentication patterns for any Zombie Delete product. -->
<!-- This section is typically inserted after the product's API endpoints and guard setup. -->

## Authentication Patterns (ICP)

Zombie Delete libraries do not implement or prescribe an authentication mechanism. On ICP, all calls carry the caller's principal as an intrinsic property of the message, verified by consensus. The canister's own access control checks this principal. The library defers entirely to the host canister's existing auth pattern.

| Pattern | How it works | Typical use case |
|---|---|---|
| **Internet Identity** | User authenticates via WebAuthn passkey, receives a unique principal per dApp. | Consumer-facing applications (e.g., OpenChat). |
| **NFID / alternative IdP** | Third-party identity provider issues delegated principals. | dApps wanting social login. |
| **Canister-to-canister** | Governance canister (e.g., SNS) calls deletion with its own principal. | DAO-governed deletion decisions. |
| **Controller principal** | The canister's controller calls directly. | Admin-triggered deletions. |

Access control on the deletion endpoint is critical and must match the canister's governance model. The Zombie Delete library does not enforce access control — the canister's existing auth pattern (e.g., `require_owner()`) applies. The product-specific integration guide specifies which endpoints require restricted access.
