# KV Storage

The runtime opens a native durable principal store and obtains KV and audit adapters from it. KV is a projection over owner-scoped immutable state, sharing the object arena and root publication mechanism with named content. SurrealKV remains a legacy migration reader and differential-test oracle; it is not the current native runtime backend.

## Ownership before names

`StateOwner` distinguishes system, principal, and fleet state. The integration boundary supplies an authority-aware namespace resolver. Storage must not create ownership by splitting an arbitrary namespace string. A capsule's scoped KV view is a convenience over this authority mapping, not permission to choose any principal.

The runtime opens the store under its singleton lock. Initial cutover imports and verifies legacy state before serving. Opening another unrelated database path is not a way to access the current runtime's authoritative state.

## Immutable state and publication

`TreeKvStore` represents state using immutable B+-tree checkpoints and compact transitions. Point changes append transitions; checkpointing folds them into tree pages while preserving logical contents. The expected owner root identifies the state against which a mutation was prepared.

Root compare-and-swap publishes a mutation only against its expected predecessor. On a root conflict, the implementation retries using a fresh view. The root and quota checks prevent concurrent updates from simply overwriting a stale view. They are not a promise of arbitrary transactions across owners.

## Compare-and-swap

Use the KV CAS operation for conditional updates. Create-if-absent uses an absent expected value. A mismatch means another value or state won; re-read and decide whether to retry. A storage error means the operation failed. Never turn an error into an unconditional set: that would discard the condition that made the operation safe.

The guest WIT/SDK determines the surfaced error or boolean form. Check the SDK version used by the capsule. The host backend's root conflict handling and a guest's value-level CAS condition are related but distinct mechanisms.

## Batches

A batch resolves all namespaces before mutation and rejects mixed owners. Conditions and mutations operate against one captured header. Validation limits the combined number of conditions and mutations to 1,024 and the batch payload to 64 MiB.

Same-owner atomicity does not imply atomicity with an external network request, payment, host process, or a second owner's state. Such compositions need their own recovery and idempotency protocol.

## Quotas and content identity

Quota policy is injected and evaluated at mutation time. Accounting can include principal-controlled metadata as well as value bytes, so empty values do not imply free unlimited state. Failure to resolve quota rejects the mutation.

Object identity commits to kind, format, canonical bytes, logical size, object class, and labelled references. Content addressing establishes identity and sharing; it neither grants access nor encrypts values.

## Persistence and recovery

Durable group commit persists the group, publishes the commit frontier, and then publishes the durable root. Error paths can require recovery before further mutation. Hosted storage distinguishes recoverable tail conditions from header or interior corruption. Do not treat the existence of bytes in a volume as proof that they are committed state.

Recovery reconstructs authoritative state from its persisted structures and committed frontier. Retention and compaction must preserve roots still needed by live state, snapshots, or recovery. Physical space reclamation is not the same operation as deleting a logical key.

## Secrets

The default `build_secret_store` uses KV even when the crate is compiled with the keychain feature. It supplies no OS-keychain encryption guarantee. The explicit keychain constructor probes at startup and selects either keychain or KV for the lifetime of the store; it logs a warning when it falls back to KV.

A deployment requiring encrypted or hardware-backed secret custody must establish that separately. Signed audit entries and content hashes are not confidentiality controls.

## Backend scope

Memory stores and compatibility adapters remain useful for tests and explicit legacy callers. Distributed multi-node placement is separate implementation work, not a switch to a second production backend. See [Workspace Copy-on-Write](vfs-overlay.md) for the filesystem projection and [Audit Chain](audit-chain.md) for signed evidence.

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-kernel/src/lib.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/lib.rs)
- [crates/astrid-storage/src/lib.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/lib.rs)
- [crates/astrid-storage/src/principal_state/store_open.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/principal_state/store_open.rs)
- [crates/astrid-storage/src/kv/principal.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/kv/principal.rs)
- [crates/astrid-storage/src/kv/tree.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/kv/tree.rs)
- [crates/astrid-storage/src/kv/tree/adapter.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/kv/tree/adapter.rs)
- [crates/astrid-storage/src/kv/batch.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/kv/batch.rs)
- [crates/astrid-storage/src/engine/durable/group/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/engine/durable/group/mod.rs)
- [crates/astrid-storage/src/secret.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/secret.rs)
