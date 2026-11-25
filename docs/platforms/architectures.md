# ULABI Architecture Mappings

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [CPU hardware](../hardware/cpu.md), [Calling convention](../abi/calling-convention.md), [Type descriptors](../type-system/type-descriptors.md)

## Scope

Define how an architecture profile turns shared type and call semantics into a specific machine ABI.

## Contract

- A mapping identifies architecture, endian mode, pointer model, data layout, calling convention and required instruction features. Resolve operating system and object-format constraints jointly with the OS profile.
- Specify scalar and aggregate arguments and returns, register classes, stack alignment, and preserved state for the supported subset. A missing lowering causes binding or compilation to fail with its operation ID and type.
- Give a new mapping its own ID when binary compatibility cannot be preserved. Use logical descriptors and explicit codecs when crossing architecture boundaries.

## Implementation sequence

1. Implement target-specific lowering against the mapping and compare generated layout and call fixtures with an independent native compiler.
2. Register tested role and target pairs. Exercise boundary values, nested aggregates and declared unsupported signatures before claiming coverage.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Same CPU offers two pointer models | Resolve as distinct mappings unless proven equivalent. |
| Endianness differs | Use a declared conversion or reject direct binding. |
| Aggregate exceeds supported return shapes | Reject export or use a declared alternate lowering. |
