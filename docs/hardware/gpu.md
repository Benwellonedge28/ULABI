I inspected the repository before drafting this. ULABI-DESIGN.md is the architectural authority and establishes the language/runtime/CPU/vendor neutrality, Minimal Core + Profiles model, transport independence, capability discovery, hardware interoperability, and the separation between semantic contracts and implementation details.

I also checked the hardware area. docs/hardware/gpu.md is currently only a placeholder, while the existing CPU and FPGA specifications establish the complementary hardware-adapter pattern.

So the GPU document should not duplicate CPU, FPGA, generic accelerator, memory, serialization, security, or distributed specifications. It should own the GPU-specific execution contract: device identity, GPU capabilities, command submission, kernels, queues, device memory, host/device transfers, synchronization, streams, events, execution/resource limits, heterogeneous CPU/GPU execution, deterministic/real-time constraints where supportable, secure loading, failure/recovery, compatibility, and conformance.

docs/hardware/gpu.md

Copy the following as the complete file:

ULABI GPU Hardware Interface Specification

Document: "docs/hardware/gpu.md"
Project: ULABI
Full Name: Universal Language Application Binary Interface
Specification Family: Hardware
Component: GPU Interface
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

This specification defines the ULABI contract for exposing Graphics Processing Units (GPUs) and GPU-resident computation through ULABI.

The purpose is not to define a universal GPU architecture, universal shader language, or universal GPU instruction set.

Instead, it defines the semantic boundary between ULABI and a GPU implementation.

ULABI Interface
       |
       v
GPU ULABI Adapter
       |
       v
GPU Execution Interface
       |
       +-----------------------+
       |                       |
   GPU Compute             GPU Memory
       |                       |
       +-----------+-----------+
                   |
              GPU Hardware

A GPU implementation MAY be:

- integrated into a CPU;
- discrete;
- embedded in a system-on-chip;
- virtualized;
- shared by multiple processes;
- used for graphics;
- used for general-purpose computation;
- used for machine learning;
- used for signal processing;
- used for scientific computing;
- used for media processing;
- heterogeneous with other accelerators.

ULABI semantics MUST remain independent of the GPU vendor, driver, operating system, shader language, compiler, and hardware generation.

---

2. Fundamental Principle

The fundamental rule is:

«ULABI defines the semantic computation contract; a GPU adapter defines how that contract is realized on a particular GPU.»

Therefore:

ULABI Function
      |
      +---- CPU implementation
      |
      +---- GPU implementation A
      |
      +---- GPU implementation B
      |
      +---- FPGA implementation
      |
      +---- software fallback
      |
      +---- future accelerator

All implementations MUST preserve the declared ULABI semantics.

A GPU implementation MAY use:

- SIMT execution;
- SIMD execution;
- vector units;
- shader cores;
- compute units;
- tensor/matrix units;
- local/shared memory;
- device memory;
- caches;
- command processors;
- hardware schedulers;
- asynchronous copy engines;
- vendor-specific instructions;
- vendor-specific runtime mechanisms.

These are implementation details unless explicitly exposed through a standardized GPU profile.

---

3. Scope

This specification defines:

1. GPU identity;
2. GPU architecture identity;
3. GPU capability discovery;
4. GPU interface identity;
5. GPU execution contexts;
6. GPU kernels;
7. kernel identity;
8. kernel arguments;
9. kernel execution semantics;
10. command submission;
11. command queues;
12. queue ordering;
13. queue synchronization;
14. GPU streams;
15. GPU events;
16. GPU device memory;
17. host memory;
18. host-to-device transfer;
19. device-to-host transfer;
20. device-to-device transfer;
21. DMA where supported;
22. unified memory where supported;
23. mapped memory;
24. zero-copy operation;
25. memory ownership;
26. memory visibility;
27. synchronization;
28. fences;
29. barriers;
30. atomic operations;
31. GPU execution limits;
32. resource accounting;
33. work-group/grid semantics;
34. GPU scheduling constraints;
35. heterogeneous CPU/GPU execution;
36. asynchronous execution;
37. cancellation semantics;
38. timeout semantics;
39. deterministic execution requirements;
40. real-time declarations;
41. GPU image/module identity;
42. secure GPU module loading;
43. GPU module compatibility;
44. GPU fault handling;
45. GPU reset behavior;
46. recovery;
47. capability fallback;
48. observability;
49. conformance testing.

This specification does not define:

