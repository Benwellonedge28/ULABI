I inspected the actual Benwellonedge28/ULABI repository before drafting this. The repository confirms that ULABI-DESIGN.md is the architectural authority and already establishes language/runtime/platform neutrality, the Minimal Core + Profiles model, transport independence, capability discovery, compatibility, hardware interoperability, and the separation between semantic ULABI contracts and implementation-specific mechanisms.

I also checked the existing hardware area. docs/hardware/fpga.md is currently only a placeholder, so this is the correct time to establish the FPGA contract rather than trying to retrofit an existing specification.  The existing CPU specification establishes the complementary principle that hardware-specific adapters lower ULABI semantics to physical hardware without changing the ULABI contract.  GPU is also still a placeholder, so FPGA should not duplicate GPU/accelerator semantics.

What this document must own

docs/hardware/fpga.md should own the FPGA execution and hardware-boundary contract:

FPGA identity and capability discovery

bitstream/image identity

static vs dynamically reconfigurable FPGA regions

ULABI interface realization on FPGA

hardware function/module boundaries

data movement

DMA

host ↔ FPGA interaction

memory-mapped interfaces

streaming interfaces

FPGA-local memory

synchronization

clocks and timing domains

reset behavior

interrupts/events

reconfiguration

partial reconfiguration

isolation

attestation/integrity

resource limits

deterministic and real-time behavior

failure handling

rollback of FPGA images

version compatibility

conformance


It should not define:

a universal FPGA architecture;

a universal HDL;

Verilog/SystemVerilog/VHDL semantics;

a particular FPGA vendor;

PCIe as the mandatory transport;

GPU semantics;

CPU instruction sets;

generic serialization rules;

generic distributed consensus;

generic security rules already owned by security specifications.


That separation is consistent with the repository's existing architecture. The CPU document explicitly avoids putting networking, serialization, GPU semantics, or a particular native CPU ABI into the CPU contract.


---

docs/hardware/fpga.md

Copy/paste the following as the complete file.

# ULABI FPGA Hardware Interface Specification

Document: `docs/hardware/fpga.md`

Project: ULABI

Full Name: Universal Language Application Binary Interface

Specification Family: Hardware

Component: FPGA Interface

Status: Architecture / Design Specification

Specification Version: 0.1.0-draft

ULABI Architecture Version: 0.2.0-draft

Normative Authority:
- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

License: Apache-2.0


---

# 1. Purpose

This document defines the ULABI contract for interfacing with Field-Programmable Gate Arrays (FPGAs).

The purpose of this specification is not to define one universal FPGA architecture.

It defines how a ULABI implementation exposes FPGA-resident computation, memory, streams, events, and hardware resources through stable ULABI contracts.

The conceptual architecture is:

ULABI Interface
       |
       v
FPGA ULABI Adapter
       |
       v
FPGA Interface Boundary
       |
       +----------------------+
       |                      |
   FPGA Fabric          FPGA Memory
       |                      |
       +----------+-----------+
                  |
             FPGA Hardware


The FPGA implementation may be:

- standalone;
- attached to a CPU;
- attached to an accelerator host;
- embedded in a system-on-chip;
- connected through a device interconnect;
- dynamically reconfigured;
- partially reconfigured;
- remotely managed.

ULABI semantics MUST remain independent of the FPGA vendor and implementation technology.


---

# 2. Fundamental Principle

The fundamental rule is:

> ULABI defines the semantic hardware contract; an FPGA adapter defines how that contract is realized in FPGA fabric.

Therefore:

ULABI Function
      |
      +---- FPGA implementation A
      |
      +---- FPGA implementation B
      |
      +---- CPU implementation
      |
      +---- software fallback
      |
      +---- future hardware implementation

All implementations MUST preserve the same declared ULABI semantics.

An FPGA implementation MAY use:

- LUTs;
- flip-flops;
- block RAM;
- distributed RAM;
- UltraRAM or equivalent;
- DSP blocks;
- hard processor systems;
- transceivers;
- vendor-specific IP;
- custom logic;
- soft processors;
- programmable interconnects;
- hardware pipelines;
- hardware state machines;
- hardware accelerators.

These are implementation details unless explicitly exposed through an FPGA-specific capability profile.


---

# 3. Scope

This specification defines:

1. FPGA identity;

2. FPGA device identity;

3. FPGA capability discovery;

4. FPGA interface identity;

5. hardware function identity;

6. FPGA image identity;

7. bitstream integrity;

8. bitstream versioning;

9. static FPGA regions;

10. dynamic FPGA regions;

11. partial reconfiguration;

12. host-to-FPGA interfaces;

13. FPGA-to-host interfaces;

14. FPGA-local interfaces;

15. memory interfaces;

16. DMA;

17. memory-mapped access;

18. streaming access;

19. queues;

20. descriptors;

21. interrupts;

22. events;

23. synchronization;

24. clock domains;

25. reset domains;

26. timing guarantees;

27. deterministic execution;

28. real-time behavior;

29. resource accounting;

30. resource limits;

31. isolation;

32. capability restrictions;

33. image authentication;

34. image integrity;

35. image rollback;

36. failure detection;

37. recovery;

38. reconfiguration safety;

39. compatibility;

40. conformance testing.

This specification does NOT define:

- a universal FPGA;
- a universal HDL;
- Verilog semantics;
- SystemVerilog semantics;
- VHDL semantics;
- a vendor-specific programming environment;
- a particular FPGA family;
- a particular board;
- a mandatory host transport;
- a universal physical interconnect;
- GPU programming semantics;
- CPU instruction-set semantics.


---

# 4. Relationship to Other ULABI Specifications

The FPGA layer is below the semantic ULABI interface layer and above physical FPGA implementation details.

The conceptual relationship is:

ULABI-DESIGN.md
       |
ULABI-SPEC.md
       |
ULABI Core ABI
       |
Hardware Interface
       |
FPGA Interface
       |
FPGA Adapter
       |
FPGA Fabric
       |
Physical Device


The FPGA specification integrates with:

- `docs/abi/core-abi.md`
- `docs/abi/calling-convention.md`
- `docs/abi/data-types.md`
- `docs/abi/memory-model.md`
- `docs/abi/register-model.md`
- `docs/runtime/runtime-interface.md`
- `docs/runtime/resource-management.md`
- `docs/memory/memory-safety.md`
- `docs/memory/shared-memory.md`
- `docs/security/security-model.md`
- `docs/security/capability-security.md`
- `docs/security/secure-loading.md`
- `docs/security/supply-chain-security.md`
- `docs/compatibility/feature-negotiation.md`
- `docs/compatibility/capability-discovery.md`
- `docs/compatibility/graceful-degradation.md`
- `docs/observability/diagnostics.md`
- `docs/observability/tracing.md`
- `docs/reliability/fault-detection.md`
- `docs/reliability/fault-isolation.md`
- `docs/reliability/recovery.md`
- `docs/reliability/rollback.md`
- `docs/standards/conformance.md`
- `docs/standards/test-suite.md`
- `docs/standards/certification.md`

These documents retain ownership of their respective generic concepts.

This document only defines their FPGA realization.

No later document should require this document to be rewritten merely because those specifications become more detailed.

FPGA-specific extensions MUST be added as profiles or explicitly versioned additions rather than silently changing existing semantics.


---

# 5. FPGA Identity

Every FPGA implementation MUST expose a stable hardware identity.

Conceptually:

```text
FPGAIdentity {
    vendor
    device_family
    device_model
    device_revision
    architecture
    fabric_capacity
    memory_capacity
    feature_set
}

The identity MUST be distinguishable from:

board identity;

operating-system identity;

host CPU identity;

compiler identity;

HDL toolchain identity.


A board containing an FPGA MUST NOT be treated as the FPGA identity itself.


---

6. FPGA Capability Identity

ULABI MUST distinguish:

FPGA identity

from:

FPGA capabilities.

A device MAY support capabilities such as:

LogicFabric
BlockMemory
DistributedMemory
DSP
HighSpeedIO
DMA
Interrupts
PartialReconfiguration
ECCMemory
SecureBoot
Attestation
HardwareCrypto
ClockManagement
Streaming
PCIe
Ethernet
CXL
AXI
VendorInterconnect

Capability names are semantic.

Physical implementation names remain vendor-specific unless standardized by a profile.


---

7. FPGA Interface Identity

Every ULABI-visible FPGA interface MUST have a stable interface identity.

Conceptually:

FPGAInterfaceIdentity {
    interface_id
    version
    semantic_contract
    data_schema
    execution_model
    resource_requirements
    capability_requirements
}

The interface identity MUST remain stable across compatible FPGA image revisions.

Changing the semantic contract requires a new compatible interface version or a new interface identity according to ULABI versioning rules.


---

8. FPGA Hardware Function

An FPGA-resident hardware function is a ULABI-visible computational component.

Example:

function matrix_multiply(
    input: Tensor,
    weights: Tensor
) -> Result<Tensor, HardwareError>

The function MAY be implemented using:

pipelined logic;

parallel datapaths;

DSP blocks;

memory engines;

custom state machines;

hardware schedulers.


The ULABI function contract remains independent of the implementation.

The implementation MUST preserve:

input semantics;

output semantics;

error semantics;

ownership;

lifetime;

capability requirements;

execution semantics.



---

9. Execution Models

An FPGA function MAY declare one or more execution models.

Supported semantic models include:

Synchronous
Asynchronous
Streaming
Batch
Pipeline
Persistent
EventDriven
Cancellable
NonCancellable
Deterministic
BestEffort
RealTime

The implementation MUST NOT advertise stronger guarantees than it can actually provide.


---

10. FPGA Execution Context

An FPGA execution context represents the resources and state associated with a ULABI hardware invocation.

Conceptually:

FPGAExecutionContext {
    context_id
    interface_id
    resource_budget
    memory_budget
    execution_deadline
    capability_set
    cancellation_policy
}

An execution context MUST have explicitly defined:

ownership;

lifetime;

resource limits;

access rights;

cancellation behavior;

failure behavior.



---

11. Host-to-FPGA Boundary

A host may communicate with an FPGA through:

memory-mapped I/O;

DMA;

queues;

shared memory;

streaming interfaces;

interrupts;

polling;

device-specific transports.


ULABI MUST NOT require one transport.

The FPGA adapter MUST expose the semantic interface independently from the transport.


---

12. FPGA-to-Host Boundary

FPGA implementations MAY return information through:

direct memory writes;

completion queues;

interrupts;

event channels;

descriptors;

shared memory;

transport-specific mechanisms.


The ULABI semantic result remains authoritative.

A transport status value MUST NOT silently replace a ULABI result or error.


---

13. Memory-Mapped Interfaces

An FPGA MAY expose memory-mapped registers or memory regions.

A ULABI implementation MUST define:

address-space identity;

access permissions;

width;

alignment;

side effects;

ordering;

volatility;

lifetime;

capability requirements.


A raw physical address MUST NOT automatically constitute a ULABI capability.


---

14. FPGA Register Interfaces

Hardware registers MAY expose:

control;

status;

configuration;

counters;

descriptors;

interrupt state;

diagnostic information.


Each ULABI-visible register interface MUST have defined:

Register {
    identifier
    width
    access
    reset_value
    side_effects
    ordering
    version
}

Undefined register behavior MUST NOT be relied upon by conforming implementations.


---

15. DMA

DMA MAY be used to transfer data between:

host memory;

FPGA memory;

device memory;

shared memory.


DMA operations MUST define:

source;

destination;

length;

ownership;

lifetime;

access permissions;

alignment;

completion semantics;

failure semantics.


DMA MUST NOT bypass ULABI memory-safety or capability rules.


---

16. DMA Ownership

A DMA buffer MUST have an explicit ownership state.

Conceptually:

HostOwned
    |
    v
TransferredToFPGA
    |
    v
FPGAOwned
    |
    v
TransferComplete
    |
    v
HostOwned

An implementation MAY use different internal states.

The semantic rule is that simultaneous conflicting ownership MUST NOT be assumed.

A buffer MUST NOT be modified concurrently by two owners unless the contract explicitly permits shared access.


---

17. Zero-Copy

Zero-copy operation MAY be supported.

Zero-copy MUST NOT mean:

> arbitrary access to another component's memory.



Zero-copy requires explicit compatibility of:

addressability;

lifetime;

ownership;

access permissions;

alignment;

cache coherence;

synchronization;

isolation.


If those requirements cannot be satisfied, the implementation MUST use a safe copy or another approved representation.


---

18. FPGA-Local Memory

FPGA implementations MAY contain:

block RAM;

distributed RAM;

external DRAM;

HBM;

SRAM;

vendor-specific memory.


The FPGA adapter MUST expose only the semantic properties required by the ULABI interface.

For example:

MemoryRegion {
    region_id
    capacity
    alignment
    access
    persistence
    coherence
    latency_class
}

Physical memory technology MUST remain implementation-specific unless explicitly exposed through a hardware profile.


---

19. Streaming Interfaces

FPGA implementations are particularly suitable for streaming computation.

ULABI streaming interfaces MAY define:

Stream<T>

A stream contract MUST define:

element type;

ordering;

backpressure;

buffering;

termination;

cancellation;

error behavior;

ownership;

flow-control behavior.



---

20. Backpressure

Streaming interfaces MUST explicitly define backpressure behavior.

Possible states include:

Ready
Blocked
Paused
Draining
Closed
Failed

A producer MUST NOT assume unlimited buffering.

An implementation MUST provide bounded behavior or explicitly declare its buffering semantics.


---

21. Queues

An FPGA interface MAY expose:

submission queues;

completion queues;

event queues;

command queues;

data queues.


Each queue MUST define:

capacity;

element type;

ownership;

ordering;

overflow behavior;

synchronization;

reset behavior;

failure behavior.



---

22. Descriptor Model

Hardware operations MAY use descriptors.

Conceptually:

Descriptor {
    operation_id
    input_reference
    output_reference
    length
    flags
    capability
    completion_reference
}

Descriptors MUST be validated before execution.

A malformed descriptor MUST NOT cause arbitrary memory access or uncontrolled hardware behavior.


---

23. Interrupts and Events

FPGA implementations MAY expose:

Completion
Error
Threshold
Timer
DataAvailable
ResourceAvailable
DeviceEvent

Events MUST have stable semantic identifiers.

An interrupt number is not itself a ULABI semantic event identity.

The adapter translates physical interrupts into ULABI events.


---

24. Polling

Polling MAY be used where interrupts are unavailable or undesirable.

Polling MUST preserve the same semantic state transitions as interrupt-driven execution.

An implementation MUST NOT produce different externally observable semantics merely because polling is used internally.


---

25. Clock Domains

FPGAs may contain multiple clock domains.

The ULABI boundary MUST distinguish:

clock-domain identity;

synchronization boundary;

timing guarantee;

data-transfer mechanism.


Cross-clock-domain communication MUST use a safe synchronization mechanism.

Unsynchronized crossing of independently clocked state MUST NOT be treated as conforming behavior.


---

26. Timing Contracts

An FPGA interface MAY specify timing guarantees.

Possible guarantees include:

BestEffort
BoundedLatency
DeterministicLatency
Periodic
DeadlineBounded
HardRealTime
SoftRealTime

A timing guarantee MUST include sufficient information to determine whether the guarantee is meaningful.

An implementation MUST NOT claim deterministic latency when scheduling, buffering, transport, or memory behavior violates the declared bound.


---

27. Deterministic FPGA Execution

Where an interface declares deterministic execution, the implementation MUST define the relevant deterministic boundary.

Determinism MAY include:

output determinism;

ordering determinism;

timing determinism;

resource-use determinism.


Physical hardware behavior that does not affect the declared semantic guarantee MAY remain implementation-specific.


---

28. Real-Time FPGA Interfaces

FPGA implementations MAY provide real-time interfaces.

A real-time contract MUST define:

deadline;

scheduling model;

maximum execution time where applicable;

queueing limits;

memory limits;

failure behavior;

overload behavior.


Failure to meet a deadline MUST have explicitly defined semantics.

A missed deadline MUST NOT silently become a successful operation.


---

29. Resource Model

FPGA resources MAY include:

Logic
Registers
BRAM
URAM
DSP
MemoryBandwidth
ExternalMemory
I/O
Transceivers
ClockResources
PowerBudget
ThermalBudget
ReconfigurationRegion

ULABI MUST treat these as resource constraints rather than universal physical units.

A resource profile MAY expose standardized abstract resource classes.


---

30. Resource Accounting

A hardware function MAY declare:

ResourceRequirements {
    logic_class
    memory
    bandwidth
    dsp
    io
    clock
    power
}

The values MUST be interpreted according to the relevant hardware profile.

A function MUST NOT assume that a named physical resource exists unless the capability has been negotiated.


---

31. Resource Limits

FPGA execution MUST respect configured resource limits.

Possible failures include:

ResourceUnavailable
ResourceLimitExceeded
MemoryUnavailable
BandwidthUnavailable
ExecutionCapacityExceeded
ThermalLimit
PowerLimit

Resource exhaustion MUST be reported through the ULABI error model.


---

32. Static FPGA Regions

A device MAY contain static regions that remain active across reconfiguration.

Static regions MAY contain:

security logic;

management logic;

host interfaces;

monitoring;

memory controllers;

isolation boundaries.


The ULABI contract for static regions MUST remain stable across compatible dynamic-region changes.


---

33. Dynamic FPGA Regions

An FPGA MAY support dynamically replaceable computation regions.

A dynamic region MUST have:

region identity;

interface contract;

resource boundary;

isolation policy;

lifecycle;

image compatibility requirements.


Replacing a dynamic region MUST NOT invalidate unrelated ULABI interfaces.


---

34. Partial Reconfiguration

Partial reconfiguration MAY replace a portion of FPGA logic while other logic remains operational.

The implementation MUST ensure:

1. affected interfaces are quiesced;


2. outstanding operations are handled according to policy;


3. memory ownership remains valid;


4. capabilities are not unintentionally expanded;


5. the new image is authenticated;


6. interface compatibility is verified;


7. the region is safely activated.



Partial reconfiguration MUST NOT create an uncontrolled execution path.


---

35. Reconfiguration Lifecycle

The semantic lifecycle is:

Prepare | Quiesce | Validate | Authenticate | Load | Initialize | Verify | Activate | Healthy

Failure MUST transition to a defined recovery state.

Possible states:

PreparationFailed
ValidationFailed
AuthenticationFailed
LoadFailed
InitializationFailed
VerificationFailed
ActivationFailed
RecoveryRequired


---

36. FPGA Image Identity

Every executable FPGA image MUST have a stable identity.

Conceptually:

FPGAImageIdentity {
    image_id
    version
    interface_version
    target_device
    required_capabilities
    build_identity
    integrity_digest
}

The image identity MUST NOT rely solely on a filename.


---

37. FPGA Image Compatibility

Before loading an image, the implementation MUST verify compatibility with:

target device;

device revision;

interface version;

required capabilities;

memory requirements;

resource requirements;

security policy.


Incompatible images MUST be rejected.


---

38. Bitstream Integrity

An FPGA image MUST be integrity-checked before activation when the security profile requires integrity verification.

Integrity verification MAY use:

cryptographic digests;

authenticated manifests;

signed images;

hardware roots of trust.


The exact cryptographic mechanism is governed by the security specifications and applicable profiles.

This document defines the FPGA lifecycle requirement, not a new cryptographic standard.


---

39. Secure FPGA Loading

FPGA images MUST be subject to the applicable secure-loading policy.

The implementation MUST support policy-controlled checks such as:

ImageIdentity
TargetDevice
Version
Integrity
Authenticity
Authorization
CapabilityRequirements

An image that fails mandatory validation MUST NOT be activated.


---

40. Rollback

A failed FPGA update MUST support rollback where the implementation declares rollback capability.

The lifecycle is:

CurrentHealthyImage | v CandidateImage | v Validation | v Activation | v Verification /   
Healthy Failed |      | v      v Commit   Rollback

Rollback MUST restore a known-good compatible state.

Rollback MUST NOT silently revert security policy or authorization state.


---

# 41. FPGA State Preservation

Before reconfiguration, the implementation MUST explicitly define which state is:

- preserved;
- migrated;
- discarded;
- reconstructed;
- invalidated.

State MUST NOT be assumed to survive reconfiguration unless the contract explicitly guarantees preservation.


---

# 42. In-Flight Operations

When reconfiguration affects active operations, the implementation MUST choose one of the declared policies:

```text
Complete
Cancel
Drain
Checkpoint
Migrate
Fail

