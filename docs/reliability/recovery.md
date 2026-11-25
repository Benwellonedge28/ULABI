# ULABI Recovery Planning and Verification

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Fault isolation](fault-isolation.md), [Rollback](rollback.md), [Self-healing](self-healing.md)

## Scope

Define how an authorized incident action is selected, attempted and independently verified.

## Contract

- A recovery plan identifies incident and generation, permitted actions, attempts, backoff, time and resource budgets, preconditions, verification and fallback. Bind each attempt to an idempotent action identity where the action supports it.
- Choose among declared actions such as restart, failover or state restoration under current limits and grants. Replay a business operation only when its contract and known outcome permit it.
- Verification checks readiness, required interface compatibility, state integrity and security policy. A successful action return does not establish recovery.

## Implementation sequence

1. Load and validate the plan, reserve budget and apply the next permitted action. Track attempt history and persist actionable state across supervisor restart where supported.
2. Verify the resulting generation before returning it to service. Invoke rollback or escalate when verification fails or the attempt budget is exhausted.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Restart succeeds but interface identity changes unexpectedly | Keep the instance out of routing and follow fallback. |
| Retry budget is exhausted | Escalate without further attempts. |
| Remote write outcome is unknown and replay is unsafe | Require reconciliation or escalation. |
