# ULABI Versioning and Compatibility

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](ULABI-SPEC.md)
- **Integration** · [Architecture](ULABI-DESIGN.md), [Governance](ULABI-GOVERNANCE.md), [Backward compatibility](docs/compatibility/backwards-compatibility.md), [Feature negotiation](docs/compatibility/feature-negotiation.md)

## Scope

Define version identities, change classification, and release controls for the specification, published interfaces, and profile mappings.

## Contract

- Record specification revision, Core ABI revision, interface identity and version, profile revision, platform mapping ID, and descriptor schema revision separately. A draft marker is part of the revision. A reference implementation or test-suite release has its own version.
- Use the Core specification’s major, minor, and patch semantics for published interfaces. Classify removed or reinterpreted fields, changed layout or calling behavior, and newly mandatory behavior that breaks compatible peers as breaking. Treat independently negotiable additions as candidates for a minor version. A patch preserves boundary behavior.
- Define compatible pairs for each role and mapping from evidence. A higher version alone does not authorize binding. Validate required capabilities, extensions and metadata before accepting a pair. Reject an unknown mandatory feature and retain historical artifacts and reports.
- Record deprecations with first revision, affected scope, supported replacement, and the earliest planned removal. Treat removal of a supported contract as a breaking change.

## Implementation sequence

1. Inventory public interface descriptors, mandatory requirements, and compatibility matrices from the previous release. Compare both machine-readable descriptors and behavioral cases.
2. Record the proposed change class and rationale in release notes. Obtain the review required by governance, then publish the pinned specification, descriptors, migration guidance, and scoped conformance evidence together.
3. Run existing peers against the new artifact in each compatibility direction claimed. Publish any reduced compatibility as a scope change.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Patch deliberately changes struct layout | Reject the patch classification and require a migration and compatibility decision. |
| New optional feature meets an older peer | Bind to the supported common behavior when permitted by both contracts. |
| Unrecognized descriptor revision | Reject without interpreting layout or binding an operation. |
