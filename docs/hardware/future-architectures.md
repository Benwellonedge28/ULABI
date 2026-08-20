I read the repository first. I did not treat the proposed tree as if it were already implemented.

The current repository confirms that ULABI-DESIGN.md is the architectural authority and is currently a 0.2.0-draft; it establishes language/runtime/platform/architecture neutrality, Minimal Core + Profiles, explicit semantics, capability limitation, determinism, compatibility, and implementation independence.

I also checked the hardware area. future-architectures.md is currently only a placeholder, while cpu.md has already been substantially developed.   The hardware directory currently contains CPU, FPGA, GPU, NPU, quantum, and future-architecture specifications.

I also checked the architecture/compatibility material so the new document does not duplicate those responsibilities. The repository already has dedicated specifications for capability discovery, feature negotiation, graceful degradation, the Core ABI, stack/memory models, calling convention, and Distributed ABI.

Important architectural decision: future-architectures.md should not attempt to predict or standardize future CPUs, GPUs, quantum machines, etc. It should define the extension mechanism that allows genuinely new computing architectures to be introduced without changing ULABI Core.

Below is the complete document to copy into:

docs/hardware/future-architectures.md


---

ULABI Future Architectures Interface Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: docs/hardware/future-architectures.md
Specification Family: Hardware / Future Architectures
Component: Future Architecture Adaptation Framework
Status: Architecture / Design Specification
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Normative Authority: ULABI-DESIGN.md, ULABI-SPEC.md, ULABI-VERSIONING.md
License: Apache-2.0


---

1. Purpose

This document defines how ULABI accommodates computing architectures that do not yet exist, or that cannot reasonably be described by the existing CPU, GPU, NPU, FPGA, or quantum-specific profiles.

The purpose is not to predict future hardware.

The purpose is to ensure that the ULABI architecture remains extensible when fundamentally new execution technologies appear.

The governing principle is:

> A new computing architecture must be able to implement ULABI without requiring ULABI Core to be redesigned merely because the hardware is novel.



The architecture therefore provides a stable boundary:

ULABI Core
                      |
              ULABI Profiles
                      |
             Future Architecture
                  Adapter
                      |
          +-----------+-----------+
          |           |           |
      Execution    Memory     Capabilities
          |           |           |
          +-----------+-----------+
                      |
             Future Hardware

The future architecture layer MUST preserve the universal semantic contracts of ULABI while allowing new hardware to expose capabilities that cannot be expressed by existing hardware profiles.


---

2. Fundamental Principle

ULABI MUST distinguish:

Universal Semantic Contract
             |
             v
Architecture-Neutral Capability Model
             |
             v
Architecture Adapter
             |
             v
Architecture-Specific Execution Model
             |
             v
Physical Hardware

The architecture-specific implementation MUST NOT redefine the meaning of an existing ULABI Core operation.

A future architecture MAY introduce:

new execution models;

new memory models;

new parallelism models;

new synchronization mechanisms;

new addressing models;

new data representations;

new accelerators;

new execution domains;

new hardware capabilities;

new security mechanisms;

new fault models;

new resource models.


However, those capabilities MUST be introduced through explicit profiles, extensions, or new architectural specifications.


---

3. Scope

This specification defines the framework for:

1. identifying future architectures;


2. describing architecture capabilities;


3. defining architecture profiles;


4. creating architecture adapters;


5. preserving ULABI Core semantics;


6. extending ULABI without destabilizing Core;


7. capability discovery;


8. feature negotiation;


9. execution-model adaptation;


10. memory-model adaptation;


11. data-model adaptation;


12. synchronization adaptation;


13. resource-model adaptation;


14. security adaptation;


15. fault-model adaptation;


16. deterministic and nondeterministic execution;


17. architecture versioning;


18. architecture-specific extensions;


19. compatibility;


20. graceful degradation;


21. conformance testing;


22. future hardware certification.



This document does not define a particular future architecture.

It does not define:

a future CPU;

a future GPU;

a future NPU;

a future FPGA;

a future quantum processor;

a particular vendor's technology;

a particular instruction set;

a particular operating system;

a particular compiler;

a particular runtime;

a particular transport.



---

4. Relationship to Other Specifications

This specification is an extension framework, not a replacement for existing hardware specifications.

The hierarchy is:

ULABI-DESIGN.md
        |
        v
ULABI-SPEC.md
        |
        v
ULABI Core
        |
        +----------------------------+
        |                            |
 Existing Hardware Profiles     Future Architecture
        |                            |
 CPU / GPU / NPU / FPGA /      Future Architecture
 Quantum                       Adapter
                                     |
                                     v
                              Architecture Profile

Existing hardware specifications remain authoritative for hardware classes they explicitly cover.

In particular:

docs/hardware/cpu.md defines CPU-specific semantics.

docs/hardware/gpu.md defines GPU-specific semantics.

docs/hardware/npu.md defines NPU-specific semantics.

docs/hardware/fpga.md defines FPGA-specific semantics.

docs/hardware/quantum.md defines quantum-specific semantics.


future-architectures.md defines what happens when a new architecture does not fit cleanly into those categories.

It MUST NOT duplicate their detailed semantics.


---

5. Architectural Independence

A future architecture MUST NOT be required to resemble:

a CPU;

a GPU;

an NPU;

an FPGA;

a quantum processor.


For example, a future architecture could potentially use:

Event-driven execution
Dataflow execution
Spatial computation
Neuromorphic execution
Photonic computation
Memristive computation
Analog computation
Reversible computation
Molecular computation
Biological computation
Optical computation
Distributed hardware execution
Hybrid classical/quantum execution
Unknown future execution models

