# ULABI Conformance Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **First evidence source** · [Native C/Rust reference implementations](reference-implementations.md) and [test-suite design](test-suite.md)

## Purpose

Define what an implementation may claim, what evidence supports that claim, and when the evidence must be renewed. A claim applies to a released artifact and a declared scope. A programming language, compiler family, or project name alone identifies too broad a scope to evaluate.

The Core specification defines the mandatory behavior. This design defines how to evaluate and record that behavior. [Compliance levels](compliance-levels.md) maps the preliminary levels to evidence. [Certification](certification.md) defines the proposed issuer and review process. This design supplies evidence for both.

## Claim scope

Record the artifact digest and release identifier, source revision, role, target and platform mapping, specification revision, and applicable profile and extension revisions. Record the build toolchain and environment that produced the artifact separately from the target on which tests ran. Distinct combinations of artifact, role, target mapping, and contract revision require their own evaluation. A single published report may contain several such evaluations if its results identify each combination.

| Claim type | Evidence required | Claim wording |
| --- | --- | --- |
| Pilot interoperability | The native test-suite gate passes for the declared pilot revision, target, roles, and artifact set. | “Passed the ULABI native interoperability pilot for `org.ulabi.pilot.byte-copy` 1.0 on Linux x86-64, pilot revision 1.” |
| Core conformance | Every mandatory requirement of the pinned Core revision applicable to the declared scope has accepted evidence. | “Conforms to ULABI Core <revision> for <role> on <target and mapping>.” |
| Profile conformance | Core coverage plus accepted evidence for the profile’s required extensions, platform and security constraints, and each optional feature the implementation claims. | “Conforms to ULABI <profile and revision> for <role> on <target and mapping>, with <claimed extensions>.” |

The publication record accompanying any claim includes the evaluated artifact digests and a link to the evidence. Use the exact draft designation when claiming against a draft. An evaluation that has not passed can publish its status and coverage progress.

## Pilot gate

Use the [test-suite design](test-suite.md) as the source of expected outputs and execution rules. The baseline matrix has 32 cases and four pairings, yielding 128 required results per complete run. Check that the report contains each expected pair and case exactly once and that every required case passed. Verify both cross-language directions, the declared artifacts and their digests, and the compiler versions used to build them. Review the clean offline repeat and the freshly built and prebuilt artifact runs required by the test-suite design. Compare ordered semantic observations for the repeat, using the suite’s rules for run-specific metadata.

Record precisely which artifact set passed each execution mode. If the claim names a released artifact, it needs a passing prebuilt run using that artifact’s digest. A successful clean rebuild supplies repeatability evidence for the source and toolchain. It cannot on its own establish that a separately distributed binary was tested.

Record accepted results as pilot evidence. The byte-copy operation exercises a bounded subset of identity, binding, type mapping, argument representation, borrowed memory, and status results. For example, manifest equality cases provide evidence for the pilot’s exact fixture binding. Core capability discovery still requires coverage of the capabilities required by the applicable Core clauses.

## Requirements inventory

Create a versioned inventory from the pinned normative text before evaluating Core or a profile. Assign a stable ID to each independently testable obligation and link it to its source clause. A section may yield several IDs. The IDs belong to the coverage inventory and do not alter the specification. The inventory lists role and feature applicability, the expected outcomes, and the required evidence for each ID. Resolve unclear requirements in the owning specification or record them as blocked.

Start with the Core Profile topics listed in section 8 of the [Core specification](../../ULABI-SPEC.md). The topics below guide the inventory. The pinned normative clauses determine the final entries and outcomes.

| Core topic | Evidence to plan |
| --- | --- |
| Interface identity and ABI version | Readable version metadata, stable interface identity across independent implementations, and mismatched identity and ABI inputs handled as specified. |
| Primitive, argument, and return representation | Positive and negative vectors for required types and widths, argument ordering, and return behavior for each declared mapping and role. |
| Calling convention metadata | Correct descriptor and operation binding, with unsupported or incompatible metadata handled according to the applicable clause. |
| Error representation | Expected boundary results for required failure categories and evidence of any role-specific error conversion. |
| Alignment and layout | Independent size, alignment, and offset checks for mandated data shapes, plus byte-level comparisons wherever a representation is prescribed. |
| Ownership metadata | Expected results for borrow, transfer, or release behavior where required. Use contract-valid storage and observe ownership through role-specific checks. |
| Capability discovery | Machine-readable evidence for declared capabilities and visibility of required profile information. |
| Compatibility information | Supported combinations and their outcomes, including older and incompatible revisions wherever the declared contract requires those cases. |

For a profile, import the Core inventory, add the required extensions and constraints, then include the obligations for each declared optional feature. Evaluate the combined inventory per scope. A missing implementation of a mandatory requirement remains open or failed. An applicability exclusion requires a clause and a reason grounded in the declared scope.