- a universal GPU;
- a universal shader language;
- a universal graphics API;
- a universal GPU instruction set;
- CUDA semantics;
- OpenCL semantics;
- Vulkan semantics;
- Metal semantics;
- Direct3D semantics;
- a particular GPU vendor;
- a particular driver;
- a particular operating system;
- generic serialization;
- generic distributed computing;
- generic cryptography;
- generic memory safety.

Those concepts remain owned by their respective ULABI specifications.

---

4. Relationship to Other ULABI Specifications

The GPU layer sits below the semantic ULABI interface and above GPU-specific hardware implementation.

ULABI-DESIGN.md
       |
ULABI-SPEC.md
       |
ULABI Core ABI
       |
Hardware Interface
       |
GPU Interface
       |
GPU Adapter
       |
GPU Runtime / Driver
       |
GPU Hardware

The GPU specification integrates with:

- "docs/abi/core-abi.md"
- "docs/abi/calling-convention.md"
- "docs/abi/data-types.md"
- "docs/abi/memory-model.md"
- "docs/abi/register-model.md"
- "docs/runtime/runtime-interface.md"
- "docs/runtime/resource-management.md"
- "docs/runtime/async-model.md"
- "docs/runtime/concurrency.md"
- "docs/memory/memory-safety.md"
- "docs/memory/shared-memory.md"
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
- "docs/standards/conformance.md"
- "docs/standards/test-suite.md"
- "docs/standards/certification.md"

These specifications retain ownership of their generic concepts.

This document defines their GPU realization.

It MUST NOT redefine their generic semantics.

---

5. GPU Identity

Every ULABI GPU implementation MUST expose a stable GPU identity.

Conceptually:

GPUIdentity {
    vendor
    architecture
    device_family
    device_model
    device_revision
    memory_capacity
    feature_set
}

GPU identity MUST be distinguishable from:

- host CPU identity;
- motherboard identity;
- operating-system identity;
- driver identity;
- application identity;
- compiler identity.

A GPU board or physical enclosure MUST NOT automatically be treated as the semantic GPU identity.

---

6. GPU Capability Identity

ULABI MUST distinguish GPU identity from GPU capabilities.

Possible capabilities include:

Compute
Graphics
Vector
Matrix
Tensor
FP16
FP32
FP64
IntegerCompute
AtomicOperations
DeviceMemory
SharedMemory
UnifiedMemory
MappedMemory
AsyncCopy
PeerToPeer
HardwareScheduling
Preemption
Virtualization
MultiProcess
SecureExecution
PerformanceCounters
HardwareQueues

Capability names are semantic.

Vendor-specific feature names MUST NOT be required for the Core ABI.

---

7. Capability Discovery

GPU capabilities MUST be discoverable through the ULABI capability-discovery mechanism.

ULABI Interface
      |
      v
Required GPU Capabilities
      |
      v
Capability Discovery
      |
   +--+----------------+
   |                   |
Supported          Unsupported
   |                   |
Execute          Fallback / Reject

An implementation MUST NOT silently assume that an optional GPU capability exists.

If a required capability is unavailable, the implementation MUST either:

1. use an approved compatible fallback;
2. negotiate an alternative implementation;
3. report an explicit incompatibility;
4. reject execution.

---

8. GPU Interface Identity

Every ULABI-visible GPU interface MUST have a stable identity.

Conceptually:

GPUInterfaceIdentity {
    interface_id
    version
    semantic_contract
    execution_model
    data_schema
    resource_requirements
    capability_requirements
}

Changing the semantic contract requires:

- a compatible interface version; or
- a new interface identity,

according to ULABI versioning rules.

---

9. GPU Kernel

A GPU kernel is a ULABI-visible computational operation executed on GPU hardware.

Conceptually:

Kernel {
    kernel_id
    interface_id
    version
    input_schema
    output_schema
    capability_requirements
    resource_requirements
    execution_model
}

Example:

kernel matrix_multiply(
    input: Tensor,
    weights: Tensor
) -> Result<Tensor, ComputeError>

The kernel MAY internally use:

- vector operations;
- matrix operations;
- tensor units;
- shared memory;
- local caches;
- parallel execution;
- asynchronous operations.

The implementation details MUST NOT change the ULABI semantic contract.

---

10. Kernel Identity

A kernel identity MUST remain distinguishable from:

- source code;
- shader source;
- compiler version;
- binary image;
- driver version.

Conceptually:

KernelIdentity {
    kernel_id
    semantic_version
    interface_version
    target_profile
}

