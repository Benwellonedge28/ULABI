I inspected the actual Benwellonedge28/ULABI repository first, including the current ULABI-DESIGN.md, the hardware area, and the surrounding ABI/compatibility architecture. The repository confirms that ULABI-DESIGN.md is the architectural authority and that ULABI is explicitly language-, runtime-, OS-, CPU-, and vendor-neutral.

I also checked the existing hardware documents. docs/hardware/npu.md is currently only a placeholder, as is docs/hardware/gpu.md; therefore this is the correct point to establish the NPU contract rather than retrofit or duplicate an existing specification.

The important architectural boundary is:

> ULABI defines the semantic accelerator contract; the NPU adapter defines how that contract is realized on a particular neural-processing architecture.



The NPU document therefore should not become a generic GPU document, a tensor serialization document, or a vendor-specific accelerator API.

1. What docs/hardware/npu.md should own

It should own:

NPU identity

NPU capability discovery

accelerator interface identity

neural-computation execution model

tensor execution boundaries

supported tensor shapes

data types and precision capabilities

quantization capabilities

model/image identity

model loading into accelerator memory

accelerator-local memory

host ↔ NPU transfers

DMA

zero-copy where safe

command submission

completion

asynchronous execution

batching

pipelines

operator/kernel identity

accelerator resource accounting

execution deadlines

deterministic execution declarations

real-time constraints

power/thermal capability reporting

isolation

secure model/image loading

integrity and attestation hooks

cancellation

fault handling

recovery

accelerator reset

fallback to another implementation

compatibility/version negotiation

heterogeneous accelerator systems

conformance


It should not own:

generic ULABI tensor semantics

generic serialization

generic distributed calls

generic service discovery

generic security policy

generic memory ownership

generic capability semantics

GPU-specific execution semantics

CPU instruction-set semantics

a particular NPU vendor API

a particular neural-network framework


Those belong to their respective specifications.


---

2. Complete docs/hardware/npu.md

Copy the following as the complete file.

ULABI NPU Hardware Interface Specification

Document: "docs/hardware/npu.md"
Project: ULABI
Full Name: Universal Language Application Binary Interface
Specification Family: Hardware
Component: Neural Processing Unit Interface
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

This document defines the ULABI hardware contract for Neural Processing Units (NPUs) and other neural-computation accelerators.

The purpose of this specification is not to define one universal NPU architecture.

It defines how an NPU implementation exposes neural computation, tensor processing, accelerator memory, execution resources, events, and hardware capabilities through ULABI contracts.

The conceptual architecture is:

ULABI Semantic Interface
          |
          v
    NPU ULABI Adapter
          |
          v
   NPU Execution Boundary
          |
     +----+----+
     |         |
 NPU Compute  NPU Memory
     |         |
     +----+----+
          |
     Neural Accelerator

An NPU implementation may be:

- integrated into a CPU;
- integrated into a system-on-chip;
- a discrete accelerator;
- embedded in a mobile device;
- attached through a device interconnect;
- virtualized;
- partitioned;
- shared by multiple execution contexts;
- combined with CPU/GPU/FPGA resources.

ULABI semantics MUST remain independent of the NPU vendor, hardware architecture, instruction set, compiler, model framework, and operating system.

---

2. Fundamental Principle

The fundamental rule is:

«ULABI defines the semantic neural-computation contract; an NPU adapter defines how that contract is physically realized by the target accelerator.»

Therefore:

ULABI Neural Function
        |
        +---- NPU implementation A
        |
        +---- NPU implementation B
        |
        +---- CPU implementation
        |
        +---- GPU implementation
        |
        +---- FPGA implementation
        |
        +---- Software fallback
        |
        +---- Future accelerator

All implementations MUST preserve the declared ULABI semantics.

An NPU MAY internally use:

- matrix engines;
- tensor cores;
- systolic arrays;
- vector units;
- scalar processors;
- DSPs;
- SRAM;
- DRAM;
- HBM;
- on-chip caches;
- command processors;
- hardware schedulers;
- dedicated neural operators;
- compressed-weight engines;
- sparse-computation engines;
- vendor-specific hardware blocks.

These are implementation details unless explicitly exposed through a standardized capability profile.

---

3. Scope

This specification defines:

1. NPU identity;
2. accelerator identity;
3. NPU architecture identity;
4. NPU capability discovery;
5. NPU interface identity;
6. neural-function identity;
7. operator/kernel identity;
8. model/image identity;
9. model/image integrity;
10. model/image versioning;
11. tensor execution boundaries;
12. tensor shape requirements;
13. supported data types;
14. precision capabilities;
15. quantization capabilities;
16. sparse-computation capabilities;
17. accelerator-local memory;
18. host-to-NPU transfers;
19. NPU-to-host transfers;
20. DMA;
21. zero-copy operation;
22. command submission;
23. command queues;
24. completion queues;
25. asynchronous execution;
26. batching;
27. pipelining;
28. synchronization;
29. events;
30. interrupts;
31. cancellation;
32. execution contexts;
33. resource accounting;
34. resource limits;
35. latency declarations;
36. throughput declarations;
37. deterministic execution declarations;
38. real-time execution declarations;
39. power and thermal capability reporting;
40. isolation;
41. secure loading;
42. attestation hooks;
43. reset behavior;
44. failure handling;
45. recovery;
46. fallback;
47. compatibility;
48. feature negotiation;
49. heterogeneous accelerator operation;
50. conformance testing.

