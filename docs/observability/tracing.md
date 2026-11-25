# ULABI Boundary Tracing

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Diagnostics](diagnostics.md), [Telemetry](telemetry.md), [Remote calls](../distributed/remote-calls.md)

## Scope

Define causality information for authorized local, asynchronous and distributed observations.

## Contract

- Use a trace ID, span ID and optional parent or linked span, plus operation and component identity. Start and finish records share a span ID. Track async completion separately from submission and link it by operation identity.
- Forward only validated metadata allowed by the transport and security profile. Create a new trace root when context is missing or untrusted. Trace context confers no identity or access.
- Record local monotonic duration where available. Preserve event ordering within a documented clock domain and bound sampling, propagation size, and buffers.

## Implementation sequence

1. Instrument boundary start, binding or dispatch and completion. Keep incoming trace parsing independent from authorization and call results.
2. Verify cancellation and peer loss produce a terminal span outcome when observable. Record missing spans or dropped events without manufacturing completion.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Peer supplies malformed trace context | Discard or restart tracing without changing the valid call. |
| Completion occurs on another executor | Correlate it with the original operation. |
| Sampling is disabled | Operation behavior remains unchanged. |