ULABI MUST NOT assume that one of these models will become dominant.

Instead, ULABI defines the contracts necessary to integrate such systems.


---

6. Architecture Identity

Every future architecture profile MUST have a stable identity.

Conceptually:

FutureArchitectureIdentity {
    architecture_id
    architecture_version
    profile_id
    profile_version
    vendor
    implementation
    execution_model
    memory_model
    capability_set
}

The following MUST be independently identifiable:

architecture;

architecture version;

profile;

profile version;

implementation;

optional capabilities.


Architecture identity MUST NOT be derived solely from:

vendor;

operating system;

compiler;

programming language;

executable format.



---

7. Architecture Class

A future architecture MAY declare a broad architecture class.

Possible classes include:

CPU
GPU
NPU
FPGA
Quantum
Dataflow
Neuromorphic
Photonic
Analog
Reversible
Spatial
DistributedHardware
Hybrid
Other

Other MUST remain available.

The architecture class is descriptive.

It MUST NOT determine semantic compatibility by itself.

For example:

Architecture Class:
    GPU

does not automatically establish:

ULABI GPU compatibility

The implementation MUST satisfy the applicable ULABI profile and conformance requirements.


---

8. Execution Model

Every future architecture MUST explicitly describe its execution model.

Possible execution models include:

Sequential
Parallel
SIMD
SIMT
Dataflow
EventDriven
ActorBased
Spatial
Pipeline
Speculative
Transactional
Reactive
Continuous
Quantum
Hybrid
Other

An architecture MAY support multiple execution models.

For example:

ExecutionModels {
    Dataflow
    EventDriven
    Parallel
}

The execution model MUST be exposed to capability discovery when it affects ULABI semantics.


---

9. Execution Semantics

A future architecture MUST NOT silently change the semantic meaning of a ULABI operation.

For example:

ULABI:
    operation()

MUST retain its defined semantics even if the target executes it using:

multiple hardware engines;

asynchronous execution;

speculative execution;

distributed hardware;

dataflow scheduling;

event-triggered execution.


If the target cannot preserve the required semantics, the implementation MUST:

1. provide a compatible implementation;


2. provide a defined fallback;


3. reject the operation;


4. or negotiate an alternative profile.



It MUST NOT silently produce incompatible behavior.


---

10. Architecture Adapter

Every future architecture implementation MUST provide an architecture adapter.

Conceptually:

ULABI Contract
      |
      v
Architecture Adapter
      |
      +---- Type Mapping
      |
      +---- Execution Mapping
      |
      +---- Memory Mapping
      |
      +---- Synchronization Mapping
      |
      +---- Resource Mapping
      |
      +---- Security Mapping
      |
      +---- Error Mapping
      |
      v
Future Architecture

The adapter is responsible for translating ULABI semantics into target-specific behavior.

The adapter MUST NOT redefine ULABI semantics.


---

11. Adapter Responsibilities

A future architecture adapter MUST define:

architecture identity;

supported ULABI profiles;

supported interface versions;

supported data types;

execution semantics;

memory semantics;

resource constraints;

synchronization semantics;

security capabilities;

error behavior;

unsupported feature behavior;

fallback behavior;

capability discovery;

feature negotiation;

conformance behavior.


Where applicable, it SHOULD also expose:

performance characteristics;

latency characteristics;

determinism characteristics;

parallelism characteristics;

power characteristics;

thermal constraints;

reliability characteristics.



---

12. Capability Model

Future architectures MUST use explicit capabilities.

Conceptually:

Capability {
    id
    version
    scope
    requirements
    limitations
    security_properties
}

Capabilities MUST have stable identifiers.

Examples:

ParallelExecution
EventExecution
DataflowExecution
PersistentExecution
NearMemoryCompute
OpticalCompute
AnalogCompute
HardwareIsolation
FineGrainedScheduling
TransactionalMemory
SpecializedStorage
HardwareFaultRecovery

These are examples only.

They do not become ULABI Core capabilities merely because this document names them.


---

13. Capability Discovery

Future architecture capabilities integrate with:

docs/compatibility/capability-discovery.md

The conceptual sequence is:

Architecture
     |
     v
Capability Discovery
     |
     v
Supported Features
     |
     v
ULABI Interface
     |
     +---- Required Capability
     |
     +---- Optional Capability
     |
     +---- Unsupported Capability

An implementation MUST NOT assume an optional capability exists without verification.

Capability discovery MUST distinguish:

Supported
Unsupported
ConditionallySupported
Emulated
Deprecated
Unavailable
Unknown


---

14. Feature Negotiation

Feature negotiation integrates with:

docs/compatibility/feature-negotiation.md

A future architecture MAY advertise:

supports:
    FeatureA
    FeatureB
    FeatureC

A ULABI interface MAY require:

requires:
    FeatureA

The implementation MAY proceed only when the requirement is satisfied.

If it is not satisfied, the implementation MUST follow the declared fallback or incompatibility behavior.


---

15. Unknown Architecture Features

An implementation MUST distinguish:

Unknown Optional Feature

from:

Unknown Mandatory Feature

An unknown optional feature MAY be ignored when the contract permits it.

An unknown mandatory feature MUST cause safe incompatibility handling.

It MUST NOT be silently ignored.


---

16. Extension Mechanism

Future architecture features MUST be introduced through versioned extensions.

Conceptually:

ULABI Core
    |
    +---- Profile
    |
    +---- Extension
    |
    +---- Vendor Extension
    |
    +---- Experimental Extension