This specification does NOT define:

- a universal NPU architecture;
- a universal neural-network language;
- a universal machine-learning framework;
- a particular model format;
- a particular vendor API;
- CUDA semantics;
- GPU semantics;
- FPGA semantics;
- CPU instruction-set semantics;
- generic serialization;
- generic distributed RPC;
- generic service discovery;
- generic cryptographic policy;
- generic memory-management policy.

---

4. Relationship to Other ULABI Specifications

The NPU layer is below the semantic ULABI interface and above physical accelerator implementation details.

ULABI-DESIGN.md
       |
ULABI-SPEC.md
       |
ULABI Core ABI
       |
Semantic Types / Tensor Contracts
       |
Hardware Interface
       |
NPU Interface
       |
NPU Adapter
       |
NPU Runtime / Driver
       |
Physical NPU

This specification integrates with:

- "docs/abi/core-abi.md"
- "docs/abi/calling-convention.md"
- "docs/abi/data-types.md"
- "docs/abi/memory-model.md"
- "docs/abi/exception-model.md"
- "docs/abi/return-values.md"
- "docs/runtime/runtime-interface.md"
- "docs/runtime/async-model.md"
- "docs/runtime/concurrency.md"
- "docs/runtime/resource-management.md"
- "docs/memory/memory-safety.md"
- "docs/memory/ownership.md"
- "docs/memory/lifetimes.md"
- "docs/memory/shared-memory.md"
- "docs/security/security-model.md"
- "docs/security/capability-security.md"
- "docs/security/secure-loading.md"
- "docs/security/supply-chain-security.md"
- "docs/compatibility/feature-negotiation.md"
- "docs/compatibility/capability-discovery.md"
- "docs/compatibility/graceful-degradation.md"
- "docs/observability/tracing.md"
- "docs/observability/diagnostics.md"
- "docs/reliability/fault-detection.md"
- "docs/reliability/fault-isolation.md"
- "docs/reliability/recovery.md"
- "docs/reliability/rollback.md"
- "docs/standards/conformance.md"
- "docs/standards/test-suite.md"
- "docs/standards/certification.md"

Those documents retain ownership of their generic concepts.

This document defines their NPU realization.

A later refinement of another specification MUST NOT require this document to be rewritten merely because that generic specification becomes more detailed.

NPU-specific additions MUST be versioned or introduced as profiles.

---

5. NPU Identity

Every ULABI-visible NPU MUST expose a stable identity.

Conceptually:

NPUIdentity {
    vendor
    device_family
    device_model
    device_revision
    architecture
    architecture_revision
    firmware_identity
    driver_compatibility
}

The identity MUST be distinguishable from:

- host CPU identity;
- host operating-system identity;
- board identity;
- compiler identity;
- model identity;
- driver identity;
- runtime identity.

A board containing an NPU MUST NOT itself be treated as the NPU identity.

---

6. NPU Capability Identity

ULABI MUST distinguish:

NPU Identity

from:

NPU Capabilities

Possible capabilities include:

MatrixCompute
TensorCompute
VectorCompute
Int8
UInt8
Int16
FP16
BF16
FP32
FP64
INT4
INT2
SparseCompute
DenseCompute
DynamicShapes
StaticShapes
Quantization
Dequantization
Compression
DMA
SharedMemory
ZeroCopy
CommandQueues
AsyncExecution
BatchExecution
PipelineExecution
Preemption
Cancellation
SecureBoot
SecureLoading
Attestation
ECCMemory
PowerManagement
ThermalMonitoring

Capabilities MUST be discoverable.

Physical vendor feature names MUST NOT be treated as semantic capability identities unless standardized by a ULABI profile.

---

7. NPU Interface Identity

Every ULABI-visible NPU interface MUST have a stable interface identity.

Conceptually:

NPUInterfaceIdentity {
    interface_id
    version
    semantic_contract
    input_schema
    output_schema
    execution_model
    resource_requirements
    capability_requirements
}

The interface identity MUST remain stable across compatible implementation revisions.

A semantic contract change requires:

- a compatible interface version change; or
- a new interface identity,

according to ULABI versioning rules.

---

8. Neural Function Identity

A neural function is a ULABI-visible computational operation executed by an NPU.

Example:

function infer(
    input: Tensor
) -> Result<Tensor, InferenceError>

The function contract MUST specify:

- function identity;
- input types;
- output types;
- tensor constraints;
- execution model;
- ownership;
- lifetime;
- effects;
- capabilities;
- resource requirements;
- cancellation behavior;
- error behavior;
- compatibility requirements.

The function identity MUST NOT depend on the vendor-specific implementation.

---

9. Operator and Kernel Identity

An NPU implementation MAY expose lower-level operators.

Examples include:

MatrixMultiply
Convolution
Attention
Activation
Pooling
Normalization
Reduction
Transpose
Reshape
Embedding
Gather
Scatter
Elementwise

A ULABI-visible operator MUST have a stable semantic identity.

A physical kernel name is NOT a semantic identity.

For example:

vendor_kernel_47

MUST NOT itself constitute a ULABI operator identity.

The adapter maps the semantic operator to an implementation-specific kernel.

---

10. Model and Accelerator Image Identity

An NPU MAY require a compiled accelerator image or executable representation.

Conceptually:

NPUImageIdentity {
    image_id
    version
    target_architecture
    required_capabilities
    interface_version
    integrity_identifier
}

An accelerator image MUST declare:

- target architecture;
- required capabilities;
- interface compatibility;
- required memory;
- required execution resources;
- integrity information;
- version information.

A model identifier and an executable accelerator image identifier MUST remain distinct.

Model Identity
      !=
NPU Image Identity

A single model MAY have multiple NPU-specific executable representations.

---

11. Model Loading

Model or accelerator image loading MUST be explicit.

The lifecycle is:

Image Available
      |
      v
Validate Metadata
      |
      v
Verify Compatibility
      |
      v
Verify Integrity
      |
      v
Verify Authorization
      |
      v
Load
      |
      v
Initialize
      |
      v
Ready

An implementation MUST NOT execute an image that has failed required validation.

---

12. Secure Loading

NPU image loading integrates with:

- "docs/security/secure-loading.md"
- "docs/security/supply-chain-security.md"
- "docs/security/capability-security.md"

The NPU adapter MUST support the security requirements applicable to the selected ULABI security profile.

Possible mechanisms include:

- cryptographic signatures;
- integrity hashes;
- authenticated metadata;
- measured loading;
- attestation;
- version restrictions;
- rollback protection.

The exact cryptographic mechanisms are owned by the security specifications.

This document defines only their NPU integration.

---

13. Tensor Boundary

Where ULABI exposes tensor semantics, the NPU adapter MUST preserve the tensor contract.

A tensor MAY be described conceptually as:

TensorDescriptor {
    element_type
    shape
    strides
    layout
    storage
    ownership
    mutability
}

The NPU adapter MUST NOT silently change:

- element meaning;
- shape;
- ordering;
- ownership;
- mutability;
- numerical semantics.

Any transformation MUST be explicitly declared or provably semantics-preserving.

---

14. Tensor Shape Requirements

An NPU function MAY declare:

StaticShape
DynamicShape
BoundedDynamicShape
ShapePolymorphic

For example:

input: Tensor<Float16, [1, 224, 224, 3]>

or:

input: Tensor<Float16, [batch, 224, 224, 3]>
constraints:
    1 <= batch <= 64

The implementation MUST validate shape constraints before execution.

Invalid shapes MUST produce a defined ULABI error.

They MUST NOT cause undefined hardware behavior.

---

15. Tensor Layout

An NPU may support layouts such as:

RowMajor
ColumnMajor
ChannelFirst
ChannelLast
Blocked
Tiled
VendorDefined

A physical layout is not automatically a semantic layout.

The adapter MUST distinguish:

Semantic Layout
        |
        v
Physical NPU Layout

Conversions MAY be performed when required.

The conversion MUST preserve semantic tensor meaning.

---

16. Data Types

NPU implementations MAY support different numerical representations.

Possible types include:

Int8
UInt8
Int16
UInt16
Int32
Float16
BFloat16
Float32
Float64
Int4
UInt4
Int2
UInt2

The NPU adapter MUST explicitly advertise supported types.

It MUST NOT silently reinterpret a semantic type as another type when that changes numerical semantics.

---

17. Precision

An NPU function MAY declare precision requirements.

Possible semantic declarations include:

Exact
BoundedError
Approximate
ImplementationDefined

Where numerical approximation is permitted, the contract SHOULD specify relevant error bounds.

An implementation MUST NOT claim exact semantics when the underlying computation is approximate.

---

18. Quantization

Quantized execution MAY be supported.

A quantized operation MUST define the semantic conversion.

Conceptually:

QuantizationParameters {
    scale
    zero_point
    range
    rounding
    saturation
}

Quantization MUST NOT be silently applied where the function contract requires exact representation.

A fallback implementation MUST preserve the declared numerical semantics within the permitted contract.

---

19. Sparse Computation

An NPU MAY support sparse computation.

Possible sparse models include:

StructuredSparse
UnstructuredSparse
BlockSparse
CompressedSparse
VendorDefined

The sparse representation MUST be explicitly identified.

A sparse representation MUST NOT change semantic tensor values merely because storage is compressed.

---

20. Accelerator Memory

NPU implementations MAY contain:

- SRAM;
- cache;
- DRAM;
- HBM;
- scratchpad memory;
- local tensor memory;
- shared memory;
- vendor-specific memory.

The adapter SHOULD expose semantic memory properties such as:

MemoryRegion {
    region_id
    capacity
    alignment
    access
    coherence
    persistence
    latency_class
}

Physical memory technology remains implementation-specific unless explicitly exposed by a hardware profile.

---

