I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI before writing this. The repository currently identifies ULABI as version 0.2.0-draft, language/runtime/platform/vendor neutral, with a Minimal Core + Standard Profiles + Extensible Ecosystem architecture. It explicitly separates ULABI from Zamani and Sankofa and requires explicit semantics, stable compatibility, capability limitation, determinism, recoverability, and implementation independence.

I also checked the existing target file: docs/architecture/design-principles.md is currently only a placeholder, so this is the correct file to replace.

Below is the complete replacement for docs/architecture/design-principles.md, followed by the implementation/file plan and the complete code-module inventory.


---

1. docs/architecture/design-principles.md

Copy everything inside this block into:

docs/architecture/design-principles.md

# ULABI Design Principles

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/architecture/design-principles.md`  
**Specification Family:** ULABI Architecture  
**Status:** Normative Architectural Foundation  
**Current ULABI Design Version:** 0.2.0-draft  
**License:** Apache-2.0

---

## 1. Purpose

This document defines the architectural principles that govern the design,
implementation, evolution, and conformance of ULABI.

ULABI is a universal, language-independent interoperability standard.

Its purpose is to define stable contracts between independently developed
software and hardware systems without requiring those systems to share:

- a programming language;
- a compiler;
- a runtime;
- an operating system;
- a CPU architecture;
- a memory-management strategy;
- a vendor;
- a deployment environment;
- or an implementation strategy.

ULABI defines interoperability contracts.

ULABI does not define the internal implementation of the systems that use
those contracts.

This document therefore governs the architectural boundaries within which
all ULABI specifications and implementations MUST operate.

---

# 2. Relationship to ULABI-DESIGN.md

`ULABI-DESIGN.md` is the master architecture document.

This document refines and formalizes the design principles established
there.

The master architectural principles include:

- language neutrality;
- runtime neutrality;
- platform neutrality;
- architecture neutrality;
- vendor neutrality;
- open specification;
- stable core;
- layered extensions;
- explicit semantics;
- secure defaults;
- failure-oriented design;
- determinism;
- compatibility;
- implementation independence;
- capability limitation;
- recoverability;
- absence of hidden semantics.

These principles are binding architectural constraints on subsequent ULABI
specifications.

A component specification MUST NOT silently introduce an architectural
assumption that contradicts these principles.

---

# 3. Normative Language

The following terms are normative:

- **MUST** — an absolute requirement.
- **MUST NOT** — an absolute prohibition.
- **REQUIRED** — equivalent to MUST.
- **SHALL** — equivalent to MUST.
- **SHALL NOT** — equivalent to MUST NOT.
- **SHOULD** — a strong recommendation that may be violated only for a
  documented technical reason.
- **SHOULD NOT** — a strong recommendation against an action.
- **MAY** — an optional capability.
- **OPTIONAL** — not required for the applicable profile.

An implementation claiming conformance MUST satisfy all applicable MUST and
MUST NOT requirements.

---

# 4. Fundamental Principle

> ULABI defines the universal interoperability contract; it does not belong
> to any one programming language, compiler, operating system, CPU
> architecture, runtime, company, or project.

ULABI exists at the boundary between systems.

It does not replace the systems themselves.

For example:

```text
Language A
    |
Compiler A
    |
Runtime A
    |
ULABI Adapter
    |
    +-------------------+
                        |
                     ULABI
                        |
    +-------------------+
    |
ULABI Adapter
    |
Runtime B
    |
Compiler B
    |
Language B

The internal implementation of Language A and Language B remains independent.


---

5. Language Neutrality

ULABI MUST remain independent of individual programming languages.

A ULABI implementation MAY be implemented using any suitable programming language.

Examples include:

C;

C++;

Rust;

Go;

Python;

Java;

Kotlin;

Swift;

Fortran;

Ada;

Zig;

or future languages.


The implementation language MUST NOT change the ULABI contract.

ULABI specifications MUST NOT require:

a particular syntax;

a particular type system;

a particular ownership model;

a particular exception mechanism;

a particular garbage collector;

a particular object model;

or a particular compiler architecture.


Source-language concepts MUST be mapped explicitly to ULABI concepts.


---

6. Project Independence

ULABI MUST remain independent from every individual project using it.

In particular:

Zamani  != ULABI
Sankofa != ULABI

Zamani MAY implement ULABI.

Sankofa MAY implement ULABI.

Neither project defines ULABI.

The same rule applies to every other language, runtime, operating system, library, company, or implementation.

A ULABI specification MUST NOT contain requirements that can only be implemented by one particular project.


---

7. Runtime Neutrality

ULABI MUST NOT require a universal runtime.

Implementations MAY use:

garbage collection;

reference counting;

ownership systems;

manual memory management;

region allocation;

arenas;

tracing;

stack allocation;

heap allocation;

managed runtimes;

native execution;

virtual machines;

interpreters;

JIT compilation;

ahead-of-time compilation.


The ULABI boundary MUST describe observable interoperability semantics rather than imposing one internal runtime model.


---

8. Platform Neutrality

ULABI MUST support interoperability across different execution environments.

The architecture MUST permit implementations targeting:

embedded systems;

mobile devices;

desktop systems;

servers;

cloud environments;

virtual machines;

containers;

WebAssembly environments;

accelerators;

specialized devices;

future computing environments.


A platform-specific implementation MAY provide a platform profile.

Platform-specific behavior MUST NOT silently become part of the universal core.


---

9. Architecture Neutrality

ULABI MUST NOT depend on one CPU architecture.

Implementations MAY target:

x86;

x86-64;

ARM;

AArch64;

RISC-V;

Power;

MIPS;

SPARC;

DSP architectures;

GPU architectures;

NPU architectures;

FPGA-based systems;

quantum interfaces;

future architectures.


Architecture-specific register sets, instruction sets, and calling conventions MUST be represented through an architecture adapter or profile.

The universal semantic contract MUST remain architecture independent.


---

10. Vendor Neutrality

ULABI MUST NOT require or privilege a particular vendor.