Extensions MUST have:

stable identifiers;

ownership information;

version;

semantic definition;

compatibility rules;

capability requirements;

security implications;

failure behavior;

conformance requirements.


Experimental features MUST be explicitly marked as experimental.


---

17. Core Protection

A future architecture MUST NOT force new hardware concepts into ULABI Core merely because they are technically novel.

A feature SHOULD enter Core only when it is:

1. broadly applicable;


2. stable;


3. language-neutral;


4. architecture-neutral;


5. independently implementable;


6. necessary for interoperability;


7. sufficiently mature;


8. compatible with long-term ULABI evolution.



This preserves the:

> Minimal Core + Standard Profiles + Extensible Ecosystem



architecture.


---

18. New Semantic Capabilities

A future architecture MAY expose capabilities that have no direct equivalent in current ULABI profiles.

For example:

New Hardware Capability
          |
          v
Semantic Definition
          |
          v
ULABI Extension
          |
          v
Capability Identifier
          |
          v
Negotiation
          |
          v
Implementation

The extension MUST first define the semantic behavior.

Only afterward should it define the hardware realization.

This prevents ULABI from becoming coupled to one implementation.


---

19. Memory Model

A future architecture MUST explicitly declare its memory semantics.

Possible models include:

FlatMemory
HierarchicalMemory
SharedMemory
DistributedMemory
LocalMemory
PersistentMemory
TaggedMemory
CapabilityMemory
TransactionalMemory
ScratchpadMemory
StreamingMemory
NonAddressableMemory
Other

The architecture MUST distinguish:

ULABI Memory Semantics

from:

Physical Hardware Memory

A future architecture MAY have no conventional addressable memory.

ULABI MUST remain capable of representing such systems through appropriate profiles.


---

20. Non-Addressable Computation

A future architecture MAY perform computation without exposing conventional memory addresses.

For example:

Input
  |
  v
Computation Graph
  |
  v
Output

In such a system, the ULABI contract MAY operate on:

values;

descriptors;

streams;

resources;

capabilities;

handles;

computation graphs.


ULABI MUST NOT require raw pointers when they are not semantically necessary.


---

21. Data Representation

Future architectures MAY use internal representations that differ from ULABI representations.

Examples:

ULABI Float
    |
    +---- Binary Floating Point
    |
    +---- Decimal Hardware
    |
    +---- Posit-like Representation
    |
    +---- Analog Representation
    |
    +---- Other

The architecture adapter MUST preserve the semantic requirements of the ULABI type.

If exact semantic preservation is impossible, the implementation MUST explicitly declare the limitation.


---

22. Numeric Semantics

Hardware-specific numeric representations MUST NOT silently change ULABI numeric semantics.

Where a future architecture uses:

approximate arithmetic;

reduced precision;

stochastic arithmetic;

analog arithmetic;

non-IEEE representations;


the adapter MUST explicitly declare the applicable semantics.

For example:

ULABI Float64

MUST NOT silently become:

ApproximateFloat

without an explicit contract permitting that behavior.


---

23. Parallelism

A future architecture MAY provide arbitrary levels of parallelism.

The implementation MUST distinguish:

Semantic Parallelism

from:

Physical Parallelism

ULABI MAY define that an operation is parallelizable without requiring a specific number of hardware execution units.

The hardware MAY execute:

1 unit
8 units
1024 units
1,000,000 units

provided the observable contract is preserved.


---

24. Ordering

Future architectures MUST explicitly define ordering semantics where ordering affects correctness.

Possible models include:

StrictOrder
ProgramOrder
WeakOrder
RelaxedOrder
EventOrder
DependencyOrder
NoGlobalOrder

The adapter MUST NOT imply stronger ordering than the target can guarantee.

Likewise, it MUST NOT weaken a ULABI contract that requires stronger ordering.


---

25. Synchronization

A future architecture MUST describe synchronization mechanisms relevant to its ULABI profile.

Possible mechanisms include:

barriers;

events;

signals;

locks;

transactions;

dependency graphs;

channels;

message passing;

hardware queues;

interrupts;

futures;

completion tokens.


The physical mechanism is architecture-specific.

The semantic synchronization contract remains ULABI-defined.


---

26. Atomicity

If an architecture supports atomic operations, it MUST expose:

atomicity guarantees;

supported sizes;

ordering semantics;

failure behavior;

scope;

supported memory domains.


If native atomicity is unavailable, the implementation MAY provide:

software emulation;

locking;

transactional mechanisms;

serialized execution.


The fallback MUST preserve the required ULABI semantics.


---

27. Concurrency

A future architecture MAY support concurrency models that differ substantially from conventional threads.

Examples:

Threads
Actors
Tasks
Fibers
Events
Dataflow Nodes
Hardware Agents
Persistent Workers
Reactive Nodes

ULABI MUST define observable concurrency semantics independently from the physical mechanism.

The architecture adapter translates the semantic model into the target model.


---

28. Persistent Execution

A future architecture MAY support computation that persists independently of a conventional process.

For example:

Application
     |
     v
Persistent Hardware Agent
     |
     v
Events

Such functionality MUST NOT be treated as equivalent to an ordinary function call.

The implementation MUST explicitly define:

lifecycle;

ownership;

authorization;

resource limits;

shutdown;

failure;

persistence;

recovery;

security boundaries.



---

29. Resource Model

Future architectures MUST explicitly describe relevant resources.

Examples:

ComputeUnits
Memory
Bandwidth
Energy
ThermalBudget
Storage
Queues
ExecutionSlots
HardwareContexts
PersistentAgents