The chosen policy MUST be deterministic for the interface profile.

An operation MUST NOT be silently lost while being reported as successful.


---

43. Isolation

An FPGA implementation MAY contain multiple independently managed hardware functions.

The implementation MUST provide isolation where required by the security profile.

Isolation MAY apply to:

memory;

DMA;

configuration;

interrupts;

reconfiguration;

control registers;

host access;

device resources.


A hardware function MUST receive only the capabilities required by its contract.


---

44. Capability Security

FPGA capabilities integrate with:

docs/security/capability-security.md

A capability MAY authorize access to:

MemoryRegion
DMAChannel
InterruptSource
DeviceRegister
Stream
Queue
ReconfigurationRegion
HardwareFunction

Possession of a physical address MUST NOT automatically grant capability access.


---

45. DMA Isolation

DMA-capable FPGA logic MUST be prevented from accessing memory outside its authorized boundary.

The implementation MAY use:

IOMMU;

memory protection;

address translation;

hardware firewalls;

capability tags;

device isolation;

equivalent mechanisms.


The exact mechanism is implementation-specific.

The semantic security requirement is mandatory.


---

46. Fault Detection

FPGA implementations SHOULD detect relevant failures such as:

configuration failure;

clock failure;

memory error;

ECC failure;

protocol violation;

queue corruption;

DMA error;

timeout;

thermal fault;

power fault;

hardware function failure.


Detected failures MUST be represented through the ULABI failure model.


---

47. Fault Containment

A hardware failure MUST be contained to the smallest safe boundary.

Possible containment levels include:

Operation
Function
DynamicRegion
Device
Host
System

The implementation SHOULD avoid escalating a localized hardware failure to the entire system when safe isolation is possible.


---

48. Recovery

Recovery MAY include:

Retry
ResetFunction
ResetRegion
RestartContext
ReinitializeMemory
ReloadImage
RollbackImage
DisableCapability
Escalate

Recovery behavior MUST be policy-controlled.

ULABI does not authorize arbitrary self-modification of FPGA logic.


---

49. Hardware Reset

Reset behavior MUST be explicitly defined.

Reset MAY apply to:

function;

region;

device;

interface;

queue;

DMA engine.


Reset MUST define the resulting state of:

ownership;

outstanding operations;

memory;

events;

capabilities;

interface state.



---

50. Watchdogs

An FPGA implementation MAY use watchdogs.

A watchdog MUST have:

scope;

timeout;

trigger condition;

recovery action;

escalation policy.


Watchdogs MUST NOT cause an unauthorized configuration change.


---

51. Observability

FPGA interfaces MAY expose:

health;

counters;

latency;

throughput;

errors;

resource utilization;

temperature;

power;

configuration state.


Observability data MUST NOT alter the semantic behavior of the interface.

Sensitive hardware information MUST be governed by the applicable security policy.


---

52. Debugging

FPGA debugging MAY expose:

trace buffers;

state snapshots;

logic analyzers;

performance counters;

hardware breakpoints;

diagnostic registers.


Debug capabilities MUST be explicitly authorized.

Production systems MUST NOT expose unrestricted debug access merely because the FPGA supports it physically.


---

53. Security Boundary

An FPGA MAY represent a security boundary.

Security-sensitive FPGA functions MUST define:

trust level;

capabilities;

memory access;

configuration authority;

reconfiguration authority;

debug authority;

attestation requirements.


The FPGA MUST NOT be assumed trustworthy merely because it is hardware.


---

54. Attestation

Where hardware attestation is supported, the FPGA implementation MAY expose:

Attestation {
    device_identity
    image_identity
    image_measurement
    configuration_state
    security_state
}

Attestation semantics MUST integrate with the security and secure-loading specifications.

An attestation result MUST NOT be claimed unless the implementation can provide evidence supporting the declared measurement.


---

55. Supply-Chain Security

FPGA images, configuration artifacts, and hardware IP MAY form part of the software/hardware supply chain.

Implementations SHOULD support:

provenance;

integrity verification;

signed artifacts;

reproducible-build metadata where possible;

dependency identification;

version verification;

