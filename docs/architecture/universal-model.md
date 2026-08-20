I read the current ULABI-DESIGN.md first, then checked the existing architecture documents and searched the repository for overlapping material. The master document establishes the Minimal Core + Standard Profiles + Extensible Ecosystem model, language/runtime/platform neutrality, semantic type separation, explicit contracts, capability limitation, deterministic behavior, and implementation independence.

I also checked ULABI-SPEC.md: it already defines the normative foundations for interface identity, versions, profiles, primitive/composite types, calling conventions, return values, errors, memory, capabilities, lifecycle, and distributed operation.

The current docs/architecture/universal-model.md is only a placeholder, so it is appropriate to replace it rather than build around existing duplicate content.

Below is the complete replacement.

1. docs/architecture/universal-model.md

Copy everything inside this block into:

docs/architecture/universal-model.md

# ULABI Universal Model

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/architecture/universal-model.md`  
**Specification Family:** ULABI Architecture  
**Status:** Normative Architectural Specification  
**Current ULABI Design Version:** 0.2.0-draft  
**License:** Apache-2.0

---

## 1. Purpose

This document defines the universal computational and interoperability model
used by ULABI.

It specifies how independently implemented languages, compilers, runtimes,
libraries, applications, operating systems, devices, accelerators, and
distributed components are represented at a ULABI boundary.

The purpose of this document is not to define the implementation internals of
those systems.

Instead, it defines the common semantic model that allows those systems to
interoperate.

ULABI therefore separates:

1. semantic meaning;
2. ULABI boundary contracts;
3. implementation representation.

The fundamental model is:

```text
Source / Implementation System
            |
            v
     ULABI Adapter
            |
            v
   ULABI Semantic Contract
            |
            v
     ULABI Adapter
            |
            v
Target / Implementation System

The systems on either side remain independently implemented.


---

2. Relationship to Other ULABI Documents

This document is part of the ULABI architecture layer.

The authority hierarchy is:

ULABI-GOVERNANCE.md
        |
        v
ULABI-VERSIONING.md
        |
        v
ULABI-DESIGN.md
        |
        v
docs/architecture/design-principles.md
        |
        v
docs/architecture/universal-model.md
        |
        +-------------------------------+
        |                               |
        v                               v
   ULABI Core                     ULABI Profiles
        |                               |
        +---------------+---------------+
                        |
                        v
                 Implementations

ULABI-DESIGN.md defines the overall architecture.

docs/architecture/design-principles.md defines the architectural constraints.

This document defines the universal model used by those constraints.

ULABI-SPEC.md defines normative Core requirements.

Component-specific specifications define detailed contracts for individual parts of the model.

This document MUST NOT duplicate detailed rules that belong to specialized specifications.

Instead, it establishes the relationships between them.


---

3. Normative Language

The following terms are normative:

MUST — absolute requirement.

MUST NOT — absolute prohibition.

REQUIRED — equivalent to MUST.

SHALL — equivalent to MUST.

SHALL NOT — equivalent to MUST NOT.

SHOULD — strong recommendation.

SHOULD NOT — strong recommendation against.

MAY — permitted but optional.

OPTIONAL — not required unless a profile requires it.


An implementation claiming conformance MUST satisfy all applicable mandatory requirements.


---

4. Fundamental Universal Model

ULABI defines a common boundary model rather than a common implementation model.

The universal model is:

+------------------------------------------------------+
|                 Implementation A                     |
|                                                      |
| Language / Runtime / Memory / Platform / Hardware    |
+--------------------------+---------------------------+
                           |
                           v
                    ULABI Adapter A
                           |
                           v
+------------------------------------------------------+
|                  ULABI CONTRACT                      |
|                                                      |
| Identity                                               |
| Version                                                |
| Types                                                  |
| Functions                                              |
| Ownership                                              |
| Lifetime                                               |
| Errors                                                 |
| Capabilities                                           |
| Effects                                                |
| Execution semantics                                    |
| Compatibility                                          |
+------------------------------------------------------+
                           |
                           v
                    ULABI Adapter B
                           |
                           v
+------------------------------------------------------+
|                 Implementation B                     |
|                                                      |
| Language / Runtime / Memory / Platform / Hardware    |
+------------------------------------------------------+

The contract is universal.

The implementations are not.


---

5. Independence Principle

ULABI MUST NOT require two implementations to share their internal design.

Two systems MAY use completely different:

programming languages;

type systems;

compilers;

runtimes;

garbage collectors;

ownership models;

memory allocators;

operating systems;

CPU architectures;

execution environments;

object models;

concurrency models;

storage systems;

deployment models.


They only need to agree on the applicable ULABI contract.

For example:

Rust
  |
  +---- ULABI ----+
                  |
               Python

is valid.

So is:

C
 |
 +---- ULABI ----+
                  |
               Java

and:

Sankofa
   |
   +---- ULABI ----+
                   |
                 Go

and:

Zamani
   |
   +---- ULABI ----+
                   |
                 C++

ULABI MUST NOT contain language-specific assumptions that prevent these relationships.


---

6. Project Independence

ULABI is independent of individual projects.

In particular:

Zamani  != ULABI
Sankofa != ULABI

Zamani MAY implement ULABI.

Sankofa MAY implement ULABI.

Neither project defines ULABI.

The same principle applies to every other implementation.

ULABI MUST remain implementable by independent organizations without requiring access to the source code of another ULABI implementation.


---

7. Universal Contract Boundary

The ULABI boundary consists of observable semantics.

The boundary MAY describe:

interface identity;

function identity;

type identity;

data representation;

ownership;

lifetime;

errors;

capabilities;

effects;

execution mode;

locality;

resource requirements;

compatibility;

versioning;

security requirements.


The boundary MUST NOT unnecessarily expose implementation details.

For example, a ULABI interface SHOULD NOT require knowledge of whether an implementation internally uses:

malloc
arena allocation
garbage collection
reference counting
ownership analysis
stack allocation
pool allocation

unless that distinction is observable and required by the contract.


---

8. Three-Level Semantic Model

ULABI defines three distinct levels.

+-------------------------------+
|      Semantic Meaning         |
+-------------------------------+
               |
               v
