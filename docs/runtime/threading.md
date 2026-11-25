# ULABI Threading and Thread Affinity

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Concurrency](concurrency.md), [Async model](async-model.md), [Stack model](../abi/stack-model.md)

## Scope

Define the scheduling facts visible to an ABI caller without imposing a host threading implementation.

## Contract

- Declare whether the operation is thread-safe, instance-confined, or bound to an executor. State callback thread and affinity obligations, including when a callback can re-enter its caller.
- Treat thread-local state and thread-owned handles as nontransferable unless the contract defines transfer. Dispatch affinity-bound work on the admitted executor, with documented failure when no suitable executor is available.
- Make detach and shutdown release thread-owned host registrations. An executor switch must respect declared blocking rules, ownership, and cancellation semantics.

## Implementation sequence

1. Put thread policy in operation metadata. Validate it during binding and check or dispatch at invocation before entering the foreign operation.
2. Track an executor or thread token for affinity-bound handles. Exercise callbacks and teardown using the same production dispatch path.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Wrong executor invokes a confined object | Reject or dispatch using the declared policy before access. |
| Callback returns on an undeclared thread | Fail the scheduling validation. |
| Thread exits with local registrations | Invalidate registrations and release their resources. |