## Evidence record and evaluation

For each evaluation, create one evidence record with these fields.

| Field | Required contents |
| --- | --- |
| Record identity | Evidence schema revision, evaluation ID, UTC time, evaluator identity, and claimed scope. |
| Baselines | Specification and profile revisions, exact source revisions used for requirements, and suite, fixture, and inventory revisions. |
| Artifact and environment | Release and source revisions, SHA-256 of the tested binary and required manifest, role, target and mapping, toolchain versions and ABI-affecting flags, plus runtime, loader, or system-library versions where relevant. Record counterpart artifacts for interoperability runs. |
| Results | Requirement ID and source clause, applicability, mapped case IDs or manual procedure, expected and observed outcomes, disposition, and evidence location. Keep raw suite reports and build logs addressable. |
| Decision | Outcome, any gaps, evidence references, and a link to the predecessor evaluation if the record replaces one. |

Use machine-executable evidence where practical. Accept a manual result only when the inventory documents why automation is impractical and supplies a repeatable procedure, expected observation, reviewer, and retained result. A manual result follows the same pass criteria as an executable check for that obligation.

Evaluate in this order.

1. Confirm that the claimed scope matches the artifact and environment in the record. Verify digests and baseline revisions before using any result.
2. Confirm that each applicable obligation has a test or approved manual procedure. Check required cases for absence and duplication. Reject stale evidence for a changed scope.
3. Assign each entry one disposition. Use `pass` for a verified match, `fail` for a verified mismatch, `blocked` for missing or unusable evidence, and `not_applicable` only for a documented scope exclusion.
4. A full claim passes only when every applicable obligation passes. A verified mismatch makes the evaluation fail. If there is no verified mismatch and an applicable obligation remains blocked, the evaluation is incomplete. Record every failed or blocked item even when another item already determines the overall decision.
5. Publish the scope, decision and evidence location together. Preserve earlier decisions for the specific artifacts and baselines they covered.

For the pilot, use the suite result and repeat evidence directly. A verified mismatch fails the evaluation. A partial run without a verified mismatch leaves the evaluation incomplete.

## Re-evaluation rules

Compare each proposed release to its last accepted record. Where a change fits multiple rows, apply all relevant checks. Evidence can be carried forward when the evaluated artifact, contract, and environment assumptions still match and the evaluator records why each reused result remains valid.

| Change | Required action before making the claim |
| --- | --- |
| Documentation wording or release packaging | Carry evidence forward only when the tested artifact and manifest digests, normative behavior, and execution assumptions match. Record the comparison. If packaging changes loading, discovery, or dependencies, evaluate the delivered package on the affected target. |
| Adapter, exported code, or manifest changes | Evaluate the changed artifact. Re-run all pilot pairings required by the pilot claim. For a full claim, re-run affected obligations and dependent boundary cases. An inherited result must reference its original artifact and document why the changed artifact preserves the behavior it tested. Re-run the full scope when the impact cannot be bounded. |
| Compiler, linker, ABI-affecting flags, runtime, loader, or system-library change | Rebuild where applicable and evaluate on the new configuration. Re-run the full pilot for pilot claims. For Core or profile claims, re-run the full affected target and role when the change may affect representation or calls across the boundary, or the impact is uncertain. |
| New CPU, operating system, or platform mapping | Open a separate evaluation. Reuse applicable semantic vectors and run all required target checks on the new mapping. Retain provenance for each separately tested target. |
| Core, profile, extension, or interface contract revision | Diff the pinned obligations, update the inventory, and run changed or newly applicable cases and their dependencies. Run the full affected scope when compatibility or impact is uncertain. Exercise old and new peers where compatibility is claimed. |
| Suite or fixture revision | If expected behavior or coverage changed, re-run affected checks under the new revision and review the rest of the evidence for applicability. An earlier report remains attributable to its original suite and fixture revisions. |

When a published artifact later fails a mandatory case, mark the corresponding evaluation superseded or withdrawn and publish the affected scope and reason. Keep the historical record. A corrected artifact or corrected evaluation gets its own identity and supporting evidence.

## Delivery order and acceptance

1. Publish a pilot evidence template using the required fields above and review it against the test-suite report format. Use the suite report as raw evidence and reference it from the evaluation record.
2. Review the first pilot evaluation for a complete matrix, clean offline repeat, both artifact execution modes, and traceable digests. Check that removing one result or replacing one digest prevents a passing evaluation.
3. Build the clause-level Core inventory against the pinned specification. Review its applicability and evidence mappings before evaluating a Core claim. Confirm that an untested mandatory obligation makes the evaluation incomplete.
4. Add each profile inventory as its specification is ready. Confirm that omission of a required extension or a claimed optional capability prevents a passing profile evaluation.

The repository’s current pilot and suite documents define planned work. A claim starts when the required evidence exists for the declared scope.