Resource requirements SHOULD be machine-readable.

An operation MAY declare:

requires:
    Compute >= X
    Memory >= Y

Unsupported resource requirements MUST produce defined behavior.


---

30. Resource Exhaustion

Resource exhaustion MUST NOT produce undefined behavior.

Possible errors include:

ResourceExhausted
CapacityExceeded
QueueFull
MemoryUnavailable
ComputeUnavailable
ThermalLimit
PowerLimit
ExecutionLimit

The exact error identifiers belong to the applicable ULABI error specification.


---

31. Scheduling

A future architecture MAY provide hardware-level scheduling.

ULABI MUST NOT require one scheduler.

Scheduling MAY be:

centralized;

distributed;

priority-based;

deadline-based;

event-driven;

data-dependent;

energy-aware;

thermal-aware;

hardware-controlled.


Where scheduling affects observable semantics, it MUST be explicitly specified.


---

32. Determinism

A future architecture MUST declare whether relevant operations are:

Deterministic
ConditionallyDeterministic
Nondeterministic

Nondeterminism MAY arise from:

hardware scheduling;

analog behavior;

random hardware;

concurrent execution;

external events;

distributed state.


Nondeterminism MUST NOT be hidden when it affects the ULABI contract.


---

33. Reproducibility

Where deterministic execution is required, the architecture adapter SHOULD provide sufficient information to reproduce execution.

This MAY include:

architecture version;

capability set;

execution configuration;

numeric mode;

scheduling mode;

randomness source;

firmware version;

microcode version;

relevant hardware configuration.



---

34. Security Model

Future architecture security integrates with:

docs/security/security-model.md

docs/security/capability-security.md

docs/security/sandboxing.md

docs/security/zero-trust.md


A future architecture MUST NOT automatically receive unrestricted authority because it implements ULABI.

Hardware capabilities MUST be explicitly represented.

Examples:

HardwareExecute
HardwareMemory
DeviceAccess
PersistentExecution
SecureStorage
CryptographicOperation
PrivilegedExecution

Security-sensitive capabilities SHOULD be least-privilege and revocable where technically possible.


---

35. Hardware Trust

ULABI MUST distinguish:

Architecture Identity

from:

Hardware Trust

Knowing that a component claims to be architecture X does not prove that:

its firmware is authentic;

its hardware is genuine;

its execution is trustworthy;

its results are correct;

its environment is uncompromised.


Trust requirements belong to the applicable security and attestation profiles.


---

36. Firmware and Microcode

A future architecture MAY depend on firmware or microcode.

If firmware or microcode affects ULABI semantics, the implementation MUST identify the relevant version.

Conceptually:

Architecture
    |
Firmware
    |
Microcode
    |
ULABI Adapter

Compatibility MUST account for semantic changes caused by firmware or microcode.


---

37. Hardware Configuration

Hardware configuration MAY affect behavior.

Examples include:

execution mode;

memory topology;

enabled accelerators;

precision mode;

security mode;

thermal mode;

power mode.


Configuration that changes ULABI-observable semantics MUST be exposed through the relevant capability or profile.


---

38. Fault Model

A future architecture MUST define relevant hardware failure modes.

Possible failures include:

ComputeFault
MemoryFault
InterconnectFault
DeviceFault
ThermalFault
PowerFault
FirmwareFault
ConfigurationFault
IntegrityFault
TransientFault
PermanentFault

The implementation MUST translate hardware failures into defined ULABI failures.

Hardware failure MUST NOT silently become corrupted application state.


---

39. Fault Containment

Where possible, the architecture adapter SHOULD isolate hardware faults.

Possible isolation boundaries include:

Execution Unit
Memory Region
Device
Process
Runtime
Host
Cluster

The architecture MUST document what can and cannot be isolated.


---

40. Recovery

Recovery integrates with the ULABI reliability architecture.

A future architecture MAY support:

retry;

failover;

reinitialization;

hardware reset;

migration;

replication;

software fallback;

degraded execution.


Recovery MUST be policy-controlled.

A hardware adapter MUST NOT arbitrarily modify itself or application state merely because it detects an error.


---

41. Self-Healing Boundary

Future hardware MAY expose self-repair or recovery capabilities.

These MUST integrate with:

docs/reliability/self-healing.md

The architecture MUST preserve the following conceptual boundary:

Failure
   |
Evidence
   |
Diagnosis
   |
Authorized Recovery Policy
   |
Recovery
   |
Verification
   |
+---------+
| Healthy |
+---------+
   |    |
  Yes   No
   |    |
 Done  Rollback /
       Escalate

Hardware-level autonomous recovery MUST remain bounded by the applicable ULABI security and reliability policies.


---

42. Virtualization

A future architecture MAY be:

physically present;

virtualized;

emulated;

simulated;

remote;

partitioned.


ULABI MUST distinguish:

Architecture Identity

from:

Execution Instance

A virtual architecture MAY implement the same ULABI profile as physical hardware.

However, virtualization MUST NOT falsely claim physical capabilities that are unavailable.


---

43. Emulation

An architecture MAY be emulated.

An emulator MUST preserve the declared ULABI semantics.

Performance characteristics MAY differ.

If timing or performance is part of a contract, those differences MUST be explicitly represented.


---

44. Simulation

Simulation MAY be used for:

development;

verification;

conformance;

testing;

formal analysis.


A simulation MUST NOT automatically claim hardware conformance unless it satisfies the relevant conformance requirements.


---

45. Hybrid Architectures

