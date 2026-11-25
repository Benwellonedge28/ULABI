# ULABI Runtime Resource Management

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Runtime boundary](runtime-interface.md), [Resource isolation](../security/sandboxing.md), [Memory allocation](../memory/allocation.md)

## Scope

Define admission, accounting, and release of host resources used across a component boundary.

## Contract

- Declare budgets per component and, where applicable, per operation for bytes, threads, handles, time and work queues. Define units, scope, enforcement points and behavior on exhaustion.
- Reserve capacity before the operation that consumes it. Track reserved and used amounts with checked arithmetic. If a child budget is created from a parent allocation, reconcile both scopes on commit or release.
- Return the documented resource-limit status, preserving caller ownership and committed state. Release reservations on completion, cancellation, failed initialization or teardown according to the operation contract.

## Implementation sequence

1. Read host policy and profile limits at admission. Attach an operation identifier and a reservation to queued work before invoking it.
2. Account resource changes through the host service boundary. Use a bounded supervisor sweep for orphaned instance reservations and expose aggregate counters.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two admissions contend for one handle | Admit one and return a resource-limit result to the other. |
| Operation fails after partial reservation | Release every acquired reservation exactly once. |
| Limit cannot be enforced by the host | Refuse a profile that requires that enforcement. |
