# ULABI Secure Component Loading

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Loader interface](../tooling/loader-interface.md), [Supply-chain security](supply-chain-security.md), [Sandboxing](sandboxing.md)

## Scope

Define the security admission decision that precedes module execution or symbol binding.

## Contract

- Validate artifact identity, digest, manifest compatibility, dependency closure, policy and required signatures before execution. Resolve dependencies using approved identities and locations.
- Bind verification to the bytes that will load. Load from a trusted immutable staging location or an equivalent host-controlled artifact handle. Reject substitutions after verification.
- Resolve required capabilities and isolation before initialization. Where opening a module may run constructors, finish pre-execution checks before opening it.

## Implementation sequence

1. Stage and verify the manifest, module, and required dependencies. Apply policy and select the execution boundary before the loader opens the module.
2. Bind declared exports on the verified artifact, record provenance, and undo partial admissions and grants on initialization failure.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Artifact changes between validation and load | Reject or re-verify the actual staged bytes. |
| Dependency resolves outside approved locations | Reject before executing module code. |
| Manifest requires a disallowed grant | Deny admission. |
