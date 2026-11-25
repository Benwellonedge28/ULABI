# ULABI ABI Validator

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Type descriptors](../type-system/type-descriptors.md), [Versioning](../../ULABI-VERSIONING.md), [Diagnostics](../observability/diagnostics.md)

## Scope

Define validation of descriptors, interfaces and mappings before source generation, linking or admission.

## Contract

- Accept a pinned descriptor or interface input, intended role and mapping, and a validation mode. Bound input, depth and issue counts. Report malformed input, invalid semantics, incompatible peer and unavailable capability separately.
- Validate structure, identities and types, then ownership, layouts, version constraints, required capabilities and profile obligations in that order. Optional fields follow their declared schema behavior.
- Return deterministically ordered diagnostics by path, issue ID and location. A valid result means checks passed for the specified mode and scope.

## Implementation sequence

1. Build one validation entry point with structured diagnostics and reuse it in compiler, linker and loader adapters. Expose the proposed offline command as `ulabi validate <descriptor>`.
2. Add fixtures with duplicate IDs, unresolved references, arithmetic overflow, invalid ownership and conflicting profile requirements. Fail before symbol binding on invalid input.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two distinct errors share a descriptor location | Emit both stable issue IDs in deterministic order. |
| Input exceeds configured bounds | Stop bounded parsing and return a resource-limit diagnostic. |
| Descriptor is valid for a different mapping | Report the mapping mismatch. |