+-------------------------------+
|    ULABI Boundary Model       |
+-------------------------------+
               |
               v
+-------------------------------+
| Implementation Representation |
+-------------------------------+

8.1 Semantic Meaning

Semantic meaning describes what an operation or value means.

Examples:

User identifier
Temperature
File contents
Timestamp
Payment amount
Image
Error

Semantic meaning is independent of a particular programming language.


---

8.2 ULABI Boundary Model

The boundary model specifies how the semantic object participates in interoperability.

It may specify:

Type
Width
Encoding
Ownership
Lifetime
Mutability
Validity
Capabilities
Errors


---

8.3 Implementation Representation

The implementation representation is private.

For example:

Language A:
struct

Language B:
class

Language C:
record

Language D:
object

Language E:
tuple

These MAY all represent the same ULABI semantic contract.


---

9. Universal Entity Model

Every externally interoperable ULABI entity SHOULD be identifiable.

The conceptual model is:

Entity
 |
 +-- Identity
 |
 +-- Version
 |
 +-- Type
 |
 +-- Contract
 |
 +-- Capabilities
 |
 +-- Lifetime
 |
 +-- State

Entities may include:

interfaces;

functions;

types;

resources;

handles;

components;

streams;

services;

devices;

accelerator contexts.


Detailed identity encoding belongs to the Core ABI and identity specifications.


---

10. Interface Model

An interface is a stable contract between one or more participants.

Conceptually:

Interface
 |
 +-- Interface Identity
 |
 +-- Version
 |
 +-- Functions
 |
 +-- Types
 |
 +-- Errors
 |
 +-- Capabilities
 |
 +-- Effects
 |
 +-- Compatibility Rules

An interface MUST be independently identifiable.

The interface identity MUST NOT depend solely on a source-language symbol name.

Detailed interface identity requirements belong to:

docs/abi/core-abi.md

docs/interoperability/symbol-resolution.md

docs/compatibility/feature-negotiation.md



---

11. Function Model

A ULABI function is an operation defined by an explicit contract.

Conceptually:

Function
 |
 +-- Identity
 +-- Version
 +-- Parameters
 +-- Return Values
 +-- Errors
 +-- Ownership
 +-- Lifetime
 +-- Effects
 +-- Capabilities
 +-- Execution Semantics

A function MUST have observable semantics sufficient for an independent implementation to invoke it safely.

The universal model does not require all languages to use functions internally.

A language MAY map:

function
method
procedure
subroutine
closure
actor operation
message handler
foreign call

to a ULABI callable operation.

The detailed binary representation is defined by:

docs/abi/calling-convention.md


---

12. Value Model

ULABI values are semantic values that can cross an interoperability boundary.

The value model includes:

Primitive
Composite
Reference
Handle
Resource
Stream
Result
Option

The universal model does not require source-language values to have identical internal representations.

The boundary representation MUST be unambiguous.

Detailed type representation belongs to:

docs/abi/data-types.md

docs/type-system/universal-types.md

docs/type-system/type-descriptors.md



---

13. Type Mapping Model

Source types map explicitly to ULABI types.

Source Type
     |
     v
Mapping Rule
     |
     v
ULABI Type
     |
     v
Boundary Representation

For example:

Source Integer
      |
      v
Explicit Mapping
      |
      v
ULABI I64

ULABI MUST NOT assume that a source-language type with a similar name has the same binary semantics.

Type compatibility MUST be determined from the ULABI contract.


---

14. Composite Value Model

Composite values consist of multiple semantic components.

Examples include:

Record
Struct
Tuple
Array
List
Map
Enum
Variant
Option
Result

A composite value MUST have a deterministic interpretation under its applicable ULABI profile.

Detailed layout rules belong to:

docs/abi/data-types.md

Detailed semantic type rules belong to:

docs/type-system/


---

15. Resource Model

A resource is an externally controlled entity whose lifetime or authority extends beyond a simple value.

Examples:

File
Socket
Device
GPU Context
Process
Database Connection
Memory Region
Cryptographic Key
Capability

Resources SHOULD be represented through controlled resource identities or handles.

A resource MUST have explicit lifecycle semantics.

Conceptually:

Discovered
    |
Verified
    |
Acquired
    |
Active
    |
Released

Invalid resource use MUST result in defined failure behavior.

Detailed resource semantics belong to:

docs/runtime/resource-management.md


---

16. Handle Model

A handle represents controlled access to an external resource or object.

A handle MUST NOT automatically imply unrestricted authority.

Conceptually:

Handle
 |
 +-- Identity
 +-- Resource Type
 +-- Authority
 +-- Lifetime
 +-- Ownership
 +-- State

A handle MAY be opaque.

An implementation MUST NOT rely on the internal binary representation of an opaque handle unless the applicable profile explicitly defines it.


---

17. Capability Model

Capabilities represent authority.

The conceptual model is:

Component
    |
    v
Capability
    |
    v
Authorized Operation

Examples include:

FileRead
FileWrite
Network
GPU
Device
Process
Storage
Secret
CryptographicKey

A component MUST NOT receive capabilities merely because it participates in ULABI.

Capabilities MUST be explicitly granted according to the applicable security profile.

Detailed capability semantics belong to:

docs/security/capability-security.md

docs/security/authorization.md

docs/security/security-model.md



---

18. Ownership Model

Ownership determines which participant has authority over a resource or memory region.

Conceptually:

Owned
 |
 +-- Borrowed
 |
 +-- Shared
 |
 +-- Transferred
 |
 +-- Released

Ownership MUST be explicit where it affects correctness or safety.

ULABI does not require one universal ownership system internally.

Instead, implementations map their own memory/resource models to the ULABI boundary model.

Detailed ownership rules belong to:

docs/memory/ownership.md

docs/memory/lifetimes.md

docs/abi/memory-model.md



---

19. Lifetime Model

Every reference, resource, stream, callback, or borrowed value that crosses a ULABI boundary MUST have defined lifetime semantics.

Conceptually:

Created
   |
Active
   |
Expired / Released

A participant MUST NOT use an object after its permitted lifetime.

Lifetime semantics MUST be explicit when the boundary involves:

borrowed memory;

