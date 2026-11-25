# ULABI Type Compatibility

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Type descriptors](type-descriptors.md), [Feature negotiation](../compatibility/feature-negotiation.md), [Versioning](../../ULABI-VERSIONING.md)

## Scope

Classify a producer-to-consumer mapping for a specific operation, role and representation.

## Contract

- Compare semantic identity and revision first, then representation and ownership under the selected mapping. Native direct binding requires agreement on width, alignment, extent, byte order, member layout and lifetime properties used by the operation.
- Classify a direction as direct, convertible under an explicit rule, or incompatible. Require conversions to preserve semantic values on their declared domain and report rejected values before publication. Identity and size alone do not establish equivalence.
- Compare enum tags, optionality, required fields, constraints, and ownership flags. A change that older writers can accept can still break older readers.

## Implementation sequence

1. Resolve both descriptor graphs and compare recursively with a memo of visited descriptor pairs. Record the first mismatch path and any conversion required.
2. Check producer-to-consumer and the reverse direction separately wherever both are claimed. Gate direct linking on the resulting binding decision.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Same size with changed signedness | Reject direct substitution and require an explicit range-checked conversion. |
| Borrow becomes transfer | Reject the changed obligation under the old contract. |
| Unknown mandatory field in a new descriptor | Reject direct binding. |
