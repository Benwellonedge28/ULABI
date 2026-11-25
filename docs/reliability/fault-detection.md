# ULABI Fault Detection

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Health monitoring](health-monitoring.md), [Fault isolation](fault-isolation.md), [Diagnostics](../observability/diagnostics.md)

## Scope

Turn validated observations into bounded, attributable fault candidates.

## Contract

- Define detection rules with source, window, threshold, freshness, and clear condition. Classify failure signals, including crash, delay, repeated boundary errors, and resource saturation.
- Associate faults with an instance generation and evidence IDs. Deduplicate observations into a fault record with first and latest times and affected resources. Distinguish unknown or stale data from a verified failure.
- Require the configured confidence or corroboration before automated actions. Detection supplies evidence and leaves authorization to reliability policy.

## Implementation sequence

1. Normalize events from lifecycle, health, and resource services. Evaluate thresholds under a bounded per-instance window.
2. Open, update or close a fault record and send it to policy evaluation. Test duplicate, reordered and stale events.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| A transient observation is below threshold | Record it without triggering the fault rule. |
| Crash is confirmed by the supervisor | Create a fault scoped to that generation. |
| Detector receives duplicate crash events | Keep one fault record and retain attributable evidence. |