callbacks;

asynchronous operations;

streams;

resources;

handles;

shared memory.


Detailed lifetime rules belong to:

docs/memory/lifetimes.md


---

20. Execution Model

ULABI supports multiple execution models.

An operation MAY be:

Synchronous
Asynchronous
Blocking
NonBlocking
Streaming
OneShot
LongRunning
Cancellable

The execution model MUST be declared when it affects caller behavior.

A local synchronous function MUST NOT silently become a remote asynchronous operation.

Execution semantics MUST remain explicit.

Detailed runtime behavior belongs to:

docs/runtime/runtime-interface.md

docs/runtime/async-model.md

docs/runtime/concurrency.md



---

21. Locality Model

ULABI distinguishes execution locality.

The conceptual locality levels are:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A locality declaration describes where an operation MAY execute.

A local operation MUST NOT silently become a network operation.

Remote execution MUST expose the additional semantics that affect correctness, including where applicable:

latency;

failure;

availability;

serialization;

authentication;

authorization;

consistency.


Detailed distributed behavior belongs to:

docs/distributed/


---

22. Transport Model

ULABI is transport-independent.

Possible transports include:

Direct Call
Shared Memory
Pipe
OS IPC
Unix Socket
Message Queue
TCP
QUIC
WebAssembly Host Call
Device Bus
Future Transport

The ULABI interface contract MUST remain independent from a specific transport unless a transport-specific profile is explicitly selected.

Conceptually:

ULABI Contract
                    |
       +------------+------------+
       |            |            |
    Direct        IPC        Network
       |            |            |
       +------------+------------+
                    |
              Same Contract

Transport-specific requirements belong to:

docs/distributed/remote-calls.md

and relevant transport/platform profiles.


---

23. Serialization Model

When a ULABI operation crosses a boundary requiring serialization, the serialized representation MUST preserve the applicable semantic contract.

Serialization MUST NOT silently alter:

type;

ownership;

lifetime;

value meaning;

error semantics;

security properties.


Canonical serialization rules belong to:

docs/distributed/serialization.md

ULABI MUST remain capable of zero-copy operation where the applicable profile allows it.


---

24. Memory Model

ULABI supports multiple memory-interoperability strategies.

Conceptually:

Copy
 |
Shared Memory
 |
Borrowed Memory
 |
Transferred Ownership
 |
Zero-Copy

The universal model does not mandate one strategy.

The chosen strategy MUST preserve the contract.

Detailed rules belong to:

docs/abi/memory-model.md

docs/memory/memory-safety.md

docs/memory/ownership.md

docs/memory/shared-memory.md

docs/memory/allocation.md



---

25. Error Model

Errors are first-class results of operations.

Conceptually:

Operation
    |
    +---- Success
    |
    +---- Failure

A failure MUST have machine-readable semantics.

ULABI does not mandate language-level exceptions.

An implementation MAY map a ULABI failure to:

Exception
Result
Status Code
Error Object
Error Value

The boundary semantics remain the same.

Detailed error behavior belongs to:

docs/abi/exception-model.md

docs/distributed/distributed-errors.md

docs/runtime/runtime-interface.md



---

26. Result Model

Operations that may succeed or fail SHOULD use an explicit result model.

Conceptually:

Result<T, E>
 |
 +-- Success(T)
 |
 +-- Failure(E)

The exact binary representation belongs to the applicable ABI and type specifications.

The universal model only requires that success and failure be unambiguous.


---

27. Option Model

Optional values use the semantic model:

Option<T>
 |
 +-- None
 |
 +-- Some(T)

An absent value MUST be distinguishable from a valid value.

ULABI MUST NOT rely on language-specific sentinel conventions unless explicitly defined by a profile.


---

28. Stream Model

A stream represents a potentially continuing sequence of values or events.

Conceptually:

Stream<T>
 |
 +-- Element
 +-- Element
 +-- Element
 +-- ...
 +-- Completion

A stream contract MUST define, where applicable:

element type;

direction;

completion;

failure;

cancellation;

ownership;

lifetime;

backpressure.


Detailed stream behavior belongs to the runtime and distributed profiles.


---

29. Callback Model

A callback is an invocation performed by one participant into another participant according to an established contract.

Callbacks MUST define:

callable identity;

argument contract;

return contract;

lifetime;

ownership;

reentrancy;

thread/concurrency semantics;

cancellation where applicable.


A callback MUST NOT remain valid beyond its declared lifetime.


---

30. State Model

A ULABI component MAY expose explicit state.

Conceptually:

Created
   |
Initialized
   |
Ready
   |
Running
   |
Paused / Degraded
   |
Stopped
   |
Unloaded

A component MUST reject invalid state transitions where those transitions could compromise safety or interoperability.

Detailed lifecycle rules belong to:

docs/runtime/process-model.md

and the relevant component specification.


---

31. Capability and State Separation

Capability MUST NOT be inferred solely from component state.

For example:

Running

does not automatically mean:

Network Access
Filesystem Access
GPU Access
Device Access

Capabilities must be explicitly granted.


---

32. Effect Model

Operations MAY declare observable effects.

Examples:

Pure
ReadsMemory
WritesMemory
ReadsResource
WritesResource
Network
Filesystem
GPU
Process
Time
Randomness
ExternalDevice
NonDeterministic

Effects provide metadata that can be consumed by:

security systems;

sandboxing;

static analysis;

policy engines;

validators;

conformance tools.


An effect declaration MUST describe observable behavior rather than internal implementation details.


---

33. Determinism Model

ULABI distinguishes deterministic and nondeterministic behavior.

Conceptually:

Deterministic Operation
        |
Same defined inputs
        |
Same defined result

When nondeterminism exists, it MUST be explicitly represented where it affects interoperability.

Sources may include:

randomness;

time;

scheduling;

external devices;

distributed state;

hardware variation.


ULABI MUST NOT falsely claim deterministic behavior where it cannot be guaranteed.


---

34. Compatibility Model

Compatibility is determined by the contract.

Conceptually:

Interface Identity
       |
Version
       |
Types
       |
Layout
       |
Calling Convention
       |
Ownership
       |
Errors
       |
Capabilities
       |
