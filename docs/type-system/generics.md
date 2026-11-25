# ULABI Generic Type Boundaries

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Type descriptors](type-descriptors.md), [Compatibility](type-compatibility.md), [FFI](../interoperability/foreign-function-interface.md)

## Scope

Define how parameterized interfaces expose concrete interoperability types while keeping language-specific generics internal.

## Contract

- Represent a generic template by stable identity, ordered type parameters and expressed constraints. Bind exported native operations to concrete instantiated descriptors unless a profile specifies a separately negotiated dynamic representation.
- Identify an instance by template revision and ordered semantic arguments. Compute its layout from the concrete descriptors for the chosen mapping. Resolve constraints before emitting an export.
- Publish a finite supported instance set or a declared negotiation mechanism. An erased or opaque value needs a validated type ID and the corresponding ownership and operation contract.

## Implementation sequence

1. Build an instance manifest from declared export roots and validate parameter counts, constraints, recursion and mapping support.
2. Emit concrete descriptors and bind them to stable interface and function identities. Resolve the expected instance before the adapter invokes the operation.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Template arguments are reordered | Resolve to a distinct instance or reject the mismatch. |
| Peer asks for an unexported instance | Reject as unsupported before invocation. |
| Two languages lower the same instance differently | Use declared conversion or reject direct binding. |