21. Host-to-NPU Data Movement

Data MAY cross the host/NPU boundary through:

- DMA;
- shared memory;
- memory mapping;
- queues;
- descriptors;
- implementation-specific transports.

ULABI MUST NOT require one mechanism.

The semantic ownership contract remains authoritative.

---

22. DMA

DMA MAY transfer data between:

Host Memory
NPU Memory
Shared Memory
Device Memory

Every DMA operation MUST define:

- source;
- destination;
- length;
- ownership;
- lifetime;
- permissions;
- alignment;
- completion;
- failure behavior.

DMA MUST NOT bypass ULABI memory-safety or capability rules.

---

23. DMA Ownership

The semantic ownership lifecycle may be:

HostOwned
    |
    v
TransferRequested
    |
    v
NPUOwned
    |
    v
ExecutionComplete
    |
    v
HostOwned

The implementation MAY use different internal states.

The semantic requirement is that conflicting ownership MUST NOT be assumed.

Shared access MUST be explicitly authorized.

---

24. Zero-Copy

Zero-copy operation MAY be supported.

Zero-copy requires compatible:

- addressability;
- ownership;
- lifetime;
- alignment;
- permissions;
- coherence;
- synchronization;
- isolation.

Zero-copy MUST NOT mean unrestricted access to another component's memory.

If the requirements cannot be satisfied safely, the implementation MUST use copying or another approved representation.

---

25. Command Submission

An NPU MAY expose command submission.

Conceptually:

Command {
    command_id
    operation_id
    input_references
    output_references
    resource_requirements
    completion_reference
    capability
}

Commands MUST be validated before execution.

Malformed commands MUST NOT result in:

- arbitrary memory access;
- capability escalation;
- uncontrolled execution;
- corruption of another execution context.

---

26. Command Queues

An NPU MAY expose:

- submission queues;
- completion queues;
- priority queues;
- event queues;
- data queues.

Each queue MUST define:

- capacity;
- ordering;
- ownership;
- synchronization;
- overflow behavior;
- reset behavior;
- failure behavior.

The queue implementation MUST NOT assume infinite buffering.

---

27. Asynchronous Execution

NPU functions MAY be asynchronous.

A function declaring asynchronous execution MUST provide a completion mechanism.

Conceptually:

Submit
  |
  v
Accepted
  |
  v
Running
  |
  +---- Failed
  |
  +---- Cancelled
  |
  v
Completed

The result MUST remain associated with the original invocation identity.

---

28. Batching

NPU implementations MAY support batching.

Batching MUST NOT change the semantic result of independent operations unless the contract explicitly permits batch-dependent behavior.

A function MAY declare:

BatchSize:
    fixed
    bounded
    dynamic

The implementation MUST enforce declared limits.

---

29. Pipelining

NPU implementations MAY pipeline operations.

Pipelining MUST preserve:

- data dependencies;
- ordering guarantees;
- ownership;
- lifetime;
- error semantics.

A pipeline MUST NOT expose partially written outputs as completed results unless the contract explicitly defines streaming or incremental output.

---

30. Synchronization

NPU synchronization MUST integrate with the ULABI runtime and concurrency models.

Synchronization MAY use:

- fences;
- events;
- semaphores;
- queues;
- barriers;
- device-specific primitives.

Physical synchronization primitives are implementation details.

Their semantic effects MUST be exposed through the ULABI contract.

---

31. Events

NPU events MAY include:

ExecutionCompleted
ExecutionFailed
DataAvailable
ResourceAvailable
DeviceReset
ThermalLimit
PowerLimit
MemoryFault
DeviceFault

Events MUST have stable semantic identifiers.

Physical interrupt numbers MUST NOT themselves become ULABI event identities.

---

32. Interrupts

NPU implementations MAY use interrupts.

Interrupt handling MUST translate physical interrupts into ULABI semantic events.

Polling and interrupt-driven implementations MUST produce equivalent observable semantics unless the profile explicitly specifies otherwise.

---

33. Cancellation

An NPU operation MAY declare:

Cancellable
NonCancellable
CancellationPointDefined

Cancellation behavior MUST be explicitly specified.

Cancellation MUST NOT leave externally visible resources in an undefined ownership state.

If hardware cannot safely cancel an operation, the adapter MAY:

1. wait for a safe cancellation point;
2. isolate the execution context;
3. reset the relevant accelerator context;
4. report cancellation failure.

---

34. Execution Context

An NPU execution context represents resources and state associated with a ULABI invocation.

Conceptually:

NPUExecutionContext {
    context_id
    interface_id
    capability_set
    memory_budget
    compute_budget
    deadline
    cancellation_policy
    isolation_policy
}

The context MUST have explicit:

- ownership;
- lifetime;
- capabilities;
- resource limits;
- failure behavior.

---

35. Resource Accounting

An NPU implementation SHOULD account for resources such as:

ComputeUnits
Memory
Bandwidth
QueueCapacity
Power
ThermalBudget
ExecutionTime

Resource accounting integrates with:

"docs/runtime/resource-management.md"

The NPU adapter MUST NOT silently consume resources outside the authorized execution context.

