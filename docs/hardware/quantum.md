I inspected the ULABI repository first, rather than treating the proposed architecture as already implemented. The current ULABI-DESIGN.md is the architectural authority and establishes language/runtime/platform/architecture neutrality, the Minimal Core + Profiles model, capability discovery, transport independence, explicit semantics, compatibility, hardware interoperability, and implementation independence.

I also checked the existing hardware/distributed/compatibility material. docs/hardware/quantum.md is currently only a placeholder, while the GPU and FPGA documents already establish the pattern that hardware-specific specifications should define the semantic hardware boundary, not duplicate generic ABI, memory, security, or interoperability rules.

The existing future-architectures.md is also important: it defines the extension mechanism for architectures that do not fit CPU/GPU/NPU/FPGA/quantum categories, so the quantum document should own quantum-specific execution semantics, not the generic future-architecture mechanism.

I would therefore make docs/hardware/quantum.md the authoritative quantum-hardware contract and keep the implementation work behind explicit interfaces.

docs/hardware/quantum.md

Copy this as the complete file:

ULABI Quantum Hardware Interface Specification

Document: "docs/hardware/quantum.md"
Project: ULABI
Full Name: Universal Language Application Binary Interface
Specification Family: Hardware
Component: Quantum Computing Interface
Status: Architecture / Design Specification
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Normative Authority:

- "ULABI-DESIGN.md"
- "ULABI-SPEC.md"
- "ULABI-VERSIONING.md"

License: Apache-2.0

---

1. Purpose

This specification defines the ULABI semantic contract for interfacing with quantum-computing hardware and quantum execution systems.

The purpose is not to define one universal quantum computer.

It defines the boundary between ULABI and quantum implementations while preserving the independence of:

- quantum hardware vendors;
- quantum instruction sets;
- quantum programming languages;
- quantum compilers;
- quantum runtimes;
- operating systems;
- control electronics;
- simulators;
- cloud quantum services;
- classical host processors;
- quantum architectures.

The conceptual architecture is:

                    ULABI Interface
                           |
                           v
                 Quantum ULABI Adapter
                           |
             +-------------+-------------+
             |             |             |
        Quantum Job    Quantum Data   Capabilities
             |             |             |
             +-------------+-------------+
                           |
                  Quantum Runtime
                           |
             +-------------+-------------+
             |                           |
      Classical Control             Quantum Device
             |                           |
             +-------------+-------------+
                           |
                  Physical Quantum System

A quantum implementation MAY be:

- superconducting;
- trapped-ion;
- neutral-atom;
- photonic;
- spin-based;
- semiconductor-based;
- topological;
- annealing-oriented;
- measurement-based;
- fault-tolerant;
- noisy intermediate-scale;
- simulator-backed;
- hybrid classical/quantum;
- distributed quantum;
- a future architecture.

ULABI semantics MUST remain independent of the physical realization.

---

2. Fundamental Principle

The fundamental rule is:

«ULABI defines the semantic quantum-computation contract; the quantum adapter defines how that contract is realized by a particular quantum system.»

Therefore:

ULABI Operation
      |
      +---- Classical implementation
      |
      +---- Quantum implementation A
      |
      +---- Quantum implementation B
      |
      +---- Quantum simulator
      |
      +---- Hybrid implementation
      |
      +---- Future quantum architecture

A quantum implementation MUST preserve the declared ULABI semantics.

Vendor-specific mechanisms MAY include:

- quantum instruction sets;
- pulse schedules;
- gate decompositions;
- control electronics;
- qubit connectivity;
- calibration data;
- error-correction hardware;
- measurement electronics;
- cryogenic control;
- compiler-specific optimizations.

Those mechanisms remain implementation-specific unless standardized through a ULABI quantum profile.

---

3. Scope

This specification defines the ULABI quantum interface for:

1. quantum device identity;
2. quantum device capabilities;
3. quantum interface identity;
4. quantum execution contexts;
5. quantum programs;
6. quantum circuits;
7. quantum operations;
8. quantum gates;
9. parameterized gates;
10. qubits;
11. logical qubits;
12. physical qubits;
13. qubit mapping;
14. quantum registers;
15. classical registers;
16. quantum measurement;
17. measurement results;
18. probability distributions;
19. observables;
20. expectation values;
21. state preparation;
22. state evolution;
23. state reset;
24. circuit submission;
25. job submission;
26. asynchronous execution;
27. cancellation;
28. execution deadlines;
29. shot-based execution;
30. deterministic execution where applicable;
31. nondeterministic execution;
32. quantum randomness;
33. quantum memory;
34. quantum/classical data boundaries;
35. hybrid execution;
36. quantum resource accounting;
37. quantum execution limits;
38. connectivity constraints;
39. fidelity declarations;
40. noise declarations;
41. error reporting;
42. calibration identity;
43. quantum module identity;
44. secure loading;
45. provenance;
46. verification;
47. fault detection;
48. recovery;
49. compatibility;
50. capability negotiation;
51. conformance testing.

This specification does not define:

- a universal quantum computer;
- a universal qubit technology;
- a universal quantum instruction set;
- a universal gate set;
- a quantum programming language;
- a specific quantum compiler;
- a specific quantum cloud provider;
- a specific quantum hardware vendor;
- a particular error-correction code;
- a particular quantum networking protocol;
- a particular cryptographic protocol;
- generic serialization;
- generic distributed consensus.

Those remain outside the ownership of this document.

---

4. Relationship to Other ULABI Specifications

The quantum layer is a hardware profile above the generic ULABI contract.

ULABI-DESIGN.md
       |
ULABI-SPEC.md
       |
ULABI Core ABI
       |
Hardware Interface
       |
Quantum Interface
       |
Quantum ULABI Adapter
       |
Quantum Runtime
       |
Quantum Hardware

The quantum specification integrates with:

- "docs/abi/core-abi.md"
- "docs/abi/calling-convention.md"
- "docs/abi/data-types.md"
- "docs/abi/memory-model.md"
- "docs/runtime/runtime-interface.md"
- "docs/runtime/async-model.md"
- "docs/runtime/concurrency.md"
- "docs/runtime/resource-management.md"
- "docs/memory/memory-safety.md"
- "docs/security/security-model.md"
- "docs/security/capability-security.md"
- "docs/security/secure-loading.md"
- "docs/security/supply-chain-security.md"
- "docs/compatibility/feature-negotiation.md"
- "docs/compatibility/capability-discovery.md"
- "docs/compatibility/graceful-degradation.md"
- "docs/observability/diagnostics.md"
- "docs/observability/tracing.md"
- "docs/observability/telemetry.md"
- "docs/reliability/fault-detection.md"
- "docs/reliability/fault-isolation.md"
- "docs/reliability/recovery.md"
- "docs/reliability/rollback.md"
- "docs/distributed/distributed-abi.md"
- "docs/distributed/remote-calls.md"
- "docs/distributed/serialization.md"
- "docs/standards/conformance.md"
- "docs/standards/test-suite.md"
- "docs/standards/certification.md"

Those documents retain ownership of their generic concepts.

This document defines their quantum realization.

It MUST NOT redefine generic semantics already owned by those specifications.

---

5. Quantum Device Identity

Every ULABI quantum implementation MUST expose a stable device identity.

Conceptually:

QuantumDeviceIdentity {
    device_id
    vendor
    architecture
    device_family
    device_model
    device_revision
    topology
    qubit_capacity
    capability_set
}

Device identity MUST be distinguishable from:

- host CPU identity;
- operating-system identity;
- quantum compiler identity;
- quantum runtime identity;
- driver identity;
- calibration identity;
- job identity.

A physical device MAY have multiple ULABI interfaces or profiles.

---

6. Quantum Architecture Identity

The physical technology MUST be separately identifiable.

Examples include:

Superconducting
TrappedIon
NeutralAtom
Photonic
Spin
Topological
Annealing
MeasurementBased
Hybrid
Simulator
Other

Architecture identity MUST NOT itself imply ULABI compatibility.

Compatibility MUST be established through:

- supported profile;
- interface version;
- capabilities;
- conformance level.

---

7. Quantum Capability Model

Quantum capabilities MUST be discoverable.

Possible capabilities include:

QuantumExecution
CircuitExecution
ParameterizedGates
Measurement
MidCircuitMeasurement
ConditionalExecution
DynamicCircuits
StatePreparation
Reset
AmplitudeAccess
ExpectationValue
Sampling
StateVectorSimulation
DensityMatrixSimulation
QuantumMemory
LogicalQubits
ErrorCorrection
FaultTolerance
Teleportation
Entanglement
QuantumRandomness
ParallelExecution
HybridExecution

Capability identifiers MUST be stable.

Vendor-specific names MUST NOT be required for Core interoperability.

---

8. Capability Discovery

Quantum capability discovery MUST integrate with:

"docs/compatibility/capability-discovery.md"

The sequence is:

Quantum Device
      |
      v
Capability Discovery
      |
      +---- Supported
      |
      +---- Unsupported
      |
      +---- Conditional
      |
      +---- Emulated
      |
      +---- Experimental

An implementation MUST NOT silently assume an optional capability exists.

If a required capability is unavailable, the implementation MUST:

1. use an explicitly permitted fallback;
2. negotiate an alternative;
3. report incompatibility;
4. or reject execution.

---

9. Quantum Interface Identity

Every ULABI-visible quantum interface MUST have a stable identity.

Conceptually:

QuantumInterfaceIdentity {
    interface_id
    version
    semantic_contract
    data_schema
    execution_model
    capability_requirements
    resource_requirements
}

Changing the semantic contract requires:

- a compatible version; or
- a new interface identity,

according to ULABI versioning rules.

---

10. Quantum Execution Model

Quantum execution differs fundamentally from conventional CPU execution.

A ULABI quantum interface MUST therefore distinguish:

Program Construction
        |
        v
Compilation / Lowering
        |
        v
Execution Submission
        |
        v
Quantum Processing
        |
        v
Measurement
        |
        v
Result Materialization

A quantum operation MUST NOT be assumed to behave like an ordinary synchronous CPU instruction.

---

11. Quantum Program

A quantum program is a semantic description of quantum computation.

Conceptually:

QuantumProgram {
    program_id
    interface_id
    version
    quantum_inputs
    classical_inputs
    operations
    measurements
    outputs
    capability_requirements
    resource_requirements
}

The representation MAY be:

- circuit;
- intermediate representation;
- instruction sequence;
- graph;
- measurement-based program;
- pulse program;
- implementation-specific representation.

The semantic contract remains authoritative.

---

12. Quantum Circuit

A circuit is an ordered or dependency-defined collection of quantum operations.

Conceptually:

QuantumCircuit {
    circuit_id
    qubits
    classical_bits
    operations
    measurements
    dependencies
}

The circuit representation MUST explicitly define:

- operation ordering;
- qubit references;
- classical dependencies;
- measurement points;
- control flow where supported.

---

13. Qubit Identity

ULABI MUST distinguish physical and logical qubits.

Conceptually:

PhysicalQubit {
    qubit_id
    device_id
    topology_position
}

and:

LogicalQubit {
    logical_id
    error_correction_profile
    physical_mapping
}

A logical qubit MAY map to:

- one physical qubit;
- multiple physical qubits;
- a dynamic set of physical qubits.

The mapping MUST NOT alter logical semantics.

---

14. Qubit Ownership

Quantum resources MUST have explicit ownership.

Possible ownership states include:

Available
Reserved
Allocated
Executing
Measured
Released
Failed

A qubit MUST NOT be concurrently allocated to incompatible operations.

---

15. Quantum Registers

A quantum register is an ordered semantic collection of qubits.

Conceptually:

QuantumRegister {
    register_id
    length
    qubits
}

A register MAY contain:

- physical qubits;
- logical qubits.

The representation MUST NOT assume contiguous physical hardware.

---

16. Classical Registers

Hybrid quantum systems require classical state.

Conceptually:

ClassicalRegister {
    register_id
    type
    width
    values
}

Classical values MUST use ULABI semantic types where they cross the ULABI boundary.