A system MAY combine multiple architectures.

For example:

CPU
 |
 +---- GPU
 |
 +---- NPU
 |
 +---- FPGA
 |
 +---- Future Accelerator
 |
 +---- Quantum Device

The integration model MUST treat each architecture as an independently identified execution domain.

ULABI MUST define explicit boundaries between them.


---

46. Cross-Architecture Execution

A ULABI interface MAY execute across multiple architectures.

Conceptually:

ULABI Interface
      |
      +---- CPU implementation
      |
      +---- GPU implementation
      |
      +---- Future Architecture implementation

The implementations MUST preserve the same semantic contract.

Differences in:

performance;

precision;

ordering;

latency;

availability;

resource consumption


MUST be explicitly declared when relevant.


---

47. Migration

A workload MAY migrate between architecture implementations.

Migration MUST preserve:

semantic state;

ownership;

resource authority;

security context;

version compatibility;

required capabilities.


Migration MUST NOT occur if the destination cannot satisfy the contract.


---

48. Graceful Degradation

Future architectures integrate with:

docs/compatibility/graceful-degradation.md

A feature MAY have:

Native Implementation
        |
        v
Compatible Emulation
        |
        v
Reduced Capability
        |
        v
Defined Failure

The implementation MUST NOT silently select an incompatible approximation.


---

49. Performance Is Not Semantic Compatibility

Two implementations MAY provide radically different performance.

For example:

Architecture A:
    latency = 1 ms

Architecture B:
    latency = 100 ms

They MAY still be semantically compatible.

Performance MUST become part of the contract only when an applicable profile explicitly requires it.


---

50. Real-Time Constraints

If a future architecture claims real-time support, it MUST explicitly define:

deadlines;

worst-case execution behavior;

scheduling guarantees;

resource reservations;

failure behavior.


It MUST NOT claim deterministic real-time guarantees merely because average execution is fast.

Integration point:

docs/platforms/embedded.md

and the applicable real-time profile.


---

51. Energy and Thermal Semantics

Future architectures MAY expose energy or thermal constraints.

Where those constraints affect execution guarantees, they SHOULD be discoverable.

Possible declarations:

EnergyBudget
ThermalBudget
PowerState
PerformanceState

A system MUST NOT promise execution that cannot be maintained under the declared resource constraints.


---

52. Architecture Versioning

Every architecture profile MUST be versioned.

Conceptually:

Architecture ID:
    example.architecture

Version:
    1.0

Profile:
    example.execution

Profile Version:
    1.0

Architecture versioning MUST integrate with:

ULABI-VERSIONING.md

Architecture changes MUST identify whether they are:

backward compatible;

conditionally compatible;

incompatible.



---

53. Profile Versioning

Architecture profiles MUST be independently versioned from ULABI Core.

For example:

ULABI Core:
    1.x

Future Architecture Profile:
    2.x

Updating the hardware profile MUST NOT automatically change ULABI Core.


---

54. Experimental Architectures

Experimental architectures MAY exist outside the stable ULABI profile set.

They MUST be explicitly marked:

Experimental

Experimental specifications MUST NOT be presented as stable ULABI guarantees.

Experimental interfaces SHOULD include:

experimental identifier;

version;

maturity;

known limitations;

security limitations;

compatibility limitations.



---

55. Vendor Extensions

Vendors MAY create architecture-specific extensions.

Vendor extensions MUST:

1. have stable identifiers;


2. declare their vendor namespace;


3. declare version;


4. define semantics;


5. define security requirements;


6. define unsupported behavior;


7. define conformance requirements.



Vendor extensions MUST NOT redefine ULABI Core semantics.


---

56. Open Architecture Extensions

ULABI SHOULD allow independent organizations to define architecture profiles.

A profile SHOULD be implementable from the published specification without requiring proprietary information.

This preserves ULABI's vendor-neutral architecture.


---

57. Compiler Integration

Future architectures integrate with:

docs/tooling/compiler-interface.md

A compiler targeting a future architecture MUST translate:

Source Semantics
      |
      v
ULABI Contract
      |
      v
Architecture Lowering
      |
      v
Target Representation

The compiler MUST preserve ULABI semantics during lowering.

The compiler MUST NOT require ULABI to understand source-language syntax.


---

58. Linker Integration

Future architecture linking integrates with:

docs/tooling/linker-interface.md

The linker MAY need to resolve:

architecture-specific symbols;

capability requirements;

architecture profile versions;

interface identities;

relocation requirements;

hardware resources.


Unsupported requirements MUST result in defined diagnostics.


---

59. Loader Integration

Future architectures integrate with:

docs/tooling/loader-interface.md

A loader MAY verify:

architecture identity;

profile compatibility;

capability requirements;

security requirements;

firmware compatibility;

interface versions.


Loading MUST fail safely when mandatory requirements cannot be satisfied.


---

60. Debugging

Future architecture debugging integrates with:

docs/tooling/debugger-interface.md

A debugger SHOULD be able to identify:

architecture;

execution domain;

profile;

instruction or execution representation;

ULABI interface identity;

source mapping where available.


Architecture-specific debugging information MUST NOT alter ULABI semantics.


---

61. Observability

Future architectures SHOULD expose relevant observability information through:

docs/observability/tracing.md

docs/observability/diagnostics.md

docs/observability/telemetry.md

Observable information MAY include:

architecture identity;

execution domain;

capability use;

resource consumption;

faults;

retries;

recovery;

fallback;

performance.


Observability MUST NOT leak protected information contrary to the applicable security policy.


---

