# ULABI Rollback and Safe Failure

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Recovery](recovery.md), [Runtime lifecycle](../runtime/runtime-interface.md), [Versioning](../../ULABI-VERSIONING.md)

## Scope

Define restoration of a verifiable component state after a failed change or recovery action.

## Contract

- Record a rollback point before the supported mutation, including artifact, compatible data schema, checkpoint identity and independently verified integrity. List irreversible effects and any declared compensation.
- Block conflicting admissions during restoration. Restore the last valid state only where the policy and data contract allow it. If restoration cannot be verified, enter a safe failure state.
- Never restore revoked access or erase audit history. Resource ownership and capability grants are reconciled under current policy. Transactions use the operation’s prepare, execute, commit and rollback rules.

## Implementation sequence

1. Validate the planned rollback inputs against the release and current schema. Fence the active generation, restore, and run the same readiness and security checks required for recovery.
2. Reconcile external effects separately. Record incomplete compensation or uncertain outcomes and escalate under the policy.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Checkpoint integrity fails | Enter safe failure and report the invalid rollback point. |
| An older snapshot references revoked grants | Do not reissue them. |
| Committed external effect has no compensation contract | Report it for reconciliation. |
