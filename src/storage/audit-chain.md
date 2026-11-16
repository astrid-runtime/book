# The Cryptographic Audit Chain

Astrid's audit log signs recorded entries and links them into histories. This supports verification of the evidence that was recorded. It does not establish that every runtime effect appears in the log, or that the supplied history is complete relative to an independently known latest head.

## Storage and append

The kernel obtains the audit projection from the native principal store's system-control state. Signed append serializes each session/principal chain, determines its predecessor, and persists using expected-head comparison.

The append path invalidates cached predecessor state before awaiting durable storage. Cancellation therefore cannot leave that cached predecessor treated as authoritative for the next append. A failed comparison or storage operation must be handled as an append failure, not as proof that an entry was committed.

## Keys and formats

Native boot loads audit policy once from the captured runtime home and selected workspace after state admission. It uses that snapshot for entry format, retention, and host-call admission. Invalid merged configuration prevents startup; neither default settings nor a workspace-free fallback replace the selected policy. An existing v2 key registry does not bypass that configuration requirement.

The runtime persists its signing material; it does not generate an unrelated ephemeral audit identity on every normal restart. The v1 path uses the runtime key. The v2 path separates `audit.key` from `runtime.key`, and a registry binds key roles and supports cross-signed rotation.

Once a v2 registry exists, opening retains v2 behavior even if configuration asks for v1. Invalid registry state, a missing required key, or mismatched key identity can prevent startup. Key custody in the inspected filesystem implementation uses owner-only Unix files and durable publication; this is not a hardware-backed custody claim.

## Verification boundary

A verifier must establish the trusted key or registry, the entry encoding and format, signature validity, and predecessor relationships. Trusting a public key supplied by an untrusted history is not independent authentication.

A valid chain can still be an older valid prefix. Detecting truncation or rollback requires an independently trusted expected head or checkpoint. Signature verification also does not prove that the recorded outcome matches every external effect; that depends on where records are produced and what was observed.

## Coverage and failure

Host effects have a separate ordered audit lane. For classes selected by `audit.host_fail_closed`, the host waits for a durable write-ahead admission record before executing the effect; admission failure refuses the operation. Supported classes are `file_read`, `file_write`, `file_delete`, `net_connect`, `net_bind`, and `process_spawn`. The default selection is empty. Other recording uses asynchronous queues with retry and loss/gap accounting. A durable admission record proves admission, not eventual completion of the effect.

Management audit attempts durable append and live publication. Persistence failure logs a warning and permits the management request to continue. The live event is best-effort. IPC tracing is another evidence surface and is not automatically part of the signed chain.

These choices mean a successful management action can lack its intended durable entry. A consumer requiring complete evidence must account for this behavior explicitly. Do not infer universal fail-closed behavior from cryptographic chain integrity.

## Retention and use

Retained histories consume storage and participate in the state lifecycle. A valid compaction process must preserve required logical state and retained evidence. A deterministic compaction proof adapter is not a proof of the entire storage engine or of all audit producers.

For research, keep three claims separate: record authenticity, chain consistency, and coverage of effects. State which one an experiment or proof actually checks. See [Policy, Budget, Approval, and Audit](../security/policy-budget-approval-audit.md).

## Implementation references

This chapter describes runtime source at `ac0e876515764ad85c9b647277faa47e96afb583`; a product distribution may pin a different runtime revision.

- [crates/astrid-audit/src/log.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-audit/src/log.rs)
- [crates/astrid-kernel/src/audit_keys.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/audit_keys.rs)
- [crates/astrid-kernel/src/lib.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/lib.rs)
- [crates/astrid-kernel/src/kernel_router/mod.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/kernel_router/mod.rs)
- [crates/astrid-crypto/src/key_storage.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-crypto/src/key_storage.rs)
- [crates/astrid-storage/src/engine/durable/compaction/proof.rs](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-storage/src/engine/durable/compaction/proof.rs)

- [Host audit admission and recording](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/audit_sink/mod.rs)
- [Audit configuration and defaults](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-config/src/audit.rs)

- [Captured-home audit policy loader](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/audit_retention.rs#L18-L44)
- [One policy snapshot for native boot](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/lib.rs#L1273-L1291)
- [Retention and host-admission policy application](https://github.com/astrid-runtime/astrid/blob/ac0e876515764ad85c9b647277faa47e96afb583/crates/astrid-kernel/src/lib.rs#L1368-L1372)
