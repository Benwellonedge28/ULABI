# ULABI Concurrency and Reentrancy

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Threading](threading.md), [Async model](async-model.md), [Shared memory](../memory/shared-memory.md)

## Scope

Define what simultaneous calls can observe across an interface and how adapters preserve those guarantees.

## Contract

- Declare permitted parallel calls per operation and per instance, reentrancy, and which state changes are ordered. If a contract promises an atomic transition, identify the observable completion point.
- Document visibility and synchronization for shared data and require a declared memory model at shared-memory boundaries. A status of success implies only the visibility promised by the operation.
- Keep callbacks outside locks that would prevent permitted reentry. For a disallowed reentrant operation, reject before acquiring a conflicting lock. Use bounded queues for serialized work.

## Implementation sequence

1. Classify each operation as independent, serialized, or externally synchronized and publish the required policy in metadata.
2. Implement admission and synchronization around the shared state. Use controlled schedules to check competing calls, cancellation and callback reentry.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two callers race a single-winner update | Exactly one observes acceptance under the declared ordering. |
| Permitted callback re-enters its host | Complete within the documented order without lock deadlock. |
| Consumer omits required external synchronization | Treat it as a caller contract violation. |
