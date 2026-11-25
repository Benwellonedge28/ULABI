# ULABI WebAssembly Host Mapping

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Architecture mappings](architectures.md), [Type descriptors](../type-system/type-descriptors.md), [Sandboxing](../security/sandboxing.md)

## Scope

Define a ULABI mapping for a declared WebAssembly execution environment.

## Contract

- Pin the chosen execution and component or module ABI, address width, function signatures, imported services, memory ownership and feature set. Bind only under the exact supported mapping.
- Represent guest memory accesses with guest offsets and checked extents. Reacquire memory views after calls that may grow or replace underlying storage. A host pointer requires explicit host-managed handle mediation.
- Declare how traps map to failures, who releases guest and host allocations, and which imports are authorized. Apply profile resource limits before exposing host services.

## Implementation sequence

1. Validate module metadata and requested imports. Instantiate inside the configured execution boundary and lower values through mapping-specific adapters.
2. Test memory geometry, growth, failure translation and unsupported imports. Record the host engine and capabilities in the evidence.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Offset plus length exceeds current guest memory | Reject before access. |
| Module requests a forbidden host service | Reject admission or deny the declared optional service. |
| Guest traps after an allocation | Release host-owned resources and apply the declared guest cleanup policy. |