Effects
       |
Execution Semantics
       |
Compatibility Decision

Two implementations MUST NOT be considered compatible merely because their function names or source-level APIs look similar.

Detailed compatibility rules belong to:

ULABI-VERSIONING.md

docs/compatibility/backwards-compatibility.md

docs/compatibility/forwards-compatibility.md

docs/compatibility/feature-negotiation.md

docs/compatibility/capability-discovery.md



---

35. Version Model

Every published ULABI interface MUST have a version.

The universal model recognizes:

Major
Minor
Patch

Version compatibility MUST be determined according to the applicable versioning specification.

An implementation MUST NOT silently interpret an incompatible interface as compatible.


---

36. Negotiation Model

Optional functionality is established through capability discovery and feature negotiation.

Conceptually:

Implementation A
      |
Capabilities
      |
      +----------------+
                       |
                       v
                Compatibility
                  Evaluation
                       |
                       v
Implementation B
      |
Capabilities

Only mutually supported optional functionality may be used.

Unknown mandatory functionality MUST produce safe incompatibility behavior.


---

37. Profile Model

ULABI uses profiles to prevent the Core from becoming unnecessarily large.

The conceptual model is:

ULABI
 |
 +-- Core
 |
 +-- Memory Profile
 |
 +-- Security Profile
 |
 +-- Async Profile
 |
 +-- Streaming Profile
 |
 +-- Distributed Profile
 |
 +-- Hardware Profile
 |
 +-- Embedded Profile
 |
 +-- Real-Time Profile
 |
 +-- Reliability Profile
 |
 +-- Other Standard Profiles

A profile consists of:

Core Requirements
+
Required Extensions
+
Optional Extensions
+
Constraints
+
Security Requirements
+
Conformance Requirements

An implementation MUST declare the profiles it supports.


---

38. Core Minimality

The universal model reinforces the principle:

> If a capability is not fundamental to universal interoperability, it should not automatically become part of the Core.



Examples that generally belong in profiles include:

GPU execution
Distributed consensus
Self-healing
Real-time scheduling
Shared memory
Hardware cryptography
Specialized accelerators

This prevents the Core from becoming tied to a particular class of machine.


---

39. Interoperability Modes

ULABI supports three principal modes.

39.1 In-Process

+---------------------------------------+
| Process                               |
|                                       |
| Component A                           |
|      |                                |
|    ULABI                              |
|      |                                |
| Component B                           |
+---------------------------------------+

Possible optimizations include:

direct calls;

shared memory;

zero-copy;

direct handles.



---

39.2 Out-of-Process

+------------------+     +------------------+
| Process A        |     | Process B        |
|                  |     |                  |
| ULABI Adapter    |<--->| ULABI Adapter    |
+------------------+     +------------------+

This mode provides stronger isolation and independent failure domains.


---

39.3 Distributed

Machine A                         Machine B

Application                      Application
     |                                |
ULABI Adapter                    ULABI Adapter
     |                                |
     +--------- Transport ------------+

Distributed execution MUST NOT be assumed equivalent to local execution.


---

40. Virtualized Location Model

The same logical ULABI interface MAY be implemented at different locations.

ULABI Interface
      |
      +-- Same Process
      |
      +-- Another Process
      |
      +-- Container
      |
      +-- Virtual Machine
      |
      +-- Another Host
      |
      +-- Accelerator
      |
      +-- Embedded Device

Location transparency MUST NOT hide meaningful changes in:

latency;

failure;

security;

consistency;

resource ownership;

availability.



---

41. Cross-Architecture Model

ULABI MUST remain independent of CPU architecture.

An implementation MAY target:

x86
x86-64
ARM
AArch64
RISC-V
Power
MIPS
SPARC
DSP
GPU
NPU
FPGA
Quantum Interfaces
Future Architectures

Architecture-specific details MUST be isolated behind architecture profiles or adapters.

The semantic ULABI contract MUST remain stable.

Detailed architecture mapping belongs to:

docs/platforms/architectures.md

and:

docs/hardware/


---

42. Cross-Platform Model

ULABI MAY operate across:

Embedded
Mobile
Desktop
Server
Cloud
WebAssembly
Virtual Machine
Container
Accelerator
Specialized Device

Platform-specific requirements MUST remain outside the universal semantic contract unless they are proven fundamental to interoperability.


---

43. Hardware Model

ULABI MAY represent hardware-backed resources through profiles.

Examples:

CPU
GPU
NPU
FPGA
Quantum Processor
Accelerator
Device

Hardware-specific execution MUST NOT redefine the universal semantic model.

Hardware profiles MUST map hardware capabilities into explicit ULABI contracts.


---

44. Distributed Model

Distributed ULABI extends, but does not erase, local semantics.

A distributed operation introduces additional concerns:

Latency
Availability
Serialization
Authentication
Authorization
Failure
Retry
Timeout
Consistency
Partitioning

These concerns MUST be represented by the applicable distributed profile.

Distributed execution MUST NOT silently change the meaning of a local operation.


---

45. Scalability Model

ULABI MUST support systems ranging from constrained embedded environments to large distributed systems.

Scalability MUST be achieved through:

Minimal Core
+
Profiles
+
Streaming
+
Partitioning
+
Negotiation
+
Resource Limits
+
Transport Independence
+
Implementation Independence

The universal model MUST NOT require every implementation to support the largest possible configuration.

An embedded implementation MAY support a minimal profile.

A distributed implementation MAY support additional profiles.

Both remain ULABI implementations.

Detailed scalability requirements belong to:

docs/architecture/scalability.md


---

46. Streaming Model for Large Data

Large values SHOULD NOT require unbounded buffering.

ULABI SHOULD support:

Producer
   |
Stream
   |
Consumer

Streaming is particularly important for:

large files;

media;

tensors;

datasets;

network data;

logs;

telemetry;

accelerator data.


A streaming implementation MUST define resource and lifetime limits.


---

47. Partitioning Model

Large logical objects MAY be partitioned.

Conceptually:

Logical Object
      |
      +---- Partition 1
      +---- Partition 2
      +---- Partition 3
      +---- ...

Partitioning MUST preserve the logical contract.

The receiver MUST be able to determine:

