# Policy, Budget, Approval, and Audit

Policy selects permitted behavior; the runtime enforces concrete authority at operation boundaries. Approval records a decision. Audit records evidence. These responsibilities interact, but they are not one mandatory pipeline for all activity.

## Policy and approval

Business decisions belong in capsules. The host provides mechanisms such as caller identity, manifest checks, scoped grants, and key custody. A receiving capsule must correlate an approval with the requested operation and caller; a response in an arbitrary payload is not authenticated authority.

Management operations use resolved principal/group grants and authenticated device scope. Signed resource tokens carry a separate grant format. Neither a valid identity nor a tool description grants unrestricted execution. Consult [Capabilities, Tokens, and Delegation](capabilities-and-tokens.md) for the distinction.

## Three different resource controls

- Execution fuel, memory limits, and epoch interruption constrain WASM guest execution.
- Storage quotas constrain owner-accounted state at mutation time. Quota resolution errors reject mutations rather than bypassing the ceiling.
- The approval library provides monetary budget accounting and atomic reservations for callers that integrate it.

These controls are not interchangeable. Fuel consumption is not a charge in currency. A storage ceiling is not a model-spend ceiling. The presence of `BudgetTracker` and `check_and_reserve` in a library does not establish that all tool or inference paths call it. The inspected kernel, capsule, and MCP paths do not establish universal monetary enforcement.

## Integrating a spending policy

A composition that promises a spending limit must identify every effecting call site, reserve before the effect, and define settlement and release behavior for success, failure, cancellation, and restart. It must also define concurrent admission and idempotency. These are integration requirements; do not infer them from a budget type alone.

The approval budget implementation provides atomic reservation primitives. Its guarantees apply to the operations that actually use those primitives. Review the concrete product and release rather than assuming a runtime library supplies a product-wide payment policy.

## Audit semantics

Host effects have a separate ordered audit lane. For classes selected by `audit.host_fail_closed`, the host waits for a durable write-ahead admission record before executing the effect; admission failure refuses the operation. Supported classes are `file_read`, `file_write`, `file_delete`, `net_connect`, `net_bind`, and `process_spawn`. The default selection is empty. Other recording uses asynchronous queues with retry and loss/gap accounting. A durable admission record proves admission, not eventual completion of the effect.

The kernel management audit helper attempts durable append and emits a live event. If append fails, it logs a security warning and continues. Event publication is also best-effort. Consumers must distinguish live notifications from persisted signed evidence.

The IPC audit helper emits tracing events. Tracing is useful for operations and diagnosis, but it does not automatically create a signed chain entry. Conversely, malformed audit key or registry state at startup can prevent opening the audit log. Failure behavior is therefore specific to the path and stage.

## What a claim must identify

For each claimed guarantee, specify the caller identity, protected operation, enforcement function, durable state involved, and behavior when that state cannot be read or written. If the claim includes audit completeness or a monetary ceiling, test the corresponding failure and concurrency paths in the product composition.

This is also the boundary for formal claims: a proof of an accounting transition does not establish that every external effect is routed through that transition.

## Implementation references

This chapter describes runtime source at `f519aaa28b305491bb766e083ccd5b8b0aff5e93`; a product distribution may pin a different runtime revision.

- [crates/astrid-approval/README.md](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-approval/README.md)
- [crates/astrid-approval/src/budget.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-approval/src/budget.rs)
- [crates/astrid-kernel/src/kernel_router/mod.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/kernel_router/mod.rs)
- [crates/astrid-capsule/src/engine/wasm/host/ipc.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-capsule/src/engine/wasm/host/ipc.rs)
- [crates/astrid-storage/src/kv/principal.rs](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-storage/src/kv/principal.rs)

- [Host audit admission and recording](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-kernel/src/audit_sink/mod.rs)
- [Audit configuration and defaults](https://github.com/astrid-runtime/astrid/blob/f519aaa28b305491bb766e083ccd5b8b0aff5e93/crates/astrid-config/src/audit.rs)