---

17. Quantum Operations

A quantum operation is a semantic operation on one or more quantum resources.

Conceptually:

QuantumOperation {
    operation_id
    operation_type
    target_qubits
    control_qubits
    parameters
    dependencies
}

Operations MAY include:

- gate application;
- measurement;
- reset;
- state preparation;
- conditional operation;
- barrier;
- synchronization;
- classical computation;
- observation.

---

18. Quantum Gates

A quantum gate MUST have stable semantic identity.

Conceptually:

QuantumGate {
    gate_id
    version
    arity
    parameters
    semantic_definition
}

The physical implementation MAY decompose a gate into another gate set.

For example:

ULABI Gate
    |
    v
Target Decomposition
    |
    +-- Gate A
    +-- Gate B
    +-- Gate C

The decomposition MUST preserve the declared semantics within the declared fidelity/error contract.

---

19. Parameterized Operations

Parameterized gates MUST explicitly define:

- parameter types;
- parameter ranges;
- units where applicable;
- precision;
- allowed values;
- invalid-value behavior.

Example:

Rotation(
    axis: Axis,
    angle: Float
)

An implementation MUST NOT silently reinterpret parameter units.

---

20. Quantum Measurement

Measurement is a semantic boundary between quantum and classical state.

Conceptually:

Quantum State
      |
      v
Measurement
      |
      v
Classical Result

Measurement MUST explicitly define:

- target qubits;
- measurement basis;
- result type;
- destructive/non-destructive behavior where supported;
- ordering;
- timing;
- failure behavior.

---

21. Measurement Results

A measurement result MAY be represented as:

MeasurementResult {
    measurement_id
    classical_values
    shot_index
    metadata
}

The semantic result MUST be independent of the hardware-specific readout mechanism.

---

22. Sampling and Shots

A quantum program MAY be executed repeatedly.

Conceptually:

Circuit
  |
  +-- Shot 0
  +-- Shot 1
  +-- Shot 2
  +-- ...
  +-- Shot N

A shot count MUST be explicitly defined.

The implementation MUST NOT silently change the requested number of shots except where the interface contract permits it.

---

23. Quantum Randomness

Quantum measurement MAY produce nondeterministic results.

ULABI MUST distinguish:

Deterministic

from:

Probabilistic

and:

HardwareQuantumRandom

A probabilistic result MUST NOT be presented as deterministic.

Where quantum randomness crosses the ULABI boundary, the implementation MUST declare its semantic source.

---

24. Probability Distributions

Where supported, an implementation MAY return a probability distribution.

Conceptually:

ProbabilityDistribution {
    outcomes
    probabilities
}

The representation MUST define:

- outcome identity;
- probability precision;
- normalization;
- ordering;
- error bounds.

If probabilities are estimated rather than exact, that fact MUST be declared.

---

25. Expectation Values

An implementation MAY expose observable expectation values.

Conceptually:

ExpectationValue {
    observable
    value
    uncertainty
    confidence
}

The interface MUST distinguish:

- exact values;
- sampled estimates;
- hardware estimates;
- simulator estimates.

---

26. Quantum State Access

Direct quantum-state access is architecture-dependent.

Possible modes include:

NoStateAccess
MeasurementOnly
StateVector
DensityMatrix
Tomography
PartialStateAccess

Physical quantum hardware MUST NOT be assumed to expose its complete internal quantum state.

If a state representation is unavailable, the implementation MUST report that limitation.

---

27. State Preparation

State preparation MUST define:

- target qubits;
- desired state;
- preparation method;
- fidelity requirements;
- resource requirements;
- failure behavior.

The implementation MUST NOT claim exact state preparation when only approximate preparation is available.

---

28. Reset

A quantum reset operation MUST explicitly define its resulting state.

Conceptually:

Unknown Quantum State
        |
      Reset
        |
        v
Defined Initial State

Reset MUST define:

- target;
- resulting state;
- completion semantics;
- resource usage;
- failure behavior.

---

29. Quantum Connectivity

Quantum hardware commonly imposes topology constraints.

The adapter MUST expose connectivity when it affects execution.

Conceptually:

Connectivity {
    qubits
    allowed_interactions
}

A compiler or runtime MAY map logical interactions to physical connectivity.

The mapping MUST preserve semantic correctness.

---

30. Qubit Mapping

Logical-to-physical mapping MUST be explicit where required.

Logical Qubit
      |
      v
Mapping Layer
      |
      v
Physical Qubit

The mapping MAY change between executions.

The logical interface MUST remain stable.

---

31. Quantum Execution Job

A submitted execution MUST have a stable job identity.

Conceptually:

QuantumJob {
    job_id
    program_id
    device_id
    execution_profile
    resource_request
    status
}

Possible states:

Created
Queued
Running
Completed
Failed
Cancelled
Expired
TimedOut
Aborted

Invalid state transitions MUST be rejected.

---

32. Asynchronous Execution

Quantum jobs SHOULD support asynchronous execution.

Conceptually:

Submit
  |
  v
Job ID
  |
  +---- Query
  |
  +---- Wait
  |
  +---- Cancel
  |
  v
Result

The runtime MUST NOT require a blocking call for long-running quantum jobs.

---

33. Cancellation

Cancellation MUST define its semantic boundary.

Possible states:

CancellableBeforeExecution
CancellableDuringExecution
CancellationPending
CancellationUnsupported

If a running physical operation cannot be interrupted safely, cancellation MAY mean:

«prevent further execution after the current non-interruptible operation.»

That behavior MUST be explicitly documented.

---

34. Timeouts

Quantum jobs MAY specify:

- queue timeout;
- execution timeout;
- result timeout;
- device reservation timeout.

Timeout behavior MUST be explicit.

A timeout MUST NOT be silently interpreted as successful completion.

---

35. Resource Model

Quantum execution MUST support explicit resource accounting.

Resources MAY include:

- physical qubits;
- logical qubits;
- circuit depth;
- gate count;
- measurement count;
- shots;
- execution time;
- device reservation time;
- classical memory;
- quantum memory;
- calibration resources;
- error-correction resources.

Conceptually:

QuantumResourceBudget {
    qubits
    logical_qubits
    depth
    operations
    measurements
    shots
    duration
}

Implementations MUST enforce declared resource limits.

---

36. Resource Exhaustion

If a quantum job exceeds its resource budget, the implementation MUST:

1. reject before execution where detectable;
2. terminate safely;
3. report an explicit resource error;
4. or use an explicitly permitted degradation strategy.

It MUST NOT silently exceed a declared hard limit.

---

37. Noise Model

Quantum hardware MAY introduce noise.

The implementation MUST distinguish:

Ideal
Noisy
Calibrated
Estimated
Unknown

Where a noise model is exposed, it SHOULD identify:

- error categories;
- estimated error rates;
- calibration epoch;
- confidence;
- validity period.

Noise metadata MUST NOT be presented as a guarantee of exact physical behavior.

---

38. Fidelity

Where relevant, an implementation MAY declare fidelity or error bounds.

Conceptually:

ExecutionQuality {
    fidelity
    error_bound
    confidence
    calibration_id
}

The semantics of these values MUST be explicitly defined.

An implementation MUST NOT claim stronger accuracy than its measurement or calibration process supports.

---

39. Calibration Identity

Calibration state MAY materially affect execution.

Where calibration affects the semantic guarantees of a quantum operation, the implementation SHOULD expose:

CalibrationIdentity {
    calibration_id
    timestamp
    validity
    device_id
}

Calibration identifiers MUST be distinguishable from device identities.

---

40. Quantum Error Handling

Quantum errors MUST use the ULABI error model.

Errors MAY include:

QuantumDeviceUnavailable
QubitUnavailable
InvalidCircuit
InvalidGate
InvalidParameter
UnsupportedOperation
ConnectivityViolation
ResourceExhausted
CalibrationUnavailable
ExecutionTimeout
MeasurementFailure
QuantumReadoutError
QuantumRuntimeError
QuantumCompilationError
QuantumHardwareFault

Vendor-specific error codes MAY be included as diagnostic metadata.

They MUST NOT replace the stable ULABI error category.

---

41. Hybrid Classical/Quantum Execution

ULABI MUST support hybrid execution.

Classical Program
       |
       +---- Classical Computation
       |
       +---- Quantum Job
       |          |
       |          v
       |      Quantum Result
       |
       +---- Classical Decision

A hybrid interface MUST explicitly define:

- classical/quantum boundaries;
- data transfer;
- synchronization;
- latency expectations;
- failure propagation;
- cancellation;
- ownership.

---

42. Classical Control

A quantum device MAY require classical control operations.

Classical control MUST remain distinguishable from quantum operations.

Examples include:

- job scheduling;
- calibration;
- device reset;
- resource reservation;
- result retrieval.

Control operations MUST NOT be represented as quantum gates unless their semantics genuinely are quantum operations.

---

43. Quantum Memory

Quantum memory is distinct from ordinary classical memory.

A quantum-memory resource MUST explicitly declare:

- capacity;
- lifetime;
- coherence requirements;
- access model;
- ownership;
- fidelity constraints;
- reset semantics;
- measurement behavior.

A classical pointer MUST NOT automatically constitute a quantum-memory reference.

---

44. Quantum/Classic Data Boundary

ULABI MUST distinguish:

Quantum Value

from:

Classical Value

Quantum values MUST NOT be serialized as ordinary classical values unless a defined measurement or representation operation permits it.

A quantum resource MAY be represented by a ULABI "Handle".

The handle MUST NOT expose raw hardware addresses.

---

45. Quantum Handles

Quantum resources MAY use opaque handles.

Conceptually:

QuantumHandle {
    resource_id
    resource_type
    interface_id
    lifetime
    capability
}

Handles MUST obey the generic ULABI ownership and capability rules.

---

46. Quantum Scheduling

Quantum scheduling MUST explicitly represent:

- queue state;
- priority;
- reservation;
- resource availability;
- execution ordering;
- deadline;
- cancellation.

The implementation MUST NOT promise deterministic start times unless it can provide them.

---

47. Determinism

Quantum systems MAY be nondeterministic by design.

ULABI MUST distinguish:

DeterministicProgram

from:

ProbabilisticProgram

and:

HardwareNondeterministicProgram

A deterministic semantic contract MUST NOT be advertised if physical execution violates it.

---

48. Real-Time Constraints

Quantum execution MAY declare timing classes:

BestEffort
BoundedLatency
DeadlineConstrained
RealTime

A system MUST NOT advertise "RealTime" merely because it can execute a quantum circuit quickly.

A real-time profile MUST define:

- deadline;
- scheduling guarantee;
- execution bound;
- failure behavior.

---

49. Quantum Module

A quantum module is a deployable collection of quantum interfaces.

Conceptually:

QuantumModule {
    module_id
    version
    interfaces
    target_profiles
    capability_requirements
    resource_requirements
    integrity_metadata
}

A module MAY contain:

- circuits;
- kernels;
- quantum functions;
- measurement definitions;
- metadata;
- classical support functions.

---

50. Quantum Module Representation

ULABI MUST NOT mandate one quantum executable format.

A module MAY be represented by:

- quantum IR;
- circuit IR;
- gate sequence;
- pulse description;
- native quantum instruction representation;
- implementation-specific binary;
- signed module;
- simulator representation.

The adapter MUST expose the semantic contract independently from representation.

---

51. Secure Loading

Quantum modules MUST integrate with:

- "docs/security/secure-loading.md"
- "docs/security/supply-chain-security.md"
- "docs/security/capability-security.md"

Before execution, an implementation MUST be able to validate, as applicable:

- module identity;
- integrity;
- authenticity;
- interface compatibility;
- capability requirements;
- target compatibility;
- resource requirements.

An untrusted quantum module MUST NOT automatically gain unrestricted device capabilities.

---

52. Quantum Capabilities and Security

Quantum resources MAY be capability-protected.

Examples:

AllocateQubit
ExecuteCircuit
MeasureQubit
AccessQuantumMemory
ReserveDevice
AccessCalibration
LoadQuantumModule
ReadResults