No vendor-specific implementation MUST become a hidden dependency of the standard.

Vendor-specific optimizations MAY exist as extensions.

Such extensions MUST:

1. be explicitly identified;


2. have stable identifiers;


3. declare their requirements;


4. have defined unsupported behavior;


5. not invalidate the base ULABI contract.




---

11. Open Specification

The ULABI specification SHOULD remain publicly inspectable.

Normative behavior MUST be documented.

Undocumented implementation behavior MUST NOT be required for interoperability.

Independent organizations MUST be able to implement ULABI from the published specification without requiring proprietary information from another implementation.


---

12. Stable Core

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



The Core MUST contain only semantics fundamental to interoperability.

The Core SHOULD evolve more slowly than extension profiles.

A new feature SHOULD NOT be placed in Core merely because it is useful.

Before adding a feature to Core, the specification process MUST establish that the feature is:

1. broadly applicable;


2. sufficiently stable;


3. language neutral;


4. platform neutral;


5. independently implementable;


6. necessary for interoperability;


7. compatible with the long-term architecture.




---

13. Layered Architecture

ULABI SHOULD be understood as a layered system:

Applications
      |
Language Bindings / Adapters
      |
ULABI Interface Contract
      |
ULABI Core
      |
Standard Profiles
      |
Transport / Execution Layer
      |
Operating System / Runtime
      |
CPU / Accelerator / Hardware

A layer MUST NOT silently assume capabilities belonging to another layer.

For example, the Core MUST NOT assume:

networking;

shared memory;

GPU access;

filesystem access;

process creation;

cryptographic hardware;

distributed consensus.


Those capabilities belong to appropriate profiles.


---

14. Explicit Semantics

ULABI MUST prefer explicit semantics over inferred semantics.

The following MUST be explicit where applicable:

type;

size;

alignment;

ownership;

lifetime;

mutability;

calling convention;

error behavior;

execution mode;

capabilities;

locality;

determinism;

cancellation;

concurrency;

resource requirements;

security requirements.


An implementation MUST NOT infer a security-sensitive or compatibility- sensitive property from an implementation-specific convention.


---

15. No Hidden Magic

ULABI MUST NOT silently alter the semantics of an operation to make two systems appear compatible.

For example:

Declared:
Local operation

Actual:
Network operation

is invalid unless the contract explicitly declares the network capability and locality semantics.

Similarly:

Declared:
Borrowed memory

Actual:
Ownership transfer

is invalid.

ULABI adapters MUST preserve the declared semantics.


---

16. Determinism

ULABI SHOULD use deterministic representations wherever deterministic behavior is required for interoperability.

This includes:

canonical encodings;

stable identifiers;

deterministic type descriptors;

deterministic field ordering;

deterministic error codes;

deterministic compatibility decisions;

deterministic validation results.


Where nondeterminism is unavoidable, it MUST be explicitly represented.

Possible sources include:

randomness;

scheduling;

external time;

hardware variation;

distributed state;

external devices.



---

17. Semantic Separation

ULABI MUST distinguish three levels:

Semantic Meaning
       |
       v
ULABI Boundary Representation
       |
       v
Implementation Representation

The implementation representation is private to an implementation.

The ULABI boundary representation is standardized.

The semantic meaning is the contract that implementations must preserve.

Two implementations MAY use completely different internal representations while remaining interoperable.


---

18. Type-System Neutrality

ULABI MUST NOT require source languages to share a common type system.

Instead, ULABI defines interoperable semantic types.

Examples include:

Bool;

signed integers;

unsigned integers;

floating-point values;

Char;

String;

Bytes;

Unit;

List;

Record;

Enum;

Variant;

Option;

Result;

Handle.


A source language MUST explicitly map its types to ULABI types.


---

19. Memory-Model Neutrality

ULABI MUST NOT force all implementations to use one memory-management model.

ULABI MUST instead define boundary semantics for:

ownership;

borrowing;

sharing;

transfer;

immutability;

mutability;

lifetime;

release;

allocation;

deallocation.


An implementation MUST NOT free or mutate memory without the authority defined by the ULABI contract.


---

20. Ownership Transparency

Ownership MUST be explicit whenever ownership affects correctness or safety.

Supported conceptual states MAY include:

Owned
Borrowed
Shared
Immutable
Mutable
Transferred
Released
Invalid

Ownership transitions MUST be defined.

For example:

Owned
  |
  | transfer
  v
Transferred
  |
  v
Owned by Receiver

An implementation MUST NOT invent ownership transitions that contradict the contract.


---

21. Lifetime Safety

References crossing a ULABI boundary MUST have a defined lifetime.

An implementation MUST NOT retain a borrowed reference beyond its permitted lifetime.

An implementation MUST NOT access released memory.

A ULABI implementation SHOULD detect invalid lifetime transitions whenever practical.


---

22. Capability Limitation

ULABI security follows the principle:

> A component receives only the capabilities required to perform its declared operations.



Capabilities SHOULD be:

explicit;

scoped;

least privilege;

auditable;

revocable where possible.


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

Loading a ULABI component MUST NOT automatically grant unrestricted system access.


---

23. Secure Defaults

Unsafe behavior MUST NOT be the implicit default.

Security-sensitive functionality SHOULD require explicit declaration.

Examples include:

unrestricted filesystem access;

arbitrary process creation;

unrestricted networking;

device access;

secret access;

executable memory;

privileged operations.


An implementation SHOULD fail closed when required security information is missing.


---

24. Boundary Trust

A ULABI boundary MUST NOT automatically imply trust.

The conceptual security sequence is:

Identity
   |
   v
Integrity
   |
   v
Capability
   |
   v
Policy
   |
   v
Operation

An implementation MUST NOT assume that a component is trusted merely because it uses ULABI.


---

25. Failure-Oriented Design

Failures are first-class architectural events.

ULABI components MUST define relevant failure behavior.

Possible failure categories include:

InvalidArgument;

TypeMismatch;

NotFound;

PermissionDenied;

ResourceExhausted;

Timeout;

Cancelled;

Unavailable;

