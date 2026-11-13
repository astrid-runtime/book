# Security Enforcement Boundaries

Astrid enforces authority at the boundary where an operation becomes possible. Capsule execution, IPC, host effects, management requests, package installation, and durable state each have their own checks. There is no universal interceptor through which every sensitive action passes.

## Execution boundary

The canonical Wasmtime component linker provides Astrid host interfaces and does not register WASI. A capsule cannot obtain an unprovided import merely by declaring it. Fuel, epoch interruption, and memory limits constrain guest execution. A host import is trusted code and must enforce its own contract; the WASM sandbox does not validate a host implementation for it.

## Caller identity

Identity belongs to an invocation or message. The local connection handshake verifies a claimed principal using a nonce challenge against registered keys. The host stamps the message envelope with caller information. `publish_as` derives its principal from host-held connection state rather than accepting a guest's asserted name; an unbound connection is anonymous.

Principal, device, and origin metadata survive forwarding. Capsule authors must read that metadata rather than treating a principal string in a payload as authentication. A capsule's installation identity is not the authority of every caller it serves.

## IPC admission

Guest publication checks active invocation authority, topic validity, and manifest publish permissions. An empty publish allowlist denies publication. Topic text is limited to 256 bytes and eight segments at this host boundary. The route matcher used for IPC permissions supports trailing-star subtree matching. Admission is bounded, and authority is checked again after admission before publication.

Subscription permissions are also checked. Permission to publish a request does not by itself authorize the requested effect: the receiving service and its host operations still have their own authority boundaries. See [Topics and Wildcards](../bus/topics-and-wildcards.md) for matcher-specific behavior.

## Host effects

The manifest gate evaluates filesystem resources, process commands, and network endpoints for the relevant host calls. A model choosing a tool is a request, not a grant. The host must enforce the operation's actual resource and caller constraints.

Durable storage resolves namespaces to explicit state owners at an authority-aware integration boundary. It does not infer authority by parsing a guest-chosen namespace. Quota resolution and root publication belong to the storage mutation path.

## Management authority

Kernel management requests resolve a principal profile and group grants, reject disabled or unresolved callers, and apply authenticated device scope. An authenticated connection can still be denied an administrative operation. These checks are distinct from signed resource capability tokens.

Package provenance is another boundary: a signature establishes who signed bytes, while local install authority decides whether those bytes may be admitted. Foreign or unsigned artifacts require an applicable decision bound to the artifact digest.

## Evidence and failure behavior

Host effects have a separate ordered audit lane. For classes selected by `audit.host_fail_closed`, the host waits for a durable write-ahead admission record before executing the effect; admission failure refuses the operation. Supported classes are `file_read`, `file_write`, `file_delete`, `net_connect`, `net_bind`, and `process_spawn`. The default selection is empty. Other recording uses asynchronous queues with retry and loss/gap accounting. A durable admission record proves admission, not eventual completion of the effect.

Audit append and audit coverage are different properties. Signed entries can authenticate the history that was recorded. Management audit persistence uses continue-and-alert behavior on failure, so a successful management effect need not have a durable audit entry. IPC tracing is telemetry, not automatically a signed entry. Audit key/registry configuration errors during boot can instead prevent startup.

Do not describe Astrid as universally fail-closed on audit failure or universally gated by monetary budgets. State the operation, enforcement point, and failure behavior. See [Policy, Budget, Approval, and Audit](policy-budget-approval-audit.md) and [The Cryptographic Audit Chain](../storage/audit-chain.md).

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-integration-tests/tests/canonical_security_model.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-integration-tests/tests/canonical_security_model.rs)
- [crates/astrid-capsule/src/engine/wasm/host/ipc.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/engine/wasm/host/ipc.rs)
- [crates/astrid-capsule/src/engine/wasm/host/net/handshake.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/engine/wasm/host/net/handshake.rs)
- [crates/astrid-capsule/src/security/manifest_gate.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/security/manifest_gate.rs)
- [crates/astrid-kernel/src/kernel_router/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/kernel_router/mod.rs)
- [crates/astrid-kernel/src/audit_keys.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/audit_keys.rs)
- [crates/astrid-capsule-install/src/authority.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule-install/src/authority.rs)

- [Host audit admission and recording](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/audit_sink/mod.rs)
- [Audit configuration and defaults](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-config/src/audit.rs)