62. Distributed Hardware

A future architecture MAY span multiple physical devices.

For example:

ULABI Interface
      |
      v
Hardware Fabric
 +----+----+----+
 |         |   |
Node A   Node B Node C

Such execution MUST NOT automatically be treated as local execution.

If execution crosses a distributed boundary, the applicable distributed specifications MUST apply.


---

63. Remote Execution

Remote future architectures integrate with:

docs/distributed/remote-calls.md

The architecture adapter MUST distinguish:

Local Hardware

from:

Remote Hardware

Latency, failure, authorization, locality, and resource semantics MUST remain explicit.


---

64. Serialization

If values cross a hardware or distributed boundary, serialization integrates with:

docs/distributed/serialization.md

The architecture's internal representation MUST NOT become the universal wire representation merely because it is efficient on one implementation.

ULABI canonical encoding remains the interoperability boundary.


---

65. Service Discovery

Future hardware exposed as an independent execution service integrates with:

docs/distributed/service-discovery.md

The discovered service MUST advertise:

architecture identity;

profile;

version;

capabilities;

security requirements;

resource availability;

supported interfaces.



---

66. Security Isolation

Future architecture execution MAY occur inside:

a process;

a VM;

a container;

a secure enclave;

a hardware partition;

a remote service;

a dedicated device.


The isolation mechanism is implementation-specific.

The security boundary MUST be explicit.


---

67. Capability Revocation

If a future architecture supports revocable capabilities, revocation MUST be observable according to the applicable security profile.

After revocation:

Capability
    |
    v
Revoked
    |
    v
Operation denied

The implementation MUST NOT continue unauthorized operations merely because a previous capability token remains cached internally.


---

68. Compatibility Matrix

Every future architecture profile SHOULD publish a compatibility matrix.

Example:

Feature	Required	Supported	Fallback

ULABI Core	Yes	Yes	None
Profile X	No	Yes	N/A
Feature Y	Yes	No	Reject
Feature Z	No	No	Software fallback


This matrix SHOULD be machine-readable in addition to documentation.


---

69. Conformance Requirements

A future architecture implementation claiming ULABI conformance MUST demonstrate:

Identity

stable architecture identity;

stable profile identity;

version reporting.


Semantics

correct ULABI type semantics;

correct execution semantics;

correct error behavior.


Compatibility

capability discovery;

feature negotiation;

version handling;

graceful degradation where applicable.


Security

capability enforcement;

authorization;

isolation;

secure loading where applicable.


Reliability

defined hardware failures;

defined resource exhaustion;

defined recovery behavior.


Tooling

compiler integration where supported;

linker integration where supported;

loader integration where supported;

diagnostics.



---

70. Conformance Testing

Future architecture conformance tests MUST include:

1. architecture identity tests;


2. version tests;


3. capability discovery tests;


4. feature negotiation tests;


5. required-feature rejection tests;


6. type mapping tests;


7. execution semantic tests;


8. memory semantic tests;


9. synchronization tests;


10. resource-limit tests;


11. failure tests;


12. security tests;


13. compatibility tests;


14. fallback tests;


15. deterministic behavior tests where applicable;


16. nondeterminism declaration tests where applicable.



These tests integrate with:

docs/standards/conformance.md

and

docs/standards/test-suite.md.


---

71. Reference Adapter

ULABI SHOULD eventually provide a reference future-architecture adapter model.

The reference implementation MUST be:

architecture-neutral where possible;

language-independent at the contract level;

independently implementable;

unsuitable as a hidden normative dependency.


The reference implementation demonstrates the contract.

It does not define the contract.


---

72. Required Machine-Readable Schemas

Future architecture profiles SHOULD eventually have schemas under:

schemas/

Recommended schemas:

schemas/architecture-identity.schema.json
schemas/architecture-profile.schema.json
schemas/architecture-capabilities.schema.json
schemas/architecture-requirements.schema.json
schemas/architecture-compatibility.schema.json
schemas/architecture-conformance.schema.json

The schemas MUST remain subordinate to the normative textual specification.

A schema MUST NOT silently introduce semantics absent from the specification.


---

73. Required Examples

Examples SHOULD be placed under:

examples/hardware/

Recommended examples:

examples/hardware/future-architecture-minimal/
examples/hardware/future-architecture-dataflow/
examples/hardware/future-architecture-event-driven/
examples/hardware/future-architecture-hybrid/
examples/hardware/future-architecture-heterogeneous/

Examples MUST clearly identify whether they are:

Normative
Informative
Experimental


---

74. Required Test Areas

The conformance test structure SHOULD eventually include:

tests/hardware/
├── architecture_identity/
├── capability_discovery/
├── feature_negotiation/
├── execution/
├── memory/
├── synchronization/
├── resources/
├── security/
├── failures/
├── compatibility/
└── fallback/

These tests MUST be reusable across independent future architecture implementations.


---

75. Required Implementation Modules

ULABI itself is a specification, so these modules are implementation targets, not language-specific mandatory source files.

A reference implementation SHOULD eventually contain:

implementations/
└── reference/
    └── hardware/
        └── future/
            ├── architecture_identity
            ├── architecture_profile
            ├── capability_registry
            ├── capability_discovery
            ├── feature_negotiation
            ├── execution_adapter
            ├── type_adapter
            ├── memory_adapter
            ├── synchronization_adapter
            ├── resource_manager
            ├── security_adapter
            ├── error_adapter
            ├── fallback_manager
            ├── compatibility_manager
            ├── conformance_adapter
            └── diagnostics