ProtocolError;

IntegrityFailure;

SecurityViolation;

HardwareFailure;

InternalFailure;

Unsupported;

Unknown.


Failure behavior MUST be deterministic enough for callers to respond safely.


---

26. Errors Over Undefined Behavior

An implementation MUST NOT use undefined behavior as an interoperability mechanism.

If a boundary cannot safely execute an operation, it SHOULD produce a defined error.

For example:

Unsupported capability
        |
        v
Defined incompatibility error

rather than:

Unsupported capability
        |
        v
Undefined behavior


---

27. Compatibility

Backward compatibility is a fundamental ULABI requirement.

A compatible implementation SHOULD continue to support previously valid contracts according to the declared compatibility profile.

Compatibility MUST be evaluated using the actual contract, including:

identifiers;

versions;

types;

layout;

calling convention;

ownership;

errors;

capabilities;

effects;

execution semantics.


Name similarity alone MUST NOT establish compatibility.


---

28. Explicit Versioning

Every published ULABI interface MUST have an identifiable version.

Compatibility MUST NOT depend on undocumented version assumptions.

Version changes MUST have defined compatibility semantics.

Implementations SHOULD expose machine-readable version information.


---

29. Capability Discovery

Implementations SHOULD be able to advertise supported capabilities.

Capability discovery SHOULD identify:

ULABI version;

supported profiles;

interface versions;

supported types;

optional extensions;

security features;

runtime capabilities;

resource constraints.


An implementation MUST NOT assume optional functionality exists without verification.


---

30. Feature Negotiation

Optional functionality MUST be negotiated or otherwise explicitly established before use.

The conceptual model is:

Capability A supports Feature X
Capability B supports Feature X
             |
             v
        Feature X may be used

If only one side supports Feature X:

Feature X
   |
   +---- Supported
   |
   +---- Unsupported

the implementation MUST follow the contract-defined fallback or failure behavior.

Unknown mandatory features MUST result in safe incompatibility handling.


---

31. Extension Discipline

ULABI extensions MUST be explicitly separated from the Core.

An extension SHOULD define:

identifier;

version;

purpose;

requirements;

interfaces;

dependencies;

security implications;

compatibility behavior;

unsupported behavior;

conformance tests.


Extensions MUST NOT silently modify Core semantics.


---

32. Profiles

Profiles allow ULABI to serve different environments without forcing every implementation to implement every feature.

Possible profiles include:

Core;

Memory;

Resource;

Async;

Streaming;

Zero-Copy;

Shared Memory;

Security;

Capability;

Sandbox;

Distributed;

Transport;

Hardware;

GPU;

Accelerator;

Tensor;

Real-Time;

Embedded;

Observability;

Debugging;

Internationalization;

Time;

Units;

Safety-Critical;

Verification;

Reliability;

Self-Healing;

Certification.


A profile MUST declare its mandatory requirements.


---

33. Locality Must Be Explicit

ULABI distinguishes execution locality.

Possible locality classifications include:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A local operation MUST NOT silently become a remote operation.

Remote execution introduces additional semantics including:

latency;

availability;

serialization;

authentication;

authorization;

transport failure;

retry;

idempotency;

consistency.


Those semantics MUST be explicit.


---

34. Transport Independence

ULABI MUST remain independent of any single transport.

Possible transports include:

direct calls;

shared memory;

operating-system IPC;

pipes;

sockets;

message queues;

TCP;

QUIC;

WebAssembly host calls;

device buses;

future transports.


Changing the transport SHOULD NOT require redesigning the logical interface.

However, transport-specific semantics MUST NOT be hidden when they affect correctness.


---

35. Execution-Model Neutrality

ULABI MUST NOT require one execution model.

Implementations MAY use:

synchronous execution;

asynchronous execution;

threads;

tasks;

actors;

event loops;

processes;

message passing;

hardware execution;

remote execution.


The boundary contract MUST explicitly describe observable execution properties.


---

36. Concurrency

ULABI MUST NOT require one concurrency model.

Concurrency-sensitive interfaces SHOULD declare:

thread safety;

reentrancy;

synchronization requirements;

ordering requirements;

atomicity;

cancellation;

ownership;

lifetime.


An implementation MUST NOT claim stronger concurrency guarantees than the contract permits.


---

37. Resource Explicitness

Resource consumption SHOULD be explicit where it affects correctness, security, or reliability.

Relevant resources MAY include:

memory;

CPU;

storage;

network;

threads;

handles;

devices;

execution time;

accelerator capacity.


Resource exhaustion MUST have defined behavior.


---

38. Zero-Copy Principle

ULABI MAY support zero-copy interoperability.

Zero-copy MUST NOT compromise:

ownership;

lifetime;

isolation;

mutability;

alignment;

security;

correctness.


A copying implementation MAY be used when zero-copy requirements cannot be safely satisfied.

Performance optimization MUST NOT override the semantic contract.


---

39. Streaming Principle

ULABI SHOULD support large data without requiring the entire dataset to be materialized in memory.

Streaming contracts SHOULD define:

element type;

direction;

ownership;

completion;

failure;

cancellation;

backpressure;

lifetime.


Streaming MUST remain optional unless required by the applicable profile.


---

40. Distributed-System Principle

Distributed ULABI MUST NOT pretend that remote execution is identical to local execution.

Distributed execution introduces:

partial failure;

network failure;

timeout;

duplication;

reordering;

partitioning;

authentication;

serialization;

availability differences.


These semantics MUST be represented by the distributed profile.


---

41. Hardware Independence

ULABI MUST allow hardware-specific implementations without making hardware specificity part of the universal Core.

CPU, GPU, NPU, FPGA, quantum, and other accelerator capabilities SHOULD be represented through profiles or adapters.

Hardware capabilities MUST be explicitly discoverable.


---

42. Real-Time and Safety-Critical Separation

Real-time and safety-critical requirements MUST NOT be assumed by the Core.

They SHOULD be represented by explicit profiles.

A safety-critical profile SHOULD define additional requirements for:

determinism;

