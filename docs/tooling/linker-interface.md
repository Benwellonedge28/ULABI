# ULABI Linker Interface

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Symbol resolution](../interoperability/symbol-resolution.md), [Compiler interface](compiler-interface.md), [Type compatibility](../type-system/type-compatibility.md)

## Scope

Define static or build-time binding decisions between ULABI imports and exports.

## Contract

- Identify imports by semantic interface and operation identity, contract revision, role, and target mapping. Use the native export name from a compatible descriptor when producing the actual link.
- Reject missing mandatory imports, ambiguous providers, unsupported features and incompatible signatures before committing output. Resolve optional imports using declared policy and record them as unresolved where permitted.
- Emit a link manifest with importer and provider artifact identities, negotiated mappings, and binding decisions. Transitive dependencies remain subject to loader and security policy.

## Implementation sequence

1. Index candidates by identity, filter and compare against each import, and choose using explicit resolution rules. Keep provider selection deterministic for pinned inputs.
2. Emit native link directives and a machine-readable decision list only after validation. Test duplicate implementations, optional omission and incompatible signatures.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two equally valid providers have no priority rule | Report ambiguous binding. |
| Names match but ownership or argument mapping differs | Reject the binding. |
| Optional import is absent under its permitted policy | Link the rest and record the missing feature. |
