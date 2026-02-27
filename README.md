# Zombie Delete Documentation

Shared integration documentation for the [Zombie Delete](https://github.com/Together-Alone-Ventures) product suite by Together Alone Ventures.

This repository contains **platform-level** content that is reused across multiple product integration guides. Product-specific guides live in their respective repos and pull from these shared sections at build time.

## Structure

```
icp/                          ← Internet Computer Protocol platform docs
├── prerequisites-platform.md     Platform prerequisites (Rust, dfx, ic-cdk, cycles, etc.)
├── module-hash-pipeline.md       Module hash deployment patterns & byte-identity warnings
├── authentication-patterns.md    ICP authentication patterns (II, NFID, C2C, controller)
├── tombstone-patterns.md         Tombstone guard installation & anti-resurrection semantics
├── deterministic-encoding.md     Deterministic CBOR checklist & State Encoding Spec
└── assumptions-platform.md       Platform-level assumptions shared across ICP products
```

Future platform families (e.g., `tee/` for TEE-based products, `db/` for standard database products) will follow the same pattern.

## How These Docs Are Used

Each product repo (MKTd02, MKTd03, etc.) contains a `docs/compose.yaml` manifest that defines which shared sections to include and in what order. A compose script fetches these files and stitches them together with the product-specific sections into a single, complete integration guide. Clients download the composed guide from the product repo's Releases page — they never need to visit this repo directly.

See any product repo's `docs/scripts/compose.py` for the composition tooling.

## Versioning

Shared docs are tagged alongside product releases. Product manifests should pin `shared_ref` to a tag for release builds and may track `main` for development.

## Contributing

Edits to shared content affect all products that reference it. Please coordinate with the product maintainers before merging changes to this repo.

## Licence

Apache-2.0