A compatible compiled kernel MAY have multiple hardware-specific representations.

---

11. GPU Module

A GPU module is a deployable collection of GPU-resident functions or kernels.

Conceptually:

GPUModule {
    module_id
    version
    interface_set
    target_capabilities
    resource_requirements
    integrity_metadata
}

A module MAY contain:

- one kernel;
- multiple kernels;
- helper functions;
- device-side runtime components;
- metadata.

Module loading MUST be validated before execution.

---

12. GPU Module Representation

ULABI MUST NOT mandate one GPU binary format.

A GPU module MAY be represented by:

- native device code;
- intermediate representation;
- portable GPU representation;
- implementation-specific binary;
- source requiring compilation;
- signed hardware image.

The adapter MUST expose the semantic module contract independently of representation.

---

13. Kernel Arguments

Kernel arguments MUST use ULABI semantic types.

For example:

kernel process(
    input: Bytes,
    count: UInt64,
    configuration: ProcessConfig
) -> Result<Unit, ComputeError>

The GPU adapter determines the physical representation.

Arguments MAY be represented through:

- registers;
- parameter memory;
- descriptors;
- device memory;
- command buffers;
- implementation-specific argument blocks.

The physical representation MUST NOT alter semantic meaning.

---

14. Work Dimensions

GPU kernels commonly execute over multiple logical work dimensions.

ULABI SHOULD support a semantic execution domain:

ExecutionDomain {
    dimensions
    global_size
    local_size
    offset
}

Dimensions MAY include:

- one-dimensional;
- two-dimensional;
- three-dimensional;
- implementation-defined higher-dimensional execution models.

The ULABI execution domain MUST remain independent of a particular GPU API.

---

15. Work Items

A GPU implementation MAY expose logical work items.

A work item represents one logical execution instance of a kernel.

Conceptually:

Kernel
  |
  +-- Work Item 0
  +-- Work Item 1
  +-- Work Item 2
  +-- ...

The implementation MUST NOT assume that logical work items correspond one-to-one with physical GPU threads.

---

16. Work Groups

Where the target GPU supports grouped execution, the implementation MAY expose work groups.

A work-group contract MUST define:

- group size;
- local identity;
- synchronization scope;
- local/shared memory;
- barrier semantics.

ULABI MUST NOT require a specific hardware scheduling unit.

---

17. Parallel Execution Semantics

A GPU kernel MUST explicitly declare whether operations are:

- deterministic;
- order-dependent;
- order-independent;
- atomic;
- associative;
- commutative;
- synchronization-dependent.

An implementation MUST NOT expose nondeterministic execution as deterministic unless the required guarantees are actually enforced.

---

18. Command Submission

GPU operations MUST be submitted through a defined command mechanism.

Conceptually:

Command {
    command_id
    operation
    arguments
    dependencies
    resource_requirements
}

Commands MAY represent:

- kernel execution;
- memory copy;
- memory fill;
- synchronization;
- module loading;
- event signaling;
- event waiting.

---

19. Command Queues

A GPU implementation MAY expose one or more command queues.

Conceptually:

CommandQueue {
    queue_id
    capacity
    ordering
    priority
    execution_mode
}

A queue MUST define:

- ordering;
- capacity;
- submission behavior;
- overflow behavior;
- synchronization;
- failure semantics;
- reset semantics.

An implementation MUST NOT assume unlimited command buffering.

---

20. Queue Ordering

Queues MUST explicitly declare their ordering semantics.

Possible models include:

Ordered
PartiallyOrdered
DependencyOrdered
Unordered

If commands are unordered, explicit dependencies MUST be used where ordering matters.

---

21. Command Dependencies

Commands MAY depend on previous commands.

Conceptually:

Command A
    |
    v
Command B
    |
    v
Command C

A command MUST NOT execute before its declared dependencies are satisfied.

Dependency cycles MUST be detected or rejected.

---

22. GPU Streams

A GPU implementation MAY expose logical execution streams.

A stream represents an ordered sequence of GPU operations.

Stream A:
  Copy
    |
  Kernel
    |
  Kernel

Stream B:
  Copy
    |
  Kernel

Different streams MAY execute concurrently when the GPU supports it.

Concurrency MUST NOT violate declared memory or synchronization semantics.

---

23. GPU Events

GPU events provide synchronization points.

Conceptually:

GPUEvent {
    event_id
    state
    source
    timestamp
}

Possible states:

Pending
Signaled
Failed
Cancelled
Expired

An event MUST have stable semantic identity.

A vendor-specific event handle MUST NOT itself constitute the ULABI event identity.

---

24. Synchronization

Synchronization MUST define the scope of visibility.

Possible scopes include:

- invocation;
- work item;
- work group;
- kernel;
- queue;
- device;
- host;
- shared memory domain.

The implementation MUST NOT provide weaker memory visibility than the declared contract.

---

25. GPU Memory

A GPU implementation MAY expose:

- device-local memory;
- host memory;
- shared memory;
- unified memory;
- mapped memory;
- managed memory;
- peer memory.

ULABI MUST distinguish the semantic memory object from the physical memory technology.

Conceptually:

GPUMemoryRegion {
    region_id
    capacity
    alignment
    access
    coherence
    locality
    lifetime
}

---

26. Memory Ownership

GPU memory ownership MUST be explicit.

Conceptually:

HostOwned
    |
    v
TransferToGPU
    |
    v
GPUOwned
    |
    v
TransferComplete
    |
    v
HostOwned

Shared ownership MAY be supported only when the memory model explicitly permits it.

Conflicting unsynchronized access MUST NOT be assumed safe.

---

27. Host-to-Device Transfer

Host-to-device transfers MUST define:

- source;
- destination;
- length;
- ownership;
- lifetime;
- alignment;
- access permissions;
- completion semantics;
- failure behavior.

A transfer MUST NOT silently change the logical contents of the object.

---

28. Device-to-Host Transfer

Device-to-host transfers MUST preserve:

- data semantics;
- type identity;
- length;
- ownership;
- synchronization state.

The host MUST NOT observe partially written data unless the interface explicitly declares streaming or partial-result semantics.

---

29. Device-to-Device Transfer

Peer or device-local transfers MAY be supported.

The implementation MUST expose whether:

- peer access is supported;
- synchronization is required;
- copies are asynchronous;
- copies are ordered;
- the destination becomes visible immediately or after an event.

---

30. Unified and Shared Memory

A GPU MAY share memory with the host.

Shared memory MUST NOT imply unrestricted access.

The contract MUST define:

- ownership;
- access permissions;
- coherence;
- synchronization;
- lifetime;
- visibility;
- revocation where applicable.

---

31. Zero-Copy

Zero-copy MAY be supported.

Zero-copy requires compatible:

- addressability;
- lifetime;
- ownership;
- alignment;
- access rights;
- coherence;
- synchronization;
- isolation.

If these conditions cannot be guaranteed, the implementation MUST use a safe copy or another valid representation.

---

32. Atomic Operations

GPU atomics MAY be exposed through ULABI.

An atomic operation MUST specify:

- data type;
- atomicity scope;
- ordering;
- visibility;
- supported operations.

Possible operations include:

Load
Store
Exchange
CompareExchange
Add
Subtract
And
Or
Xor
Min
Max

The GPU implementation MUST NOT claim stronger atomic semantics than the hardware/runtime actually provides.

---

33. Resource Model

GPU resources MUST be explicitly bounded.

Relevant resources include:

- device memory;
- shared/local memory;
- registers;
- compute units;
- execution slots;
- command queues;
- streams;
- events;
- bandwidth;
- execution time.

Conceptually:

GPUResourceBudget {
    memory
    local_memory
    execution_units
    queues
    streams
    time
}

An implementation MUST detect resource exhaustion rather than allowing silent corruption.

---

34. Resource Limits

A GPU execution context MAY declare:

max_memory
max_execution_time
max_concurrent_kernels
max_queue_depth
max_transfer_size

Exceeding a limit MUST produce an explicit ULABI failure.

Possible errors include:

ResourceLimit
OutOfMemory
ExecutionTimeout
QueueFull
UnsupportedConfiguration

---

35. CPU/GPU Heterogeneous Execution

ULABI MUST support heterogeneous execution without requiring a specific CPU/GPU architecture.

Conceptually:

Application
    |
    +---- CPU Function
    |
    +---- GPU Function
    |
    +---- FPGA Function
    |
    +---- Other Accelerator

A function MAY declare:

preferred_execution: GPU
fallback_execution: CPU

The fallback MUST preserve semantic behavior.

---

36. GPU Dispatch

A dispatch operation represents the submission of a GPU computation.

Conceptually:

GPUDispatch {
    kernel_id
    execution_domain
    arguments
    dependencies
    resource_budget
    completion_event
}

