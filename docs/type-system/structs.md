# ULABI Struct Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Data types](../abi/data-types.md), [Descriptors](type-descriptors.md), [Ownership](../memory/ownership.md)

## Scope

Define the semantic fields and native layout checks for struct values crossing an ABI boundary.

## Contract

- Keep stable field IDs, order, semantic types, alignment, offset or layout rule, optionality and ownership. Materialize offsets and total size from a pinned profile. Use the same checked layout on both sides of a binding.
- Validate that fields fit within the total extent and satisfy alignment. Reject overlapping value fields. Keep padding semantically unused and exclude it from portable serialization. Validate optional fields through their declared presence representation.
- Preserve nested ownership and lifetimes through conversion. Appending or reordering fields can break native compatibility even when field IDs stay stable.

## Implementation sequence

1. Resolve field descriptors, compute layout and check size and alignment against independent native declarations.
2. Lower field-by-field where layouts differ and a conversion contract permits it. Track initialization and unwind only resources owned by the converting side.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Equal field names with different offsets | Reject direct binding or use a declared conversion. |
| Packed source field lacks required native alignment | Use a compliant declared conversion without unaligned typed access. |
| Conversion fails after moving an owned field | Release or restore it according to the explicit transfer contract. |
