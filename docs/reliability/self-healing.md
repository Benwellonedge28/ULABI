# ULABI Policy-Controlled Self-Healing

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Fault detection](fault-detection.md), [Fault isolation](fault-isolation.md), [Recovery](recovery.md), [Rollback](rollback.md)

## Scope

Define the optional orchestration state machine that joins failure detection, isolation, recovery and rollback.

## Contract

- Use the Core sequence Healthy → Failure Detected → Evidence Collection → Diagnosis → Policy Check. If approved, proceed through Isolation → Recovery → Verification. A verified outcome returns to Healthy. Failure follows the bounded policy to Rollback and escalation.
- Every transition records incident, generation, evidence, policy revision, action, attempt and outcome. Allow only actions and resources authorized for that incident. Revalidate scope when the generation or policy changes.
- Prevent concurrent supervisors from applying the same incident action through an exclusive transition or fenced owner. Preserve remaining budgets and the audit trail across restart. Keep the ABI contract stable through recovery.

## Implementation sequence

1. Implement a transition table with guards for evidence, version, scope and budget. Persist guards and decisions and make resumption reconcile any uncertain action outcome.
2. Route verification and rollback to their dedicated interfaces. Expose an authorized incident record and escalate when no approved path exists.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| No approved action exists | Escalate without writing component state. |
| Two supervisors resume the same incident | Only one fenced owner executes the transition. |
| Recovery requires an unauthorized capability | Reject the action and follow policy. |
