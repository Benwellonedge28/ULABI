# ULABI Structured Diagnostics

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Exception model](../abi/exception-model.md), [Tracing](tracing.md), [Authorization](../security/authorization.md)

## Scope

Define stable records for errors, incompatibility and recovery observations at the interoperability boundary.

## Contract

- Include schema revision, diagnostic ID, severity, phase, component and operation IDs, safe reason code, event time and correlation ID where available. Keep the machine-readable category independent of localized text.
- Distinguish a stable operation error from an optional diagnostic. Define allowed fields and redact sensitive arguments, credentials, identifiers and memory content according to access policy.
- Bound record size and collection. An unavailable diagnostic sink leaves the operation result intact. Expose sink failures through a separate health signal where allowed.

## Implementation sequence

1. Construct diagnostics at boundary conversion points from approved metadata. Attach operation and artifact context before redacting and routing.
2. Expose a read-only authorized inspection path and exercise diagnostics with both localized text and machine-readable clients.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| A message is translated | Keep diagnostic ID and outcome stable. |
| Unauthorized caller requests sensitive details | Return only the allowed fields. |
| Sink fails during a call | Preserve the call’s specified status. |