partition identity;

ordering where required;

completeness;

boundaries;

integrity;

termination.


Partitioning MUST NOT silently change semantic meaning.


---

48. Resource-Bounded Model

ULABI implementations MUST NOT assume infinite resources.

Relevant resources may include:

Memory
CPU
Storage
Network
Handles
Threads
Descriptors
GPU Memory
Device Capacity
Execution Time

When resources are exhausted, the implementation MUST produce a defined failure or controlled degradation according to the applicable profile.


---

49. Backpressure Model

Streaming and asynchronous systems MAY use backpressure.

Conceptually:

Producer
    |
    v
Buffer
    |
    v
Consumer

If the consumer cannot accept additional data, the contract MUST define what happens.

Possible outcomes include:

Block
Buffer
Slow Producer
Reject
Drop according to policy
Cancel
Fail

The behavior MUST NOT be implementation-specific when it affects correctness.


---

50. Zero-Copy Model

ULABI MAY support zero-copy interoperability.

Zero-copy MUST NOT override:

ownership;

lifetime;

security;

mutability;

isolation;

alignment;

memory safety.


A zero-copy optimization is valid only when the underlying contract remains unchanged.


---

51. Security Model Boundary

The universal model treats security as a first-class property.

Conceptually:

Identity
   |
Integrity
   |
Capability
   |
Policy
   |
Operation

ULABI participation MUST NOT imply trust.

Detailed security semantics belong to:

docs/security/


---

52. Reliability Model

ULABI supports explicit reliability semantics.

A component MAY expose:

Healthy
Degraded
Recovering
Failed
Unavailable

Reliability behavior MUST be bounded and policy-controlled.

Self-healing MUST NOT imply unrestricted self-modification.

Detailed reliability behavior belongs to:

docs/reliability/


---

53. Observability Model

ULABI implementations MAY expose standardized observability metadata.

Possible observability concepts include:

Trace
Span
Diagnostic
Metric
Health
Event
Failure Evidence

Observability MUST NOT silently become a mandatory dependency of the Core.

Detailed observability behavior belongs to:

docs/observability/


---

54. Tooling Model

ULABI SHOULD be inspectable by tooling.

Tools MAY consume:

Interface Metadata
Type Descriptors
Version Information
Capabilities
Effects
Compatibility Information
Debug Metadata

Possible tools include:

Compiler
Linker
Loader
Validator
Debugger
Profiler
ABI Difference Analyzer
Conformance Tester

Detailed tooling contracts belong to:

docs/tooling/


---

55. Validation Model

Before an interface is accepted for use, implementations SHOULD validate:

Identity
Version
Types
Layout
Calling Convention
Ownership
Lifetime
Capabilities
Security
Compatibility

Validation MUST fail safely when required information is invalid or missing.


---

56. Conformance Model

A ULABI implementation is not considered universally conformant merely because it implements one function or serialization format.

Conformance is profile-specific.

Conceptually:

Implementation
      |
      +-- Core
      |
      +-- Types
      |
      +-- Memory
      |
      +-- FFI
      |
      +-- Security
      |
      +-- Async
      |
      +-- Streaming
      |
      +-- Distributed
      |
      +-- Reliability
      |
      +-- Other Profiles

Each claimed capability MUST be testable.

Detailed conformance requirements belong to:

docs/standards/conformance.md

docs/standards/compliance-levels.md

docs/standards/test-suite.md

docs/standards/certification.md



---

57. Reference Implementation Boundary

Reference implementations MUST implement the published contract.

A reference implementation MUST NOT become the hidden definition of ULABI.

The specification remains authoritative.

Multiple independent implementations SHOULD be encouraged.


---

58. Implementation Adapter Model

Each implementation SHOULD expose a dedicated ULABI adapter layer.

Conceptually:

+---------------------------+
| Implementation            |
+---------------------------+
| ULABI Adapter             |
+---------------------------+
| ULABI Contract            |
+---------------------------+

The adapter is responsible for translating:

Source Semantics
       |
       v
ULABI Semantics
       |
       v
Boundary Representation

The adapter MUST NOT alter the meaning of the ULABI contract.


---

59. Adapter Isolation

ULABI adapters SHOULD isolate implementation-specific assumptions.

For example:

Rust Ownership
       |
       v
ULABI Ownership

Java Object
       |
       v
ULABI Resource / Value

C Pointer
       |
       v
ULABI Borrowed / Owned Memory

The adapter translates semantics.

It does not redefine them.


---

60. Universal Model Invariants

The following invariants apply to the universal model.

UMODEL-001 — Independence

ULABI MUST remain independent from individual languages and projects.

UMODEL-002 — Explicit Mapping

Source-language concepts MUST be explicitly mapped to ULABI concepts.

UMODEL-003 — Stable Semantics

Compatible implementations MUST preserve the semantics of the published contract.

UMODEL-004 — No Hidden Transport

A transport change MUST NOT silently alter the contract.

UMODEL-005 — No Hidden Authority

ULABI participation MUST NOT automatically grant capabilities.

UMODEL-006 — No Hidden Ownership

Ownership MUST NOT silently change across a boundary.

UMODEL-007 — No Hidden Lifetime Extension

A participant MUST NOT retain a value beyond its permitted lifetime.

UMODEL-008 — Explicit Failure

Unsupported or invalid operations MUST produce defined behavior.

UMODEL-009 — Profile Isolation

Profile-specific behavior MUST NOT silently become Core behavior.

UMODEL-010 — Implementation Independence

Independent organizations MUST be able to implement the model independently.

UMODEL-011 — Deterministic Contract

The same valid contract MUST produce the same compatibility interpretation.

UMODEL-012 — Semantic Preservation

Adapters MUST preserve the semantic meaning of interoperable operations.


---

61. Security Requirements

Implementations of the universal model:

MUST:

validate interface identity;

validate applicable versions;

enforce declared ownership;

enforce declared lifetimes;

enforce declared capabilities;

reject malformed boundary data;

prevent unauthorized resource access;

distinguish unsupported functionality from successful functionality.


Implementations SHOULD:

use least privilege;

validate before execution;

fail closed for security-sensitive unknown states;

