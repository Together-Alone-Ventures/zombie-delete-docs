<!-- shared:icp/tombstone-patterns.md -->
<!-- Composable section: tombstone guard installation and anti-resurrection semantics. -->
<!-- Applies to all Zombie Delete products using the Tombstone (Explicit) erasure model. -->
<!-- Products using different erasure models (e.g., ZKP-based) skip this section. -->

## Tombstone Protection

### Guard installation

Every update method that modifies PII fields must reject writes when the canister is tombstoned. Three approaches are available; choose based on your crate structure:

#### Approach A: Guard function (recommended when error type is in a separate crate)

```rust
fn mktd_guard_check() -> Result<(), MyError> {
    if !mktd02::is_initialised() { return Err(MyError::NotReady); }
    if mktd02::is_tombstoned() { return Err(MyError::Deleted); }
    Ok(())
}

#[ic_cdk::update]
fn upsert_profile(input: ProfileInput) -> Result<ProfileInfo, MyError> {
    mktd_guard_check()?;
    // ... existing business logic unchanged ...
}
```

#### Approach B: `#[mktd_guard]` macro (when error type implements `GuardError`)

```rust
use mktd02_macros::mktd_guard;

#[mktd_guard]
#[ic_cdk::update]
fn upsert_profile(input: ProfileInput) -> Result<ProfileInfo, MyError> {
    // ... existing business logic unchanged ...
}
```

The macro requires the function to return `Result<T, E>` where `E: mktd02::GuardError`. It parses the return type at compile time and emits a `compile_error!` if the signature doesn't match. If the error type is defined in a separate crate (e.g., a shared library), Rust's orphan rules prevent implementing `GuardError`; use Approach A instead.

#### Approach C: `assert_can_write()` (for non-Result functions)

```rust
mktd02::assert_can_write(); // Traps if tombstoned or uninitialised
```

### Guard ordering

When the canister has both access control (e.g., `require_owner()`) and tombstone protection, **access control should fire first.** This ensures unauthenticated callers are rejected before the tombstone check, and tombstone violations are only reported to authorised callers:

```rust
require_owner()?;       // auth first
mktd_guard_check()?;    // then tombstone
```

### Guard coverage checklist

All of the following must be guarded:

- All Candid-exposed update methods that insert/update/remove PII data
- Internal helpers invoked by update methods (batch updates, maintenance)
- Admin/controller-only methods that touch PII
- Upgrade hooks and migration logic that modify PII fields
- Timers/heartbeat-driven routines that may mutate PII state

### Tombstone semantics

The Zombie Delete library uses Explicit tombstone-type with two distinct checks:

1. **`mktd02::is_tombstoned()`** reads the engine-owned `tombstoned_at` timestamp from stable memory. This is the **write guard**, used to prevent PII writes after deletion. It is unambiguous and does not depend on inspecting field values.

2. **`adapter.is_tombstoned()`** checks whether all PII fields contain the tombstone constant. This is the **post-condition check** used during the deletion flow to verify `tombstone_state()` completed correctly.

These must not be confused. The engine sets `tombstoned_at` later in the deletion flow, so at post-condition check time, only the adapter's PII-field check is meaningful.

This is procedural enforcement, not platform-level prevention. A canister controller who upgrades the code to remove the guard can bypass it. This is documented in the Residual Trust Statement (RT2) and is detectable via module hash verification (V3).

### Anti-resurrection semantics

Tombstoning is permanent for the identity (principal) that owns the canister. The same principal cannot re-register or have data re-written after deletion — the tombstone guard enforces this for every PII-mutating write path.

If the enterprise's UX allows re-registration, it must be via a **new identity** (new principal) mapped to a **new canister**. Any "rejoin" flow that unmaps and remaps the same principal to a fresh canister would defeat the purpose of the CVDR, since the original receipt attests to permanent deletion for that identity.
