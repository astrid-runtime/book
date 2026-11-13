# The Kernel Is Dumb

Astrid's kernel supplies execution, communication, authority, and state mechanisms. Agent reasoning, prompt construction, model-provider behavior, and product-specific approval policy belong in capsules. A model can request an operation; it cannot grant itself the host authority to perform it.

## What the kernel owns

The runtime composes the event bus, capsule registry, component execution engine, caller identities, capability checks, state services, audit facilities, and lifecycle coordination. These are mechanisms for hosting software. They do not make the kernel an agent loop or a tool broker.

KV and named content use the native durable principal store. Scoped views resolve to explicit owners, with quotas and root publication enforced in the storage path. The audit adapter uses system-control state. See [KV Storage](../storage/kv.md).

## Communication and authority

Capsules communicate through versioned interfaces and IPC topics. Host-controlled envelope metadata carries caller context through forwarding. The event bus provides both broadcast consumers and routed subscriptions; queue admission and per-principal scheduling are transport mechanisms.

Guest publish and subscribe operations have manifest permission checks. Management requests additionally resolve principal/group authority and authenticated device scope. Effects such as filesystem access, network access, and process execution have their own host-boundary checks. The management ACL is therefore one part of enforcement, not the whole security model.

The kernel need not understand a tool's business meaning to reject an unauthorized host operation. Conversely, routing a request successfully is not proof that the requested effect is authorized. The receiver and its host calls must enforce the relevant contract.

## Storage and workspace views

Canonical workspace branches are owner-internal durable graph views. Host filesystem and overlay compatibility paths are distinct implementations. Copy-on-write, owner isolation, and resource authorization are different properties and must be established at their respective boundaries.

## Lifecycle

Boot opens authoritative state, initializes key and audit configuration, establishes connection credentials, and loads the required capsule views. Other profiles can warm in the background. A package on disk is not the same as a ready capsule with published routes.

Restart prepares a replacement before publishing it and checks the expected runtime generation. Removal fences lifecycle changes and quiesces routes; outstanding references can delay physical reclamation. These mechanisms protect runtime identity and admission without interpreting application behavior.

## Evidence

Audit and tracing provide different forms of evidence. Management audit persistence can fail while the operation continues with a warning; a signed chain authenticates its recorded history rather than proving complete coverage. Monetary budget primitives also require explicit caller integration. See [Security Enforcement Boundaries](../security/five-layer-gate.md).

The boundary to preserve when adding a feature is concrete: host mechanisms route, enforce, account, and maintain state; capsules decide application behavior.

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-kernel/src/lib.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/lib.rs)
- [crates/astrid-kernel/src/kernel_router/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/kernel_router/mod.rs)
- [crates/astrid-capsule/src/engine/wasm/host/ipc.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/engine/wasm/host/ipc.rs)
- [crates/astrid-kernel/src/capsule_restart.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/capsule_restart.rs)
- [crates/astrid-kernel/src/capsule_removal.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/capsule_removal.rs)