No programming language is mandated for these modules.


---

76. Module Responsibilities

architecture_identity

Responsible for:

architecture ID;

version;

profile ID;

profile version;

implementation identity.


architecture_profile

Responsible for:

execution model;

memory model;

supported ULABI profiles;

architecture requirements.


capability_registry

Responsible for:

capability identifiers;

versions;

capability metadata;

capability lifecycle.


capability_discovery

Responsible for:

querying hardware capabilities;

reporting support state;

exposing resource limits.


feature_negotiation

Responsible for:

required features;

optional features;

compatibility decisions;

fallback selection.


execution_adapter

Responsible for:

mapping ULABI execution semantics;

scheduling;

execution lifecycle.


type_adapter

Responsible for:

ULABI type mapping;

representation conversion;

semantic preservation.


memory_adapter

Responsible for:

memory semantics;

ownership;

lifetime;

addressability;

resource boundaries.


synchronization_adapter

Responsible for:

events;

barriers;

atomicity;

ordering;

synchronization primitives.


resource_manager

Responsible for:

compute resources;

memory;

bandwidth;

energy;

thermal limits;

quotas.


security_adapter

Responsible for:

capability enforcement;

isolation;

authorization;

secure hardware access.


error_adapter

Responsible for mapping architecture failures to ULABI errors.

fallback_manager

Responsible for:

software fallback;

emulation;

reduced-capability execution;

safe rejection.


compatibility_manager

Responsible for:

version compatibility;

profile compatibility;

architecture compatibility.


conformance_adapter

Responsible for exposing the implementation to ULABI conformance tests.

diagnostics

Responsible for:

architecture diagnostics;

capability diagnostics;

resource failures;

hardware errors;

compatibility failures.



---

77. Required Integration Files

The future architecture specification intentionally integrates with the following existing/future ULABI documents.

Core authority

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md

Hardware

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md

ABI

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

Compatibility

docs/compatibility/capability-discovery.md
docs/compatibility/feature-negotiation.md
docs/compatibility/graceful-degradation.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

Security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/zero-trust.md
docs/security/secure-loading.md

Reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md

Distributed

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/validator.md

Observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md

Standards

docs/standards/conformance.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md
docs/standards/certification.md


---

78. What Belongs in This File vs Other Files

To prevent repetition, the boundary is:

Concern	Owner

Universal architecture	ULABI-DESIGN.md
Normative Core rules	ULABI-SPEC.md
ABI semantics	docs/abi/
CPU semantics	docs/hardware/cpu.md
GPU semantics	docs/hardware/gpu.md
NPU semantics	docs/hardware/npu.md
FPGA semantics	docs/hardware/fpga.md
Quantum semantics	docs/hardware/quantum.md
Unknown/new architecture adaptation	future-architectures.md
Capability discovery	docs/compatibility/capability-discovery.md
Feature negotiation	docs/compatibility/feature-negotiation.md
Serialization	docs/distributed/serialization.md
Remote execution	docs/distributed/remote-calls.md
Hardware failures	docs/reliability/
Security	docs/security/
Compiler lowering	docs/tooling/compiler-interface.md
Conformance	docs/standards/


This is important: future-architectures.md should be the extensibility contract, not another giant hardware encyclopedia.


---

79. Architectural Invariants

The following invariants are mandatory:

Invariant 1 — Core independence

A future architecture MUST NOT require modification of ULABI Core merely because it is novel.

Invariant 2 — Semantic preservation

Architecture lowering MUST preserve ULABI semantics.

Invariant 3 — Explicit capabilities

Hardware capabilities MUST be discoverable.

Invariant 4 — Explicit incompatibility

Unsupported mandatory functionality MUST produce defined incompatibility behavior.

Invariant 5 — No hidden fallback

An approximation MUST NOT silently replace an exact semantic contract.

Invariant 6 — No vendor ownership

A vendor MUST NOT control the universal semantics.

Invariant 7 — Version stability

Architecture evolution MUST be explicitly versioned.

Invariant 8 — Security isolation

Hardware capability MUST NOT imply unrestricted authority.

Invariant 9 — Failure containment

Hardware failure MUST NOT become undefined ULABI behavior.

Invariant 10 — Implementation independence

Independent organizations MUST be capable of implementing the profile from the published specification.


---

80. Security Requirements

A conforming future architecture implementation:

MUST:

authenticate security-sensitive hardware where required;

enforce declared capabilities;

respect authorization;

reject unauthorized operations;

isolate execution according to the applicable profile;

expose security-relevant limitations;

prevent unsupported privileged operations.


MUST NOT:

grant implicit unrestricted hardware access;

silently bypass capability restrictions;

silently downgrade security;

claim unavailable security properties;

treat hardware identity as proof of trust.



---

81. Failure Requirements

A conforming implementation MUST define behavior for relevant:

unsupported features;

resource exhaustion;

invalid configuration;

hardware faults;

capability failures;

version mismatches;

security failures;

firmware incompatibility;

execution failures.


Undefined behavior MUST NOT be used as the compatibility mechanism.


---

82. Recovery Requirements

Recovery MAY be provided.

When provided, recovery MUST:

1. have an explicit policy;


2. have bounded scope;


3. preserve security constraints;


4. preserve ownership;


5. preserve lifecycle semantics;


6. verify successful recovery;


7. rollback when required;


8. escalate when recovery is unsafe or unknown.




---

83. Compatibility Requirements

A future architecture profile MUST define:

supported ULABI Core versions;

supported profiles;

supported extensions;

incompatible versions;

fallback behavior;

