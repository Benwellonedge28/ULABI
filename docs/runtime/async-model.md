# ULABI Asynchronous Operations

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Runtime boundary](runtime-interface.md), [Concurrency](concurrency.md), [Memory lifetimes](../memory/lifetimes.md)

## Scope

Define job lifecycle, completion and cancellation for an interface that explicitly supports asynchronous calls.

## Contract

- Declare accepted completion style, cancellation semantics, inputs kept after submission, result ownership, and retry behavior. Use a generation-scoped operation token. Return accepted only after the provider has taken any declared ownership or reservation.
- Use pending and running states and one terminal outcome: completed, failed, or cancelled. Cancellation is a request until the terminal race is resolved. It grants rollback only where the operation explicitly supplies it.
- A stream declares element type, direction, ownership, credit or bounded buffering, and end, failure and cancellation behavior. A producer stays within credit and releases queued elements according to ownership on termination.

## Implementation sequence

1. Validate inputs and reserve resources at submission. Copy retained data or obtain a valid long-lived handle before a call-scoped borrow ends.
2. Complete through one state transition, publish the result and release resources once. Define poll, wait or callback behavior after completion and during shutdown.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Cancel and success race | Exactly one terminal outcome is published and resources release once. |
| Consumer supplies a call-scoped borrow for retained input | Reject or make an authorized copy before returning. |
| Slow streaming consumer exhausts credit | Pause admission without exceeding the declared bound. |