Capabilities MUST be granted according to the generic ULABI security model.

Possession of a device identifier MUST NOT grant control of the device.

---

53. Multi-Tenant Quantum Devices

A quantum device MAY be shared between multiple applications.

Isolation MUST prevent unauthorized access to:

- another job's quantum resources;
- another job's measurement results;
- another job's private metadata;
- protected calibration information;
- restricted device controls.

The implementation MUST define whether qubit reuse occurs between jobs and how state reset is guaranteed.

---

54. Quantum State Isolation

Before reallocating a quantum resource, the implementation MUST ensure that the resource is placed into a state consistent with the new allocation contract.

A stale quantum state MUST NOT leak across security boundaries.

Where complete reset cannot be guaranteed, the resource MUST remain unavailable until the required isolation condition is satisfied.

---

55. Observability

Quantum execution SHOULD expose standardized observability metadata through the ULABI observability specifications.

Possible metadata includes:

- job ID;
- device ID;
- interface ID;
- circuit ID;
- calibration ID;
- execution duration;
- queue duration;
- shot count;
- error category;
- resource usage;
- execution status.

Observability MUST NOT automatically expose protected quantum data.

---

56. Tracing

Quantum operations MAY participate in ULABI distributed and local traces.

A trace SHOULD distinguish:

Classical Operation
Quantum Submission
Quantum Execution
Measurement
Result Retrieval

Trace identifiers MUST remain independent of vendor-specific job identifiers.

---

57. Failure Detection

Quantum failures MUST be detected through explicit mechanisms.

Possible failure sources include:

- device failure;
- qubit failure;
- calibration failure;
- communication failure;
- compilation failure;
- invalid operation;
- resource exhaustion;
- timeout;
- measurement failure.

Failure detection MUST integrate with:

"docs/reliability/fault-detection.md"

---

58. Failure Isolation

A failure MUST be isolated according to its scope.

Possible scopes:

Operation
Circuit
Job
Execution Context
Qubit
Device Region
Device
Runtime
Host

A failure in one quantum job MUST NOT automatically terminate unrelated jobs unless the device or security model requires it.

---

59. Recovery

Quantum recovery MUST be bounded and policy-controlled.

A conforming implementation MUST NOT arbitrarily modify quantum programs or hardware configuration in response to a failure.

The recovery sequence is:

Failure Detected
      |
      v
Evidence Collected
      |
      v
Known Recovery Policy?
   +------+------+
   |             |
  YES            NO
   |             |
Recover        Escalate
   |
   v
Verify
   |
   v
Healthy?
 +---+---+
 |       |
YES      NO
 |       |
Done   Rollback / Escalate

Recovery MUST integrate with:

- "docs/reliability/recovery.md"
- "docs/reliability/rollback.md"
- "docs/reliability/fault-isolation.md"

---

60. Quantum Device Reset

A device-level reset MUST be treated as a potentially destructive operation.

It MUST define:

- scope;
- affected jobs;
- affected resources;
- state destruction;
- calibration impact;
- security consequences;
- recovery behavior.

A device reset MUST NOT silently occur as a normal recovery action unless policy explicitly permits it.

---

61. Graceful Degradation

Quantum implementations MAY provide fallbacks.

Examples:

Quantum Hardware
       |
       +---- Supported
       |
       +---- Quantum Simulator
       |
       +---- Classical Approximation
       |
       +---- Reject

A fallback MUST NOT silently change semantics.

If the fallback is approximate, the approximation MUST be declared.

---

62. Quantum Simulator

A simulator MAY implement the same ULABI quantum interface.

The implementation MUST identify itself as a simulator.

Conceptually:

QuantumDeviceIdentity {
    architecture = Simulator
}

A simulator MUST NOT claim physical-hardware guarantees it cannot provide.

---

63. Simulator Compatibility

A simulator MAY support:

- exact state-vector simulation;
- density-matrix simulation;
- stabilizer simulation;
- tensor-network simulation;
- noisy simulation;
- approximate simulation.

The semantic profile MUST declare which behavior is provided.

---

64. Distributed Quantum Execution

Quantum execution MAY span:

- multiple quantum devices;
- classical control systems;
- remote quantum processors;
- quantum network components.

Distributed quantum semantics MUST integrate with the distributed ULABI specifications.

A remote quantum device MUST NOT be treated as an ordinary local memory resource.

Network latency, failure, security, and availability MUST remain explicit.

---

65. Quantum Network Resources

If future ULABI quantum networking profiles are introduced, resources MAY include:

- entangled pairs;
- quantum channels;
- quantum memories;
- classical control channels.

Those resources MUST be represented through explicit capabilities and interfaces.

This document does not define the complete quantum-network protocol.

---

66. Serialization Boundary

Quantum state MUST NOT be serialized as ordinary bytes unless the semantic contract explicitly defines the representation.

Possible representations include:

- measurement results;
- classical parameters;
- circuit descriptions;
- state tomography;
- implementation-specific state representations.

A serialized circuit is not equivalent to a serialized physical quantum state.

Generic serialization remains owned by:

"docs/distributed/serialization.md"

---

67. Compatibility

Quantum interface compatibility MUST be determined by:

- interface identity;
- interface version;
- type compatibility;
- operation compatibility;
- capability compatibility;
- resource compatibility;
- execution-model compatibility.

Device model equality MUST NOT be required for semantic compatibility.

---

68. Backward Compatibility

A newer quantum implementation SHOULD continue supporting older compatible interfaces.

Breaking changes MUST use:

- a new interface version;
- a new interface identity;
- or an explicitly negotiated incompatible profile.

Older clients MUST NOT be forced to understand unknown optional features.

---

69. Forward Compatibility

Unknown optional quantum features MAY be ignored when the contract permits.

Unknown mandatory quantum features MUST produce safe incompatibility behavior.

An implementation MUST NOT silently assume that an unknown quantum operation is equivalent to a known operation.

---

70. Versioning

This specification uses ULABI versioning rules.

Quantum interfaces SHOULD independently identify:

ULABI Version
Quantum Profile Version
Interface Version
Device Architecture Version
Module Version
Calibration Version

