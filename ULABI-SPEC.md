I checked Benwellonedge28/ULABI first. The repository currently contains ULABI-DESIGN.md, but it is still only a placeholder rather than the full architecture document we drafted.

So Design Document 2 should be ULABI-SPEC.md. It should turn the architecture into normative rules that implementations can actually conform to.

Copy and paste the following as ULABI-SPEC.md:

# ULABI — Universal Language Application Binary Interface
# Normative Specification

**Document:** ULABI-SPEC.md  
**Project:** ULABI  
**Specification:** Core Normative Specification  
**Version:** 0.1.0  
**Status:** Draft  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the normative requirements of the Universal Language
Application Binary Interface (ULABI).

ULABI provides a language-independent and implementation-independent binary
interoperability contract.

This specification defines the minimum rules required for independently
developed implementations to exchange compatible data, invoke compatible
interfaces, identify components, negotiate capabilities, handle errors, and
maintain binary compatibility.

This document is normative.

The words:

- MUST
- MUST NOT
- REQUIRED
- SHALL
- SHALL NOT
- SHOULD
- SHOULD NOT
- MAY
- OPTIONAL

are used according to their conventional standards meaning.

---

# 2. Relationship to ULABI-DESIGN.md

`ULABI-DESIGN.md` defines the overall architecture and design philosophy.

This document defines normative requirements derived from that architecture.

Where an implementation detail is not explicitly defined by this document,
implementations remain free to choose their internal design.

The specification takes precedence over implementation-specific assumptions.

---

# 3. Fundamental Principle

ULABI defines interoperability contracts.

It does not define:

- a programming language;
- a compiler;
- an operating system;
- a CPU;
- a runtime;
- a garbage collector;
- a particular object model;
- a particular development environment;
- a particular vendor implementation.

An implementation MUST be able to implement ULABI without adopting the internal
semantics of another programming language.

---

# 4. Language Independence

ULABI MUST remain independent of individual programming languages.

A ULABI implementation MAY be written in:

- C;
- C++;
- Rust;
- Go;
- Python;
- Java;
- Kotlin;
- Swift;
- Fortran;
- Ada;
- Zig;
- or another language.

The language used to implement ULABI MUST NOT alter the ULABI binary contract.

---

# 5. Project Independence

ULABI is independent from all individual software projects.

In particular:

- Zamani is independent from ULABI.
- Sankofa is independent from ULABI.

Zamani MAY implement ULABI.

Sankofa MAY implement ULABI.

Neither language defines ULABI.

ULABI MUST remain usable by unrelated languages and projects.

---

# 6. Conformance Model

An implementation is ULABI-conformant only when it satisfies the mandatory
requirements applicable to the ULABI profile it claims to implement.

Conformance MUST be demonstrated through machine-executable tests where
practical.

A project MUST NOT claim complete ULABI compatibility solely because it:

- exposes a similar API;
- uses the ULABI name;
- implements a subset without declaring the subset;
- uses a compatible serialization format;
- or passes informal interoperability tests.

---

# 7. ULABI Profiles

ULABI uses profiles to avoid forcing every environment to implement features
that it cannot support.

A profile consists of:

```text
Core Requirements
+
Required Extensions
+
Optional Extensions
+
Platform Constraints
+
Security Requirements

An implementation MUST identify the profile or profiles it supports.


---

8. Core Profile

The initial ULABI Core Profile requires:

1. Interface identity


2. ABI version identification


3. Primitive type representation


4. Calling convention metadata


5. Argument representation


6. Return-value representation


7. Error representation


8. Alignment rules


9. Data layout rules


10. Ownership metadata


11. Capability discovery


12. Compatibility information




---

9. Interface Identity

Every externally visible ULABI interface MUST have an interface identity.

The identity MUST contain enough information to distinguish the interface from unrelated interfaces.

A logical interface identity consists of:

Namespace
Interface Name
Interface Identifier
Major Version
Minor Version

An implementation MAY include additional identity metadata.


---

10. Interface Identifier

An Interface Identifier MUST be stable across compatible implementations.

An implementation MUST NOT generate a different identifier for the same published interface solely because the interface was implemented in a different programming language.

Interface identifiers SHOULD be generated using a collision-resistant identifier scheme.

The final canonical identifier encoding will be defined by a future stable ULABI specification.


---

11. Versioning

Every published interface MUST have a version.

ULABI uses semantic compatibility concepts based on:

Major
Minor
Patch

A major version change indicates that compatibility MAY be broken.

A minor version change SHOULD preserve compatibility with previous compatible implementations.

A patch version change MUST NOT intentionally change the binary contract.


---

12. ABI Version

Every ULABI binary component MUST expose the ULABI ABI version it targets.

The version information MUST be machine-readable.

A component MUST NOT assume that another component supports an ABI version without verification.


---

13. Capability Discovery

A ULABI component MUST be able to expose its supported capabilities.

Capability information SHOULD include:

ULABI Version
Profile
Interfaces
Interface Versions
Supported Types
Security Features
Runtime Features
Optional Extensions
Resource Constraints


---

14. Feature Negotiation

When two components support different feature sets, they MUST negotiate before using optional functionality.

A component MUST NOT invoke an optional feature that the other side has not advertised as supported.

Unknown optional features SHOULD be safely ignored.

Unknown mandatory features MUST cause safe incompatibility handling.


---

15. Primitive Types

ULABI defines a language-neutral primitive type model.

The initial primitive types are:

Bool
I8
I16
I32
I64
I128
U8
U16
U32
U64
U128
F32
F64
Char
Byte

Additional primitive types MAY be defined by future specifications.


---

16. Integer Representation

Signed and unsigned integers MUST have explicitly defined widths.

An implementation MUST NOT assume that a source-language integer type maps directly to a ULABI integer.

For example:

Language Integer
        ↓
ULABI I64

must be an explicit mapping.


---

17. Floating-Point Representation

Floating-point types MUST identify their width.

The initial types are:

F32
F64

Floating-point interoperability MUST define:

width;

representation;

byte order;

special values;

NaN handling;

infinity handling.



---

18. Boolean Representation

ULABI Boolean values MUST have exactly two semantic states:

false
true

The binary encoding of Boolean values MUST be defined by the applicable ULABI binary representation.

Implementations MUST NOT treat arbitrary non-zero values as equivalent to true across an ABI boundary unless the applicable representation explicitly allows it.


---

19. Character Representation

ULABI character interoperability MUST define the character encoding.

Character values MUST NOT depend on a language-specific internal character representation.

Text strings and individual character values MUST be distinguishable.


---

20. Byte Representation

A ULABI byte represents exactly eight binary bits.

A byte MUST have a value range of:

0..255


---

21. Composite Types

ULABI supports composite data types including:

Array
Vector
Struct
Union
Enum
Optional
Result
Map
String
Buffer

Composite types MUST contain sufficient metadata to determine their binary layout.


---

22. Structs

A ULABI struct consists of an ordered collection of fields.

Each field MUST define:

Field Identifier
Field Type
Offset or Layout Rule
Alignment Requirement
Optionality
Ownership

Struct layout MUST be deterministic for a given ULABI ABI profile.


---

23. Unions

A ULABI union represents one of multiple possible data variants.

A union MUST include a discriminant or equivalent mechanism identifying the active variant.

An implementation MUST NOT read a union variant that is not active.


---

24. Enums

ULABI enums consist of named variants with explicitly defined values.

The representation width MUST be known.

An implementation MUST NOT assume that a source-language enum has the same representation as a ULABI enum.


---

25. Optional Values

ULABI supports optional values.

An optional value MUST distinguish:

Present
Absent

The representation MUST make the state unambiguous.


---

26. Result Values

ULABI supports explicit result values for operations that may succeed or fail.

A result MUST distinguish:

Success
Failure

A result SHOULD contain structured machine-readable error information when the operation fails.


---

27. Strings

ULABI strings MUST explicitly define their encoding and length.

A string MUST NOT depend on a terminating zero byte unless the specific ABI representation explicitly specifies such behaviour.

Strings SHOULD carry explicit length information.


---

28. Buffers

Binary buffers MUST expose sufficient metadata to establish:

Address or Handle
Length
Capacity where applicable
Ownership
Mutability
Lifetime
Alignment


---

29. Arrays

ULABI arrays MUST have a determinable element type.

An array representation MUST allow the receiver to determine its length.

An implementation MUST NOT assume that an array is null-terminated.


---

30. Memory Model

ULABI memory interfaces MUST explicitly define ownership and lifetime.

The following concepts are supported:

Owned
Borrowed
Shared
Immutable
Mutable
Transferred

An implementation MUST NOT free memory it does not own.


---

31. Ownership Transfer

When ownership is transferred across a ULABI boundary, the transfer MUST be explicit.

After ownership transfer:

Original Owner
      ↓
No Ownership

and:

Receiving Component
      ↓
New Owner

The original component MUST NOT perform operations that violate the new ownership state.


---

32. Borrowed Memory

Borrowed memory remains owned by another component.

A borrower MUST NOT:

free the memory;

retain it beyond its permitted lifetime;

mutate it unless mutation is explicitly permitted.



---

33. Mutable Memory

Mutable memory MUST explicitly identify whether mutation is permitted.

An implementation MUST NOT infer write permission solely from the existence of a memory address.


---

34. Memory Lifetime

Every borrowed or shared memory region MUST have a defined lifetime model.

A component MUST NOT retain a reference after its lifetime has expired.


---

35. Alignment

ULABI data structures MUST define required alignment.

An implementation MUST NOT access data using an alignment assumption that is not guaranteed by the ABI contract.


---

36. Byte Order

Multi-byte binary representations MUST specify byte order where the format requires it.

ULABI implementations MUST NOT assume that two systems use the same native byte order unless the interface contract guarantees it.


---

37. Calling Convention

Every callable ULABI function MUST have a defined calling convention.

The calling convention MUST specify:

Argument Order
Argument Representation
Return Representation
Register Usage
Stack Usage
Memory Arguments
Alignment
Error Handling
Variadic Behaviour where applicable


---

38. Argument Passing

Arguments MUST be passed according to the declared ULABI calling convention.

The caller and callee MUST agree on:

argument count;

argument types;

argument layout;

ownership;

lifetime;

mutability.



---

39. Return Values

Return values MUST use an explicitly defined ULABI representation.

Large return values MAY be returned indirectly through memory or handles.

The mechanism MUST be defined by the applicable calling convention.


---

40. Function Identity

Every exported function MUST have a stable identifier.

The identifier MUST NOT depend solely on source-language name mangling.

A ULABI function identity SHOULD include:

Interface ID
Function ID
Version
Signature


---

41. Function Signatures

A function signature MUST define:

Function Identifier
Arguments
Return Type
Error Model
Capability Requirements
Ownership Requirements

Optional metadata MAY include:

Async
Pure
Deterministic
Idempotent
Thread Safe
Cancellation Support


---

42. Variadic Functions

Variadic functions are OPTIONAL.

A ULABI implementation MUST NOT expose a variadic function without explicitly declaring its ABI representation.

Language-specific variadic mechanisms MUST NOT be assumed to be interoperable.


---

43. Error Model

ULABI requires structured errors.

An error SHOULD contain:

Error Code
Error Category
Severity
Origin
Retryability
Context
Machine-readable Data

A human-readable message MAY be included.

Human-readable text MUST NOT be the only error information relied upon by programmatic consumers.


---

44. Error Categories

Initial error categories include:

InvalidArgument
TypeMismatch
NotFound
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
Unsupported
Unknown

Future specifications may extend this list.


---

45. Error Stability

Error codes intended for programmatic handling MUST remain stable across compatible interface versions.

Human-readable messages MAY change.

Programs SHOULD use error codes rather than matching error messages.


---

46. Exceptions

ULABI does not mandate language-level exceptions.

A language implementation MAY map ULABI errors to:

exceptions;

error values;

result types;

status codes;

other mechanisms.


The binary contract MUST remain compatible.


---

47. Asynchronous Operations

An asynchronous ULABI operation MUST explicitly identify itself as asynchronous.

An asynchronous interface MUST define:

Completion
Failure
Cancellation
Resource Ownership
Lifetime


---

48. Cancellation

If an operation supports cancellation, the interface MUST define cancellation semantics.

Cancellation MUST NOT automatically imply rollback unless rollback is explicitly defined.


---

49. Streams

ULABI streams MUST define:

Element Type
Direction
Completion
Failure
Cancellation
Backpressure where applicable
Ownership


---

50. Thread Safety

Thread-safety properties SHOULD be declared in interface metadata.

An implementation MUST NOT claim thread safety unless concurrent use is actually supported by the contract.


---

51. Reentrancy

Interfaces that support reentrant invocation SHOULD explicitly declare it.

Implementations MUST NOT assume that a callback cannot re-enter a component unless the contract prohibits reentrancy.


---

52. Concurrency

ULABI does not mandate one concurrency model.

Implementations MAY use:

threads;

tasks;

actors;

event loops;

message passing;

asynchronous execution.


The exposed ABI semantics MUST remain deterministic enough for compatible consumers.


---

53. Capability Model

Security-sensitive operations SHOULD use explicit capabilities.

A capability represents authorized access to a resource or operation.

Examples include:

FileRead
FileWrite
Network
GPU
Device
Process
Secret
CryptographicKey
Storage


---

54. Capability Properties

Capabilities SHOULD be:

explicit;

scoped;

least-privilege;

auditable;

revocable where possible.


A component MUST NOT obtain unrestricted access merely because it is loaded through ULABI.


---

55. Security Boundary

A ULABI boundary MUST NOT automatically imply trust.

Before accessing a protected resource, the implementation SHOULD perform:

Identity Verification
       ↓
Capability Check
       ↓
Policy Check
       ↓
Operation


---

56. Secure Loading

A ULABI loader SHOULD verify:

Component Identity
Integrity
Signature where required
ULABI Compatibility
Required Capabilities
Security Policy
Dependencies

An invalid component MUST be rejected according to the applicable security profile.


---

57. Component Lifecycle

A component MAY transition through:

Discovered
Verified
Loaded
Initialized
Ready
Running
Paused
Degraded
Recovering
Stopped
Unloaded
Failed

Invalid lifecycle transitions MUST be rejected.


---

58. Resource Limits

Implementations SHOULD support explicit resource limits.

Possible limits include:

Memory
CPU
Threads
Storage
Network
Execution Time
Handles
Devices

Resource exhaustion MUST result in a defined failure.


---

59. Distributed Interoperability

Distributed ULABI is an extension of the local ABI model.

A distributed call MUST NOT be treated as equivalent to a local call with respect to:

latency;

failure;

serialization;

availability;

security.


The distinction MUST be explicit.


---

60. Serialization

Serialized ULABI data MUST contain sufficient information for the receiving side to determine:

Type
Version
Length
Encoding
Integrity where required

Unknown mandatory fields MUST be handled safely.

Unknown optional fields SHOULD be ignored.


---

61. Remote Calls

Remote ULABI calls MUST define:

Request
Response
Timeout
Failure
Authentication
Authorization
Serialization
Cancellation where supported


---

62. Compatibility

Compatibility MUST be evaluated at multiple levels.

These include:

ABI Compatibility
Type Compatibility
Interface Compatibility
Version Compatibility
Security Compatibility
Semantic Compatibility

An implementation MUST NOT claim full compatibility when only one level is supported.


---

63. Backward Compatibility

A newer implementation SHOULD support older compatible interface versions.

Removing a mandatory field, changing its binary representation, or changing its ownership semantics is a potentially breaking change.

Breaking changes MUST use an explicit major-version boundary.


---

64. Forward Compatibility

Older implementations SHOULD safely process future extensions when possible.

Unknown optional fields MUST NOT cause failure merely because they are unknown.

Unknown mandatory requirements MUST result in explicit incompatibility.


---

65. Graceful Degradation

An implementation MAY operate with a reduced feature set.

Reduced functionality MUST be discoverable.

An implementation MUST NOT silently claim capabilities it cannot provide.


---

66. Introspection

Authorized tooling SHOULD be able to inspect:

Interfaces
Types
Versions
Capabilities
Dependencies
Security Requirements
Component State

Introspection MUST obey access-control policies.


---

67. Diagnostics

ULABI implementations SHOULD expose structured diagnostics.

Diagnostics may include:

Component
Interface
Function
Error
Timestamp
Execution Context
Dependency
Recovery State

Sensitive information MUST be protected according to the security policy.


---

68. Observability

Implementations MAY expose:

Metrics
Logs
Traces
Events
Health Information

Observability information MUST NOT alter ABI semantics.


---

69. Self-Healing

Self-healing is an optional ULABI reliability extension.

Self-healing MUST be based on explicit policies.

The required conceptual state machine is:

HEALTHY
   |
   v
FAILURE_DETECTED
   |
   v
EVIDENCE_COLLECTION
   |
   v
DIAGNOSIS
   |
   v
POLICY_CHECK
   |
   +---- no approved recovery ----> ESCALATED
   |
   v
ISOLATION
   |
   v
RECOVERY
   |
   v
VERIFICATION
   |
   +---- healthy ----> HEALTHY
   |
   v
ROLLBACK
   |
   v
ESCALATED


---

70. Self-Healing Restrictions

Self-healing MUST NOT:

bypass authorization;

disable security controls;

grant new privileges without policy authorization;

modify unrelated components;

remove audit information;

exceed resource limits;

repeatedly retry without a defined limit;

silently alter the ABI contract.



---

71. Recovery Policies

Recovery policies MAY define:

Maximum Attempts
Retry Delay
Allowed Recovery Actions
Resource Limits
Rollback Strategy
Escalation Target
Verification Requirements


---

72. Recovery Verification

A recovery action MUST NOT be considered successful merely because the operation returned without an immediate error.

The implementation SHOULD verify relevant health conditions.

Example:

Recovery
   ↓
Restart Component
   ↓
Health Check
   ↓
Interface Check
   ↓
Security Check
   ↓
Healthy


---

73. Rollback

Where recovery changes component state, rollback SHOULD be supported when practical.

Rollback MUST restore the last known valid state or transition to a safe failure state.


---

74. Fault Isolation

A failure in one component SHOULD remain within its defined failure domain.

Isolation MAY occur at:

Thread
Process
Component
Memory Region
Capability
Machine
Service


---

75. Resource Isolation

A component MUST NOT consume unlimited resources merely because it is ULABI-compatible.

Resource limits SHOULD be enforced independently of component language.


---

76. Security and Self-Healing

Self-healing actions MUST be evaluated against security policy.

If a recovery action weakens the security boundary, it MUST be rejected unless explicitly authorized.


---

77. Determinism

ULABI operations SHOULD be deterministic where practical.

When an operation is inherently nondeterministic, the nondeterministic properties SHOULD be declared.


---

78. Idempotency

An interface MAY declare an operation idempotent.

An implementation MUST NOT assume idempotency unless it is explicitly declared.

This is particularly important for retries and self-healing.


---

79. Transactional Operations

Operations that support transactions MUST define:

Prepare
Execute
Commit
Rollback

Transactional semantics MUST NOT be assumed for ordinary functions.


---

80. Hardware Independence

ULABI MUST NOT require a specific processor architecture.

Architecture-specific mappings MUST exist outside the universal core.


---

81. Platform Independence

ULABI MUST NOT require a specific operating system.

Operating-system functionality SHOULD be represented through optional capabilities.


---

82. Hardware Acceleration

Hardware acceleration is optional.

An implementation MAY expose:

GPU
NPU
FPGA
DSP
SIMD
Cryptographic Accelerator
Other Accelerator

Applications MUST be able to discover whether the capability exists.


---

83. Embedded Systems

A ULABI implementation MAY operate without a traditional operating system.

The core ABI MUST remain implementable in constrained environments.

Optional functionality MAY be omitted when the relevant profile permits it.


---

84. WebAssembly

WebAssembly MAY be used as a ULABI execution environment.

ULABI MUST NOT depend on WebAssembly.

WebAssembly is one possible implementation target.


---

85. Quantum Systems

Quantum interoperability is an optional future extension.

ULABI Core MUST NOT require quantum hardware.

Quantum extensions SHOULD define:

Quantum Resource
Quantum Job
Measurement
Classical/Quantum Boundary
Execution State
Result


---

86. Cryptographic Agility

ULABI security interfaces MUST support algorithm replacement.

The core specification MUST NOT permanently depend on a single cryptographic algorithm.

Implementations SHOULD support cryptographic version negotiation.


---

87. Post-Quantum Cryptography

Implementations SHOULD be capable of adopting post-quantum cryptographic algorithms without changing unrelated ABI interfaces.


---

88. Supply-Chain Metadata

A secure ULABI component SHOULD expose:

Component Identifier
Version
Publisher
Hash
Dependencies
Build Information
Signature
Provenance


---

89. Dependency Validation

Dependencies MUST be validated before a component is considered ready.

A missing mandatory dependency MUST cause loading or initialization to fail safely.


---

90. ABI Metadata

A ULABI component MUST expose enough metadata to determine:

ABI Version
Profile
Architecture Mapping
Interfaces
Types
Capabilities
Dependencies
Security Requirements


---

91. Binary Stability

Once an interface version is declared stable, its binary layout MUST NOT change incompatibly within that version.

Any incompatible binary change requires a new compatibility boundary.


---

92. Undefined Behaviour

ULABI implementations MUST minimize undefined behaviour at interoperability boundaries.

Invalid input SHOULD produce a defined error rather than causing arbitrary memory access or uncontrolled execution.


---

93. Validation

A component SHOULD validate external ABI data before use.

Validation SHOULD include:

Lengths
Offsets
Types
Versions
Alignment
Ownership
Capabilities
Security Metadata


---

94. Integer Safety

Implementations MUST guard against:

integer overflow;

integer underflow;

truncation;

invalid conversions;

length calculation overflow.


This is especially important when processing externally supplied binary data.


---

95. Bounds Safety

Memory accesses MUST respect declared bounds.

A malformed length or offset MUST NOT permit an implementation to access memory outside the permitted region.


---

96. Fuzzing Requirements

ULABI parsers, loaders, metadata processors, serializers, and protocol implementations SHOULD be fuzz-tested.

Security-critical implementations SHOULD use continuous fuzzing.


---

97. Conformance Testing

A ULABI conformance suite SHOULD test:

Interface Identity
Versioning
Types
Layouts
Calling Conventions
Memory
Ownership
Errors
Capabilities
Security
Serialization
Compatibility
Runtime
Async
Self-Healing
Distributed Behaviour


---

98. Cross-Language Testing

Conformance testing SHOULD include implementations written in multiple programming languages.

For example:

Implementation A
        ↕
      ULABI
        ↕
Implementation B

The implementations should be independently developed where practical.


---

99. Cross-Platform Testing

Conformance testing SHOULD cover multiple:

operating systems;

processor architectures;

runtimes;

execution environments.



---

100. Reference Implementation

A reference implementation MAY be provided.

The reference implementation is informative unless explicitly incorporated into a normative specification.

The reference implementation MUST NOT redefine the specification.


---

101. Independent Implementations

ULABI SHOULD encourage multiple independent implementations.

A mature specification SHOULD demonstrate interoperability among at least two independent implementations before being considered stable.


---

102. Certification

Certification is separate from basic implementation.

A future certification program MAY evaluate:

Core Compliance
Type Compliance
Memory Compliance
Security Compliance
Runtime Compliance
Reliability Compliance
Distributed Compliance
Extension Compliance

Certification requirements will be defined in:

docs/standards/certification.md


---

103. Compliance Declaration

An implementation claiming ULABI support SHOULD publish a machine-readable compliance declaration.

Example:

ULABI Core       ✓
ULABI Types      ✓
ULABI Memory     ✓
ULABI FFI        ✓
ULABI Security   ✓
ULABI Async      ✓
ULABI Self-Heal  -
ULABI Distributed -

A dash indicates that the capability is not implemented.


---

104. Partial Implementations

Partial implementations are permitted.

They MUST clearly identify unsupported features.

A partial implementation MUST NOT represent unsupported mandatory features as supported.


---

105. Experimental Extensions

Experimental extensions MUST be clearly marked.

Experimental features MUST NOT be treated as stable compatibility guarantees.


---

106. Vendor Extensions

Vendors MAY create extensions.

Vendor extensions MUST:

use a vendor-specific namespace;

identify their version;

document compatibility;

avoid silently changing core ULABI semantics.


A vendor extension MUST NOT claim to be part of ULABI Core without approval through the project's governance process.


---

107. Core Stability

The ULABI Core should remain intentionally small.

New features SHOULD first be developed as extensions.

A feature should enter Core only after demonstrating:

Generality
Implementability
Security
Interoperability
Testing
Stability


---

108. Specification Evolution

ULABI evolves through versioned specifications.

Changes SHOULD follow:

Proposal
   ↓
Design Review
   ↓
Prototype
   ↓
Interoperability Testing
   ↓
Security Review
   ↓
Conformance Tests
   ↓
Specification Revision


---

109. Breaking Changes

Breaking changes MUST be explicitly documented.

A breaking change SHOULD require a major specification version.

Implementations MUST NOT silently reinterpret existing binary contracts.


---

110. Deprecation

Deprecated functionality MUST be documented.

Deprecation SHOULD include:

Deprecated Feature
Reason
Replacement
Migration Path
Removal Timeline


---

111. Security Disclosure

Security vulnerabilities affecting ULABI implementations SHOULD be reported through the project's documented security process.

Security fixes SHOULD be coordinated without unnecessarily exposing exploitation details before fixes are available.


---

112. Privacy

ULABI MUST NOT require unnecessary collection of personal information.

Implementations SHOULD minimize:

telemetry;

identifying metadata;

unnecessary logs;

user information.



---

113. Auditability

Security-sensitive ULABI operations SHOULD be auditable.

Audit records SHOULD identify:

Component
Operation
Capability
Result
Security Decision
Recovery Action where applicable

Audit mechanisms MUST respect privacy requirements.


---

114. Resource-Constrained Profiles

ULABI profiles MAY remove optional functionality for:

microcontrollers;

embedded devices;

real-time systems;

low-memory environments.


The Core contract must remain internally consistent.


---

115. Real-Time Profiles

A future real-time profile MAY define:

Deadline
Priority
Latency Bound
Resource Reservation
Deterministic Scheduling

Real-time requirements MUST NOT be imposed on non-real-time profiles.


---

116. Distributed Profiles

A distributed profile MAY add:

Remote Calls
Serialization
Discovery
Authentication
Network Failure
Timeouts
Retries
Distributed Recovery

Distributed features remain optional.


---

117. AI and Autonomous Systems

ULABI does not grant autonomous authority to AI systems.

An AI-powered implementation remains subject to:

Capabilities
Authorization
Resource Limits
Isolation
Security Policy
Auditability
Recovery Policy

AI-generated decisions MUST NOT automatically override ULABI security rules.


---

118. Formal Verification

Critical implementations SHOULD support formal verification.

Potential verification targets include:

Type Layout
Memory Safety
Ownership
Capability Rules
State Machines
Serialization
Compatibility
Recovery


---

119. Formal Invariants

A conformant implementation MUST preserve the following fundamental properties:

Invariant 1

An invalid memory access MUST NOT become valid merely because the request crossed a ULABI boundary.

Invariant 2

An unowned memory region MUST NOT be freed by a component.

Invariant 3

An unsupported mandatory capability MUST NOT be treated as supported.

Invariant 4

An incompatible ABI version MUST NOT be silently interpreted as compatible.

Invariant 5

A failed recovery action MUST NOT be reported as successful without required verification.

Invariant 6

A self-healing mechanism MUST NOT bypass security policy.

Invariant 7

A ULABI implementation MUST NOT depend on the identity of a particular programming language.


---

120. Minimum Interoperability Guarantee

Two implementations claiming the same ULABI Core profile and compatible interface version MUST agree on:

Interface Identity
Function Identity
Type Identity
Type Layout
Calling Convention
Argument Representation
Return Representation
Error Representation
Ownership Rules
Version Rules


---

121. Universal Interoperability Test

A ULABI implementation should ultimately be able to demonstrate:

Language A
    │
    ▼
Compiler A
    │
    ▼
ULABI
    │
    ▼
Runtime B
    │
    ▼
Language B

without requiring Language A and Language B to share their internal implementation.


---

122. Example Interface

Conceptual ULABI interface:

interface Calculator {

    add(
        I64 left,
        I64 right
    ) -> Result<I64>;

    subtract(
        I64 left,
        I64 right
    ) -> Result<I64>;
}

The source-language representation may differ.

The ULABI contract remains constant.


---

123. Example Ownership Contract

function create_buffer(
    U64 size
) -> Owned<Buffer>;

The returned buffer is owned by the caller.

The caller becomes responsible for releasing or transferring the buffer according to the interface contract.


---

124. Example Borrowed Contract

function inspect_buffer(
    Borrowed<Buffer> buffer
) -> Result<Inspection>;

The callee may inspect the buffer but does not own it.

The callee MUST NOT retain the buffer after the permitted lifetime.


---

125. Example Capability Contract

function read_resource(
    Capability<FileRead> capability,
    ResourceID resource
) -> Result<Buffer>;

The operation requires an explicit capability.

Possession of the function alone does not grant resource access.


---

126. Example Feature Negotiation

Client:
    ULABI Core
    Types
    Async
    Security

Server:
    ULABI Core
    Types
    Async
    Security
    Distributed

Negotiated:
    Core
    Types
    Async
    Security

The client MUST NOT invoke Distributed functionality unless it discovers and accepts that capability.


---

127. Example Self-Healing

Component Failure
       ↓
Health Monitor
       ↓
Diagnosis
       ↓
Known Recovery Policy
       ↓
Restart
       ↓
Verification
       ↓
Healthy

If verification fails:

Verification Failure
       ↓
Rollback
       ↓
Escalation


---

128. Implementation Freedom

ULABI defines the externally observable contract.

An implementation MAY use any internal architecture provided that externally observable behaviour remains compliant.

For example, an implementation MAY internally use:

Garbage Collection
Reference Counting
Ownership Checking
Manual Allocation
Virtual Machines
JIT Compilation
Ahead-of-Time Compilation
Interpretation
Native Code
Bytecode


---

129. No Hidden Language Dependency

A ULABI specification MUST NOT require knowledge of a source language's:

syntax;

compiler;

type checker;

garbage collector;

object model;

module system;

exception implementation;

package manager.


Only the published ULABI contract matters at the boundary.


---

130. No Hidden Platform Dependency

A ULABI Core interface MUST NOT silently depend on:

one operating system;

one CPU;

one ABI vendor;

one runtime;

one hardware platform.


Platform-specific dependencies MUST be declared.


---

131. Universal Boundary Principle

The ULABI boundary is the stable interoperability layer:

┌──────────────────────┐
│ Implementation A    │
└──────────┬───────────┘
           │
           ▼
╔══════════════════════╗
║       ULABI          ║
║ Universal Contract   ║
╚══════════════════════╝
           │
           ▼
┌──────────────────────┐
│ Implementation B    │
└──────────────────────┘


---

132. Specification Authority

ULABI specifications define the official contract.

Reference implementations, examples, libraries, and tools are subordinate to the specification.

If an implementation disagrees with the normative specification, the implementation is non-conformant unless the specification itself is revised.


---

133. Future Extensions

ULABI is intentionally extensible.

Future specifications may define:

advanced distributed systems;

advanced hardware;

quantum systems;

AI accelerators;

secure enclaves;

confidential computing;

advanced real-time profiles;

formal verification metadata;

deterministic replay;

advanced self-healing;

advanced zero-copy systems.


Future extensions MUST preserve Core compatibility unless explicitly versioned otherwise.


---

134. Implementation Roadmap

The first implementation should proceed in this order:

1. Interface Identity
        ↓
2. Primitive Types
        ↓
3. Structs and Composite Types
        ↓
4. Calling Convention
        ↓
5. Memory Ownership
        ↓
6. Error Model
        ↓
7. Versioning
        ↓
8. Capability Discovery
        ↓
9. FFI Prototype
        ↓
10. Cross-Language Tests
        ↓
11. Conformance Suite

Advanced features should be implemented after the Core is experimentally validated.


---

135. Conformance Levels

The following preliminary levels are defined:

Level 0 — Metadata

Supports:

identity;

version;

profile;

capabilities.


Level 1 — Core ABI

Adds:

calling convention;

primitive types;

functions;

errors.


Level 2 — Data Interoperability

Adds:

composite types;

type descriptors;

memory ownership.


Level 3 — Runtime Interoperability

Adds:

async;

streams;

concurrency;

lifecycle.


Level 4 — Security

Adds:

capabilities;

secure loading;

authorization;

integrity.


Level 5 — Reliability

Adds:

health monitoring;

fault detection;

recovery;

self-healing.


Level 6 — Distributed

Adds:

remote calls;

serialization;

discovery;

distributed failure handling.


These levels are preliminary and may be revised before ULABI 1.0.


---

136. Compatibility Matrix

Implementations SHOULD publish a matrix such as:

Capability	Supported	Version

Core ABI	Yes	0.1
Types	Yes	0.1
Memory	Yes	0.1
FFI	Yes	0.1
Async	No	—
Security	Yes	0.1
Self-Healing	No	—
Distributed	No	—



---

137. Required Documentation

Every conformant implementation SHOULD document:

Supported Profiles
Supported Extensions
Supported Architectures
Supported Platforms
Memory Model
Calling Convention
Security Model
Known Limitations
Conformance Results


---

138. Testing Philosophy

ULABI testing must test interoperability rather than merely implementation internals.

The primary question is:

> Can independently implemented components exchange data and invoke interfaces correctly according to the ULABI contract?




---

139. Interoperability Matrix

The conformance project should eventually test:

Implementation A
                     ↕
             Implementation B
                     ↕
             Implementation C

and:

Language A ↔ Language B
Language A ↔ Language C
Language B ↔ Language C

across supported platforms.


---

140. Final Normative Principle

The fundamental rule of ULABI is:

> If two independently implemented components conform to the same applicable ULABI contract and compatible interface version, they MUST be able to interoperate according to that contract without requiring either component to adopt the other's internal programming language or implementation model.




---

141. Final Architectural Separation

ULABI remains independent from every language and project.

ULABI
                   │
       ┌───────────┼───────────┐
       │           │           │
    Language A  Zamani      Sankofa
       │           │           │
       └───────────┼───────────┘
                   │
            Independent
            Implementations

Zamani and Sankofa are peers in the ecosystem, not dependencies of the standard.


---

142. Status

This specification is currently:

DRAFT

It defines the initial normative direction for ULABI 0.1.

Before ULABI 1.0, the following must be made substantially more precise:

binary encoding;

canonical ABI layout;

calling convention;

type descriptors;

interface identifiers;

serialization;

memory ownership representation;

capability encoding;

conformance tests;

security requirements.


Those details MUST be finalized through implementation and interoperability testing rather than being invented solely on paper.


---

143. Next Specifications

After this document, the priority specifications are:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md

docs/runtime/runtime-interface.md

docs/security/security-model.md

docs/reliability/self-healing.md

docs/compatibility/feature-negotiation.md

docs/standards/conformance.md
docs/standards/test-suite.md

These documents will progressively turn the ULABI architecture into an implementable universal ABI.


---

END OF ULABI-SPEC.md

One important correction from the repository check: **your GitHub `ULABI-DESIGN.md` is currently only a placeholder**, so before we build the component documents, the full master document we drafted should actually be committed there. 

I would make **Document 3 `docs/abi/core-abi.md`** next, because that is where we should stop speaking mainly architecturally and start defining the actual binary contract.