The dispatch MUST be validated before execution.

Malformed dispatch information MUST NOT cause arbitrary GPU or host memory access.

---

37. Asynchronous Execution

GPU execution SHOULD support asynchronous operation where hardware permits.

A function declared:

Asynchronous

MUST NOT be treated as synchronous merely because a particular implementation lacks asynchronous support.

If asynchronous execution is unavailable, the implementation MUST either:

- provide an approved fallback;
- reject the contract;
- negotiate a compatible mode.

---

38. Cancellation

A GPU operation MAY declare:

Cancellable
NonCancellable
BestEffortCancellation

Cancellation semantics MUST be explicit.

Cancellation MUST NOT imply that already committed side effects are automatically reversed.

If cancellation cannot safely interrupt an operation, the implementation MUST report that fact.

---

39. Timeouts

A GPU operation MAY specify a deadline or timeout.

The implementation MUST distinguish:

- timeout requested;
- timeout detected;
- execution terminated;
- execution still running;
- execution result unknown.

A timeout MUST NOT falsely report successful completion.

---

40. Determinism

A GPU implementation MAY declare deterministic execution.

If an operation is declared deterministic, repeated execution with identical:

- inputs;
- configuration;
- environment;
- declared capabilities;

MUST produce semantically equivalent results according to the function contract.

If hardware scheduling or floating-point behavior prevents deterministic guarantees, the implementation MUST NOT advertise deterministic execution.

---

41. Floating-Point Semantics

GPU floating-point execution MUST respect the ULABI floating-point contract.

GPU-specific differences involving:

- precision;
- rounding;
- fused operations;
- NaN behavior;
- signed zero;
- denormals;
- flush-to-zero;

MUST be explicitly declared where they can affect observable semantics.

A GPU implementation MUST NOT silently substitute weaker floating-point semantics for a stricter ULABI contract.

---

42. Tensor and Matrix Operations

GPU implementations MAY expose specialized tensor or matrix capabilities.

Examples include:

TensorMultiply
MatrixMultiply
Convolution
Reduction
VectorOperation

These capabilities belong to the GPU/hardware profile.

Generic tensor semantics MUST be defined by the appropriate ULABI type or accelerator specifications rather than duplicated here.

A GPU tensor implementation MUST preserve the declared ULABI tensor data contract.

---

43. GPU Scheduling

GPU scheduling is implementation-defined.

A scheduler MAY use:

- priority;
- fairness;
- occupancy;
- resource availability;
- deadlines;
- workload classes;
- power constraints.

The ULABI interface MUST expose only guarantees that the implementation can enforce.

Scheduling policy MUST NOT silently change semantic correctness.

---

44. Priority

A GPU operation MAY declare a priority.

Priority MUST be treated as a scheduling hint unless the implementation explicitly guarantees stronger behavior.

Possible semantic classes:

Background
Normal
Interactive
High
RealTime

A priority declaration MUST NOT automatically override security, isolation, or resource policies.

---

45. Real-Time GPU Execution

A GPU implementation MAY advertise real-time capabilities.

A real-time declaration MUST specify:

- deadline model;
- scheduling guarantees;
- maximum execution latency;
- resource reservation;
- preemption guarantees;
- failure behavior.

If these guarantees cannot be verified, the implementation MUST NOT claim real-time conformance.

---

46. GPU Isolation

GPU execution MUST respect ULABI security and capability boundaries.

An execution context MUST receive only the GPU resources and memory it is authorized to access.

One context MUST NOT gain access to another context's:

- memory;
- command queues;
- kernel state;
- private events;
- capabilities;

unless explicitly authorized.

---

47. GPU Capabilities as Security Capabilities

GPU capabilities MAY include:

GPUCompute
GPUDeviceMemory
GPUSharedMemory
GPUPeerAccess
GPUPerformanceCounters
GPUDeviceControl
GPUReconfiguration

A capability MUST represent an explicit authorization.

Possessing a GPU device identifier MUST NOT automatically grant permission to use every GPU operation.

---

48. Secure Module Loading

GPU modules SHOULD support integrity verification where the platform provides it.

A secure-loading implementation MAY validate:

- module identity;
- version;
- integrity hash;
- signature;
- target GPU;
- required capabilities;
- security policy;
- provenance.

A module failing required verification MUST NOT execute.

---

49. GPU Module Compatibility

A GPU module MUST declare its target requirements.

Conceptually:

GPUModuleCompatibility {
    interface_version
    architecture_requirements
    capability_requirements
    memory_requirements
    execution_requirements
}

A module MUST NOT execute merely because its binary format can be loaded.

Semantic compatibility MUST also be established.

---

50. Failure Model

GPU failures MUST be explicit.

Possible failures include:

DeviceUnavailable
DeviceLost
OutOfMemory
KernelFailure
InvalidKernel
InvalidArgument
InvalidMemoryAccess
QueueFailure
ExecutionTimeout
UnsupportedCapability
ModuleLoadFailure
SynchronizationFailure
ResetRequired
ResourceLimit

Failures MUST integrate with the generic ULABI error model.

GPU-specific errors MUST NOT replace the generic ULABI error contract.

---

51. Device Loss

A GPU MAY become unavailable during execution.

Examples include:

- hardware reset;
- driver failure;
- power event;
- device removal;
- virtualization failure;
- unrecoverable execution fault.

The implementation MUST report device loss explicitly.

It MUST NOT fabricate a successful result.

---

52. GPU Reset

A GPU implementation MAY support reset and recovery.

A reset MUST define its effect on:

- running kernels;
- queues;
- streams;
- events;
- memory;
- execution contexts;
- loaded modules.

Objects invalidated by reset MUST NOT remain usable as if nothing happened.

---

53. Recovery

Recovery MAY include:

Detect
  |
  v
Isolate
  |
  v
Drain / Cancel
  |
  v
Reset
  |
  v
Reinitialize
  |
  v
Reload Verified Modules
  |
  v
Verify
  |
  v
Resume or Escalate

Recovery MUST follow the generic ULABI reliability and recovery specifications.

A GPU implementation MUST NOT silently retry an operation if retrying can duplicate externally observable side effects.

---

54. Idempotency

GPU operations SHOULD declare whether they are:

Idempotent
NonIdempotent
Unknown

Automatic retry MUST be permitted only when the operation is known to be safe to retry or an explicit retry policy authorizes it.

---

55. Observability

GPU implementations SHOULD expose observability information through the generic ULABI observability model.

Possible information includes:

- dispatch identifiers;
- kernel identifiers;
- queue state;
- memory usage;
- execution duration;
- completion status;
- resource usage;
- device errors;
- reset events.

Observability data MUST NOT automatically grant additional GPU privileges.

---

56. Performance Counters

GPU performance counters MAY be exposed.

Examples include:

ExecutionTime
MemoryBandwidth
CacheActivity
Occupancy
ComputeUtilization
TransferRate
PowerUsage

Performance counters MUST be treated as implementation capabilities.

They MUST NOT alter the semantic result of a ULABI operation.

---

57. Error Containment

A GPU failure MUST NOT corrupt unrelated ULABI execution contexts where isolation is supported.

Implementations SHOULD isolate:

- command queues;
- memory regions;
- execution contexts;
- module state.

Where hardware limitations prevent complete isolation, the implementation MUST declare the limitation.

---

58. Graceful Degradation

If GPU execution is unavailable, an implementation MAY use an approved fallback.

Example:

GPU available?
   |
 +---+---+
 |       |
YES      NO
 |       |
GPU      CPU
 |       |
 +---+---+
     |
 Semantic Result

Fallback behavior MUST preserve the ULABI contract.

The fallback MUST NOT silently change:

- result types;
- error semantics;
- ownership;
- security requirements;
- required effects.

---

59. Version Compatibility

GPU interfaces MUST use ULABI versioning rules.

Compatibility MUST distinguish:

- interface version;
- kernel version;
- module version;
- GPU architecture version;
- capability set;
- driver/runtime version.

A newer GPU MUST NOT automatically imply compatibility with every older GPU module.

---

60. Feature Negotiation

GPU feature negotiation integrates with:

"docs/compatibility/feature-negotiation.md"

Example:

Required:
    Tensor
    FP16
    AsyncCopy

Available:
    Tensor
    FP16

Missing:
    AsyncCopy

The implementation MUST then:

1. use an approved fallback;
2. negotiate a weaker supported execution mode;
3. or reject execution.

Silent capability substitution is prohibited.

---

61. Security Requirements

A conforming GPU implementation MUST:

1. validate GPU commands before execution;
2. validate memory references;
3. enforce capability boundaries;
4. prevent unauthorized device-memory access;
5. validate module compatibility;
6. prevent malformed descriptors from causing uncontrolled access;
7. protect execution-context isolation where supported;
8. report device-loss and execution failures accurately;
9. follow secure-loading requirements where enabled;
10. preserve ULABI security semantics across CPU/GPU boundaries.