bounded execution;

resource limits;

verification;

fault containment;

diagnostics;

certification evidence.



---

43. Observability

ULABI implementations SHOULD expose sufficient information for:

diagnostics;

tracing;

debugging;

health monitoring;

compatibility analysis;

conformance testing.


Observability MUST NOT require exposing secrets or violating security boundaries.


---

44. Deterministic Diagnostics

Machine-readable diagnostics SHOULD be stable across compatible versions.

Human-readable diagnostic messages MAY change.

Programmatic consumers MUST rely on stable identifiers rather than message text.


---

45. Self-Healing Principle

ULABI MAY support self-healing through an explicit Reliability or Self-Healing profile.

Self-healing MUST be:

bounded;

policy-controlled;

auditable;

reversible where possible;

explicitly authorized.


Self-healing MUST NOT permit arbitrary autonomous modification of an implementation.

The conceptual lifecycle is:

Failure
   |
   v
Evidence Collection
   |
   v
Diagnosis
   |
   v
Known Recovery Policy?
   |
 +-----+------+
 |            |
YES           NO
 |            |
 v            v
Recover     Escalate
 |
 v
Verify
 |
 +-----+------+
 |            |
Healthy       Failed
 |            |
 v            v
Continue    Rollback
              |
              v
           Escalate

Recovery MUST NOT exceed the authority granted by the applicable policy.


---

46. No Uncontrolled Self-Modification

A ULABI implementation MUST NOT treat failure detection as authorization to modify arbitrary code, interfaces, security policies, or contracts.

Any recovery mechanism that modifies executable state MUST be explicitly defined by the applicable profile and policy.

Recovery MUST preserve:

interface identity;

security invariants;

ownership rules;

compatibility guarantees;

auditability.



---

47. Implementation Independence

Multiple independent implementations MUST be possible.

The standard MUST NOT depend on:

one compiler;

one runtime;

one operating system;

one vendor;

one programming language;

one repository;

one reference implementation.


Reference implementations demonstrate the standard.

They do not define it.


---

48. Reference Implementations

A reference implementation SHOULD demonstrate:

correct interpretation of the specification;

interoperability;

error handling;

security enforcement;

versioning;

compatibility;

conformance testing.


A reference implementation MUST NOT become the normative source of behavior when its implementation conflicts with the published specification.


---

49. Testability

Every normative requirement SHOULD have a corresponding conformance test where practical.

A specification component SHOULD map to:

Requirement
    |
    v
Test Case
    |
    v
Expected Result
    |
    v
Conformance Status

Non-testable requirements MUST have a documented verification method.


---

50. Formal Verification

Critical ULABI components SHOULD support formal verification where practical.

Particularly important candidates include:

memory safety;

ownership transitions;

capability enforcement;

type compatibility;

binary encoding;

decoder correctness;

calling conventions;

security boundaries;

lifecycle transitions.


Formal verification is complementary to executable conformance testing.


---

51. Fuzzing

Boundary implementations SHOULD be fuzz tested.

Fuzz targets SHOULD include:

decoders;

type descriptors;

interface identifiers;

capability descriptors;

version negotiation;

malformed calls;

malformed errors;

memory metadata;

serialization;

transport messages.


Malformed input MUST produce defined failure behavior rather than undefined behavior.


---

52. Security by Architecture

Security MUST be considered at the architectural boundary rather than added only after implementation.

Security-sensitive specifications MUST address:

identity;

integrity;

authorization;

capabilities;

isolation;

resource limits;

secure loading;

dependency trust;

cryptographic agility;

failure behavior.



---

53. Cryptographic Agility

ULABI MUST NOT permanently bind the universal Core to one cryptographic algorithm.

Cryptographic profiles SHOULD permit algorithm evolution.

Implementations SHOULD be able to negotiate supported cryptographic mechanisms without changing the semantic interface.

Post-quantum cryptographic mechanisms MAY be supported through profiles.


---

54. Privacy

ULABI implementations SHOULD minimize unnecessary exposure of:

application data;

metadata;

identifiers;

credentials;

secrets;

diagnostic information.


Observability MUST respect the security and privacy policy of the component.


---

55. Backward Compatibility

Compatible evolution MUST preserve previously valid contracts according to the applicable compatibility rules.

Changes MUST be classified according to their effect on:

binary layout;

function signatures;

identifiers;

ownership;

lifetime;

errors;

capabilities;

execution semantics;

security requirements.


Compatibility MUST be determined from the complete contract.


---

56. Forward Compatibility

Implementations SHOULD tolerate explicitly designated future extensions.

Unknown optional fields or features MAY be ignored when the contract permits it.

Unknown mandatory requirements MUST result in safe incompatibility handling.

Forward compatibility MUST NOT permit unsafe interpretation of unknown data.


---

57. Graceful Degradation

When optional functionality is unavailable, an implementation SHOULD degrade according to an explicitly defined fallback.

Example:

Preferred capability
       |
       v
Available?
  /       \
YES       NO
 |         |
 v         v
Use       Defined fallback
           |
           v
        Safe failure

An implementation MUST NOT silently substitute behavior that changes the semantic contract.


---

58. Explicit Effects

Interfaces SHOULD declare externally observable effects where applicable.

Possible effects include:

Pure;

ReadsMemory;

WritesMemory;

ReadsResource;

WritesResource;

Network;

Filesystem;

GPU;

Process;

NonDeterministic;

Time;

Randomness;

ExternalDevice.


Effect declarations MAY be used by:

security systems;

static analyzers;

sandboxing;

validators;

conformance tools.



---

59. Interface Identity

Every externally visible ULABI interface MUST have stable identity.

Identity MUST NOT depend on:

source-language name mangling;

compiler-generated addresses;

implementation-specific symbols.


Identity SHOULD remain stable across compatible implementations.


---

60. Separation of Semantic and Physical Interfaces

ULABI distinguishes:

Logical Interface
       |
       v
ULABI Contract
       |
       v
Physical Transport / ABI

The same logical interface MAY be represented through:

in-process calls;

process IPC;

