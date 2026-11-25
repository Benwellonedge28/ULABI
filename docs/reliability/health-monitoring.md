# ULABI Health and Readiness Monitoring

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Runtime lifecycle](../runtime/runtime-interface.md), [Fault detection](fault-detection.md), [Telemetry](../observability/telemetry.md)

## Scope

Define instance health observations used by admission, diagnostics and reliability policy.

## Contract

- Represent startup, liveness, readiness and degradation separately with an instance generation, observation time, deadline, and safe reason code. Stale observations become unknown under the configured policy.
- Use bounded checks and declare which dependencies influence readiness. A healthy dependency and a ready component are separate observations.
- Publish observations without expanding access or exposing protected details. Prevent probe work from exhausting the component resources it measures.

## Implementation sequence

1. Register profile-defined checks and run them within an independent probe budget. Evaluate freshness and thresholds before updating routing readiness.
2. Emit state changes and evidence for the detector. Keep supervision separate from decisions to isolate, restart, or rollback.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Health probe exceeds its deadline | Report timeout and apply the configured readiness threshold. |
| Instance is alive but mandatory dependency is down | Mark readiness according to its dependency policy. |
| Old generation sends a healthy result | Discard it for the current instance. |