revocation policy.


Supply-chain requirements remain governed by:

docs/security/supply-chain-security.md


---

56. Compatibility

FPGA compatibility MUST be evaluated across at least:

Interface
Image
Device
Capabilities
Resources
Security Policy

A compatible image MUST preserve the declared interface semantics.

An image that requires unsupported hardware capabilities MUST NOT be treated as compatible.


---

57. Graceful Degradation

If an FPGA capability is unavailable, the implementation MAY:

1. use a software implementation;


2. use another hardware implementation;


3. use a reduced-capability FPGA implementation;


4. disable an optional feature;


5. reject the operation.



The fallback MUST NOT silently change required semantics.


---

58. Software Fallback

A software fallback MAY implement the same ULABI interface.

Example:

ULABI Function
      |
   +--+----------------+
   |                   |
 FPGA Implementation  CPU Implementation

The fallback MUST preserve the semantic contract.

Performance MAY differ unless performance is itself part of the contract.


---

59. Versioning

FPGA interface versions MUST follow ULABI versioning rules.

An image update MUST distinguish:

image version;

interface version;

hardware target version;

capability version.


Changing implementation internals without changing externally visible semantics SHOULD NOT require a new interface identity.


---

60. Feature Negotiation

FPGA capabilities integrate with:

docs/compatibility/feature-negotiation.md

Example:

Required:
    Streaming
    DMA

Optional:
    PartialReconfiguration
    ECCMemory

The implementation MUST distinguish:

Required capability from Optional capability.

Missing required capabilities MUST cause negotiation failure or an approved fallback.


---

61. Capability Discovery

FPGA capability discovery integrates with:

docs/compatibility/capability-discovery.md

Conceptually:

FPGA | v Capability Query | +-------------------+ |                   | Supported          Unsupported |                   | Use                 Fallback / Reject

Capabilities MUST NOT be inferred solely from the FPGA model name.


---

# 62. Concurrency

Multiple FPGA functions MAY execute concurrently.

The implementation MUST define:

- shared resource policy;
- memory ownership;
- queue isolation;
- ordering;
- synchronization;
- priority;
- starvation behavior.

Concurrency MUST NOT create undefined ownership or memory behavior.


---

# 63. Atomicity

Where FPGA hardware provides atomic operations, the implementation MAY expose them through an appropriate profile.

Atomicity MUST be explicitly defined at the ULABI boundary.

A vendor-specific hardware atomic primitive MUST NOT automatically become a universal ULABI primitive.


---

# 64. Ordering

The implementation MUST distinguish:

```text
Program Order
Memory Order
Queue Order
Stream Order
Completion Order

These orders MUST NOT be conflated.

If ordering is observable at the ULABI boundary, it MUST be explicitly specified.


---

65. Memory Coherence

FPGA and host memory MAY be:

Coherent
NonCoherent
ExplicitlySynchronized
DeviceLocal

The implementation MUST explicitly declare the applicable memory model.

A conforming implementation MUST NOT assume cache coherence where it is not guaranteed.


---

66. Cache Synchronization

For non-coherent memory, ownership transitions MUST include required synchronization.

Conceptually:

HostWrite
   |
Flush / Synchronize
   |
FPGARead
   |
FPGAWrite
   |
Invalidate / Synchronize
   |
HostRead

The exact mechanism is implementation-specific.

The semantic visibility requirements are mandatory.


---

67. Error Model

FPGA errors MUST map into the ULABI error model.

Possible hardware-specific errors include:

DeviceUnavailable
ImageInvalid
ImageIncompatible
ConfigurationFailed
ReconfigurationFailed
DMAError
MemoryError
ClockFailure
Timeout
ResourceUnavailable
ThermalFault
PowerFault
ProtocolViolation
HardwareFunctionFailed
SecurityViolation

Hardware-specific errors MUST remain distinguishable where applications need to react differently.


---

68. Error Recovery

An FPGA operation that fails MUST have a defined recovery policy.

Possible policies:

Retry
Reset
Restart
Fallback
Rollback
Escalate
Fail

The implementation MUST NOT report an operation as successful after an unrecovered hardware failure.


---

69. Cancellation

A cancellable FPGA operation MUST define:

cancellation request;

cancellation acknowledgment;

cleanup;

memory ownership;

completion state.


Cancellation MUST NOT create dangling DMA operations or inaccessible buffers.


---

70. Resource Revocation

Capabilities and resources MAY be revoked.

Revocation MUST define the behavior of active operations.

Possible outcomes include:

Complete
Cancel
Fail
Migrate
Quiesce

Resource revocation MUST NOT silently grant the function a replacement capability.


---

71. Security and Reconfiguration Authority

Reconfiguration MUST require explicit authorization.

The following MUST NOT automatically imply reconfiguration authority:

access to a ULABI function;

access to FPGA memory;

access to DMA;

host administrator privileges;

possession of an image.


The applicable security policy determines who may reconfigure the device.


---

72. Power and Thermal Constraints

Where exposed by the hardware profile, FPGA execution MAY be constrained by:

power;

temperature;

cooling;

voltage;

thermal throttling.


Thermal or power limitations MUST produce explicit resource or hardware errors where they affect operation.


---

73. Multi-FPGA Systems

A system MAY contain multiple FPGAs.

Each FPGA MUST have an independent hardware identity.

Cross-FPGA communication MUST be represented as an explicit interface or transport.

One FPGA MUST NOT implicitly access another FPGA's resources without an authorized capability.


---

74. FPGA + CPU Systems

An FPGA may operate alongside one or more CPUs.

The architecture is:

CPU | ULABI Adapter | FPGA Interface | FPGA Fabric

The CPU and FPGA MAY use different internal execution models.

ULABI defines only the shared semantic contract.


---

75. FPGA + Accelerator Systems

An FPGA MAY cooperate with:

CPU;

GPU;

NPU;

DSP;

ASIC;

other accelerators.


Cross-accelerator semantics MUST use explicit ULABI interfaces.

This document does not define GPU or NPU semantics.


---

76. FPGA as a ULABI Provider

An FPGA implementation MAY provide ULABI interfaces.

Example:

ULABI Registry
      |
      v
FPGA Provider
      |
      +-- Function A
      +-- Function B
      +-- Stream C
      +-- Memory D

The provider MUST expose:

interface identity;

version;

capabilities;

resource requirements;

security requirements;

lifecycle state.



---

77. FPGA as a ULABI Consumer

An FPGA implementation MAY consume ULABI interfaces from:

CPU;

runtime;

operating system;

another FPGA;

accelerator;

distributed service.


The FPGA MUST receive only the capabilities required by the consumer contract.


---

78. Distributed FPGA Operation

An FPGA MAY participate in distributed execution.

However:

Local FPGA Execution
        !=
Distributed Execution

Network communication, serialization, service discovery, and distributed failure semantics belong to the distributed specifications.

An FPGA-specific implementation MUST use those contracts rather than redefining them.


---

79. Transport Independence

Possible transports include:

PCIe;

CXL;

AXI;

Ethernet;

shared memory;

device-local interconnect;

custom vendor interconnect.


No transport is mandatory.

The ULABI interface MUST remain stable when the transport changes.


---

80. Persistence

An FPGA configuration MAY be:

Volatile
Persistent
HostManaged
DeviceManaged
SecurePersistent

Persistence semantics MUST be explicitly declared.

A persistent configuration MUST NOT be assumed to survive a power cycle unless guaranteed by the hardware profile.


---

81. Lifecycle

The complete semantic FPGA lifecycle is:

Discover | Validate | Authorize | Load | Initialize | Verify | Activate | Execute | Monitor | Quiesce | Reconfigure / Shutdown | Release

Each transition MUST have defined success and failure behavior.


---

82. State Machine

The FPGA device lifecycle MAY be represented as:

Unavailable
    |
    v
Discovered
    |
    v
Validated
    |
    v
Authorized
    |
    v
Loaded
    |
    v
Initialized
    |
    v
Verified
    |
    v
Active
    |
    +------> Faulted
    |           |
    |           v
    |        Recovering
    |           |
    |           v
    +-------- Active
    |
    v
Quiesced
    |
    +----> Reconfigured
    |
    v
Released


---

83. Invariants

A conforming FPGA implementation MUST preserve the following invariants.

Invariant 1 — Semantic Preservation

FPGA lowering MUST preserve the ULABI interface semantics.

Invariant 2 — Capability Restriction

FPGA functions MUST NOT receive capabilities beyond those authorized.

Invariant 3 — Memory Safety

DMA and memory accesses MUST remain within authorized boundaries.

Invariant 4 — Image Validation

An incompatible or unauthorized image MUST NOT become active.

Invariant 5 — Explicit Ownership

Shared buffers MUST have explicit ownership semantics.

Invariant 6 — Explicit Timing

Timing guarantees MUST NOT exceed actual implementation guarantees.

Invariant 7 — Failure Transparency

Unrecovered hardware failures MUST NOT be reported as successful operations.

Invariant 8 — Reconfiguration Safety

Reconfiguration MUST preserve unaffected interface contracts.

Invariant 9 — Transport Independence

Changing the physical transport MUST NOT require changing the semantic ULABI interface.

Invariant 10 — Version Integrity

Image and interface versions MUST remain distinguishable.

Invariant 11 — No Implicit Authority

Physical hardware access MUST NOT automatically grant ULABI capabilities.

Invariant 12 — Deterministic Contract

Declared deterministic interfaces MUST have deterministic externally observable semantics.


---

84. Security Requirements

A conforming implementation MUST:

1. validate FPGA images where security policy requires validation;


2. enforce memory and DMA boundaries;


3. enforce capability restrictions;


4. protect reconfiguration authority;


5. protect privileged control interfaces;


6. define debug authorization;


7. prevent unauthorized image activation;


8. expose security failures through the ULABI error model;


9. prevent malformed descriptors from causing unauthorized memory access;


10. preserve security state across supported rollback operations.




---

85. Failure Modes

FPGA implementations MUST consider at least:

DeviceUnavailable
DeviceReset
ConfigurationFailure
ImageCorruption
ImageIncompatibility
AuthenticationFailure
DMAFailure
MemoryFailure
ClockFailure
QueueFailure
Timeout
Deadlock
ResourceExhaustion
ThermalFault
PowerFault
ProtocolViolation
CapabilityViolation
SecurityViolation
PartialReconfigurationFailure
HardwareFunctionFailure

Each supported failure MUST have defined:

detection;

reporting;

containment;

recovery;

escalation.



---

86. Recovery Contract

The recovery sequence is:

Failure Detected | Evidence Collected | Contain | Known Recovery Policy? /           
Yes            No |              | Recover       Escalate | Verify | Healthy? /    
Yes    No |      | Done   Rollback / Escalate

Recovery MUST be bounded and policy-controlled.

ULABI MUST NOT permit arbitrary autonomous FPGA modification merely because a failure occurred.


---

87. Compatibility Requirements

A conforming implementation MUST distinguish:

Binary/Image Compatibility

Whether an FPGA image can physically execute on the target device.

Interface Compatibility

Whether the image provides the required ULABI interface.

Semantic Compatibility

Whether the image preserves the required behavior.

Security Compatibility

Whether the image satisfies the applicable security policy.

Resource Compatibility

Whether sufficient FPGA resources exist.

These forms of compatibility MUST NOT be conflated.


---

88. Conformance Requirements

An implementation claiming conformance to this specification MUST demonstrate:

1. FPGA identity;


2. capability discovery;


3. interface identity;


4. function invocation;


5. memory-boundary enforcement;


6. DMA safety where DMA is supported;


7. image validation;


8. version compatibility;


9. error propagation;


10. failure detection;


11. recovery behavior;


12. reconfiguration safety where supported;


13. capability enforcement;


14. transport independence;


15. resource-limit enforcement;


16. deterministic behavior where declared;


17. timing guarantees where declared;


18. rollback where declared;


19. observability behavior;


20. security-policy enforcement.




---

89. Conformance Test Categories

The ULABI test suite SHOULD include:

FPGAIdentityTests
CapabilityDiscoveryTests
InterfaceIdentityTests
FunctionInvocationTests
DataTransferTests
DMASafetyTests
MemoryOwnershipTests
StreamTests
QueueTests
InterruptTests
TimingTests
DeterminismTests
ResourceLimitTests
ImageValidationTests
ImageCompatibilityTests
ReconfigurationTests
PartialReconfigurationTests
RollbackTests
FailureInjectionTests
FaultContainmentTests
SecurityTests
IsolationTests
VersionCompatibilityTests
TransportIndependenceTests

Tests SHOULD include both:

positive cases;

negative cases.


Negative testing is mandatory for security-sensitive boundaries.


---

90. Reference Implementation Boundary

A future reference implementation SHOULD be divided into:

ULABI FPGA Interface
        |
        +-- Identity
        +-- Capability Discovery
        +-- Interface Registry
        +-- Function Invocation
        +-- Memory Manager
        +-- DMA Manager
        +-- Stream Manager
        +-- Queue Manager
        +-- Event Manager
        +-- Image Manager
        +-- Reconfiguration Manager
        +-- Security Manager
        +-- Resource Manager
        +-- Health Monitor
        +-- Recovery Manager

The reference implementation MUST remain an implementation of this specification rather than becoming the specification itself.


---

91. Required FPGA Adapter Modules

A conforming reference implementation SHOULD provide the following logical modules:

fpga/
├── mod
├── identity
├── capabilities
├── interfaces
├── functions
├── execution
├── memory
├── dma
├── streams
├── queues
├── descriptors
├── events
├── interrupts
├── clocks
├── synchronization
├── timing
├── resources
├── images
├── validation
├── loading
├── reconfiguration
├── isolation
├── security
├── attestation
├── diagnostics
├── health
├── faults
├── recovery
├── rollback
├── compatibility
└── transport

These are logical module boundaries, not requirements for one programming language or repository layout.


---

92. Required Schemas

The schema layer SHOULD eventually define machine-readable representations for:

FPGAIdentity
FPGACapabilities
FPGAInterface
FPGAFunction
FPGAExecutionContext
FPGAMemoryRegion
FPGADescriptor
FPGAQueue
FPGAStream
FPGAEvent
FPGAImage
FPGAImageManifest
FPGAResourceRequirements
FPGAReconfigurationPolicy
FPGAFailure
FPGAAttestation

These schemas belong under:

schemas/

The schemas MUST remain language-neutral.


---

93. Required Examples

The examples area SHOULD eventually contain:

examples/fpga/
├── basic-function/
├── dma/
├── streaming/
├── shared-memory/
├── image-loading/
├── partial-reconfiguration/
├── failure-recovery/
├── rollback/
├── capability-negotiation/
└── software-fallback/

Examples MUST demonstrate contracts rather than vendor-specific programming techniques unless explicitly labeled as implementation examples.


---

94. Required Test Files

The conformance test structure SHOULD eventually contain:

tests/hardware/fpga/
├── identity
├── capabilities
├── interfaces
├── functions
├── memory
├── dma
├── streams
├── queues
├── events
├── timing
├── resources
├── images
├── loading
├── reconfiguration
├── isolation
├── security
├── faults
├── recovery
├── rollback
├── compatibility
└── transport

The test suite MUST test the semantic contract, not merely one FPGA vendor's API.


---

95. Required Conformance Modules

The conformance layer SHOULD contain:

conformance/hardware/fpga/
├── identity
├── capabilities
├── interface
├── memory
├── dma
├── streaming
├── execution
├── resources
├── image-security
├── reconfiguration
├── recovery
├── compatibility
└── security

A conforming implementation MUST be able to report its supported FPGA capabilities separately from unsupported optional capabilities.


---

96. Implementation Independence

ULABI FPGA support MAY be implemented in:

C;

C++;

Rust;

Go;

Ada;

SystemVerilog;

VHDL;

Chisel;

SpinalHDL;

other implementation technologies.


The ULABI contract MUST remain independent of the implementation language.


---

97. Vendor Independence

The specification MUST remain applicable to FPGA devices from different vendors.

Vendor-specific features MAY be exposed through capabilities or vendor profiles.

Vendor-specific behavior MUST NOT silently become part of the ULABI Core.


---

98. FPGA Profile Model

FPGA implementations SHOULD use profiles.

Possible profiles include:

ULABI-FPGA-Core
ULABI-FPGA-Memory
ULABI-FPGA-DMA
ULABI-FPGA-Streaming
ULABI-FPGA-Reconfiguration
ULABI-FPGA-Security
ULABI-FPGA-Realtime
ULABI-FPGA-Deterministic
ULABI-FPGA-Embedded
ULABI-FPGA-Accelerator

A profile MUST specify:

required capabilities;

optional capabilities;

mandatory tests;

compatibility requirements;

security requirements.



---

99. Minimal FPGA Core

The minimum FPGA profile SHOULD contain only:

Identity
Capability Discovery
Interface Identity
Function Invocation
Basic Memory Transfer
Error Reporting
Lifecycle
Compatibility

DMA, streaming, reconfiguration, attestation, and other advanced capabilities SHOULD remain optional profiles unless required by a future ULABI Core revision.


---

100. Architectural Boundary

The final architectural boundary is:

ULABI
               |
        Semantic Contract
               |
       FPGA Adapter Layer
               |
       +-------+-------+
       |       |       |
    Memory   DMA    Streams
       |       |       |
       +-------+-------+
               |
          FPGA Fabric
               |
       Physical Hardware

ULABI defines the contract.

The FPGA adapter defines the realization.

The FPGA fabric defines the implementation.

The physical device defines the hardware constraints.


---

101. Non-Goals Reaffirmed

This document does not attempt to:

standardize FPGA hardware;

standardize HDL languages;

standardize synthesis tools;

standardize place-and-route;

standardize vendor IP;

standardize board layouts;

standardize one physical transport;

replace PCIe;

replace CXL;

replace AXI;

replace existing FPGA ecosystems;

define CPU semantics;

define GPU semantics;

define distributed consensus;

define universal serialization.


ULABI integrates with those systems rather than replacing them.


---

102. Final Contract

The FPGA contract can be summarized as:

Discover
   |
Identify
   |
Negotiate
   |
Authorize
   |
Validate
   |
Load
   |
Initialize
   |
Verify
   |
Execute
   |
Observe
   |
Recover if required
   |
Quiesce
   |
Reconfigure or Release

At every stage:

Capabilities
+
Ownership
+
Security
+
Compatibility
+
Resource Limits
+
Failure Semantics

MUST remain explicit.

The FPGA layer MUST NEVER weaken the semantic guarantees of the ULABI contract.


---

103. Normative Summary

A conforming FPGA implementation:

MUST preserve ULABI semantics;

MUST expose explicit hardware identity;

MUST expose capabilities explicitly;

MUST validate FPGA images according to applicable security policy;

MUST enforce memory and DMA boundaries;

MUST define ownership for shared data;

MUST define failure behavior;

MUST define reconfiguration behavior where supported;

MUST enforce resource limits;

MUST respect ULABI versioning;

MUST support compatibility negotiation;

MUST not silently expose vendor-specific behavior as universal behavior;

MUST NOT require one FPGA vendor;

MUST NOT require one HDL;

MUST NOT require one transport;

MUST NOT redefine generic ULABI semantics;

MUST remain independently implementable.

The governing architectural principle is:

> ULABI defines the universal interoperability contract; FPGA implementations define how that contract is realized in programmable hardware.



## File-by-file implementation plan

The important point is that **`docs/hardware/fpga.md` can be completed now without waiting for the other files**. Its integration contracts are already declared above. That satisfies your requirement that we do not have to reopen it merely because another specification is subsequently expanded.

### Phase 1 — independent specification files

These should be completed independently before writing FPGA implementation code:

| Order | File | Owns |
|---|---|---|
| 1 | `docs/hardware/fpga.md` | FPGA semantic hardware contract |
| 2 | `docs/hardware/cpu.md` | CPU realization |
| 3 | `docs/hardware/gpu.md` | GPU realization |
| 4 | `docs/hardware/npu.md` | NPU realization |
| 5 | `docs/hardware/future-architectures.md` | future hardware model |
| 6 | `docs/platforms/architectures.md` | architecture taxonomy |
| 7 | `docs/platforms/embedded.md` | embedded constraints |
| 8 | `docs/abi/register-model.md` | abstract register semantics |
| 9 | `docs/abi/memory-model.md` | universal memory boundary |
| 10 | `docs/compatibility/capability-discovery.md` | capability discovery |
| 11 | `docs/compatibility/feature-negotiation.md` | negotiation |
| 12 | `docs/security/capability-security.md` | capability authorization |
| 13 | `docs/security/secure-loading.md` | secure artifact loading |
| 14 | `docs/reliability/fault-detection.md` | fault detection |
| 15 | `docs/reliability/recovery.md` | recovery |
| 16 | `docs/reliability/rollback.md` | rollback |
| 17 | `docs/standards/conformance.md` | conformance model |
| 18 | `docs/standards/test-suite.md` | test methodology |

The FPGA document already references these interfaces without duplicating their definitions.

### Phase 2 — machine-readable contracts

Create:

```text
schemas/
├── hardware/
│   ├── fpga-identity.schema.*
│   ├── fpga-capabilities.schema.*
│   ├── fpga-interface.schema.*
│   ├── fpga-function.schema.*
│   ├── fpga-execution-context.schema.*
│   ├── fpga-memory-region.schema.*
│   ├── fpga-dma.schema.*
│   ├── fpga-stream.schema.*
│   ├── fpga-queue.schema.*
│   ├── fpga-event.schema.*
│   ├── fpga-image.schema.*
│   ├── fpga-image-manifest.schema.*
│   ├── fpga-resource-requirements.schema.*
│   ├── fpga-reconfiguration.schema.*
│   ├── fpga-failure.schema.*
│   └── fpga-attestation.schema.*

