# ULABI Deterministic Boundary Debugging

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Debugger interface](../tooling/debugger-interface.md), [Tracing](tracing.md), [Concurrency](../runtime/concurrency.md)

## Scope

Define when a bounded replay of boundary interactions can reproduce observations.

## Contract

- Capture artifact and mapping identities, operation order, declared nondeterministic inputs, observed outcomes and permitted test state. Record an explicit missing-input marker when policy omits replay-critical information.
- Replay only under an authorized test context with isolated or substituted external services. Restore a verified snapshot before applying recorded inputs. Do not reissue external side effects as part of replay.
- Compare semantic outcomes and order under the declared deterministic scope. Record first divergence and mark unrecorded or inherently nondeterministic activity as outside the replay guarantee.

## Implementation sequence

1. Choose an operation and record a bounded event sequence. Pin the runtime and build information and sanitize allowed data.
2. Load the matching isolated environment, inject recorded inputs and scheduling points where modeled, then compare resulting events and states.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| A required time or randomness input was omitted | Mark replay incomplete. |
| Replay would reach a live external service | Use an approved substitute or stop. |
| Matching snapshots yield a different boundary status | Report the first divergence with operation and event IDs. |