These versions MUST NOT be conflated.

---

71. Security Requirements

A conforming implementation MUST:

1. authenticate protected quantum modules where required;
2. validate interface compatibility;
3. enforce quantum resource ownership;
4. enforce capability restrictions;
5. isolate jobs according to policy;
6. prevent unauthorized measurement access;
7. prevent unauthorized device control;
8. validate execution requests;
9. reject malformed quantum programs;
10. prevent resource-limit bypass;
11. protect sensitive device metadata;
12. preserve ULABI security invariants.

---

72. Safety Requirements

Quantum hardware MAY have physical operational constraints.

The adapter MUST enforce applicable:

- device safety constraints;
- thermal constraints;
- timing constraints;
- calibration constraints;
- control constraints;
- resource constraints.

A ULABI caller MUST NOT be able to bypass hardware safety rules merely because it possesses a ULABI interface.

---

73. Resource Safety

Quantum resource exhaustion MUST be bounded.

Implementations MUST NOT assume:

- unlimited qubits;
- unlimited shots;
- unlimited execution time;
- unlimited memory;
- unlimited queue capacity;
- unlimited calibration availability.

Resource limits MUST be explicit.

---

74. Error Mapping

Vendor-specific quantum errors MUST map to stable ULABI errors.

Conceptually:

Vendor Error
     |
     v
Quantum Adapter
     |
     v
ULABI Error
     |
     +---- category
     +---- code
     +---- diagnostic metadata

Vendor information MAY be retained as diagnostic metadata.

The semantic ULABI error category remains authoritative.

---

75. Invariants

A conforming quantum implementation MUST preserve the following invariants.

Invariant 1 — Semantic Preservation

A supported quantum operation MUST preserve its declared ULABI semantics.

Invariant 2 — Resource Ownership

A quantum resource MUST NOT be simultaneously owned by incompatible operations.

Invariant 3 — Capability Enforcement

An operation requiring a capability MUST NOT execute without that capability.

Invariant 4 — Measurement Honesty

Probabilistic or noisy results MUST NOT be represented as exact deterministic results.

Invariant 5 — Version Integrity

An implementation MUST NOT claim compatibility with an incompatible interface.

Invariant 6 — State Isolation

Quantum resources MUST NOT leak protected state across security boundaries.

Invariant 7 — Bounded Recovery

Recovery MUST remain within an explicitly authorized policy.

Invariant 8 — No Hidden Transport Semantics

Remote execution MUST NOT silently become local execution or vice versa when that distinction affects observable behavior.

Invariant 9 — No Raw Hardware Dependency

ULABI interfaces MUST NOT require callers to depend on vendor-specific physical addresses or control mechanisms.

Invariant 10 — Implementation Independence

The semantic contract MUST remain independent of the physical quantum implementation.

---

76. Failure Modes

Implementations MUST define behavior for at least:

Failure| Required behavior
Invalid circuit| Reject
Unsupported gate| Reject or negotiated lowering
Invalid parameter| Reject
Missing capability| Fallback, negotiate, or reject
Qubit unavailable| Queue, remap, or reject according to policy
Resource exhaustion| Explicit failure
Device unavailable| Explicit failure
Calibration unavailable| Reject or approved fallback
Execution timeout| Explicit timeout
Measurement failure| Explicit error
Device fault| Isolate and recover/escalate
Security violation| Reject and report
Module integrity failure| Reject
Version incompatibility| Reject or negotiate
Cancellation failure| Report actual state
Remote communication failure| Explicit distributed error

---

77. Recovery Semantics

Recovery MUST NOT fabricate successful results.

If execution fails before a valid result is produced:

Result = Failure

must remain distinguishable from:

Result = Success

A partially executed quantum job MUST NOT automatically be represented as a completed successful job.

---

78. Transactional Semantics

Quantum execution SHOULD be treated as transactional at the job boundary where practical.

The implementation SHOULD distinguish:

Accepted
Started
PartiallyExecuted
Completed
Failed
Cancelled

A failed job MUST NOT be reported as successful merely because some operations completed.

---

79. Conformance Requirements

A conforming quantum implementation MUST implement or explicitly declare support for:

- device identity;
- capability discovery;
- quantum interface identity;
- quantum program identity;
- quantum operation identity;
- qubit allocation;
- measurement;
- execution jobs;
- result handling;
- error handling;
- version compatibility;
- resource limits;
- security enforcement.

Optional capabilities MUST be explicitly declared.

---

80. Conformance Levels

Quantum implementations MAY declare:

ULABI-Quantum-Core
ULABI-Quantum-Extended
ULABI-Quantum-Hybrid
ULABI-Quantum-FaultTolerant
ULABI-Quantum-Distributed

A conformance level MUST NOT imply support for features that have not been tested.

---

81. Required Conformance Tests

The ULABI test suite SHOULD include tests for:

1. device identity;
2. capability discovery;
3. interface identity;
4. interface versioning;
5. qubit allocation;
6. qubit release;
7. gate validation;
8. parameter validation;
9. circuit submission;
10. circuit rejection;
11. measurement;
12. shot execution;
13. result ordering;
14. asynchronous execution;
15. cancellation;
16. timeout;
17. resource exhaustion;
18. unsupported capability;
19. error mapping;
20. secure module loading;
21. capability enforcement;
22. state isolation;
23. simulator identification;
24. hybrid execution;
25. distributed execution where supported;
26. recovery;
27. rollback;
28. backward compatibility;
29. forward compatibility;
30. malformed-input rejection.

---

82. Reference Test Scenarios

Scenario A — Valid Circuit

Create circuit
      |
Allocate qubits
      |
Submit
      |
Execute
      |
Measure
      |
Return result

Expected:

Success

with valid measurement results.

Scenario B — Unsupported Gate

Submit
   |
Unsupported Gate
   |
Reject / Lower / Negotiate

The implementation MUST NOT silently substitute an incompatible operation.

Scenario C — Missing Qubit

Allocate
   |
Qubit unavailable
   |
Queue / Remap / Reject

The behavior MUST follow the declared policy.