The exact schema technology should be chosen by the repository's existing schema strategy rather than invented inside fpga.md.

Phase 3 — implementation modules

The logical implementation boundary is:

implementations/
└── fpga/
    ├── mod
    ├── identity
    ├── capabilities
    ├── interfaces
    ├── functions
    ├── execution
    ├── memory
    ├── dma
    ├── streams
    ├── queues
    ├── descriptors
    ├── events
    ├── interrupts
    ├── clocks
    ├── synchronization
    ├── timing
    ├── resources
    ├── images
    ├── validation
    ├── loading
    ├── reconfiguration
    ├── isolation
    ├── security
    ├── attestation
    ├── diagnostics
    ├── health
    ├── faults
    ├── recovery
    ├── rollback
    ├── compatibility
    └── transport

These are required logical modules, not necessarily 30 separate source files. The eventual implementation language can determine whether some are combined.

Phase 4 — conformance tests

tests/
└── hardware/
    └── fpga/
        ├── identity
        ├── capabilities
        ├── interfaces
        ├── functions
        ├── memory
        ├── dma
        ├── streams
        ├── queues
        ├── events
        ├── timing
        ├── resources
        ├── images
        ├── reconfiguration
        ├── isolation
        ├── security
        ├── faults
        ├── recovery
        ├── rollback
        └── compatibility

