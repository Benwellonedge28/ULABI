# ULABI Authorization Decisions

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Authentication](authentication.md), [Capability security](capability-security.md), [Diagnostics](../observability/diagnostics.md)

## Scope

Define a policy enforcement point for requested operations and protected resources.

## Contract

- Evaluate the principal, component instance, operation, resource identity, capability and contextual policy. Default to deny for protected access when required context or policy is missing.
- Bind the decision to the resource and action actually performed. Recheck when authorization state changes before a delayed or queued action begins. A cached decision includes the policy revision and validity conditions.
- Distinguish authorization denial from authentication failure. Record a correlation ID, policy revision, and safe reason category without exposing credential material.

## Implementation sequence

1. Normalize the operation and resource identity before policy lookup. Validate any relevant capability and evaluate the profile policy.
2. Enforce the decision at the actual resource-use boundary, including callbacks and retries. Define when already-admitted work may continue after policy changes.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Policy changes while a write is queued | Re-evaluate before the write starts. |
| Same principal requests another resource | Evaluate the new resource scope independently. |
| Policy service is unavailable for a protected operation | Deny access and report a stable failure. |