shared memory;

network transport;

accelerator invocation.


The physical representation MUST preserve the logical contract.


---

61. Failure Containment

A failure in one ULABI component SHOULD NOT automatically corrupt another component.

Isolation MAY be provided through:

process boundaries;

sandboxing;

capability boundaries;

memory protection;

transaction boundaries;

resource quotas.


The selected isolation mechanism depends on the applicable profile.


---

62. Recovery Must Preserve Contracts

Recovery MUST NOT restore a component by silently changing its public ABI.

If recovery changes:

interface identity;

version;

type contract;

ownership;

security policy;

capability requirements;


the change MUST be exposed through the appropriate compatibility mechanism.


---

63. Minimality

ULABI SHOULD avoid unnecessary complexity.

Every Core feature SHOULD justify its existence through interoperability requirements.

A feature SHOULD be placed into an extension when it can be implemented without requiring universal Core semantics.

Minimality is a long-term compatibility strategy.


---

64. Extensibility

ULABI MUST remain extensible.

Extensions SHOULD use:

stable identifiers;

versioned contracts;

explicit dependencies;

capability discovery;

feature negotiation.


Extensions MUST NOT require modification of unrelated implementations when they are unsupported.


---

65. No Language-Specific Shortcuts

A specification MUST NOT define behavior solely by referring to a language-specific concept.

For example, a specification MUST NOT require:

Rust ownership

or:

C ABI

or:

Python object semantics

as the universal definition.

Instead, it MUST define the language-neutral semantic property.

For example:

ownership transfer

is a ULABI concept.

A language implementation then maps its native model to that concept.


---

66. No Architecture-Specific Shortcuts

Likewise, ULABI MUST NOT define a universal requirement solely in terms of:

x86 registers;

ARM registers;

RISC-V registers;

one native stack layout;

one native pointer width;

one native endianness.


Architecture-specific details belong in architecture profiles and adapters.


---

67. No Runtime-Specific Shortcuts

ULABI MUST NOT require:

a garbage collector;

reference counting;

a particular scheduler;

a particular event loop;

a particular thread implementation;

a particular exception runtime.


The ULABI contract MUST describe externally observable behavior.


---

68. Security and Compatibility Are Part of the Contract

Security MUST NOT be treated as an optional afterthought when security properties affect interoperability.

Similarly, compatibility MUST NOT be treated as merely documentation.

The contract includes:

Identity
+
Types
+
Calling Convention
+
Memory
+
Errors
+
Capabilities
+
Effects
+
Execution Semantics
+
Versioning
+
Security Requirements


---

69. Independent Implementation Principle

A competent independent organization SHOULD be able to implement ULABI using only:

1. the normative specification;


2. applicable profiles;


3. published schemas;


4. published conformance requirements.



It SHOULD NOT need access to another implementation's private source code.


---

70. Conformance Claims

An implementation MUST NOT claim:

> "ULABI compatible"



without specifying the applicable conformance scope.

Conformance claims SHOULD identify:

ULABI Core
ULABI Types
ULABI Calls
ULABI Memory
ULABI Errors
ULABI Security
ULABI Async
ULABI Streaming
ULABI Distributed
ULABI Hardware
ULABI Reliability
ULABI Self-Healing

only where the relevant requirements have actually been satisfied.


---

71. Conformance Levels

ULABI SHOULD eventually define formal conformance levels.

A possible structure is:

Level 0 — Informational
Level 1 — Core
Level 2 — Interoperability
Level 3 — Secure
Level 4 — Extended
Level 5 — Profile-Specific

The exact level definitions MUST be established by the conformance specification rather than inferred from this document.


---

72. Specification Traceability

Every major ULABI requirement SHOULD be traceable to:

Architecture Principle
        |
        v
Normative Requirement
        |
        v
Schema / Interface
        |
        v
Implementation Module
        |
        v
Conformance Test
        |
        v
Certification Evidence

This prevents architectural requirements from becoming disconnected from implementation and testing.


---

73. Documentation Completeness

Every major ULABI component specification SHOULD contain:

1. Purpose


2. Scope


3. Terminology


4. Requirements


5. Interfaces


6. Data structures


7. Invariants


8. Security requirements


9. Failure modes


10. Recovery behavior


11. Compatibility requirements


12. Conformance tests


13. Reference implementation requirements


14. Integration requirements


15. Versioning requirements



A document MUST NOT rely on a future document to define a requirement that is necessary to implement the current component.

Cross-references MAY refine behavior, but MUST NOT create circular dependencies that make independent implementation impossible.


---

74. Dependency Direction

ULABI specifications SHOULD follow a dependency hierarchy.

Architecture Principles
        |
        v
Universal Model
        |
        v
Core ABI
        |
        +---- Types
        |
        +---- Calling Convention
        |
        +---- Memory
        |
        +---- Errors
        |
        +---- Identity
        |
        v
Profiles
        |
        +---- Security
        +---- Async
        +---- Streaming
        +---- Distributed
        +---- Hardware
        +---- Reliability

Higher-level specifications MUST NOT redefine lower-level contracts.


---

75. Avoiding Circular Specification Dependencies

A component MUST have enough information to be implemented without waiting for every other component.

For example:

calling-convention.md MAY reference the abstract type model.

It MUST NOT require the final implementation of a specific language binding.

Likewise:

memory-model.md MAY define ownership semantics.

It MUST NOT require a specific garbage collector.


---

76. Integration Contract Principle

Every ULABI specification MUST document its integration points before the dependent implementation is written.

Integration documentation SHOULD identify:

upstream contracts;

downstream consumers;

required schemas;

required interfaces;

required identifiers;

compatibility implications;

tests;

reference modules.


This allows implementation work to proceed file-by-file without requiring retroactive redesign.


---

77. Code Independence

The specification is independent from the reference implementation.

Code MUST implement the specification.

The specification MUST NOT be rewritten merely because one implementation chose a particular internal architecture.

Implementation-specific optimizations MUST remain behind the ULABI boundary.


---

78. Reference Implementation Independence

