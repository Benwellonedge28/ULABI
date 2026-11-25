# ULABI Tagged Union Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Data types](../abi/data-types.md), [Type descriptors](type-descriptors.md), [Memory lifetimes](../memory/lifetimes.md)

## Scope

Define representation and validation for one active variant, including optional and result types that adopt this model.

## Contract

- Declare the tag representation, tag-to-variant map, variant descriptors, payload location and alignment rule. Every discriminant selects at most one active variant. An empty variant has no readable payload.
- Validate the tag before reading or releasing a payload. Closed descriptors reject unknown tags. Preserve an unknown variant only under an explicit extensible encoding with bounded, independently delimited payload and defined ownership.
- Treat tag, payload initialization and transfer as one logical transition. Keep destruction and rollback scoped to the active payload.

## Implementation sequence

1. Validate the descriptor’s full tag map and layout. Construct a variant by initializing its payload before publishing a usable value.
2. Decode the tag, check payload bounds and alignment, then convert or borrow the active member using its descriptor.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Tag selects an absent optional value | Read and release no payload. |
| Invalid tag in a closed result | Reject without interpreting or freeing payload bytes. |
| A larger variant changes union extent | Require a compatible new representation or a breaking revision. |
