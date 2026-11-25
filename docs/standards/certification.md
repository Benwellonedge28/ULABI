# ULABI Certification Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Conformance](conformance.md), [Compliance levels](compliance-levels.md), [Governance](../../ULABI-GOVERNANCE.md)

## Scope

Describe a future independent evaluation process for claims backed by versioned ULABI conformance evidence. Certification starts when an authorized program and rules exist.

## Contract

- A submission identifies artifact and manifest digests, release, target and platform mapping, role, pinned specification and profile revisions, claimed capabilities, evidence records, and reproduction instructions.
- An authorized evaluator verifies scope and provenance, checks completeness against the normative inventory, and reproduces the program’s required checks. Record conflicts and route conflicted evaluations elsewhere. An implementer may publish its own conformance result under its own identity.
- An issued record identifies issuer, certificate ID, exact scope, evidence, issue date, validity conditions, and status. Record suspension, withdrawal, and supersession with a dated reason and preserve historical records.
- Use the conformance re-evaluation rules when an artifact or relevant environment changes. The issuer reviews corrected submissions under a new or superseding record.

## Implementation sequence

1. Validate an incoming dossier and return missing scope or evidence to the submitter. Route a complete submission to an independent reviewer.
2. Record the assessment and reproducibility checks. Publish the issuer decision and exact certificate scope after authorization. Hold unready submissions as pending.
3. Provide a lookup by certificate ID and artifact digest. Reassess published records when evidence is withdrawn or a material issue affects their scope.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| A request cites only the byte-copy pilot | Return insufficient evidence for a full Core or profile certificate. |
| Artifact digest differs from the submission | Hold the evaluation until the scope and reproducible evidence agree. |
| A withdrawn certificate is looked up | Show withdrawn status, reason, date, and historical scope. |
