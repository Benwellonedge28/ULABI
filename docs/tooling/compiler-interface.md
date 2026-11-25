# ULABI Compiler Interface

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Validator](validator.md), [Calling convention](../abi/calling-convention.md), [Type descriptors](../type-system/type-descriptors.md)

## Scope

Define the compiler’s inputs, stages and output artifacts for a native provider or consumer adapter.

## Contract

- Accept a versioned interface description, declared role, target mapping, and feature set. Emit target declarations or stubs, machine-readable descriptors, an export or import inventory, and diagnostics.
- Resolve semantic identities first, then lower layouts and calls for the selected mapping. Validate ownership, lifetime and status conversion in generated adapters. A stable semantic ID stays independent of native symbol spelling.
- Record compiler and mapping versions and ABI-affecting options in the output manifest. Keep deterministic output ordering and reject unsupported operations instead of silently emitting a different ABI.

## Implementation sequence

1. Run shared validation before lowering. Generate the adapter and metadata from the same resolved representation and ensure they agree.
2. Test generated bindings against independent declarations and link and call fixtures. Include diagnostics for unsupported types and target mappings.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Metadata says 32-bit length but generated adapter uses 64-bit | Fail the compiler consistency check. |
| Mandatory convention feature lacks lowering | Reject the affected export before publication. |
| Same input compiles with another architecture mapping | Keep semantic identities stable and publish the distinct mapping. |
