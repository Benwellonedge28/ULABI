# ULABI Compliance Levels

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Conformance](conformance.md), [Certification](certification.md), [Versioning](../../ULABI-VERSIONING.md)

## Scope

Map the preliminary levels in Core §135 to evidence and machine-readable declarations. Evaluate levels per artifact, target, role and revision.

## Contract

- Level 0 adds identity, version, profile and capability metadata. Level 1 adds calling convention, primitive types, functions and errors. Level 2 adds composite types, descriptors and memory ownership. Level 3 adds async, streams, concurrency and lifecycle.
- Level 4 adds capabilities, secure loading, authorization and integrity. Level 5 adds health, fault detection, recovery and self-healing. Level 6 adds remote calls, serialization, discovery and distributed failure handling.
- The level requirements are cumulative for the pinned draft. Bind each topic to an obligation inventory and retain results per level. A level passes when its applicable obligations and every lower level pass. The named Core or profile claim still uses its full normative scope.
- Declare supported, unsupported, or unevaluated for each capability and include the highest demonstrated level if one exists, plus evaluation IDs. A standalone pilot pass is reported as pilot evidence.

## Implementation sequence

1. Map each level topic to pinned clauses and the conformance inventory. Record applicability with its rationale instead of counting missing evidence as unsupported.
2. Generate the declaration from evaluated results and validate it against the evidence scope. Publish a readable matrix alongside a machine-readable declaration with a schema revision.
3. Recalculate claims for changed artifacts and revisions using the conformance re-evaluation rules.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Level 3 cases pass while a Level 2 mandatory case fails | Withhold Level 2 and higher claims for that scope. |
| Two target mappings pass different levels | Publish separate level results keyed by mapping. |
| Only the pilot passed | Publish the pilot result and report unverified level coverage. |