provide auditable capability decisions;

isolate untrusted components.



---

62. Failure Requirements

The universal model recognizes failures as normal architectural events.

Relevant failures include:

InvalidArgument
TypeMismatch
Unsupported
PermissionDenied
ResourceExhausted
Timeout
Cancelled
Unavailable
ProtocolError
IntegrityFailure
SecurityViolation
HardwareFailure
InternalFailure

Failure behavior MUST remain compatible with the applicable interface contract.

An implementation MUST NOT convert a defined failure into successful behavior without an explicit compatibility rule.


---

63. Recovery Requirements

Recovery MAY occur when the applicable profile permits it.

Recovery MUST be:

bounded;

policy-controlled;

observable where required;

verifiable;

compatible with ownership and lifetime;

compatible with security policy.


Recovery MUST NOT silently mutate the interface contract.


---

64. Extensibility

The universal model MUST support future technologies.

Future technologies MAY include:

new processors;

new memory architectures;

new accelerators;

new programming languages;

new execution models;

new transports;

new distributed systems;

new storage models.


Future extensions MUST preserve the fundamental semantic model.

An extension MUST NOT require existing conformant implementations to adopt unknown semantics merely to remain valid.


---

65. Unknown Extension Behavior

Implementations encountering an unknown extension MUST distinguish between:

Optional Extension

and:

Mandatory Extension

Unknown optional functionality MAY be ignored.

Unknown mandatory functionality MUST result in safe incompatibility handling.


---

66. Universal Model and Future Evolution

ULABI is designed to evolve without repeatedly redefining its fundamental semantic model.

Future specifications SHOULD extend:

Core
 |
 +-- Profile
 |
 +-- Extension

rather than redefine the underlying model.

The semantic principles of:

Identity
Types
Contracts
Ownership
Lifetime
Capabilities
Errors
Compatibility

should remain stable across generations.


---

67. Integration Contract

This document integrates with the following specifications.

Architecture

ULABI-DESIGN.md
        |
        +-- docs/architecture/design-principles.md
        |
        +-- docs/architecture/universal-model.md
        |
        +-- docs/architecture/scalability.md
        |
        +-- docs/architecture/overview.md

ABI

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

Type System

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md

Interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Runtime

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Memory

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

Security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Distributed

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Compatibility

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Reliability

docs/reliability/self-healing.md
docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md

Standards

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md


---

68. Non-Duplication Rule

This document defines the universal model.

It MUST NOT become the detailed specification for every subsystem.

Specifically:

calling convention details belong in docs/abi/calling-convention.md;

stack details belong in docs/abi/stack-model.md;

register details belong in docs/abi/register-model.md;

memory details belong in docs/abi/memory-model.md;

type details belong in docs/type-system/;

FFI details belong in docs/interoperability/;

runtime details belong in docs/runtime/;

security details belong in docs/security/;

distributed details belong in docs/distributed/;

reliability details belong in docs/reliability/;

compatibility details belong in docs/compatibility/;

conformance details belong in docs/standards/.


This prevents the architecture from accumulating contradictory duplicate definitions.


---

69. Required Conformance Tests

The universal model MUST eventually be testable through conformance tests.

At minimum, the test suite SHOULD verify:

UMODEL-001  Language independence
UMODEL-002  Explicit type mapping
UMODEL-003  Semantic preservation
UMODEL-004  Transport independence
UMODEL-005  Capability isolation
UMODEL-006  Ownership preservation
UMODEL-007  Lifetime preservation
UMODEL-008  Defined failure behavior
UMODEL-009  Profile isolation
UMODEL-010  Independent implementation
UMODEL-011  Deterministic compatibility
UMODEL-012  Adapter semantic preservation

Tests SHOULD include implementations written in different programming languages.

Tests SHOULD also include:

in-process interoperability;

out-of-process interoperability;

serialized interoperability;

streaming interoperability;

incompatible-version scenarios;

malformed-input scenarios;

capability violations;

lifetime violations;

ownership violations.



---

70. Reference Implementation Requirements

A future reference implementation SHOULD provide:

ULABI Contract Parser
ULABI Type System
ULABI Interface Registry
ULABI Capability Model
ULABI Version Negotiator
ULABI Compatibility Engine
ULABI Boundary Validator
ULABI Error Model
ULABI Adapter Interface

The reference implementation MUST implement the specification rather than define it.


---

71. Completion Criteria

This document is complete for ULABI 0.2.x when:

the universal semantic model is defined;

implementation independence is preserved;

the relationship between Core and Profiles is explicit;

identity and contracts are defined conceptually;

types, values, resources, handles, ownership, and lifetime are separated;

execution and locality semantics are explicit;

transport independence is defined;

security and capability boundaries are explicit;

failure and recovery boundaries are explicit;

scalability is separated from the universal model;

subsystem ownership is assigned to specialized specifications;

integration dependencies are documented;

conformance requirements are identifiable.


This document MUST NOT be repeatedly rewritten merely because a specialized component specification is implemented.

Specialized specifications MUST conform to this universal model rather than modify it implicitly.


---

72. Final Architectural Rule

> ULABI defines the universal semantic interoperability contract. Implementations define how that contract is realized.



No programming language owns ULABI.

No compiler owns ULABI.

No runtime owns ULABI.

No operating system owns ULABI.

No processor architecture owns ULABI.

No company owns the technical meaning of ULABI.

No individual implementation is the definition of ULABI.

ULABI exists as the stable contract between independently developed systems.

---

# 2. Important result of the repository review

There is an important architectural point here.

The repository already has substantial material in `ULABI-DESIGN.md` and `ULABI-SPEC.md`; therefore **we should not keep expanding `universal-model.md` with detailed ABI rules**. The existing master document already establishes the universal semantic/type/function/memory model, while `ULABI-SPEC.md` has normative requirements for those areas.  

The new `universal-model.md` above therefore acts as the **architectural bridge**: it defines how the pieces relate without stealing detailed ownership from `core-abi.md`, `calling-convention.md`, `memory-model.md`, etc.

That is important for your requirement that we should be able to finish one file without later reopening it because another file introduced a conflicting definition.

---

# 3. What should be done under which files

