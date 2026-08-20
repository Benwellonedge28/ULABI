Yes. I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI first. The current architecture explicitly establishes ULABI as language-, runtime-, OS-, CPU-, and vendor-neutral, with a minimal stable Core + profiles architecture. It also explicitly keeps Zamani and Sankofa separate; neither is the model for ULABI.

Below is the complete document for:

docs/abi/register-model.md

I have also included the implementation file/module plan and integration contracts in advance, so this document can be completed without requiring the register-model implementation to be redesigned later when calling conventions, memory, types, security, hardware, or distributed components are added.


---

docs/abi/register-model.md

# ULABI Register Model

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** Benwellonedge28/ULABI  
**Document:** `docs/abi/register-model.md`  
**Specification Area:** ABI / Register Model  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the ULABI Register Model.

The ULABI Register Model specifies how registers are represented at the ULABI interoperability boundary without requiring all programming languages, compilers, runtimes, operating systems, processors, or hardware platforms to expose identical physical registers.

The purpose of this model is to provide a stable abstraction for:

- function arguments;
- return values;
- temporary values;
- preserved values;
- pointers;
- handles;
- capability references;
- scalar values;
- floating-point values;
- vector values;
- aggregate values;
- execution state;
- calling conventions;
- context switching;
- debugging;
- exception handling;
- asynchronous execution;
- accelerator execution;
- heterogeneous execution;
- register spilling;
- register virtualization.

ULABI defines the contract.

A concrete implementation maps the ULABI register model onto the physical resources available on the target platform.

---

# 2. Relationship to ULABI-DESIGN.md

This document is subordinate to:

`/ULABI-DESIGN.md`

The master architecture establishes that ULABI is:

- language neutral;
- runtime neutral;
- operating-system neutral;
- CPU-architecture neutral;
- vendor neutral;
- implementation independent;
- based on a minimal stable Core;
- extensible through profiles;
- explicitly versioned;
- designed for interoperability rather than replacement of existing ABIs.

The register model therefore MUST NOT assume:

- x86 registers;
- ARM registers;
- RISC-V registers;
- PowerPC registers;
- MIPS registers;
- SPARC registers;
- a particular register count;
- a particular word size;
- a particular instruction set;
- a particular compiler;
- a particular operating system;
- a particular language runtime.

ULABI implementations may map the abstract register model onto any suitable target architecture.

---

# 3. Fundamental Principle

ULABI distinguishes three levels:

```text
ULABI Register
      |
      v
Implementation Register
      |
      v
Physical Hardware Register

The ULABI Register is a semantic ABI concept.

An implementation register is an implementation-level representation.

A physical register is hardware-specific.

ULABI MUST NOT expose hardware-specific register identities as part of the portable Core ABI.


---

4. Goals

The register model MUST support:

1. Language-independent calling conventions.


2. Architecture-independent ABI definitions.


3. Scalar argument passing.


4. Aggregate argument passing.


5. Return values.


6. Temporary values.


7. Preserved values.


8. Pointer values.


9. Handles.


10. Capability references.


11. Floating-point values.


12. Vector values.


13. Optional accelerator values.


14. Register spilling.


15. Register restoration.


16. Nested calls.


17. Recursion.


18. Exceptions.


19. Asynchronous execution.


20. Context switching.


21. Debugging.


22. Deterministic ABI behavior.


23. Security validation.


24. Compatibility negotiation.


25. Hardware-specific optimization without changing the ULABI contract.


26. Future CPU architectures.


27. Future execution architectures.


28. Heterogeneous execution.


29. Sandboxed execution.


30. Out-of-process execution where registers are represented by serialized call state.




---

5. Non-Goals

The ULABI Register Model does NOT attempt to:

define a CPU instruction set;

define machine instructions;

replace physical CPU registers;

require a universal number of registers;

require register-based execution;

require stack-based execution;

require a specific compiler;

require a specific operating system;

require a specific runtime;

require register allocation algorithms;

expose proprietary hardware;

require every value to be passed through registers.


An implementation MAY use:

registers;

stack memory;

shared memory;

message buffers;

heap objects;

indirect references;

hardware-specific mechanisms.


The implementation must nevertheless satisfy the ULABI contract.


---

6. Register Classes

ULABI defines semantic register classes.

The initial classes are:

General
Integer
Pointer
Handle
Capability
FloatingPoint
Vector
Predicate
Status
Control
Argument
Return
Temporary
Preserved
Special
Extension

These classes describe semantic roles rather than physical hardware.


---

7. General Registers

General registers represent values that do not require a more specialized semantic class.

Examples include:

integer values
boolean values
enumerations
small records
opaque values
implementation-neutral scalar values

A target implementation MAY map general registers onto:

integer registers;

general-purpose registers;

virtual registers;

stack slots.



---

8. Integer Registers

Integer registers represent:

signed integers;

unsigned integers;

integer identifiers;

lengths;

offsets;

indexes;

counts.


ULABI integer values MUST have explicit:

signedness;

width;

representation;

range;

overflow semantics.


The ULABI contract MUST NOT assume that the host machine's native integer size is portable.


---

9. Pointer Registers

Pointer registers represent references to memory.

A ULABI pointer MUST NOT automatically imply unrestricted access.

A pointer contract MUST specify:

target object;

access permissions;

mutability;

ownership;

lifetime;

bounds;

alignment;

address-space identity where applicable.


Example:

Pointer {
    address
    bounds
    permissions
    lifetime
}

A concrete implementation may represent this using:

a native pointer;

a fat pointer;

a handle;

an index;

a capability;

a managed reference.



---

10. Handle Registers

Handles represent opaque resources.

Examples:

FileHandle
SocketHandle
ProcessHandle
ThreadHandle
DeviceHandle
MemoryHandle
GPUBufferHandle
StreamHandle

A handle MUST NOT be interpreted as a raw memory address unless the applicable ULABI profile explicitly permits that behavior.

Handles MUST be:

opaque;

validated;

scoped;

revocable where applicable;

associated with an ownership policy.



---

11. Capability Registers

Capability registers represent authority.

A capability is not merely a pointer.

A capability may encode authority to:

read;

write;

execute;

communicate;

access a device;

access memory;

access a resource;

invoke an interface.


Capability semantics belong primarily to the ULABI security and capability profiles.

The register model provides a location-neutral representation for capability references.


---

12. Floating-Point Registers

Floating-point registers represent floating-point values defined by the ULABI type system.

The implementation MUST preserve the semantic requirements of the ULABI floating-point type.

The register model MUST account for:

precision;

NaN;

infinities;

signed zero;

rounding;

conversion;

canonical representation where required.


A target MAY use:

hardware floating-point registers;

vector registers;

software floating-point;

memory-backed values.



---

13. Vector Registers

Vector registers represent multiple logically related values.

Examples:

Vector<Int32>
Vector<Float32>
Vector<Float64>

Vector registers may support:

SIMD;

cryptographic acceleration;

tensor operations;

packed values;

accelerator interfaces.


Vector width MUST be described explicitly.

ULABI MUST NOT assume a fixed hardware vector width.


---

14. Predicate Registers

Predicate registers represent boolean masks or execution predicates.

They may be used for:

vector masking;

conditional execution;

lane selection;

hardware-specific predication.


Predicate registers are an optional capability of the target implementation unless required by a ULABI profile.


---

15. Status Registers

Status information MUST be represented semantically rather than exposing arbitrary machine status flags.

Possible semantic status values include:

Success
Failure
Pending
Cancelled
Interrupted
Retryable
Fatal

Hardware flags such as CPU condition codes MUST NOT become part of the portable ULABI Core unless explicitly standardized.


---

16. Control Registers

Control registers represent execution-control information where required by a ULABI profile.

Possible information includes:

execution mode;

privilege context;

scheduling state;

exception state;

asynchronous state;

transaction state.


Control registers are generally implementation-specific.


---

17. Argument Registers

Argument registers are logical locations used to transport function arguments.

ULABI does not mandate a fixed number.

An ABI profile determines:

which argument classes may use registers;

allocation order;

register capacity;

aggregate handling;

alignment;

overflow behavior.


Example:

Function:

calculate(Int, Float, Pointer)

ULABI argument state:

ARG0 = Int
ARG1 = Float
ARG2 = Pointer

A concrete implementation may map these to physical registers or stack locations.


---

18. Return Registers

Return registers represent function return values.

Example:

Return:
    Result<Int, Error>

The implementation may return this through:

one register;

multiple registers;

a hidden return buffer;

a caller-provided memory location;

an out-of-process message.


The semantic result MUST remain identical.


---

19. Temporary Registers

Temporary registers contain values that need not survive a call.

They are commonly used for:

intermediate calculations;

scratch data;

temporary addresses;

expression evaluation.


Temporary-register preservation is defined by the applicable calling convention.


---

20. Preserved Registers

Preserved registers contain values that MUST remain logically unchanged across a call when the applicable ABI contract requires preservation.

Preservation may be achieved through:

physical callee-saved registers;

stack spilling;

shadow storage;

runtime context storage.


The caller MUST NOT assume a physical register is preserved merely because it belongs to a particular hardware class.

Preservation is a ULABI contract property.


---

21. Special Registers

Special registers may represent:

program state;

execution context;

thread-local state;

runtime state;

exception state;

stack metadata;

capability context.


Special registers MUST NOT be exposed as portable hardware registers.

They MAY be exposed through semantic ULABI interfaces.


---

22. Virtual Register File

ULABI defines a logical register namespace.

A logical register MAY be represented as:

RegisterID
RegisterClass
Width
Type
Role
Lifetime
Ownership
Preservation

Example:

Register {
    id: ARG0,
    class: Argument,
    width: 64,
    type: UInt64,
    role: Input,
    lifetime: Call,
    preservation: CallerClobbered
}

The actual physical mapping is implementation-specific.


---

23. Register Identity

ULABI register identities MUST be stable within the ABI profile that defines them.

A register identity MUST NOT depend solely on a physical register name such as:

RAX
RBX
X0
X1
A0
T0

Instead ULABI uses semantic identities such as:

ARG0
ARG1
RET0
RET1
TEMP0
TEMP1
PRESERVED0
PRESERVED1


---

24. Register Width

A register contract MUST specify its logical width.

Examples:

8-bit
16-bit
32-bit
64-bit
128-bit
256-bit
512-bit
dynamic

A dynamic-width value MUST include sufficient metadata to determine its representation.


---

25. Register Alignment

Where alignment matters, the ABI contract MUST specify:

required alignment;

natural alignment;

minimum alignment;

maximum alignment;

misalignment behavior.


Implementations MUST NOT silently reinterpret misaligned values.


---

26. Register Type Association

A register may carry a type contract.

Example:

ARG0 : UInt64
ARG1 : Pointer<Buffer>
ARG2 : Float64
RET0 : Result<Int32, Error>

Type compatibility is defined by the ULABI type system.

The register model must not independently redefine the universal type system.


---

27. Register Lifetime

Register contents may have different lifetimes:

Instruction
Expression
Call
Frame
Thread
Process
Context

The lifetime MUST be explicit where interoperability depends on it.


---

28. Register Ownership

Register ownership describes which execution context controls a register value.

Possible ownership states:

CallerOwned
CalleeOwned
Shared
Borrowed
Transferred
RuntimeOwned
HardwareOwned

Ownership semantics must integrate with the ULABI memory and resource models.


---

29. Caller-Saved and Callee-Saved Semantics

ULABI defines semantic preservation classes:

CallerClobbered
CalleePreserved
Immutable
Shared
Volatile

The target calling convention maps these onto physical resources.

A caller-clobbered value MUST be assumed destroyed across the call unless otherwise specified.

A callee-preserved value MUST remain logically unchanged across the call.


---

30. Register Allocation

ULABI does not define a compiler register allocator.

An implementation may use:

graph coloring;

linear scan;

greedy allocation;

SSA-based allocation;

runtime allocation;

interpreter state;

JIT allocation.


The ABI boundary is concerned only with the resulting contract.


---

31. Register Spilling

If insufficient physical registers are available, an implementation MAY spill logical registers into memory.

Spilling MUST preserve:

value;

type;

lifetime;

ownership;

alignment;

security properties.


A spilled ULABI register remains semantically a register value even when physically stored in memory.


---

32. Register Restoration

When a spilled value becomes active again, restoration MUST produce a value equivalent to the original logical register value.

Restoration MUST NOT change:

type;

ownership;

permissions;

lifetime;

semantic value.



---

33. Register-to-Stack Transition

ULABI permits an argument or return value to move between register and stack representations.

Example:

Argument
   |
   +--> Register
   |
   +--> Stack
   |
   +--> Indirect Memory

The semantic contract remains unchanged.

This is essential for supporting targets with different numbers of registers.


---

34. Aggregate Values

Large aggregates SHOULD generally be passed indirectly when required by the target ABI.

Example:

Record {
    field_a
    field_b
    field_c
    ...
}

The ABI profile determines whether the value is:

register-passed;

split across registers;

passed indirectly;

passed through memory.


The semantic representation remains the same.


---

35. Multi-Register Values

A logical ULABI value may occupy multiple registers.

Example:

RET0 = lower part
RET1 = upper part

The mapping MUST be deterministic.

Multi-register values MUST define:

register order;

component order;

width;

alignment;

type;

reconstruction rules.



---

36. Hidden Arguments

ULABI permits implementation-required hidden arguments.

Examples include:

return buffers;

runtime context;

capability context;

exception context;

metadata references.


Hidden arguments MUST be explicitly declared by the ABI profile.

They MUST NOT silently alter the public semantic function signature.


---

37. Return Buffers

A function returning a large value MAY use a caller-provided return buffer.

Conceptually:

Caller
  |
  | return buffer
  v
Callee
  |
  | writes result
  v
Caller

The ownership and lifetime of the buffer MUST be defined.


---

38. Register State and Exceptions

Exception handling may require preservation of register state.

An exception context SHOULD be represented through a ULABI semantic context object rather than requiring a universal hardware register layout.

Example:

ExceptionContext {
    instruction_state
    call_state
    register_state
    stack_state
    error
}

The detailed exception model belongs to:

docs/abi/exception-model.md


---

39. Register State and Asynchronous Execution

Asynchronous execution may suspend and resume a computation.

The runtime MUST preserve all state required by the applicable ULABI contract.

Possible mechanisms:

Register Snapshot
Stack Frame
Continuation
Coroutine Frame
Heap Context
Runtime Context Object

ULABI does not require one mechanism.


---

40. Register State and Threads

Each execution context that exposes independent register state MUST have a well-defined register context.

Conceptually:

Thread
 |
 +-- Register Context
 |      |
 |      +-- Arguments
 |      +-- Temporaries
 |      +-- Preserved state
 |      +-- Control state
 |
 +-- Stack
 |
 +-- Runtime Context

Thread-local register state MUST NOT be confused with process-global state.


---

41. Context Switching

A context switch MUST preserve all state required by the execution model.

The implementation MAY save:

registers;

stack state;

program state;

runtime state;

capability context;

exception state.


The exact mechanism is platform-specific.


---

42. Debugging Register State

Debugging interfaces SHOULD expose logical ULABI registers.

Example:

ARG0
ARG1
RET0
TEMP0
PRESERVED0

rather than requiring debugger tooling to understand every physical architecture.

A debugger MAY additionally expose physical registers.


---

43. Security Requirements

Register state MUST be treated as potentially sensitive.

Implementations MUST consider:

secret values;

capability references;

cryptographic material;

authentication state;

pointers;

memory addresses;

process identifiers.


Sensitive registers SHOULD be cleared or protected when their lifetime ends.

Register state MUST NOT be leaked across security boundaries.


---

44. Capability Register Security

Capability-bearing register values MUST be validated before use when crossing a trust boundary.

An invalid capability MUST result in a defined failure.

An implementation MUST NOT convert an invalid capability into unrestricted authority.


---

45. Pointer Register Security

A pointer crossing an ABI boundary MUST NOT automatically grant access beyond its declared bounds or permissions.

The receiving implementation MUST respect:

bounds;

permissions;

lifetime;

ownership;

address-space restrictions.



---

46. Register Sanitization

When register state crosses a security boundary, implementations SHOULD sanitize values that are not part of the declared ABI contract.

Examples:

unused argument registers
unused return registers
reserved registers
temporary registers
privileged control state

Undefined state MUST NOT become an implicit communication channel.


---

47. Determinism

The mapping between logical ABI locations and the calling convention MUST be deterministic for a given ABI profile.

Given the same:

ABI version
function signature
target profile
feature set

the ABI allocation rules MUST produce a deterministic result.


---

48. Compatibility

Register compatibility MUST be evaluated at the semantic level.

Compatible:

ULABI ARG0 = UInt32

and an implementation's physical mapping:

R0

is valid if the ABI contract is satisfied.

An implementation MUST NOT declare incompatibility merely because another implementation uses different physical registers.


---

49. ABI Profiles

The register model is extended by ABI profiles.

Possible profiles include:

ULABI-CORE
ULABI-64
ULABI-32
ULABI-SIMD
ULABI-VECTOR
ULABI-ACCELERATOR
ULABI-REALTIME
ULABI-EMBEDDED
ULABI-SAFETY
ULABI-CAPABILITY
ULABI-SANDBOX
ULABI-DISTRIBUTED

Profiles MUST clearly define:

register classes;

width;

allocation rules;

preservation rules;

supported value types;

optional features;

conformance requirements.



---

50. Architecture Mapping

A target implementation maps ULABI registers to hardware.

Example:

ULABI:

ARG0
ARG1
RET0
RET1

        |
        v

Target Architecture:

physical register A
physical register B
physical register C
physical register D

Another architecture may map them differently.

Both remain ULABI-compatible if they satisfy the same semantic contract.


---

51. Example Mapping

Example logical ABI:

function add(
    a: Int64,
    b: Int64
) -> Int64

ULABI representation:

ARG0 = Int64(a)
ARG1 = Int64(b)

RET0 = Int64(result)

Target implementation A:

ARG0 -> physical register 0
ARG1 -> physical register 1
RET0 -> physical register 0

Target implementation B:

ARG0 -> physical register 3
ARG1 -> physical register 4
RET0 -> physical register 5

Both are valid.


---

52. Example Aggregate Mapping

Function:

function distance(
    Point
) -> Float64

If Point is small:

ARG0 = Point.x
ARG1 = Point.y

If Point is large:

ARG0 = pointer to Point

The selection is determined by the target ABI profile.


---

53. Register File for Distributed Execution

A distributed call does not literally transfer CPU registers.

Instead the logical register state is encoded into an ABI message.

Conceptually:

Logical Registers
       |
       v
Canonical Encoding
       |
       v
Transport
       |
       v
Decoded Logical Registers

The receiving system maps the logical values onto its own local representation.

This is essential to ULABI's transport independence.


---

54. Register Model and Serialization

The register model MUST integrate with the canonical encoding system.

A register value crossing a serialization boundary MUST be represented according to its ULABI type and ABI contract.

Physical register names MUST NOT appear in portable canonical encodings.


---

55. Register Model and Zero-Copy

Zero-copy execution MAY bypass serialization.

However, zero-copy is permitted only when:

both sides share an appropriate memory domain;

ownership is defined;

lifetime is defined;

permissions are compatible;

alignment is valid;

security requirements are satisfied.


Zero-copy MUST NOT be assumed merely because both components run on the same host.


---

56. Register Model and Shared Memory

Shared-memory profiles MAY allow logical register values to reference shared memory.

The shared-memory contract MUST define:

ownership;

synchronization;

visibility;

mutation;

lifetime;

invalidation.



---

57. Register Model and Accelerators

Accelerator profiles MAY define additional logical register classes.

Examples:

TensorRegister
MatrixRegister
VectorRegister
DeviceRegister
QueueRegister
EventRegister

These are extensions and MUST NOT contaminate the portable Core ABI.


---

58. Register Model and GPU Execution

GPU-oriented implementations MAY map ULABI logical values onto:

scalar registers;

vector registers;

lane state;

shared memory;

device memory.


ULABI does not require a CPU-like register architecture.

The accelerator profile defines the necessary semantics.


---

59. Register Model and Future Hardware

The register model MUST permit future hardware designs including:

many-core processors;

capability processors;

vector processors;

tensor processors;

neuromorphic systems;

quantum interfaces;

reconfigurable processors;

heterogeneous processors;

architectures with few or no conventional registers.


The semantic ABI MUST remain stable even when hardware changes.


---

60. Quantum and Non-Classical Execution

A future quantum profile MAY define semantic execution state that does not map to conventional classical registers.

The portable Core MUST NOT assume that all execution state is representable as ordinary integer or floating-point registers.

Quantum-specific semantics belong to:

docs/hardware/quantum.md

and related profiles.


---

61. Register Reservation

ABI profiles MAY reserve logical registers for:

runtime context;

exception context;

capability context;

thread state;

asynchronous state.


Reserved registers MUST be documented.

An implementation MUST NOT expose reserved state as ordinary user arguments.


---

62. Register Exhaustion

When the available register resources are insufficient, the implementation MUST use a defined fallback.

Possible fallbacks:

stack
memory
indirect argument
heap context
runtime-managed storage

Register exhaustion MUST NOT change the semantic function contract.


---

63. Failure Conditions

The register subsystem MUST define failures for:

invalid register ID;

invalid register class;

incompatible width;

incompatible type;

invalid alignment;

invalid pointer;

invalid capability;

invalid register state;

malformed register encoding;

unsupported register profile;

unsupported vector width;

unsupported accelerator feature.


Failures MUST use the ULABI error model.


---

64. Validation

Before a register state crosses a trust or ABI boundary, an implementation SHOULD validate:

Register identity
Register class
Width
Type
Alignment
Ownership
Lifetime
Permissions
Value representation
Profile compatibility

Validation failures MUST NOT produce undefined behavior.


---

65. Register State Validation Contract

A logical register state can be represented conceptually as:

RegisterState {
    profile
    version
    registers[]
}

Each register entry contains:

RegisterEntry {
    id
    class
    type
    width
    value
    ownership
    lifetime
    permissions
}

The exact wire representation belongs to the canonical encoding specification.


---

66. Register Model and Calling Convention

The register model provides the vocabulary.

The calling convention specifies the allocation rules.

Therefore:

Register Model
      |
      v
Calling Convention
      |
      v
Function ABI

The calling convention MUST NOT redefine register semantics.

The calling convention document is:

docs/abi/calling-convention.md


---

67. Register Model and Memory Model

The register model interacts with:

pointers;

ownership;

borrowing;

lifetimes;

stack;

heap;

shared memory;

zero-copy.


These semantics are defined by:

docs/abi/memory-model.md

and the memory subsystem specifications.


---

68. Register Model and Type System

Register values carry semantic types.

The register model therefore depends on:

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md

The register model MUST NOT create duplicate type definitions.


---

69. Register Model and Error Model

Invalid register states MUST use standardized ULABI errors.

The register model therefore integrates with:

docs/abi/exception-model.md
ULABI-SPEC.md


---

70. Register Model and Security

Register state can contain authority and secrets.

Security requirements integrate with:

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md


---

71. Register Model and Debugging

Debuggers SHOULD understand the logical ULABI register namespace.

Integration target:

docs/tooling/debugger-interface.md
docs/observability/deterministic-debugging.md


---

72. Register Model and Self-Healing

The register model MUST NOT permit arbitrary self-modification.

If a register-related failure is detected, recovery MUST follow the ULABI reliability policy:

Detection
    |
Diagnosis
    |
Isolation
    |
Approved Recovery
    |
Verification
    |
+----+----+
|         |
Healthy   Failed
|         |
Done    Rollback/Escalate

Register restoration MUST be based on known-good state.

Integration target:

docs/reliability/self-healing.md
docs/reliability/recovery.md
docs/reliability/rollback.md


---

73. Distributed Register Semantics

A distributed implementation MUST NOT pretend that remote registers are local CPU registers.

Remote values are logical ABI values.

The implementation MUST explicitly account for:

latency;

failure;

serialization;

transport;

cancellation;

timeout;

retry;

idempotency.


Integration targets:

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md


---

74. ABI Negotiation

Before using an extended register feature, implementations SHOULD negotiate:

ULABI version
ABI profile
register classes
widths
vector support
accelerator support
security features
optional extensions

Unsupported features MUST be rejected or gracefully degraded according to compatibility rules.


---

75. Feature Discovery

A runtime MAY expose:

supports_register_class(General)
supports_register_class(Vector)
supports_register_class(Capability)
supports_width(64)
supports_profile("ULABI-VECTOR")

Feature discovery belongs to:

docs/compatibility/capability-discovery.md


---

76. Backward Compatibility

A newer implementation SHOULD continue supporting older register contracts where the compatibility rules permit.

A newer physical architecture MUST NOT require existing ULABI applications to understand new physical register names.


---

77. Forward Compatibility

Older implementations MAY encounter new register classes.

They MUST use the applicable compatibility policy:

Supported
Optional
Ignorable
Unsupported
Fatal

Unknown critical register semantics MUST NOT be silently ignored.


---

78. Versioning

The register model MUST be versioned independently from physical architectures.

Example:

ULABI Register Model 1.0
Target Architecture: Architecture-X

The architecture may change without changing the semantic register contract.


---

79. Security Boundary Rule

Crossing an ABI boundary MUST be treated as a potential trust boundary.

Implementations MUST NOT assume:

register value == trusted value

Instead:

Incoming Register State
        |
        v
Validation
        |
        v
Authorization
        |
        v
Type Validation
        |
        v
Execution


---

80. Undefined Register State

Any register not explicitly defined by the ABI contract is considered undefined.

Undefined state MUST NOT be relied upon for:

communication;

security;

compatibility;

deterministic behavior;

persistence.



---

81. Reserved Register State

Reserved state MUST be treated as unavailable to ordinary ABI calls unless the relevant profile explicitly exposes it.


---

82. Canonical Register Ordering

When register state is serialized, registers MUST have a deterministic canonical ordering.

The ordering MUST be based on semantic register identity rather than physical register numbering.

Example:

ARG0
ARG1
ARG2
RET0
RET1


---

83. Register State Hashing

A future verification profile MAY define cryptographic hashes over canonical register state.

This can support:

deterministic debugging;

replay;

verification;

checkpointing;

recovery;

distributed consistency checks.


The hash MUST operate on canonical semantic state rather than physical register names.


---

84. Checkpointing

A reliability implementation MAY checkpoint logical register state.

A checkpoint MUST contain sufficient information to reconstruct the permitted execution state.

It MUST NOT expose secrets unnecessarily.


---

85. Register State and Replay

Deterministic replay MAY use logical register snapshots.

Replay systems SHOULD record:

ABI profile;

version;

logical register values;

execution context;

required external effects.


Physical register names MUST NOT be required for portable replay.


---

86. Register Model Invariants

The following invariants are normative design requirements.

R1 — Semantic Independence

ULABI register semantics MUST NOT depend on physical register names.

R2 — Deterministic Mapping

A conforming ABI profile MUST define deterministic mapping rules.

R3 — Type Safety

Register values MUST conform to their declared semantic types.

R4 — Ownership Safety

Register ownership MUST be respected.

R5 — Lifetime Safety

Register values MUST NOT outlive their permitted lifetime.

R6 — Capability Safety

Capabilities MUST NOT gain authority through register conversion.

R7 — Pointer Safety

Pointers MUST NOT gain authority beyond their declared bounds.

R8 — Compatibility

Physical register differences MUST NOT by themselves cause ULABI incompatibility.

R9 — Undefined-State Isolation

Undefined register state MUST NOT be relied upon.

R10 — Profile Isolation

Architecture-specific register extensions MUST remain within their profiles.

R11 — Deterministic Serialization

Serialized register state MUST have deterministic ordering.

R12 — Security Boundary Enforcement

Register state crossing trust boundaries MUST be validated.

R13 — No Hidden Semantics

The register mapping MUST NOT silently change the meaning of a function contract.

R14 — Failure Explicitness

Register-related failures MUST have defined outcomes.

R15 — Future Extensibility

The model MUST permit future register architectures without redesigning the Core ABI.


---

87. Conformance Requirements

An implementation claiming conformance to the ULABI Register Model MUST demonstrate:

1. Logical register identification.


2. Register classification.


3. Type association.


4. Width handling.


5. Argument handling.


6. Return-value handling.


7. Preservation semantics.


8. Register spilling.


9. Register restoration.


10. Aggregate handling.


11. Pointer safety.


12. Capability safety where supported.


13. Error handling.


14. Profile negotiation.


15. Compatibility handling.


16. Deterministic behavior.


17. Security validation.


18. Undefined-state isolation.




---

88. Minimum Conformance Profile

The minimum implementation SHOULD support:

General registers
Integer registers
Pointer references
Argument registers
Return registers
Temporary registers
Preserved registers
Register widths
Register typing
Register spilling
Register restoration

Advanced features are optional unless required by a profile.


---

89. Extended Conformance

Extended profiles MAY additionally support:

FloatingPoint
Vector
Predicate
Capability
Accelerator
Tensor
RealTime
Embedded
Distributed
Security
Verification
Recovery


---

90. Conformance Tests

Tests MUST verify semantic behavior rather than physical register names.

Example:

Input:

ARG0 = 10
ARG1 = 20

Operation:

add(ARG0, ARG1)

Expected:

RET0 = 30

The test remains valid regardless of which physical registers are used.


---

91. Required Test Categories

The test suite SHOULD include:

register_identity
register_classification
integer_values
pointer_values
handle_values
argument_mapping
return_mapping
caller_saved
callee_saved
register_spilling
register_restoration
aggregate_values
multi_register_values
alignment
invalid_registers
invalid_types
invalid_widths
security_validation
capability_validation
serialization
deserialization
version_negotiation
feature_negotiation
compatibility
distributed_mapping
debugging
checkpointing
recovery


---

92. Fuzz Testing

Register state parsers and validators MUST be suitable for fuzz testing.

Fuzz targets SHOULD include:

malformed register IDs;

malformed register types;

invalid widths;

invalid values;

oversized register states;

duplicate register identifiers;

invalid profiles;

malicious capability references;

malformed serialized state.



---

93. Formal Verification

Critical register invariants SHOULD be formally verifiable.

Potential verification targets include:

type preservation
ownership preservation
lifetime safety
register mapping determinism
canonical ordering
capability non-escalation
pointer-bound preservation


---

94. Implementation Independence

At least two independent implementations SHOULD eventually be able to implement the register model without sharing internal code.

This requirement is important for preventing accidental coupling between the specification and one implementation.


---

95. Reference Implementation Strategy

The initial reference implementation SHOULD be written in Rust.

Reason:

strong memory safety;

explicit data modeling;

good systems-programming support;

suitable for ABI tooling;

suitable for parsers and validators;

cross-platform support;

good testing ecosystem.


Rust is an implementation language only.

Rust MUST NOT become part of the ULABI specification.


---

96. Required Code Architecture

The initial reference implementation SHOULD use:

src/
├── lib.rs
├── abi/
│   ├── mod.rs
│   ├── register.rs
│   ├── register_class.rs
│   ├── register_id.rs
│   ├── register_width.rs
│   ├── register_value.rs
│   ├── register_state.rs
│   ├── register_mapping.rs
│   ├── preservation.rs
│   ├── lifetime.rs
│   ├── ownership.rs
│   ├── spill.rs
│   └── restore.rs
│
├── types/
│   ├── mod.rs
│   ├── primitive.rs
│   ├── pointer.rs
│   ├── handle.rs
│   ├── capability.rs
│   └── type_ref.rs
│
├── calling_convention/
│   ├── mod.rs
│   ├── convention.rs
│   ├── argument.rs
│   ├── return_value.rs
│   ├── aggregate.rs
│   └── allocator.rs
│
├── encoding/
│   ├── mod.rs
│   ├── canonical.rs
│   ├── register_encoding.rs
│   └── decoder.rs
│
├── validation/
│   ├── mod.rs
│   ├── register_validator.rs
│   ├── type_validator.rs
│   ├── ownership_validator.rs
│   └── capability_validator.rs
│
├── compatibility/
│   ├── mod.rs
│   ├── version.rs
│   ├── profile.rs
│   ├── negotiation.rs
│   └── feature.rs
│
└── errors/
    ├── mod.rs
    └── register_error.rs


---

97. Independent Implementation Files

The implementation MUST be developed in dependency order.

The first files should be independently understandable.

Recommended order:

1. src/abi/register_id.rs
2. src/abi/register_class.rs
3. src/abi/register_width.rs
4. src/abi/preservation.rs
5. src/abi/lifetime.rs
6. src/abi/ownership.rs
7. src/abi/register.rs
8. src/abi/register_value.rs
9. src/abi/register_state.rs
10. src/abi/register_mapping.rs
11. src/abi/spill.rs
12. src/abi/restore.rs

These form the register-model foundation.


---

98. Integration Files

After the register foundation:

src/types/type_ref.rs
src/types/primitive.rs
src/types/pointer.rs
src/types/handle.rs
src/types/capability.rs

src/calling_convention/convention.rs
src/calling_convention/argument.rs
src/calling_convention/return_value.rs
src/calling_convention/aggregate.rs
src/calling_convention/allocator.rs

src/encoding/register_encoding.rs
src/encoding/canonical.rs
src/encoding/decoder.rs

src/validation/register_validator.rs
src/validation/type_validator.rs
src/validation/ownership_validator.rs
src/validation/capability_validator.rs

src/compatibility/profile.rs
src/compatibility/version.rs
src/compatibility/feature.rs
src/compatibility/negotiation.rs


---

99. Module Responsibilities

register_id.rs

Defines semantic register identifiers.

Examples:

ARG0
ARG1
RET0
RET1
TEMP0
PRESERVED0
SPECIAL0

It MUST NOT contain CPU-specific registers.


---

register_class.rs

Defines semantic register classes.

Examples:

General
Integer
Pointer
Handle
Capability
FloatingPoint
Vector
Predicate
Status
Control
Argument
Return
Temporary
Preserved
Special
Extension


---

register_width.rs

Defines logical register widths.

It MUST support future extensibility.


---

preservation.rs

Defines:

CallerClobbered
CalleePreserved
Immutable
Shared
Volatile


---

lifetime.rs

Defines logical register lifetimes.


---

ownership.rs

Defines register ownership semantics.


---

register.rs

Combines:

RegisterID
RegisterClass
Width
Type
Role
Lifetime
Ownership
Preservation

This becomes the primary register descriptor.


---

register_value.rs

Represents the value stored in a logical register.

It MUST preserve the distinction between:

type
value
metadata


---

register_state.rs

Represents a complete logical register context.

It MUST support:

lookup;

insertion;

replacement;

validation;

snapshot;

restoration.



---

register_mapping.rs

Maps logical ULABI registers to implementation-specific storage.

It MUST isolate physical architecture knowledge from the Core register model.


---

spill.rs

Defines register-to-memory spilling.


---

restore.rs

Defines restoration from spilled state.


---

100. Calling Convention Integration

src/calling_convention/ MUST consume the register model.

It MUST NOT redefine:

register classes;

register identities;

register widths;

ownership;

preservation semantics.


It only defines allocation policy.


---

101. Type-System Integration

src/types/ provides semantic types.

The register implementation references those types.

It MUST NOT duplicate type compatibility rules.


---

102. Encoding Integration

src/encoding/ serializes logical register state.

Physical register names MUST NOT be required in canonical encoding.


---

103. Validation Integration

Validation MUST occur before:

ABI calls;

deserialization;

trust-boundary crossing;

capability use;

pointer use;

restoration from external state.



---

104. Compatibility Integration

The compatibility layer determines whether:

register model version
profile
register class
width
feature

can be used by both parties.


---

105. Test File Architecture

The register subsystem SHOULD eventually contain:

tests/
├── register_id_tests.rs
├── register_class_tests.rs
├── register_width_tests.rs
├── register_descriptor_tests.rs
├── register_value_tests.rs
├── register_state_tests.rs
├── register_mapping_tests.rs
├── preservation_tests.rs
├── ownership_tests.rs
├── lifetime_tests.rs
├── spill_tests.rs
├── restore_tests.rs
├── calling_convention_register_tests.rs
├── register_encoding_tests.rs
├── register_validation_tests.rs
├── compatibility_tests.rs
├── security_register_tests.rs
├── fuzz_register_state.rs
└── conformance/
    └── register_model.rs


---

106. Integration Contract

The register subsystem is considered complete when:

1. Register identities are stable.


2. Register classes are stable.


3. Width semantics are defined.


4. Register descriptors are defined.


5. Register values are defined.


6. Register state is defined.


7. Preservation semantics are defined.


8. Ownership semantics are defined.


9. Lifetime semantics are defined.


10. Spilling is defined.


11. Restoration is defined.


12. Validation is defined.


13. Serialization is defined.


14. Compatibility is defined.


15. Calling conventions can consume the model without redefining it.


16. Memory semantics can consume the model without redefining it.


17. Security profiles can consume the model without redefining it.


18. Distributed profiles can serialize it without changing its semantics.


19. Conformance tests exist.


20. Physical architecture mappings remain outside the semantic Core.




---

107. No-Rework Integration Rule

Once this document is accepted as the register-model contract, later ULABI documents MUST integrate with the interfaces defined here rather than changing their meaning.

Later documents MUST NOT silently redefine:

RegisterID
RegisterClass
RegisterWidth
RegisterOwnership
RegisterLifetime
RegisterPreservation
RegisterState

If a new requirement cannot be represented by the current model, it MUST be introduced through:

1. a new profile;


2. an explicit extension;


3. a versioned specification change.



Existing semantics MUST NOT be silently changed.


---

108. Required Future Documentation

The register model integrates with these specifications:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/shared-memory.md

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md

docs/runtime/runtime-interface.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md

docs/tooling/debugger-interface.md
docs/observability/deterministic-debugging.md

docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/backwards-compatibility.md

docs/reliability/self-healing.md
docs/reliability/recovery.md
docs/reliability/rollback.md

docs/standards/conformance.md
docs/standards/test-suite.md

These documents MUST consume the register contract rather than redefine it.


---

109. Complete Required Code Module List

The initial ULABI reference implementation should eventually contain at least:

src/lib.rs

src/abi/mod.rs
src/abi/register.rs
src/abi/register_id.rs
src/abi/register_class.rs
src/abi/register_width.rs
src/abi/register_value.rs
src/abi/register_state.rs
src/abi/register_mapping.rs
src/abi/preservation.rs
src/abi/lifetime.rs
src/abi/ownership.rs
src/abi/spill.rs
src/abi/restore.rs

src/types/mod.rs
src/types/primitive.rs
src/types/type_ref.rs
src/types/pointer.rs
src/types/handle.rs
src/types/capability.rs

src/calling_convention/mod.rs
src/calling_convention/convention.rs
src/calling_convention/argument.rs
src/calling_convention/return_value.rs
src/calling_convention/aggregate.rs
src/calling_convention/allocator.rs

src/encoding/mod.rs
src/encoding/canonical.rs
src/encoding/register_encoding.rs
src/encoding/decoder.rs

src/validation/mod.rs
src/validation/register_validator.rs
src/validation/type_validator.rs
src/validation/ownership_validator.rs
src/validation/capability_validator.rs

src/compatibility/mod.rs
src/compatibility/version.rs
src/compatibility/profile.rs
src/compatibility/feature.rs
src/compatibility/negotiation.rs

src/errors/mod.rs
src/errors/register_error.rs


---

110. Files That Must NOT Be Coupled to a Specific Language

The following modules MUST remain language-neutral:

register.rs
register_id.rs
register_class.rs
register_width.rs
register_value.rs
register_state.rs
register_mapping.rs
preservation.rs
lifetime.rs
ownership.rs
calling_convention/*
encoding/*
validation/*
compatibility/*

They must not contain assumptions about:

C
C++
Rust
Go
Java
Python
Swift
Kotlin
Fortran
Ada
Zamani
Sankofa

Language-specific adapters belong outside the semantic Core.


---

111. Language Adapter Architecture

Language implementations should eventually use adapters such as:

adapters/
├── c/
├── cpp/
├── rust/
├── go/
├── java/
├── python/
├── swift/
├── kotlin/
├── fortran/
├── ada/
├── zamani/
└── sankofa/

These adapters are independent implementations.

ULABI does not require them to share implementation code.


---

112. Zamani and Sankofa Separation

ULABI MUST preserve the following architectural rule:

Zamani  != Sankofa
Zamani  != ULABI
Sankofa != ULABI

Zamani may implement a ULABI adapter.

Sankofa may independently implement a ULABI adapter.

Neither language defines the register model.

Neither language becomes the reference semantic model for ULABI.


---

113. Physical Architecture Adapter Architecture

Physical architectures SHOULD be isolated similarly:

targets/
├── generic/
├── x86_64/
├── aarch64/
├── riscv64/
├── wasm/
├── gpu/
└── accelerator/

These modules map the semantic ULABI register model onto physical execution environments.


---

114. Architecture Adapter Rule

A physical target adapter MUST NOT modify the semantic meaning of:

ARG0
ARG1
RET0
TEMP0
PRESERVED0

It only determines where those logical values are physically stored or transported.


---

115. Final Architecture

The intended architecture is:

ULABI SPECIFICATION
                         |
                         v
                 REGISTER MODEL
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
     Type System   Calling Convention  Security
          |              |              |
          +--------------+--------------+
                         |
                         v
                 Logical Register State
                         |
              +----------+----------+
              |                     |
              v                     v
       Local Execution       Serialized State
              |                     |
              v                     v
       Physical Target          Transport
              |                     |
       +------+-------+       +-----+------+
       |      |       |       |            |
      CPU    GPU    NPU    IPC         Network


---

116. Final Design Principle

The ULABI Register Model is a semantic abstraction.

It is not a universal CPU.

It is not a universal machine.

It is not a replacement for existing hardware ABIs.

It is the stable contract that allows different execution environments to agree on the meaning and handling of ABI-level values.

The fundamental rule is:

> ULABI standardizes what a register means at the interoperability boundary, not which physical register must implement it.




---

117. Status

This document is an architectural specification.

It becomes normative only when incorporated into the corresponding version of:

ULABI-SPEC.md

Until then, implementation work MUST treat unresolved details as draft requirements rather than final standards.


---

118. Summary

ULABI's register architecture provides:

Semantic Registers
        +
Typed Values
        +
Explicit Ownership
        +
Explicit Lifetime
        +
Explicit Preservation
        +
Deterministic Mapping
        +
Safe Spilling
        +
Safe Restoration
        +
Security Validation
        +
Profile Extensions
        +
Architecture Independence
        +
Language Independence
        +
Distributed Representation

This allows one ULABI contract to support radically different programming languages and hardware architectures without forcing them to become identical.

ULABI remains the interoperability contract.

The implementation remains independent.

## Implementation order for this component

For the **register subsystem specifically**, I recommend we do **not** create all the modules at once. The clean dependency order is:

| Order | File | Role | Depends on |
|---:|---|---|---|
| 1 | `src/abi/register_id.rs` | Stable logical IDs | Nothing |
| 2 | `src/abi/register_class.rs` | Semantic classes | Nothing |
| 3 | `src/abi/register_width.rs` | Width model | Nothing |
| 4 | `src/abi/preservation.rs` | Caller/callee preservation | Nothing |
| 5 | `src/abi/lifetime.rs` | Register lifetimes | Nothing |
| 6 | `src/abi/ownership.rs` | Ownership states | Nothing |
| 7 | `src/abi/register.rs` | Complete register descriptor | 1–6 |
| 8 | `src/abi/register_value.rs` | Typed register values | 3, 7 |
| 9 | `src/abi/register_state.rs` | Complete register context | 7–8 |
| 10 | `src/abi/register_mapping.rs` | Logical → physical mapping | 7–9 |
| 11 | `src/abi/spill.rs` | Register spilling | 9–10 |
| 12 | `src/abi/restore.rs` | Register restoration | 9–11 |
| 13 | `src/types/type_ref.rs` | Type references | Independent type foundation |
| 14 | `src/calling_convention/convention.rs` | Convention contract | Register model |
| 15 | `src/calling_convention/argument.rs` | Argument allocation | 14 |
| 16 | `src/calling_convention/return_value.rs` | Return allocation | 14 |
| 17 | `src/calling_convention/aggregate.rs` | Aggregate passing | 14 + types |
| 18 | `src/calling_convention/allocator.rs` | Allocation algorithm | 15–17 |
| 19 | `src/encoding/register_encoding.rs` | Register serialization | Register state |
| 20 | `src/validation/register_validator.rs` | Register validation | Register state |
| 21 | `src/compatibility/profile.rs` | Profile compatibility | Register model |
| 22 | `src/compatibility/negotiation.rs` | Feature negotiation | Profile/version |

### The important architectural boundary

The first **12 files** are the core register-model implementation. They should be completed before the calling-convention implementation begins.

Then:

```text
Register Model
      ↓
Calling Convention
      ↓
Type Integration
      ↓
Encoding
      ↓
Validation
      ↓
Compatibility
      ↓
Architecture Adapters
      ↓
Language Adapters
      ↓
Conformance Tests

This ordering is deliberate. It means when we later write calling-convention.md, memory-model.md, or the language adapters, they consume the already-defined register contract instead of forcing us to rewrite the register model.

And most importantly, Zamani and Sankofa remain two separate programming languages throughout this architecture. They can each independently implement ULABI; neither is part of ULABI's definition.