---

36. Resource Limits

An NPU function MAY declare:

MaximumMemory
MaximumExecutionTime
MaximumBatchSize
MaximumQueueDepth
MaximumPower
MaximumThermalBudget

If a limit cannot be satisfied, the implementation MUST reject, defer, throttle, or safely fall back according to the applicable profile.

It MUST NOT silently violate the declared contract.

---

37. Latency and Throughput

An NPU implementation MAY expose performance characteristics.

Examples:

LatencyClass
ThroughputClass
BandwidthClass

Performance characteristics MUST NOT be represented as correctness guarantees unless explicitly declared as such.

If a hard deadline is declared, the implementation MUST either:

- provide the required guarantee; or
- reject the contract.

---

38. Deterministic Execution

An NPU MAY declare deterministic execution.

A deterministic contract MUST identify relevant sources of nondeterminism, including:

- scheduling;
- floating-point behavior;
- reduced-precision arithmetic;
- concurrent execution;
- random operations;
- hardware race conditions.

An implementation MUST NOT advertise deterministic execution if required deterministic guarantees cannot be established.

---

39. Real-Time Execution

An NPU MAY support real-time profiles.

A real-time contract MUST explicitly define:

- deadline;
- worst-case execution requirement;
- resource reservation;
- scheduling behavior;
- failure behavior.

Average latency MUST NOT be presented as a real-time guarantee.

---

40. Power and Thermal State

An NPU MAY expose:

PowerState
ThermalState
PerformanceState

Possible semantic states include:

Normal
Throttled
PowerLimited
ThermallyLimited
Unavailable

Thermal or power restrictions MUST NOT silently violate correctness semantics.

If performance guarantees can no longer be met, the implementation MUST report the condition according to the applicable runtime/reliability contract.

---

41. Isolation

NPU resources MAY be shared by multiple execution contexts.

The implementation MUST ensure that one context cannot access another context's:

- memory;
- commands;
- capabilities;
- execution state;
- private outputs.

Isolation may be implemented through:

- hardware partitions;
- virtual contexts;
- address-space isolation;
- command validation;
- firmware enforcement;
- runtime isolation.

The exact mechanism is implementation-specific.

---

42. Capability Security

NPU operations MUST integrate with:

"docs/security/capability-security.md"

A function requiring an NPU capability MUST receive only the capabilities necessary for the operation.

Examples:

NPUExecute
NPUMemoryRead
NPUMemoryWrite
NPUModelLoad
NPUModelExecute
NPUReconfigure
NPUReset

Capability names and semantics MUST remain independent of vendor-specific handles.

---

43. Secure Execution

Where supported, an NPU MAY provide secure execution facilities.

Possible mechanisms include:

- isolated execution contexts;
- protected memory;
- secure firmware;
- measured execution;
- hardware attestation;
- encrypted model storage.

These mechanisms integrate with the ULABI security specifications.

This document does not define a universal trusted-execution architecture.

---

44. Attestation

An NPU MAY provide attestation.

Conceptually:

NPUAttestation {
    device_identity
    firmware_identity
    image_identity
    configuration_identity
    measurement
}

Attestation MUST NOT be treated as valid merely because an implementation claims to provide it.

Verification MUST follow the applicable ULABI security profile.

---

45. Reset

An NPU implementation MUST define reset behavior.

Reset MAY be:

ContextReset
QueueReset
EngineReset
DeviceReset
SystemReset

The reset scope MUST be explicit.

A reset MUST NOT silently leave resources in an ambiguous ownership state.

---

46. Failure Model

NPU failures MAY include:

UnsupportedCapability
InvalidTensor
InvalidShape
InvalidDataType
InvalidCommand
MemoryFault
DMAFault
ExecutionFault
DeviceFault
FirmwareFault
ThermalLimit
PowerLimit
Timeout
CancellationFailure
ResourceExhaustion
SecurityViolation
IntegrityFailure

Failures MUST be represented using the ULABI error model.

Vendor-specific hardware error codes MAY be retained as diagnostic metadata but MUST NOT replace the semantic ULABI error.

---

47. Fault Isolation

When an NPU operation fails, the implementation SHOULD isolate the smallest affected scope.

Preferred hierarchy:

Operation
   |
Context
   |
Queue
   |
Engine
   |
Device

A device-wide reset SHOULD NOT be required for a recoverable operation-level failure.

The implementation MUST NOT claim isolation that the hardware cannot actually provide.

---

48. Recovery

NPU recovery integrates with:

- "docs/reliability/fault-detection.md"
- "docs/reliability/fault-isolation.md"
- "docs/reliability/recovery.md"
- "docs/reliability/rollback.md"

A permitted recovery sequence may be:

Failure
  |
  v
Capture Evidence
  |
  v
Classify Failure
  |
  v
Isolate
  |
  v
Known Recovery Policy?
  |             |
 YES            NO
  |             |
 Recover      Escalate
  |
  v
Verify
  |
  +---- Failed ---> Rollback / Escalate
  |
  v
Healthy

NPU recovery MUST be bounded and policy-controlled.