Reference implementations SHOULD be modular.

A reference implementation SHOULD separate:

identifiers;

type system;

ABI contracts;

memory;

calling conventions;

errors;

capability enforcement;

negotiation;

serialization;

validation;

transport;

observability;

testing.


This makes it possible for independent implementations to replace individual components.


---

79. Testing Principle

ULABI testing MUST occur at multiple levels:

Unit Tests
    |
Integration Tests
    |
Cross-Language Tests
    |
Cross-Platform Tests
    |
Conformance Tests
    |
Fuzz Tests
    |
Security Tests
    |
Failure Tests
    |
Compatibility Tests

A component MUST NOT be considered complete solely because its unit tests pass.


---

80. Cross-Implementation Interoperability

At least two independently structured implementations SHOULD eventually interoperate through the same published ULABI contract.

The goal is to prove that ULABI describes a real interoperability standard rather than one implementation's internal API.


---

81. Compatibility Testing

Compatibility tests SHOULD cover:

old producer/new consumer;

new producer/old consumer;

same-version interoperability;

different compatible versions;

unsupported optional features;

unsupported mandatory features;

malformed inputs;

changed types;

changed layouts;

changed ownership;

changed capabilities.



---

82. Security Testing

Security testing SHOULD include:

unauthorized capability access;

malformed descriptors;

invalid identities;

signature failures;

integrity failures;

privilege escalation;

resource exhaustion;

memory violations;

sandbox escapes;

replay where applicable;

downgrade attacks;

negotiation attacks.



---

83. Resource-Bounded Design

No ULABI component SHOULD assume infinite:

memory;

CPU;

storage;

network bandwidth;

handles;

threads;

execution time.


Operations with potentially unbounded input MUST define resource behavior.

This is particularly important for:

parsers;

decoders;

serializers;

streams;

distributed messages;

type descriptors.



---

84. Safe Failure

When an implementation cannot safely satisfy a contract, it MUST fail according to a defined error model.

It MUST NOT:

guess silently;

reinterpret incompatible data;

bypass security;

violate ownership;

access invalid memory;

invoke unsupported capabilities.



---

85. Predictability

ULABI implementations SHOULD make behavior predictable across:

languages;

operating systems;

architectures;

runtimes;

transports.


Where behavior necessarily differs, the difference MUST be represented by the applicable profile or contract metadata.


---

86. Performance Without Semantic Compromise

ULABI MAY support:

direct calls;

inline calls;

zero-copy;

shared memory;

batching;

vectorization;

accelerator execution;

caching;

transport optimization.


Performance optimization MUST NOT alter the semantic contract.


---

87. Optimization Transparency

An optimization MAY change implementation strategy.

It MUST NOT change observable behavior.

For example:

Copying implementation
        |
        v
Zero-copy implementation

is valid if both preserve:

ownership;

lifetime;

mutability;

contents;

synchronization;

security.



---

88. Future-Proofing

ULABI SHOULD be designed for future technologies without making speculative features mandatory today.

Future technologies MAY be represented through:

extensions;

profiles;

capability discovery;

feature negotiation;

versioned schemas.


The Core SHOULD remain stable even as new execution technologies emerge.


---

89. Universal Boundary Principle

The most important architectural boundary is:

Implementation
      |
      | private
      v
ULABI Contract
      |
      | standardized
      v
ULABI Contract
      |
      | private
      v
Implementation

Everything above and below the contract may differ.

The contract itself must remain precise.


---

90. Architectural Invariants

The following are permanent architectural invariants unless explicitly changed through the ULABI governance process:

1. ULABI remains language neutral.


2. ULABI remains implementation neutral.


3. ULABI remains vendor neutral.


4. ULABI remains platform neutral.


5. ULABI remains architecture neutral.


6. ULABI does not become a programming language.


7. ULABI does not require one universal runtime.


8. ULABI Core remains intentionally minimal.


9. Advanced functionality is implemented through profiles or extensions.


10. Security-sensitive behavior is explicit.


11. Ownership and lifetime are explicit.


12. Compatibility is explicit.


13. Failures are explicit.


14. Unsupported functionality fails safely.


15. Reference implementations do not define the specification.


16. Conformance must be testable.


17. Independent implementations must remain possible.


18. ULABI remains independent from Zamani.


19. ULABI remains independent from Sankofa.


20. No single project or implementation owns the universal contract.




---

91. Architectural Completion Criteria

A ULABI component is not architecturally complete merely because a document exists.

It is complete when:

Principle
   |
   v
Specification
   |
   v
Normative Requirements
   |
   v
Schema
   |
   v
Interface
   |
   v
Implementation
   |
   v
Tests
   |
   v
Conformance

has been established.

For each major component, the project SHOULD be able to answer:

What is the contract?

What data crosses the boundary?

What invariants apply?

What can fail?

What happens on failure?

What security properties apply?

How is compatibility determined?

How is it tested?

How is it implemented?

How does it integrate with the other ULABI components?



---

92. Final Architectural Rule

ULABI exists to make independent systems interoperable without making them identical.

Therefore:

> Standardize the boundary, not the implementation.



And:

> Preserve independence while guaranteeing interoperability.



These two principles govern the design, implementation, testing, evolution, and governance of ULABI.

---

# 2. What I found in the repository

The important point is that the repository already has the **documentation skeleton**, including `core-abi.md`, `calling-convention.md`, `data-types.md`, `memory-model.md`, `register-model.md`, `exception-model.md`, `return-values.md`, and the architecture documents.  

However, `design-principles.md` was still just a placeholder. 

Also, the current normative `ULABI-SPEC.md` already establishes important requirements for profiles, calling conventions, memory ownership, errors, capabilities, versioning, and compatibility. 

So **this design-principles document should be completed before we start creating implementation code**.

---

# 3. Correct implementation order

You asked specifically to start with files that can be completed independently and then integrate them.

I recommend this dependency order.

## Phase 0 — Architectural foundation

These are the files that can be completed independently.

### 1. `docs/architecture/design-principles.md`
**Status after the document above:** complete.