I recommend treating the repository as **three layers**:

```text
LAYER 1 — Architecture
        ↓
LAYER 2 — Normative Specifications
        ↓
LAYER 3 — Executable Implementation / Conformance

And we should work dependency-first, not in arbitrary directory order.

Phase 1 — Independent architectural foundation

These files establish concepts that other documents depend upon.

Order	File	What it owns

1	ULABI-DESIGN.md	Master architecture
2	docs/architecture/design-principles.md	Immutable architectural principles
3	docs/architecture/universal-model.md	Universal semantic/interoperability model
4	docs/architecture/overview.md	Public architectural map
5	docs/architecture/scalability.md	Scaling constraints and strategies
6	ULABI-VERSIONING.md	Version lifecycle and compatibility
7	ULABI-GOVERNANCE.md	Specification change authority/process


The first three are especially important because everything else should reference them rather than redefine them.


---

4. ABI layer

After the architecture is locked:

docs/abi/core-abi.md

Owns:

interface identity;

ABI identity;

binary boundary;

canonical ABI metadata;

fundamental ABI contract;

Core ABI invariants;

ABI validation boundary.


docs/abi/data-types.md

Owns:

primitive representations;

composite layouts;

alignment;

size;

encoding;

canonical representation.


docs/abi/calling-convention.md

Owns:

argument passing;

return passing;

registers;

stack interaction;

indirect arguments;

ABI frames;

variadic ABI;

architecture-independent calling semantics.


docs/abi/memory-model.md

Owns:

boundary memory;

ownership;

borrowing;

sharing;

transfer;

mutation;

release;

lifetime interaction.


docs/abi/stack-model.md

Owns:

logical ULABI stack model;

frames;

frame ownership;

frame lifetime;

call nesting;

stack safety;

stack-independent execution.


docs/abi/register-model.md

Owns:

abstract registers;

register classes;

volatile/nonvolatile semantics;

architecture mapping.


docs/abi/return-values.md

Owns:

scalar returns;

aggregate returns;

indirect returns;

multiple returns;

result values;

ownership of returned values.


docs/abi/exception-model.md

Owns:

exception interoperability;

error-to-exception mappings;

unwinding boundaries;

non-exceptional error paths.



---

5. Type-system layer

These should not redefine the ABI representation.

docs/type-system/universal-types.md

Owns semantic types.

docs/type-system/type-descriptors.md

Owns machine-readable type metadata.

docs/type-system/type-compatibility.md

Owns semantic type compatibility.

docs/type-system/generics.md

Owns generic type interoperability.

docs/type-system/enums.md

Owns enum semantics.

docs/type-system/structs.md

Owns structured records.

docs/type-system/unions.md

Owns union/variant semantics.

The division should be:

universal-types.md
        ↓
What does the type mean?

data-types.md
        ↓
How is it represented?

type-descriptors.md
        ↓
How is the type described?

type-compatibility.md
        ↓
When are two types compatible?

This avoids repetition.


---

6. Interoperability layer

docs/interoperability/language-interoperability.md

Owns source-language mapping.

docs/interoperability/foreign-function-interface.md

Owns FFI boundary behavior.

docs/interoperability/object-model.md

Owns object/reference interoperability.

docs/interoperability/name-mangling.md

Owns mapping between source names and stable ULABI identities.

docs/interoperability/symbol-resolution.md

Owns symbol lookup.

docs/interoperability/cross-language-data.md

Owns practical cross-language data mapping.


---

7. Runtime layer

docs/runtime/runtime-interface.md

Owns runtime-facing ULABI contract.

docs/runtime/process-model.md

Owns component/process lifecycle.

docs/runtime/threading.md

Owns thread semantics.

docs/runtime/async-model.md

Owns async semantics.

docs/runtime/concurrency.md

Owns concurrency semantics.

docs/runtime/resource-management.md

Owns resource lifecycle.


---

8. Memory layer

docs/memory/memory-safety.md

Safety guarantees.

docs/memory/ownership.md

Ownership transitions.

docs/memory/lifetimes.md

Lifetime semantics.

docs/memory/allocation.md

Allocation/deallocation.

docs/memory/virtual-memory.md

Virtual memory integration.

docs/memory/shared-memory.md

Cross-component shared memory.

The important distinction is:

docs/abi/memory-model.md
        =
ABI boundary contract

docs/memory/*
        =
detailed memory subsystem contracts


---

9. Security layer

docs/security/security-model.md

Overall security architecture.

docs/security/capability-security.md

Capabilities.

docs/security/sandboxing.md

Isolation.

docs/security/cryptography.md

Cryptographic primitives/agility.

docs/security/authentication.md

Identity authentication.

docs/security/authorization.md

Permission decisions.

docs/security/zero-trust.md

Trust boundaries.

docs/security/secure-loading.md

Component loading.

docs/security/supply-chain-security.md

Dependency and artifact integrity.


---

10. Reliability layer

docs/reliability/fault-detection.md

Detection.

docs/reliability/fault-isolation.md

Isolation.

docs/reliability/recovery.md

Controlled recovery.

docs/reliability/rollback.md

Rollback semantics.

docs/reliability/health-monitoring.md

Health.

docs/reliability/self-healing.md

Orchestrates:

Detect
 ↓
Diagnose
 ↓
Isolate
 ↓
Recover
 ↓
Verify
 ↓
Rollback
 ↓
Escalate

Self-healing must not redefine the universal model.


---

11. Distributed layer

docs/distributed/distributed-abi.md

Distributed ABI contract.

docs/distributed/remote-calls.md

Remote invocation.

docs/distributed/serialization.md

Canonical serialization.

docs/distributed/service-discovery.md

Discovery.

docs/distributed/distributed-errors.md

Remote failures.

docs/distributed/consensus-boundaries.md

Explicit boundary between ABI and distributed consensus.


---

12. Platform and hardware layer

These should translate ULABI into environments rather than modify ULABI.

Platforms

docs/platforms/operating-systems.md
docs/platforms/architectures.md
docs/platforms/embedded.md
docs/platforms/mobile.md
docs/platforms/desktop.md
docs/platforms/server.md
docs/platforms/cloud.md
docs/platforms/webassembly.md
docs/platforms/accelerators.md

Hardware

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md
docs/hardware/future-architectures.md


---

13. Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md

These should consume the specifications rather than redefine them.


---

14. Observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md


---

15. Compatibility

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

These should consume the identity/version/type/interface definitions rather than duplicate them.


---

16. Standards

Finally:

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

These define how an implementation proves what it supports.


---

17. Required schemas

The specification eventually needs machine-readable schemas.

I recommend:

schemas/
├── interface.schema.json
├── function.schema.json
├── type.schema.json
├── type-descriptor.schema.json
├── error.schema.json
├── capability.schema.json
├── profile.schema.json
├── version.schema.json
├── contract.schema.json
├── resource.schema.json
├── stream.schema.json
├── compatibility.schema.json
├── component.schema.json
└── conformance.schema.json

These schemas should be derived from the normative documents, not become a second undocumented specification.


---

18. Required examples

examples/
├── basic/
├── types/
├── functions/
├── memory/
├── errors/
├── ffi/
├── async/
├── streaming/
├── capabilities/
├── security/
├── distributed/
├── embedded/
├── accelerator/
└── cross-language/

Examples should demonstrate the specification and should not introduce new rules.


---

19. Required tests

tests/
├── abi/
├── types/
├── calling-convention/
├── memory/
├── stack/
├── registers/
├── returns/
├── errors/
├── ffi/
├── runtime/
├── security/
├── compatibility/
├── distributed/
├── streaming/
├── async/
├── resources/
└── malformed/


---

20. Required conformance tree

conformance/
├── core/
├── types/
├── memory/
├── ffi/
├── runtime/
├── security/
├── async/
├── streaming/
├── distributed/
├── reliability/
├── profiles/
└── certification/


---

21. Required reference implementation

I recommend eventually creating:

reference/
├── parser/
├── metadata/
├── types/
├── identity/
├── versioning/
├── compatibility/
├── validation/
├── capabilities/
├── memory/
├── calls/
├── errors/
├── resources/
├── serialization/
├── transport/
└── profiles/

The reference implementation should be deliberately modular.


---

22. Required implementation modules

The eventual implementation should not be one giant ulabi.rs, ulabi.cpp, or equivalent file.

A language-neutral logical module inventory should be:

ulabi-core
├── identity
├── version
├── interface
├── contract
├── metadata
├── types
├── values
├── functions
├── calls
├── returns
├── errors
├── ownership
├── lifetime
├── capabilities
├── effects
├── execution
├── locality
├── resources
├── handles
├── validation
└── compatibility

Then:

ulabi-runtime
├── component
├── lifecycle
├── process
├── threading
├── async
├── concurrency
├── cancellation
├── resources
└── scheduler-adapter

Then:

ulabi-memory
├── ownership
├── lifetime
├── allocator
├── region
├── shared-memory
├── zero-copy
├── buffer
└── safety

Then:

ulabi-ffi
├── binding
├── symbol
├── name-mangling
├── calling-convention
├── type-mapping
├── object-mapping
└── adapter

Then:

ulabi-security
├── identity
├── authentication
├── authorization
├── capability
├── policy
├── sandbox
├── secure-loader
├── integrity
└── supply-chain

Then:

ulabi-distributed
├── remote-call
├── serialization
├── transport
├── discovery
├── distributed-error
├── timeout
├── retry
├── consistency
└── partition

Then:

ulabi-validation
├── contract-validator
├── type-validator
├── layout-validator
├── capability-validator
├── compatibility-validator
├── schema-validator
└── security-validator

Then:

ulabi-conformance
├── test-runner
├── profile-tests
├── interoperability-tests
├── negative-tests
├── fuzz-tests
├── compatibility-tests
└── certification-report


---

23. Dependency order for the actual code

This is the order I recommend when implementation eventually begins.

Stage 1 — No dependencies

identity
version
error
primitive-types

Stage 2

type-descriptor
interface
function-signature
capability
effect

Stage 3

contract
validation
compatibility

Stage 4

ownership
lifetime
memory
resource
handle

Stage 5

calling-convention
arguments
returns
stack
register-abstraction

Stage 6

ffi
language-adapter
symbol-resolution
object-model

Stage 7

runtime
process
threading
async
concurrency
cancellation

Stage 8

serialization
transport
remote-call
distributed
discovery

Stage 9

security
sandbox
secure-loader
authorization

Stage 10

streaming
shared-memory
zero-copy
accelerator

Stage 11

reliability
fault-detection
recovery
rollback
self-healing

Stage 12

conformance
certification
reference implementation

This dependency order is important because it means, for example, the FFI implementation does not have to be rewritten later because identity was defined differently.


---

24. The "finish one file once" rule

I recommend locking this engineering rule into ULABI:

> Every normative document owns one clearly defined contract and must declare its dependencies and consumers before implementation begins.



Every future specification should therefore contain these sections:

1. Purpose
2. Scope
3. Non-goals
4. Terminology
5. Normative language
6. Dependencies
7. Contract
8. Interfaces
9. Data structures
10. Invariants
11. Security requirements
12. Failure modes
13. Recovery
14. Compatibility
15. Versioning
16. Integration contract
17. Conformance requirements
18. Required schemas
19. Required tests
20. Reference implementation boundary
21. Completion criteria
22. Non-duplication rule

That directly addresses your requirement that a file should be implementable without discovering later that another file requires its fundamental contract to be rewritten.


---

One important correction to the earlier plan

The repository currently has the architecture files appearing as actual files, but several are still placeholders; for example, universal-model.md is explicitly just a placeholder.

So we should not create another parallel architecture tree.

The correct strategy is:

Existing file
      ↓
Read it
      ↓
Determine what it owns
      ↓
Remove placeholder/duplicate material
      ↓
Give it one authoritative contract
      ↓
Declare dependencies
      ↓
Implement
      ↓
Conformance test

This is especially important because the repository already contains both the large master architecture and normative specification.

For the next file, I would proceed in dependency order with docs/architecture/overview.md only after universal-model.md is accepted, then docs/architecture/scalability.md, and only then begin the ABI implementation specifications. This prevents the architecture documents from contradicting each other.