The NPU adapter MUST NOT autonomously change its executable image or hardware configuration merely because it detected a fault.

---

49. Image Rollback

If NPU firmware or executable images support rollback, rollback MUST be explicit and policy-controlled.

A rollback target MUST have:

- compatible interface identity;
- verified integrity;
- authorized provenance;
- supported hardware target.

Rollback MUST NOT silently downgrade below a required security or compatibility level.

---

50. Software Fallback

A ULABI NPU implementation MAY provide fallback execution.

Possible fallback targets include:

CPU
GPU
FPGA
Software Runtime
Another NPU

Fallback MUST preserve the semantic ULABI contract.

The implementation MUST NOT silently weaken:

- security;
- correctness;
- ownership;
- error semantics;
- capability restrictions.

A fallback SHOULD be declared through capability negotiation or implementation metadata.

---

51. Heterogeneous Accelerators

A system MAY contain:

CPU + GPU + NPU
CPU + FPGA + NPU
CPU + GPU + FPGA + NPU
Multiple NPUs

ULABI MUST allow each accelerator to implement the same semantic interface independently.

The system MUST distinguish:

Interface Identity
        |
        +---- CPU Implementation
        +---- GPU Implementation
        +---- NPU Implementation
        +---- FPGA Implementation

The presence of one accelerator MUST NOT make another mandatory.

---

52. Capability Negotiation

NPU capability negotiation integrates with:

"docs/compatibility/feature-negotiation.md"

A function MAY declare:

requires:
    FP16
    DynamicShapes
    AsyncExecution

If those capabilities are unavailable, the implementation MUST:

1. select an approved compatible implementation;
2. use a valid fallback;
3. negotiate a compatible feature set; or
4. reject the operation.

Unsupported capabilities MUST NOT be silently assumed.

---

53. Capability Discovery

Capability discovery integrates with:

"docs/compatibility/capability-discovery.md"

Conceptually:

ULABI Interface
       |
Required Capabilities
       |
Capability Discovery
       |
   +---+---+
   |       |
Supported Unsupported
   |          |
Execute    Negotiate /
           Fallback /
           Reject

Capability discovery MUST occur before execution whenever the required capability cannot otherwise be established safely.

---

54. Compatibility

NPU interfaces MUST follow:

- semantic versioning rules defined by ULABI;
- interface identity rules;
- feature negotiation;
- capability discovery;
- backward compatibility requirements.

An implementation MUST NOT infer compatibility merely because:

- two NPUs are from the same vendor;
- two devices expose similar APIs;
- two models have the same name;
- two accelerator images have the same file extension.

Compatibility MUST be established from the declared ULABI contract.

---

55. Observability

NPU implementations SHOULD expose diagnostics for:

- execution latency;
- queue state;
- memory usage;
- resource usage;
- failures;
- resets;
- thermal state;
- power state;
- capability changes.

Observability integrates with:

- "docs/observability/tracing.md"
- "docs/observability/diagnostics.md"

Observability data MUST NOT silently become application-visible semantic state unless explicitly defined by the interface.

---

56. Conformance Requirements

A conforming NPU implementation MUST demonstrate, as applicable:

- stable NPU identity;
- stable interface identity;
- capability discovery;
- capability negotiation;
- correct tensor validation;
- correct shape validation;
- correct data-type handling;
- correct ownership handling;
- safe memory access;
- validated command submission;
- bounded queue behavior;
- correct asynchronous completion;
- correct cancellation behavior;
- correct error propagation;
- isolation;
- resource enforcement;
- secure loading where claimed;
- reset behavior;
- recovery behavior where claimed;
- fallback behavior where claimed;
- version compatibility.

An implementation MUST NOT claim support for an NPU profile whose mandatory requirements it cannot satisfy.

---

57. Required Conformance Tests

The NPU conformance suite MUST eventually include tests for:

NPU identity
Capability discovery
Capability negotiation
Interface identity
Tensor validation
Shape validation
Data-type validation
Precision declarations
Quantization semantics
Sparse execution
Memory ownership
DMA safety
Zero-copy safety
Command validation
Queue overflow
Asynchronous execution
Completion ordering
Cancellation
Resource limits
Deadline enforcement
Isolation
Secure loading
Integrity failure
Device reset
Context reset
Execution failure
Recovery
Rollback
Fallback
Version compatibility

Negative tests are mandatory.

The test suite MUST verify that malformed inputs cannot cause:

- arbitrary memory access;
- capability escalation;
- undefined execution;
- cross-context data leakage;
- silent semantic corruption.

---

58. Reference Implementation Boundary

The reference implementation SHOULD be divided into:

ULABI Semantic Layer
        |
NPU Adapter
        |
NPU Runtime/Driver Adapter
        |
Hardware

The reference implementation MUST NOT require a specific:

- NPU vendor;
- operating system;
- programming language;
- model framework;
- neural-network compiler.

A software emulation backend MAY be provided for environments without an NPU.

The emulator MUST preserve the ULABI semantic contract.

---

59. Required Integration Interfaces

The implementation should expose logical interfaces equivalent to:

NpuDevice
NpuCapabilitySet
NpuInterface
NpuFunction
NpuImage
NpuExecutionContext
NpuTensorView
NpuMemoryRegion
NpuCommand
NpuCommandQueue
NpuCompletion
NpuEvent
NpuResourceBudget
NpuAttestation
NpuRecoveryPolicy

These are logical contracts, not mandatory source-language types.

Individual implementations MAY map them into idiomatic types.

---

60. Required Invariants

The following invariants are normative:

1. ULABI semantics MUST NOT depend on a particular NPU vendor.

2. An NPU MUST NOT execute a command that has failed required validation.

3. An NPU MUST NOT bypass ULABI memory ownership rules.

4. Zero-copy MUST require explicit memory compatibility.

5. An unsupported capability MUST NOT be silently assumed.

6. An unsupported tensor type MUST NOT be silently reinterpreted when semantics change.

7. An invalid tensor shape MUST produce a defined failure.

8. NPU execution contexts MUST remain isolated according to their declared security profile.

9. Resource limits MUST be enforced.

10. Hardware error codes MUST NOT replace the ULABI semantic error model.

11. Cancellation MUST preserve resource and ownership invariants.

12. Reset MUST have explicitly defined scope.

13. Recovery MUST be bounded and policy-controlled.

14. Automatic image replacement MUST NOT occur merely because a fault was detected.

15. Fallback MUST preserve the ULABI semantic contract.

16. Physical accelerator details MUST NOT leak into the universal contract unless explicitly standardized.

17. A conforming implementation MUST NOT advertise guarantees it cannot provide.

---

61. Integration Stability Rule

This document is intentionally designed so that the following documents can evolve independently:

ABI
Types
Memory
Runtime
Security
Compatibility
Reliability
Observability
Standards

Each owns its generic semantics.

This document owns only the NPU realization.

Therefore, implementation work on this file is considered complete when:

- the NPU contract above is implemented;
- the listed interfaces exist;
- the listed invariants are enforced;
- the conformance tests exist;
- integration adapters satisfy the referenced contracts.

Another specification MUST NOT require reopening this file merely to add its own generic rules.

NPU-specific changes MUST use a new specification/profile/version when they alter existing semantics.

---

62. Summary

ULABI NPU support follows:

Semantic ULABI Contract
          |
          v
Capability Discovery
          |
          v
NPU Interface Selection
          |
          v
Resource Validation
          |
          v
Tensor / Command Validation
          |
          v
Memory Preparation
          |
          v
NPU Execution
          |
          v
Completion / Failure
          |
     +----+----+
     |         |
 Success     Failure
     |         |
     v         v
 Result     Isolation
               |
               v
            Recovery?
            /      \
          Yes       No
           |         |
        Verify    Escalate
           |
        Healthy?
        /      \
      Yes       No
       |         |
     Done     Rollback /
              Escalate

The essential architectural rule is:

«ULABI standardizes what an NPU-visible operation means; it does not standardize how an NPU vendor builds the hardware that performs it.»3. What files should be implemented around this document

I would not immediately modify all of the other specifications. The NPU document above deliberately contains its integration contracts in advance, so it can be implemented without waiting for the other documents to be rewritten.

The implementation should be staged in dependency order.

Phase 1 — independent NPU contracts

These can be created first because they establish the NPU-specific abstractions without depending on a concrete vendor.

implementations/
└── npu/
    ├── mod.*
    ├── device.*
    ├── capabilities.*
    ├── interface.*
    ├── function.*
    ├── image.*
    ├── tensor.*
    ├── memory.*
    ├── command.*
    ├── queue.*
    ├── execution.*
    ├── events.*
    ├── resources.*
    ├── attestation.*
    ├── recovery.*
    └── errors.*

Recommended logical modules:

Module	Responsibility

device	NPU identity and lifecycle
capabilities	capability representation/discovery
interface	stable ULABI NPU interface identity
function	neural-function contracts
image	executable/model-image metadata
tensor	tensor boundary and validation
memory	NPU memory and ownership
command	command construction/validation
queue	submission/completion queues
execution	execution contexts and async state
events	semantic event translation
resources	resource budgets and limits
attestation	attestation abstraction
recovery	bounded recovery policy integration
errors	NPU-specific errors mapped to ULABI errors
mod	public module surface


These modules should be designed against interfaces, not against CUDA, ROCm, OpenVINO, Core ML, Qualcomm APIs, Apple APIs, ARM APIs, Intel APIs, etc.


---

4. Integration modules

After the independent NPU layer is stable, these adapters connect it to the rest of ULABI:

implementations/
├── abi/
│   ├── npu_call_adapter.*
│   └── npu_type_adapter.*
│
├── runtime/
│   ├── npu_runtime_adapter.*
│   ├── npu_async_adapter.*
│   └── npu_resource_adapter.*
│
├── memory/
│   ├── npu_memory_adapter.*
│   ├── npu_dma_adapter.*
│   └── npu_zero_copy_adapter.*
│
├── security/
│   ├── npu_capability_adapter.*
│   ├── npu_secure_loader.*
│   └── npu_attestation_adapter.*
│
├── compatibility/
│   ├── npu_capability_discovery.*
│   └── npu_feature_negotiation.*
│
├── reliability/
│   ├── npu_fault_adapter.*
│   ├── npu_recovery_adapter.*
│   └── npu_rollback_adapter.*
│
└── observability/
    ├── npu_tracing.*
    └── npu_diagnostics.*

