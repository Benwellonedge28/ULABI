# ULABI Supply-Chain Evidence

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Secure loading](secure-loading.md), [Governance](../../ULABI-GOVERNANCE.md), [Conformance](../standards/conformance.md)

## Scope

Define the metadata and dependency verification needed to admit a component under a supply-chain profile.

## Contract

- Record component and build identities, source revision, artifact digest, build inputs, toolchain and dependency identities. Bind required signatures and attestations to the artifact digest and review policy.
- Resolve a declared dependency closure and check allowed versions, sources and digests. Treat undeclared resolved dependencies as failures where the policy requires complete closure.
- Freeze evidence by release and keep provenance queries independent of runtime capability grants. A trusted producer identity does not by itself authorize execution.

## Implementation sequence

1. Generate provenance from the build environment and package it with the manifest and dependency inventory.
2. Verify required attestations and closure before admission. Store the exact consulted digests and the policy decision for later review.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Build attestation covers a different digest | Reject the evidence and block required admission. |
| A required dependency is missing from the inventory | Fail closure verification under the strict profile. |
| An offline host lacks required trust evidence | Defer or reject according to the profile. |
