# ULABI Process and Isolation Model

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Runtime boundary](runtime-interface.md), [Sandboxing](../security/sandboxing.md), [Remote calls](../distributed/remote-calls.md)

## Scope

Define process-local and process-external component identity, startup, shutdown, and failure boundaries.

## Contract

- Record component identity, instance ID, generation, isolation boundary and lifecycle. Treat a process ID as host-local routing metadata requiring an instance check. Raw pointers stay in their address space.
- In-process execution uses the admitted native mapping. Across processes, use a declared transport and serialized data or explicitly granted shared memory handles with lifetime and access limits.
- A host defines startup readiness, peer authentication, stop deadlines and cleanup. The process supervisor owns termination and observes exit status. A restarted process receives a new generation.

## Implementation sequence

1. Start an instance with an explicit manifest, resource budget, and minimal inherited handles. Connect and validate its readiness and peer identity before routing calls.
2. On peer exit, invalidate generation-scoped handles, classify in-flight calls under their delivery semantics, and release local resources. Route restart decisions to reliability policy.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Child process exits during a request | Report peer loss with delivery state and invalidate its handles. |
| OS reuses a process ID | Reject stale instance identity. |
| Pointer exported across processes | Reject the descriptor or require a declared shared memory handle. |