Defines:

- neutrality;
- stability;
- explicit semantics;
- security;
- compatibility;
- extensibility;
- failure handling;
- implementation independence.

**Dependencies:** `ULABI-DESIGN.md`

**Downstream consumers:** essentially every ULABI specification.

---

### 2. `docs/architecture/universal-model.md`

Defines the universal abstract model:

```text
Interface
Function
Type
Value
Resource
Capability
Error
Component
Transport
Profile
Version

This should become the conceptual vocabulary used by the rest of ULABI.


---

3. docs/architecture/overview.md

Defines the complete layer architecture.


---

4. docs/architecture/scalability.md

Defines how the ABI scales from:

embedded
→ mobile
→ desktop
→ server
→ distributed
→ accelerator
→ future systems


---

5. ULABI-VERSIONING.md

Defines version semantics independently from individual ABI components.


---

6. ULABI-GOVERNANCE.md

Defines how the standard itself evolves.

This is important because otherwise the "stable Core" principle has no formal mechanism behind it.


---

4. Core specification files

Once the architectural foundation is fixed:

docs/abi/core-abi.md
docs/abi/data-types.md
docs/abi/calling-convention.md
docs/abi/memory-model.md
docs/abi/register-model.md
docs/abi/stack-model.md
docs/abi/return-values.md
docs/abi/exception-model.md

Dependency relationship

design-principles
        |
        v
universal-model
        |
        v
core-abi
   |
   +---- data-types
   |
   +---- calling-convention
   |          |
   |          +---- register-model
   |          |
   |          +---- stack-model
   |          |
   |          +---- return-values
   |
   +---- memory-model
   |
   +---- exception-model

Each document should be written as a complete contract, not a placeholder.


---

5. Type-system files

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/type-compatibility.md

These should define the semantic type layer rather than copying any individual programming language.


---

6. Interoperability files

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

These connect language implementations to the already-defined Core.


---

7. Runtime files

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

These should consume the Core contracts rather than redefine them.


---

8. Memory files

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

These refine the memory boundary defined by docs/abi/memory-model.md.


---

9. Security files

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Security should depend on the Core, not modify the meaning of Core types or calls.


---

10. Reliability files

docs/reliability/self-healing.md
docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md

These consume the error, lifecycle, resource and security contracts.


---

11. Distributed files

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Distributed ULABI must build on the local contract rather than redefining it.


---

12. Platform files

docs/platforms/operating-systems.md
docs/platforms/architectures.md
docs/platforms/embedded.md
docs/platforms/mobile.md
docs/platforms/desktop.md
docs/platforms/server.md
docs/platforms/cloud.md
docs/platforms/webassembly.md
docs/platforms/accelerators.md

These are adapters/profiles.

They must not contaminate the universal Core.


---

13. Hardware files

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md
docs/hardware/future-architectures.md

Again, these consume the universal contracts.


---

14. Tooling files

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md


---

15. Observability files

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md


---

16. Compatibility files

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

These become particularly important after the Core and type contracts exist.


---

17. Standards files

Finally:

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

These should consume all preceding normative specifications.


---

18. Required schemas

The documentation alone is not enough.

ULABI needs machine-readable schemas.

I recommend:

schemas/
├── abi/
│   ├── interface.schema.json
│   ├── function.schema.json
│   ├── signature.schema.json
│   ├── type.schema.json
│   ├── type-descriptor.schema.json
│   ├── field.schema.json
│   ├── enum.schema.json
│   ├── union.schema.json
│   ├── error.schema.json
│   ├── capability.schema.json
│   ├── effect.schema.json
│   ├── ownership.schema.json
│   ├── lifetime.schema.json
│   └── profile.schema.json
│
├── compatibility/
│   ├── version.schema.json
│   ├── feature.schema.json
│   ├── negotiation.schema.json
│   └── compatibility-result.schema.json
│
├── security/
│   ├── identity.schema.json
│   ├── authorization.schema.json
│   ├── policy.schema.json
│   └── security-profile.schema.json
│
└── diagnostics/
    ├── diagnostic.schema.json
    └── trace.schema.json


---

19. Required reference implementation modules

This is the important part of your request: the names of the actual code files/modules.

Because ULABI must remain language-neutral, these are ULABI module contracts, not a requirement that ULABI itself be implemented in Rust, C++, Sankofa, Zamani, etc.

A reference implementation can later choose a language.

I recommend:

reference/
└── core/
    ├── abi/
    │   ├── interface.rs
    │   ├── function.rs
    │   ├── signature.rs
    │   ├── calling_convention.rs
    │   ├── return_values.rs
    │   ├── register_model.rs
    │   ├── stack_model.rs
    │   └── exception_model.rs
    │
    ├── types/
    │   ├── primitive.rs
    │   ├── composite.rs
    │   ├── struct_type.rs
    │   ├── enum_type.rs
    │   ├── union_type.rs
    │   ├── option.rs
    │   ├── result.rs
    │   ├── string.rs
    │   ├── bytes.rs
    │   └── type_descriptor.rs
    │
    ├── memory/
    │   ├── memory_region.rs
    │   ├── ownership.rs
    │   ├── lifetime.rs
    │   ├── borrowing.rs
    │   ├── allocation.rs
    │   ├── alignment.rs
    │   └── shared_memory.rs
    │
    ├── errors/
    │   ├── error.rs
    │   ├── error_code.rs
    │   ├── error_category.rs
    │   └── diagnostics.rs
    │
    ├── identity/
    │   ├── interface_id.rs
    │   ├── function_id.rs
    │   ├── component_id.rs
    │   └── version.rs
    │
    ├── capabilities/
    │   ├── capability.rs
    │   ├── capability_id.rs
    │   ├── capability_set.rs
    │   ├── policy.rs
    │   └── authorization.rs
    │
    ├── compatibility/
    │   ├── compatibility.rs
    │   ├── feature.rs
    │   ├── negotiation.rs
    │   └── discovery.rs
    │
    ├── encoding/
    │   ├── encoder.rs
    │   ├── decoder.rs
    │   ├── canonical.rs
    │   ├── layout.rs
    │   └── validation.rs
    │
    ├── runtime/
    │   ├── runtime.rs
    │   ├── component.rs
    │   ├── lifecycle.rs
    │   ├── resource_limits.rs
    │   └── execution.rs
    │
    └── lib.rs

These names are deliberately implementation-neutral in meaning even though the example uses .rs for the reference implementation.

If the eventual reference implementation is C++, the same conceptual modules can become .hpp/.cpp; if another organization implements ULABI in Go, they become packages.


---

20. Required adapter modules

ULABI needs a clear adapter boundary.

implementations/
├── adapters/
│   ├── language/
│   │   ├── c/
│   │   ├── cpp/
│   │   ├── rust/
│   │   ├── go/
│   │   ├── python/
│   │   ├── java/
│   │   ├── kotlin/
│   │   ├── swift/
│   │   ├── fortran/
│   │   └── ada/
│   │
│   ├── platform/
│   │   ├── linux/
│   │   ├── windows/
│   │   ├── macos/
│   │   ├── android/
│   │   ├── ios/
│   │   └── wasm/
│   │
│   └── architecture/
│       ├── x86_64/
│       ├── arm64/
│       ├── riscv64/
│       └── other/

Important: these are implementations of ULABI. They are not part of the ULABI specification itself.


---

21. Required test modules

The test architecture should mirror the specification.

tests/
├── abi/
│   ├── interface_tests
│   ├── function_tests
│   ├── calling_convention_tests
│   ├── return_value_tests
│   ├── register_tests
│   ├── stack_tests
│   └── exception_tests
│
├── types/
│   ├── primitive_tests
│   ├── composite_tests
│   ├── struct_tests
│   ├── enum_tests
│   ├── union_tests
│   ├── option_tests
│   ├── result_tests
│   └── descriptor_tests
│
├── memory/
│   ├── ownership_tests
│   ├── lifetime_tests
│   ├── borrowing_tests
│   ├── allocation_tests
│   ├── alignment_tests
│   └── shared_memory_tests
│
├── compatibility/
│   ├── version_tests
│   ├── negotiation_tests
│   ├── backward_compatibility_tests
│   └── forward_compatibility_tests
│
├── security/
│   ├── capability_tests
│   ├── authorization_tests
│   ├── isolation_tests
│   ├── secure_loading_tests
│   └── policy_tests
│
├── errors/
│   ├── error_code_tests
│   ├── error_mapping_tests
│   └── failure_tests
│
├── encoding/
│   ├── encoding_tests
│   ├── decoding_tests
│   ├── canonicalization_tests
│   └── malformed_input_tests
│
├── runtime/
│   ├── lifecycle_tests
│   ├── resource_tests
│   └── execution_tests
│
└── integration/
    ├── cross_language_tests
    ├── cross_platform_tests
    ├── producer_consumer_tests
    └── end_to_end_tests


---

22. Required conformance modules

Separate the normal tests from formal conformance.

conformance/
├── core/
│   ├── interface/
│   ├── types/
│   ├── calls/
│   ├── errors/
│   └── compatibility/
│
├── memory/
│   ├── ownership/
│   ├── lifetime/
│   └── sharing/
│
├── security/
│   ├── capabilities/
│   ├── authorization/
│   └── isolation/
│
├── async/
├── streaming/
├── distributed/
├── hardware/
├── reliability/
└── self-healing/

Each conformance test should have:

Requirement ID
Test ID
Input
Expected Behavior
Expected Error
Security Expectations
Compatibility Expectations
Pass/Fail


---

23. Required examples

examples/
├── hello/
├── cross-language/
├── structs/
├── enums/
├── unions/
├── option/
├── result/
├── memory/
├── ownership/
├── borrowing/
├── callbacks/
├── errors/
├── async/
├── streaming/
├── capabilities/
├── distributed/
└── hardware/

The examples should be normative demonstrations where explicitly marked, otherwise they remain explanatory examples.


---

24. Required implementation-independent identifiers

ULABI should eventually have dedicated modules for:

Interface ID
Function ID
Type ID
Error ID
Capability ID
Profile ID
Feature ID
Component ID
Transport ID

These should not be based on source-language symbol names.


---

25. The final dependency graph

The complete architecture should eventually look like:

ULABI-DESIGN.md
                        |
                        v
             design-principles.md
                        |
                        v
               universal-model.md
                        |
                        v
                    core-abi
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
   data-types     calling-convention   memory-model
       |                |                |
       v          +-----+------+         v
 type-system      |            |      ownership
                  v            v         |
             register       stack        v
               model        model      lifetime
                  |
                  v
            return-values
                  |
                  v
           exception-model
                  |
       +----------+----------+
       |          |          |
       v          v          v
  interoperability runtime security
       |          |          |
       +----------+----------+
                  |
                  v
          compatibility
                  |
       +----------+----------+
       |          |          |
       v          v          v
 distributed  reliability  hardware
       |          |          |
       +----------+----------+
                  |
                  v
              tooling
                  |
                  v
             observability
                  |
                  v
              conformance
                  |
                  v
          reference implementations
                  |
                  v
              adapters
                  |
                  v
               languages


---

26. The most important rule for the remaining work

We should not write the implementation code first.

The correct sequence is:

Architecture
      ↓
Normative specification
      ↓
Schemas
      ↓
Interfaces
      ↓
Reference modules
      ↓
Tests
      ↓
Conformance
      ↓
Language/platform adapters

That gives you exactly the property you asked for:

> When a file is declared complete, another later file should not require us to go back and redesign it.



To achieve that, every document we create from this point should contain its integration contract in advance, including its upstream dependencies, downstream consumers, identifiers, schemas, invariants, failure behavior, compatibility behavior, tests, and reference modules.

For design-principles.md, the implementation work is now blocked only on the architecture/specification layer, which is correct: this document defines rules rather than executable code. The next independent specification should therefore be docs/architecture/universal-model.md, followed by the Core ABI contracts.