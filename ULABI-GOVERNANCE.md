# ULABI Specification Governance

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](ULABI-SPEC.md)
- **Integration** · [Architecture](ULABI-DESIGN.md), [Versioning](ULABI-VERSIONING.md), [Conformance](docs/standards/conformance.md), [Certification](docs/standards/certification.md)

## Scope

Define how ULABI proposes, reviews, records, and releases technical decisions while retaining independent implementations and publicly traceable specifications.

## Contract

- Maintain a public change register with proposal ID, owner, affected clauses, compatibility impact, security and privacy impact, test evidence, status, and decision rationale. Track draft, review, accepted, rejected, and withdrawn decisions.
- Maintain a visible roster of people authorized by repository governance to approve specification changes. A proposal author requests an independent reviewer. Conflicts are declared in the record and conflicted reviewers recuse. Keep draft decisions provisional when independent review is unavailable.
- Review affected Core and profile changes for interoperability, scope and evidence. Stable Core changes require an explicit compatibility decision and evidence from independent implementations as described by the Core specification.
- Allocate shared identifiers in a reviewed registry. Give experimental and vendor extensions distinct namespaces and lifecycle labels. Releasing a proposal does not grant certification or assign implementation authority to a single vendor.
- Receive security disclosures through a declared private channel. Limit early access to parties coordinating remediation. Publish the resulting change and rationale after disclosure coordination.

## Implementation sequence

1. Open a proposal with alternative designs, affected scope, release class, migration path and verification plan. Ask affected maintainers and independent implementers to review.
2. Record the reviewed decision and related test results before updating normative documents. Update the version and release notes together, with pinned links to the proposal and evidence.
3. If a decision is contested, retain the written objection and disposition. A later change references and supersedes the original record without erasing it.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Breaking change without a compatibility analysis | Return it to review with the missing analysis recorded. |
| Two extensions request the same identity | Reject the collision and assign independently scoped identifiers. |
| Conflicted proposal author is the only available reviewer | Keep the proposal provisional. |
