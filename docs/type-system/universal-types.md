# ULABI Universal Type Semantics

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [ABI data types](../abi/data-types.md), [Type descriptors](type-descriptors.md), [Type compatibility](type-compatibility.md)

## Scope

Define the language-neutral types an adapter resolves before choosing a native or wire representation.

## Contract

- Use the Core primitive identities Bool, I8 through I128, U8 through U128, F32, F64, Char, and Byte. Preserve signedness and width. Byte represents eight bits. Char carries the encoding defined by the contract. A language’s integer, character, or Boolean layout requires an explicit mapping.
- Resolve composite types using member types and ordered identities. Represent absence and success/failure explicitly. Distinguish strings by encoding and length, arrays by element type and extent, and buffers by ownership, lifetime, access and bounds.
- Keep semantic type, platform layout, and wire encoding separately addressable. Require the representation to define byte order, integer encoding, Boolean values, and floating-point special values where relevant.

## Implementation sequence

1. Resolve aliases and type references to semantic IDs. Build a representation mapping for each supported role and target.
2. Use checked conversions at the adapter boundary. Reject values outside the required range or invalid for the selected encoding before invoking the foreign operation.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Unsigned source value exceeds the destination range | Return a conversion failure without narrowing. |
| Boolean payload uses an unassigned encoding | Reject it under that representation. |
| A buffer contains a zero byte | Preserve the byte and use the declared length. |