required capabilities;

optional capabilities.


A future architecture MUST NOT claim general ULABI compatibility merely because it can execute one ULABI example.


---

84. Conformance Levels

Future architectures SHOULD eventually support conformance levels such as:

Level 0 — Experimental
Level 1 — Basic ULABI Adapter
Level 2 — Profile Conformance
Level 3 — Full Architecture Conformance
Level 4 — Certified Architecture

Exact compliance levels belong to:

docs/standards/compliance-levels.md

and MUST NOT be independently redefined here.


---

85. Certification

Certification MAY verify:

architecture identity;

profile compliance;

semantic correctness;

capability correctness;

security;

reliability;

compatibility;

conformance tests.


Certification MUST be based on published criteria.

It MUST NOT require ownership of a particular implementation.


---

86. Future-Proofing Rule

The most important rule of this specification is:

> ULABI MUST evolve by adding explicit capabilities and profiles before changing Core semantics.



The preferred evolution path is:

New Hardware
     |
     v
Architecture Analysis
     |
     v
Existing Profile?
   /       \
 Yes       No
 |          |
Use        New Profile /
Existing   Extension
Profile       |
              v
       Capability Definition
              |
              v
       Compatibility Rules
              |
              v
       Conformance Tests
              |
              v
       Independent Implementations

Only when a concept proves to be universally necessary should it be considered for Core.


---

87. Reference Architecture Contract

A future architecture profile is complete only when the following are defined:

Architecture Identity
        +
Execution Model
        +
Memory Model
        +
Type Mapping
        +
Capability Model
        +
Resource Model
        +
Synchronization Model
        +
Security Model
        +
Failure Model
        +
Compatibility Model
        +
Fallback Model
        +
Conformance Tests

If any mandatory element is missing, the architecture profile is incomplete.


---

88. Definition of Done

docs/hardware/future-architectures.md is considered complete when:

the future-architecture boundary is defined;

existing hardware profiles remain authoritative for their domains;

new architectures can be introduced without modifying Core unnecessarily;

architecture identity is defined;

profile identity is defined;

capability discovery is defined;

feature negotiation is defined;

execution-model adaptation is defined;

memory-model adaptation is defined;

resource handling is defined;

security boundaries are defined;

failure handling is defined;

recovery boundaries are defined;

compatibility behavior is defined;

conformance requirements are defined;

machine-readable schemas are identified;

implementation modules are identified;

integration points are identified.


No subsequent hardware specification should need to rewrite the fundamental future-architecture contract merely because a new architecture is added.


---

89. Final Architectural Rule

> ULABI must be capable of accommodating computing architectures that its designers cannot yet predict.



Therefore:

ULABI
 |
 +-- Current Architectures
 |
 +-- Current Profiles
 |
 +-- Extensions
 |
 +-- Future Architectures
 |
 +-- Unknown Future Computing Models

The existence of a new hardware paradigm MUST NOT invalidate the fundamental ULABI architecture.

The universal contract remains stable.

The implementation adapts.


---

File/integration plan

For this document, the only file that needs to be completed now is:

docs/hardware/future-architectures.md

It can be completed independently because all of its integration points are explicitly defined above.

The next implementation work should then proceed in dependency order rather than editing this document repeatedly.

Phase 1 — independent normative foundations

These should establish contracts before implementation modules:

ULABI-SPEC.md
ULABI-VERSIONING.md

docs/abi/core-abi.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/calling-convention.md

docs/compatibility/capability-discovery.md
docs/compatibility/feature-negotiation.md
docs/compatibility/graceful-degradation.md

Phase 2 — hardware contracts

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md
docs/hardware/future-architectures.md

future-architectures.md is deliberately not dependent on a particular implementation of those files. It defines the extension boundary in advance.

Phase 3 — distributed contracts

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Phase 4 — implementation contracts

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/validator.md

Phase 5 — conformance

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md
docs/standards/reference-implementations.md


---

Required code/module inventory

When ULABI moves from specification into implementation, the future-hardware subsystem should have these logical modules:

architecture_identity
architecture_profile
capability_registry
capability_discovery
feature_negotiation
execution_adapter
type_adapter
memory_adapter
synchronization_adapter
resource_manager
security_adapter
error_adapter
fallback_manager
compatibility_manager
conformance_adapter
diagnostics

And the repository-level implementation structure should eventually be approximately:

implementations/
└── reference/
    ├── core/
    ├── types/
    ├── abi/
    ├── compatibility/
    ├── security/
    ├── distributed/
    ├── runtime/
    └── hardware/
        ├── cpu/
        ├── gpu/
        ├── npu/
        ├── fpga/
        ├── quantum/
        └── future/
            ├── architecture_identity
            ├── architecture_profile
            ├── capability_registry
            ├── capability_discovery
            ├── feature_negotiation
            ├── execution_adapter
            ├── type_adapter
            ├── memory_adapter
            ├── synchronization_adapter
            ├── resource_manager
            ├── security_adapter
            ├── error_adapter
            ├── fallback_manager
            ├── compatibility_manager
            ├── conformance_adapter
            └── diagnostics

These are logical module names, not a demand that ULABI be implemented in Rust, C++, Python, or any other language.

Key result

The important thing we have now established is that future hardware does not become a reason to keep rewriting ULABI-DESIGN.md. The master architecture defines the principle; future-architectures.md defines the extension contract; architecture-specific profiles define concrete behavior; implementations consume those contracts.

That keeps ULABI independent of Zamani, Sankofa, any existing CPU, and even hardware paradigms that do not exist yet.