---

62. Invariants

The following invariants are mandatory.

Invariant 1 — Semantic Preservation

GPU execution MUST preserve the ULABI function contract.

Invariant 2 — Capability Isolation

A GPU context MUST NOT gain capabilities it was not granted.

Invariant 3 — Memory Safety

GPU operations MUST NOT bypass the applicable ULABI memory contract.

Invariant 4 — Explicit Failure

GPU failure MUST NOT be represented as successful completion.

Invariant 5 — Explicit Synchronization

Concurrent access MUST obey the declared synchronization model.

Invariant 6 — Deterministic Declaration

An implementation MUST NOT advertise deterministic behavior it cannot guarantee.

Invariant 7 — Version Integrity

A module MUST NOT execute when required compatibility constraints are unsatisfied.

Invariant 8 — Resource Boundedness

GPU resource consumption MUST remain within declared or enforced limits.

Invariant 9 — Isolation

Failure in one execution context MUST NOT silently corrupt unrelated contexts.

Invariant 10 — Vendor Independence

ULABI semantics MUST NOT depend on a particular GPU vendor.

---

63. Conformance Requirements

A conforming GPU implementation MUST demonstrate:

- GPU identity;
- capability discovery;
- kernel identity;
- module identity;
- kernel argument validation;
- command submission;
- queue semantics;
- synchronization;
- memory ownership;
- memory transfer;
- execution failure handling;
- resource-limit handling;
- compatibility checking;
- capability negotiation;
- security enforcement;
- device-loss handling where applicable;
- observability integration.

---

64. Required Conformance Tests

The ULABI test suite SHOULD contain at least:

gpu_identity
gpu_capability_discovery
gpu_interface_identity
gpu_kernel_identity
gpu_module_validation
gpu_argument_validation
gpu_command_submission
gpu_queue_ordering
gpu_queue_overflow
gpu_command_dependencies
gpu_stream_ordering
gpu_event_completion
gpu_event_failure
gpu_memory_allocation
gpu_memory_ownership
gpu_host_to_device
gpu_device_to_host
gpu_device_to_device
gpu_zero_copy
gpu_synchronization
gpu_atomic_operations
gpu_resource_limits
gpu_out_of_memory
gpu_execution_timeout
gpu_cancellation
gpu_device_loss
gpu_reset
gpu_recovery
gpu_module_compatibility
gpu_feature_negotiation
gpu_secure_loading
gpu_context_isolation
gpu_determinism
gpu_fallback
gpu_observability

Each test MUST identify:

- required capability;
- preconditions;
- operation;
- expected result;
- expected error;
- cleanup requirements.

---

65. Reference Implementation Boundary

A reference implementation SHOULD be divided into:

GPU Adapter
     |
     +-- Device Discovery
     +-- Capability Discovery
     +-- Module Manager
     +-- Kernel Manager
     +-- Command Manager
     +-- Queue Manager
     +-- Memory Manager
     +-- Synchronization Manager
     +-- Resource Manager
     +-- Security Manager
     +-- Error Manager
     +-- Recovery Manager
     +-- Observability Adapter

The reference implementation MUST NOT be treated as the specification.

The specification defines the contract.

The implementation demonstrates one valid realization.

---

66. Recommended Implementation Modules

The repository SHOULD eventually provide implementation modules with responsibilities separated as follows:

gpu/
├── mod
├── identity
├── capabilities
├── device
├── context
├── module
├── kernel
├── arguments
├── dispatch
├── command
├── queue
├── stream
├── event
├── memory
├── transfer
├── synchronization
├── atomic
├── resources
├── scheduler
├── compatibility
├── security
├── validation
├── errors
├── recovery
├── observability
└── conformance

Language-specific implementations MAY use different module names and structures.

The semantic responsibilities MUST remain equivalent.

---

67. Required Integration Files

The GPU implementation MUST integrate with the following generic ULABI modules:

Core ABI
    -> function contracts
    -> interface identity
    -> errors

Calling Convention
    -> kernel arguments
    -> host-side dispatch

Data Types
    -> kernel data
    -> buffers
    -> descriptors

Memory Model
    -> ownership
    -> lifetime
    -> sharing

Runtime Interface
    -> execution contexts
    -> scheduling
    -> lifecycle

Async Model
    -> asynchronous dispatch
    -> events
    -> completion

