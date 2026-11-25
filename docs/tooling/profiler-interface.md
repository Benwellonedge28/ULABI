# ULABI Boundary Profiler Interface

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Telemetry](../observability/telemetry.md), [Tracing](../observability/tracing.md), [Runtime limits](../runtime/resource-management.md)

## Scope

Define collection and interpretation of performance observations at the ULABI boundary.

## Contract

- Identify measurement source, units, clock domain, sampling policy, aggregation window, artifact and mapping. Distinguish queue delay, execution time and end-to-end duration when supplied.
- Bound instrumentation overhead and event cardinality. Follow profile restrictions for real-time or constrained execution and record dropped samples. Preserve operation results independently of collector availability.
- Correlate asynchronous submissions with completions using operation IDs and the clock policy. Treat durations across independent clocks as unavailable unless the profile supplies a valid correlation.

## Implementation sequence

1. Instrument entry, dispatch and completion points allowed by the runtime. Record metadata with samples and aggregate before export where the policy allows.
2. Check that disabling collection preserves results. Use known workloads to validate units, phase attribution and drop counts.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Collector queue fills | Drop or aggregate according to policy and increment the loss counter. |
| Two clock domains lack synchronization evidence | Keep phase observations separate. |
| Profile disallows instrumentation in a critical path | Disable that probe for the selected profile. |