These modules are integration modules, not additional specifications.


---

5. Vendor/backend modules

Only after the semantic NPU layer is stable should concrete backends be added.

The architecture should look like:

ULABI NPU API
      |
      +------------------+
      |                  |
 Software Emulator    Backend Adapters
                         |
          +--------------+--------------+
          |              |              |
       Vendor A       Vendor B       Vendor C

Do not make the ULABI NPU API depend on one vendor.

A future structure can be:

implementations/
└── npu/
    ├── backend/
    │   ├── generic/
    │   │   └── ...
    │   ├── vendor_a/
    │   │   └── ...
    │   ├── vendor_b/
    │   │   └── ...
    │   └── vendor_c/
    │       └── ...
    │
    └── emulator/
        └── ...

The actual vendor names should be introduced only when there is a concrete implementation target.


---

6. Tests that belong specifically to NPU

Create:

tests/
└── npu/
    ├── device_identity.*
    ├── capability_discovery.*
    ├── capability_negotiation.*
    ├── interface_identity.*
    ├── tensor_validation.*
    ├── shape_validation.*
    ├── datatype_validation.*
    ├── quantization.*
    ├── sparse_execution.*
    ├── memory_ownership.*
    ├── dma.*
    ├── zero_copy.*
    ├── command_validation.*
    ├── queues.*
    ├── async_execution.*
    ├── cancellation.*
    ├── resource_limits.*
    ├── isolation.*
    ├── secure_loading.*
    ├── reset.*
    ├── failure_handling.*
    ├── recovery.*
    ├── rollback.*
    ├── fallback.*
    └── compatibility.*

And:

conformance/
└── npu/
    ├── core.*
    ├── memory.*
    ├── execution.*
    ├── security.*
    ├── compatibility.*
    ├── reliability.*
    └── hardware.*

The distinction is important:

tests/npu/ = implementation tests.

conformance/npu/ = standardized tests that independent implementations must pass.


That supports the eventual claim:

ULABI NPU Profile
    Core             ✓
    Tensor           ✓
    Memory           ✓
    Async            ✓
    Security         ✓
    Reliability      ✓
    Compatibility    ✓

rather than simply "NPU supported."


---

7. Schemas required

The NPU specification also implies these schema artifacts:

schemas/
└── npu/
    ├── npu-identity.schema.*
    ├── npu-capabilities.schema.*
    ├── npu-interface.schema.*
    ├── npu-function.schema.*
    ├── npu-image.schema.*
    ├── tensor-descriptor.schema.*
    ├── npu-memory.schema.*
    ├── npu-command.schema.*
    ├── npu-execution-context.schema.*
    ├── npu-event.schema.*
    ├── npu-resource-budget.schema.*
    └── npu-attestation.schema.*

The exact schema language should follow whatever schema convention the repository establishes; npu.md should not invent a second schema system.


---

8. Examples

Create:

examples/
└── npu/
    ├── basic-inference.*
    ├── asynchronous-inference.*
    ├── tensor-transfer.*
    ├── zero-copy.*
    ├── batching.*
    ├── capability-negotiation.*
    ├── fallback.*
    ├── resource-limited-execution.*
    └── failure-recovery.*

These examples should demonstrate the ULABI contract, not a particular NPU vendor.


---

9. Files that should NOT be modified merely because npu.md was added

This is important for your "finish one file without reopening it later" rule.

Do not reopen:

ULABI-DESIGN.md
ULABI-SPEC.md
docs/abi/core-abi.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/runtime/runtime-interface.md
docs/security/security-model.md
docs/compatibility/feature-negotiation.md
docs/reliability/recovery.md

just to add an NPU-specific paragraph.

Their generic contracts already exist above this specialization layer. The current architecture explicitly uses a Minimal Core + Profiles model and treats hardware interoperability as an extension/profile rather than forcing hardware-specific semantics into the Core.

If a future change genuinely changes a generic ULABI invariant, that is a separate specification change and should be handled as such—not smuggled into npu.md.


---

10. Final dependency order

For the hardware subsystem, I recommend this order:

1. docs/hardware/npu.md
        ↓
2. implementations/npu/core modules
        ↓
3. schemas/npu/*
        ↓
4. tests/npu/*
        ↓
5. conformance/npu/*
        ↓
6. implementations/npu/integration/*
        ↓
7. implementations/npu/backend/*
        ↓
8. examples/npu/*

And the conceptual dependency is:

ULABI Core
    |
    +---- CPU
    +---- GPU
    +---- FPGA
    +---- NPU
    +---- Quantum
    |
    +---- Future Hardware

None of these hardware specifications should depend on one another for their fundamental semantics. They should instead implement the same universal ULABI contract.

That keeps NPU, GPU, FPGA and CPU as peers, rather than accidentally making ULABI an NPU/GPU-oriented ABI.