Scenario D — Device Failure

Execution
   |
Device Failure
   |
Isolate
   |
Recover / Rollback / Escalate

No false successful result may be produced.

---

83. Reference Data Structures

The following are semantic structures rather than language-specific implementation types:

QuantumDeviceIdentity
QuantumInterfaceIdentity
QuantumCapability
QuantumProgram
QuantumCircuit
QuantumOperation
QuantumGate
QuantumRegister
LogicalQubit
PhysicalQubit
ClassicalRegister
MeasurementResult
ProbabilityDistribution
ExpectationValue
QuantumJob
QuantumResourceBudget
CalibrationIdentity
QuantumModule
QuantumHandle
QuantumExecutionContext
QuantumError

Implementations MAY represent these structures differently.

Their observable semantics MUST remain compatible.

---

84. Reference Interface

A conceptual ULABI quantum interface MAY look like:

interface QuantumDevice {

    discover_capabilities()
        -> Result<CapabilitySet, QuantumError>

    allocate_qubits(
        count: UInt
    )
        -> Result<QuantumRegister, QuantumError>

    submit(
        program: QuantumProgram,
        resources: QuantumResourceBudget
    )
        -> Result<QuantumJob, QuantumError>

    wait(
        job: QuantumJob
    )
        -> Result<QuantumResult, QuantumError>

    cancel(
        job: QuantumJob
    )
        -> Result<Unit, QuantumError>

    release(
        resources: QuantumRegister
    )
        -> Result<Unit, QuantumError>
}

This is illustrative.

It does not define a programming-language-specific API.

---

85. Reference Execution Flow

Application
    |
    v
ULABI Quantum Interface
    |
    v
Capability Discovery
    |
    v
Compatibility Validation
    |
    v
Resource Allocation
    |
    v
Program Validation
    |
    v
Quantum Compilation / Lowering
    |
    v
Execution Submission
    |
    v
Quantum Hardware
    |
    v
Measurement
    |
    v
Result Validation
    |
    v
ULABI Result

At every boundary:

- capabilities MUST be checked;
- ownership MUST be enforced;
- errors MUST be explicit;
- security policy MUST remain active.

---

86. Integration Contract

This file is complete independently.

No later document is required to rewrite the quantum specification merely because another ULABI document becomes more detailed.

The following ownership boundaries are fixed:

Concern| Owning specification
Generic ABI| "docs/abi/core-abi.md"
Calling convention| "docs/abi/calling-convention.md"
Generic data types| "docs/abi/data-types.md"
Generic memory| "docs/abi/memory-model.md"
Runtime| "docs/runtime/runtime-interface.md"
Async behavior| "docs/runtime/async-model.md"
Resources| "docs/runtime/resource-management.md"
Generic security| "docs/security/security-model.md"
Capabilities| "docs/security/capability-security.md"
Secure loading| "docs/security/secure-loading.md"
Capability discovery| "docs/compatibility/capability-discovery.md"
Feature negotiation| "docs/compatibility/feature-negotiation.md"
Serialization| "docs/distributed/serialization.md"
Distributed execution| "docs/distributed/distributed-abi.md"
Fault detection| "docs/reliability/fault-detection.md"
Recovery| "docs/reliability/recovery.md"
Rollback| "docs/reliability/rollback.md"
Conformance| "docs/standards/conformance.md"
Test framework| "docs/standards/test-suite.md"
Quantum hardware semantics| This document

This document MUST own only quantum-specific semantics.

---

87. Implementation Boundary

The quantum implementation MUST be replaceable without changing the ULABI semantic contract.

                    ULABI
                      |
              Quantum Interface
                      |
              +-------+-------+
              |       |       |
          Vendor A Vendor B Simulator
              |       |       |
              +-------+-------+
                      |
                 Quantum Device

The following MUST remain outside the ULABI Core:

- vendor SDKs;
- vendor driver APIs;
- vendor instruction sets;
- vendor pulse languages;
- vendor hardware addresses;
- vendor compiler internals.

---

88. Implementation Independence

ULABI quantum support MUST be implementable independently by:

- C;
- C++;
- Rust;
- Go;
- Java;
- Python;
- Swift;
- Kotlin;
- Fortran;
- Ada;
- Zamani;
- Sankofa;
- other languages.

No language-specific feature may be required by this specification.

---

89. Extension Policy

Quantum features not sufficiently mature for the base profile MUST be introduced as extensions.

Examples:

ULABI-Quantum/ErrorCorrection
ULABI-Quantum/FaultTolerance
ULABI-Quantum/DynamicCircuits
ULABI-Quantum/QuantumMemory
ULABI-Quantum/Distributed
ULABI-Quantum/LogicalQubits

Extensions MUST have:

- stable identifiers;
- versions;
- semantic definitions;
- compatibility rules;
- security implications;
- resource requirements;
- conformance tests.

---

90. Final Architectural Rule

The following rule is normative:

«ULABI defines the universal interoperability contract. Quantum hardware defines its own physical implementation. The ULABI quantum interface exists between them and MUST NOT make ULABI dependent on any particular quantum technology, vendor, compiler, runtime, operating system, or programming language.»

A quantum implementation is conformant when independent implementations can exchange quantum interface contracts, execute compatible operations, exchange results, handle failures, enforce security, and negotiate capabilities without relying on vendor-specific semantics.

---What should be done under which files

The important point is that quantum.md should be completed before its implementation modules, because it establishes the semantic contract. It should not later be rewritten just because Rust/C++/Python implementations are added.

Phase 1 — independent specification files

These are the files that establish contracts and should be finished before implementation coupling:

Priority	File	Responsibility

1	docs/hardware/quantum.md	Quantum hardware contract — this file
2	docs/hardware/future-architectures.md	Novel architecture extension mechanism
3	docs/compatibility/capability-discovery.md	Capability discovery
4	docs/compatibility/feature-negotiation.md	Negotiation
5	docs/abi/data-types.md	Cross-boundary types
6	docs/abi/memory-model.md	Memory/resource boundaries
7	docs/runtime/resource-management.md	Resource quotas/lifetimes
8	docs/security/capability-security.md	Capability enforcement
9	docs/security/secure-loading.md	Quantum module loading
10	docs/distributed/serialization.md	Circuit/result serialization
11	docs/standards/conformance.md	Conformance model
12	docs/standards/test-suite.md	Test architecture


