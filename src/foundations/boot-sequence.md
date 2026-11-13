# The Boot Sequence

Boot establishes authoritative state and credentials before serving ready capsule views. This chapter describes the major dependency boundaries rather than a fixed list of line-numbered constructor steps.

## Open state under exclusive ownership

The runtime's singleton lock protects opening the authoritative principal store. Kernel construction supplies the principal directory and quota resolver, then calls `open_runtime_principal_store_with_directory`. Initial cutover imports and verifies legacy SurrealKV state before serving. KV and audit adapters come from the opened native store.

An open or recovery error prevents successful construction. Recoverable tail conditions are distinguished from corruption in headers or committed interior structures. Do not repair a running installation by creating an unrelated empty database and treating it as the same state.

## Establish signing and audit identity

The runtime loads or creates its persisted runtime key. The audit projection is opened from system-control state, then audit format and key-registry rules are applied. The v2 path has a separate audit key; an existing registry prevents silent reversion to v1 or arbitrary replacement of a missing registered key.

Invalid registry or key state can fail startup. This differs from management audit append failures after startup, which use continue-and-alert behavior. See [The Cryptographic Audit Chain](../storage/audit-chain.md).

## Establish connection credentials

The secure Unix socket is bound while the singleton lock is held. Binding is distinct from listening and serving. The session token is generated before a capsule can accept connections. A best-effort PID file supports stop/restart recovery, but its existence is not the lock itself.

Connection authentication and caller stamping remain necessary after transport setup. An accepted connection does not imply permission for every management operation.

## Construct services and load views

Runtime resources compose the bus, state, audit, signing, and capsule execution mechanisms. Capsule loading establishes the required boot views first. Other principal profiles can warm in the background, while the full-load compatibility path remains a distinct entry point.

A readiness claim must refer to the required view and route availability. Merely finding a capsule artifact or beginning initialization does not establish readiness. Product distributions can select different members and pin different runtime revisions.

## Replacement and removal

Restart prepares a replacement with route admission closed. It waits for readiness and checks the expected runtime generation before publication. A superseded restart must not replace a newer runtime. Preparation failure leaves the existing running view available.

Removal takes the keyed lifecycle fence, unregisters the view, and quiesces it. Outstanding references can retain a cancelled runtime after logical removal. Treat route closure, task cancellation, resource release, and durable state retirement as separate events.

## Operational interpretation

Persistent daemons do not have an idle-shutdown policy merely because there are no current clients. Readiness, liveness, and absence of clients answer different questions. Shutdown and recovery must respect active storage ownership and lifecycle fences.

For the release-specific user commands and platform contract, see [Operating the 2026.9 Runtime](../operating-2026-9.md). For a product such as AOS, use the runtime revision recorded by that distribution when reproducing startup behavior.

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-kernel/src/lib.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/lib.rs)
- [crates/astrid-kernel/src/audit_keys.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/audit_keys.rs)
- [crates/astrid-storage/src/principal_state/store_open.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/principal_state/store_open.rs)
- [crates/astrid-storage/src/volume/hosted/open.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/volume/hosted/open.rs)
- [crates/astrid-kernel/src/capsule_restart.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/capsule_restart.rs)
- [crates/astrid-kernel/src/capsule_removal.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/capsule_removal.rs)
