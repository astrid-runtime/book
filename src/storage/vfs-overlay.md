# Workspace Copy-on-Write and the VFS Overlay

A canonical Astrid workspace is an owner-internal branch of durable content state. It carries a base catalog root and a current catalog root. It is not itself a host directory. Unchanged content can be shared while modifications create new content and update the branch view.

## Content representation

Files become content-defined chunks and trees. The default FastCDC 2020 profile uses a 16 KiB minimum, 64 KiB target, and 256 KiB maximum chunk size, with tree fanout 128. These parameters describe chunking, not a guaranteed deduplication ratio.

Object identities cover canonical content and structure. The owning state root selects what is authoritative. A hash identifies an object but does not authorize reading it or provide encryption.

## Branches and effects

A workspace binding associates an invocation's view with the appropriate durable branch. The runtime composes `WorkspaceBranchService` and `AstridStorageVfs` for this canonical path. Reads and writes use the branch's current state; promotion and rollback are explicit operations with their own authority and concurrency conditions.

A rollback can discard branch-local state. It cannot undo an external network effect or arbitrary host process action. Applications must distinguish effects inside the workspace from effects outside its transaction boundary.

## Projections and compatibility

A materialized host view is a projection of state, not the definition of ownership. Host-backed VFS paths and git or platform copy-on-write compatibility implementations remain separate from the durable graph branch mechanism. Which implementation an invocation receives determines its actual behavior.

Direct host writes do not acquire rollback semantics merely because they pass through a VFS interface. Likewise, a git checkpoint is not automatically the authoritative native owner root. Document the selected implementation when describing recovery or isolation.

## Concurrency and lifecycle

Branch roots refer to immutable state; publication must obey the expected-root and owner rules of the storage service. Keeping a snapshot or another branch can retain objects after they disappear from the current visible tree. Compaction must preserve retained roots before reclaiming their backing objects.

A cancelled capsule can still have outstanding references during teardown. Logical removal, stopping further admission, and physical reclamation are separate lifecycle events. Do not delete backing state simply because a user-facing view has disappeared.

## Security boundary

Filesystem operations remain subject to caller authority and the relevant host/VFS checks. Path normalization, resource matching, and owner resolution are enforcement responsibilities, while copy-on-write provides state separation. Neither replaces the other.

See [KV Storage](kv.md) for shared roots, quota and commit behavior, and [Security Enforcement Boundaries](../security/five-layer-gate.md) for host-effect authority.

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-storage/src/content/store/workspace/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/content/store/workspace/mod.rs)
- [crates/astrid-storage/src/content/store/workspace/model.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/content/store/workspace/model.rs)
- [crates/astrid-storage/src/content_dag/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/content_dag/mod.rs)
- [crates/astrid-capsule/src/engine/wasm/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/engine/wasm/mod.rs)
- [crates/astrid-kernel/src/capsule_removal.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/capsule_removal.rs)