The repository already contains the corresponding architecture/specification areas, including core ABI, calling convention, register model, compatibility, and distributed ABI documents.


---

Required code/module architecture

Because ULABI is language-independent, do not put quantum implementation code into docs/ and do not make the specification depend on Rust, C++, Python, Zamani, or Sankofa.

I recommend the implementation layer be structured as follows:

implementations/
└── quantum/
    │
    ├── core/
    │   ├── device/
    │   │   ├── quantum_device
    │   │   ├── device_identity
    │   │   ├── device_capabilities
    │   │   └── device_registry
    │   │
    │   ├── interface/
    │   │   ├── quantum_interface
    │   │   ├── interface_identity
    │   │   └── interface_version
    │   │
    │   ├── qubits/
    │   │   ├── physical_qubit
    │   │   ├── logical_qubit
    │   │   ├── qubit_register
    │   │   ├── qubit_allocator
    │   │   └── qubit_mapper
    │   │
    │   ├── circuit/
    │   │   ├── quantum_program
    │   │   ├── quantum_circuit
    │   │   ├── quantum_operation
    │   │   ├── quantum_gate
    │   │   ├── gate_registry
    │   │   └── circuit_validator
    │   │
    │   ├── measurement/
    │   │   ├── measurement
    │   │   ├── measurement_result
    │   │   ├── probability_distribution
    │   │   └── expectation_value
    │   │
    │   ├── jobs/
    │   │   ├── quantum_job
    │   │   ├── job_scheduler
    │   │   ├── job_state
    │   │   ├── cancellation
    │   │   └── timeout
    │   │
    │   ├── resources/
    │   │   ├── resource_budget
    │   │   ├── resource_manager
    │   │   └── resource_limits
    │   │
    │   ├── errors/
    │   │   ├── quantum_error
    │   │   ├── error_codes
    │   │   └── error_mapping
    │   │
    │   └── security/
    │       ├── quantum_capabilities
    │       ├── quantum_handles
    │       └── module_validation
    │
    ├── adapters/
    │   ├── simulator/
    │   ├── superconducting/
    │   ├── trapped_ion/
    │   ├── neutral_atom/
    │   ├── photonic/
    │   └── vendor/
    │
    ├── compilation/
    │   ├── quantum_lowering
    │   ├── gate_decomposition
    │   ├── qubit_mapping
    │   ├── circuit_optimization
    │   └── target_validation
    │
    ├── hybrid/
    │   ├── classical_quantum_bridge
    │   ├── hybrid_job
    │   └── hybrid_scheduler
    │
    ├── distributed/
    │   ├── remote_quantum_device
    │   ├── remote_job
    │   └── quantum_transport_adapter
    │
    └── observability/
        ├── quantum_tracing
        ├── quantum_diagnostics
        ├── quantum_metrics
        └── quantum_audit

Schemas

These should be implementation-neutral:

schemas/
└── quantum/
    ├── quantum-device.schema
    ├── quantum-interface.schema
    ├── quantum-capability.schema
    ├── quantum-program.schema
    ├── quantum-circuit.schema
    ├── quantum-operation.schema
    ├── quantum-gate.schema
    ├── quantum-register.schema
    ├── measurement-result.schema
    ├── probability-distribution.schema
    ├── expectation-value.schema
    ├── quantum-job.schema
    ├── resource-budget.schema
    ├── calibration.schema
    ├── quantum-module.schema
    └── quantum-error.schema

Conformance tests

conformance/
└── quantum/
    ├── device_identity
    ├── capability_discovery
    ├── interface_versioning
    ├── qubit_allocation
    ├── qubit_release
    ├── gate_validation
    ├── circuit_validation
    ├── measurement
    ├── sampling
    ├── result_integrity
    ├── async_execution
    ├── cancellation
    ├── timeout
    ├── resource_limits
    ├── error_mapping
    ├── security
    ├── isolation
    ├── simulator
    ├── hybrid_execution
    ├── distributed_execution
    ├── recovery
    ├── rollback
    ├── backward_compatibility
    └── forward_compatibility

Examples

examples/
└── quantum/
    ├── minimal-circuit
    ├── measurement
    ├── parameterized-circuit
    ├── multi-qubit-circuit
    ├── asynchronous-job
    ├── hybrid-classical-quantum
    ├── capability-negotiation
    ├── simulator-fallback
    └── failure-recovery

Integration order

The dependency direction should be:

ULABI-DESIGN.md
      ↓
ULABI-SPEC.md
      ↓
Core ABI
      ↓
Generic Types / Memory / Runtime / Security
      ↓
Compatibility + Capability Discovery
      ↓
Quantum Specification
      ↓
Quantum Schemas
      ↓
Quantum Core Interfaces
      ↓
Quantum Adapter Interface
      ↓
Individual Hardware Adapters
      ↓
Conformance Tests
      ↓
Reference Implementations

Do not reverse this order.

In particular, a vendor adapter should never become the source of the ULABI quantum semantics.

The repository's existing architecture explicitly follows the same separation: the hardware specifications describe hardware-specific realization while generic concerns remain owned by their respective specifications.

One important correction to the overall ULABI plan

I would not make quantum.md define a giant quantum ABI in the Core.

Quantum should be a Hardware/Quantum Profile.

That preserves the architecture already established in ULABI-DESIGN.md:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



That means an ordinary C/Rust/Python application does not need quantum support merely to be ULABI Core compliant, while a quantum-capable implementation can advertise:

ULABI Core              ✓
ULABI Types             ✓
ULABI Memory            ✓
ULABI Security          ✓
ULABI Async             ✓
ULABI Hardware          ✓
ULABI Quantum           ✓
ULABI Quantum-Hybrid    ✓
ULABI Quantum-Distributed ✓

That is the cleaner path toward making ULABI a genuinely universal standard rather than making every implementation carry every possible hardware capability.