# ULABI Type Descriptors

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Universal types](universal-types.md), [Native layouts](../abi/data-types.md), [Validation](../tooling/validator.md)

## Scope

Specify what descriptor producers emit and what consumers validate before lowering or decoding a type.

## Contract

- Identify the schema revision, semantic type ID, kind, member references, and descriptor revision. Bind any layout section to a mapping ID and include representation size, alignment, byte order when applicable, and layout rules. Include qualifiers for ownership and mutability where required.
- Maintain a descriptor graph keyed by ID. A by-value cycle is invalid. Indirect recursion is supported only through an explicitly modeled reference or handle with a lifetime contract.
- Read with bounded bytes, member count, graph depth and total references. Use checked arithmetic for offsets, extents and array sizes. Reject unresolved mandatory types and unknown mandatory fields.

## Implementation sequence

1. Decode structure and index IDs before resolving references. Detect cycles and validate member layouts before exposing descriptors to compilation or loading.
2. Compare canonical descriptor content for a pinned schema and mapping. Record semantic identity separately from any content digest.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Duplicate ID maps to different definitions | Reject the graph with the conflicting ID. |
| Alignment or extent overflows | Reject descriptor validation before allocating by the declared size. |
| Self-reference through a declared handle | Resolve the descriptor without recursively expanding handle storage. |