Resource Management
    -> GPU budgets
    -> memory limits
    -> execution limits

Security Model
    -> isolation
    -> authorization

Capability Security
    -> GPU capabilities
    -> device access

Secure Loading
    -> GPU modules

Feature Negotiation
    -> GPU capability compatibility

Capability Discovery
    -> hardware feature discovery

Graceful Degradation
    -> CPU/software fallback

Reliability
    -> device failure
    -> recovery
    -> reset

Observability
    -> execution diagnostics
    -> tracing
    -> telemetry

Conformance
    -> GPU compliance

Test Suite
    -> GPU conformance tests

This document intentionally defines these integration points now so that later generic specifications do not require redesigning the GPU contract.

---

68. Non-Interference Rule

Future ULABI specifications MUST NOT modify the meaning of this GPU specification merely by adding detail to another subsystem.

If another specification introduces a stronger generic requirement, a GPU implementation MUST adapt through:

- an implementation profile;
- capability negotiation;
- a versioned extension;
- or a new GPU profile.

Existing compatible GPU contracts MUST remain stable.

---

69. Implementation Independence

A GPU implementation may be written in:

- C;
- C++;
- Rust;
- Go;
- Zig;
- Java;
- another systems language;
- another implementation language.

ULABI does not prescribe the implementation language.

Similarly, ULABI does not prescribe:

- CUDA;
- HIP;
- OpenCL;
- Vulkan;
- Metal;
- Direct3D;
- one GPU vendor;
- one operating system.

Those technologies MAY be adapters beneath the ULABI boundary.

---

70. Summary

The ULABI GPU specification establishes a stable semantic boundary between ULABI and GPU hardware.

The architecture is:

ULABI Contract
      |
      v
GPU Adapter
      |
      v
GPU Runtime / Driver
      |
      v
GPU Hardware

The GPU layer provides:

- hardware discovery;
- capability discovery;
- kernel execution;
- command submission;
- asynchronous execution;
- streams;
- events;
- memory management;
- host/device transfers;
- synchronization;
- resource accounting;
- heterogeneous execution;
- secure module loading;
- compatibility;
- failure containment;
- recovery;
- observability;
- conformance.

The central rule is:

«GPU hardware is an implementation target of ULABI, not the definition of ULABI.»

A ULABI implementation MUST preserve semantic interoperability regardless of the GPU architecture used underneath.Files/modules that this document establishes

The documentation file is:

docs/hardware/gpu.md

The eventual implementation should be split rather than creating one monolithic gpu module:

implementations/
└── <implementation>/
    └── gpu/
        ├── mod
        ├── identity
        ├── capabilities
        ├── device
        ├── context
        ├── module
        ├── kernel
        ├── arguments
        ├── dispatch
        ├── command
        ├── queue
        ├── stream
        ├── event
        ├── memory
        ├── transfer
        ├── synchronization
        ├── atomic
        ├── resources
        ├── scheduler
        ├── compatibility
        ├── security
        ├── validation
        ├── errors
        ├── recovery
        ├── observability
        └── conformance

These are semantic module responsibilities, not a requirement that every implementation use exactly these filenames.

Integration order

To satisfy your "finish one file without later reopening it" requirement, I would treat the GPU work in this order:

1. docs/hardware/gpu.md — independent GPU contract


2. docs/hardware/cpu.md — host-side CPU realization


3. docs/hardware/fpga.md — separate programmable-logic realization


4. docs/hardware/npu.md — neural-processor realization


5. docs/hardware/future-architectures.md — future/unknown hardware boundary


6. docs/abi/memory-model.md + docs/memory/shared-memory.md — shared memory semantics


7. docs/runtime/resource-management.md — resource accounting


8. docs/runtime/async-model.md — asynchronous execution semantics


9. docs/security/capability-security.md — authorization


10. docs/compatibility/capability-discovery.md — capability discovery


11. docs/compatibility/feature-negotiation.md — GPU feature negotiation


12. docs/reliability/recovery.md / rollback.md — recovery behavior


13. docs/standards/conformance.md — compliance classification


14. docs/standards/test-suite.md — executable GPU tests


15. implementation modules — actual adapters.



That separation is important because the repository already treats the hardware documents as specialization layers rather than places to redefine the ULABI Core. The existing architecture explicitly supports hardware/accelerator profiles, and the repository contains the corresponding CPU, FPGA, ABI, compatibility, and architecture documents that establish those boundaries.