And:

conformance/
└── hardware/
    └── fpga/
        ├── identity
        ├── capabilities
        ├── interface
        ├── memory
        ├── dma
        ├── streaming
        ├── execution
        ├── resources
        ├── image-security
        ├── reconfiguration
        ├── recovery
        └── compatibility

What does not need to be edited for this file

I deliberately do not require immediate modifications to ULABI-DESIGN.md, ULABI-SPEC.md, core-abi.md, or the other documents just to make fpga.md valid.

That is important because your workflow is:

> Finish one file → it is complete → later files integrate against it rather than forcing a rewrite.



The current architecture already provides the necessary foundation: ULABI explicitly separates semantic contracts from implementation mechanisms, supports hardware/accelerator profiles, and treats hardware interoperability as an extension rather than making a particular hardware architecture part of the Core.

Also, the existing repository confirms that fpga.md was only a placeholder, so replacing that placeholder with this specification is a clean ownership change rather than a competing rewrite.

Important architectural decision

I would lock this in:

> FPGA is a ULABI hardware realization profile, not part of the ULABI Core ABI.



That prevents ULABI from becoming accidentally dependent on FPGA hardware while still allowing FPGA implementations to achieve first-class conformance.

And, critically, Zamani and Sankofa remain completely outside this specification. They may implement the FPGA ULABI contract through their own adapters, but neither language is referenced as a dependency of the standard.

[ULABI repository](https://github.com/Benwellonedge28/ULABI?utm_source=chatgpt.com)