I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI. The existing architecture establishes the key principles we need to preserve: language neutrality, runtime/platform independence, a minimal stable Core, profiles/extensions, explicit semantics, compatibility, security, and independent implementations. It also explicitly keeps Zamani and Sankofa separate, with each potentially implementing a ULABI adapter.

Below is Design Document 6, for:

docs/abi/calling-convention.md

I am also designing the implementation structure in advance so this document does not need to be rewritten merely because later components are added.

Copy everything below into that file.

# ULABI Calling Convention Specification

**Document:** `docs/abi/calling-convention.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** `Benwellonedge28/ULABI`  
**Specification Family:** ABI  
**Component:** Calling Convention  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.1.0-draft  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the ULABI Calling Convention.

The calling convention specifies how independently implemented components exchange function calls across the ULABI boundary.

It defines the universal contract for:

- function invocation;
- parameter passing;
- return values;
- argument ordering;
- argument classification;
- scalar values;
- structured values;
- references;
- handles;
- ownership transfer;
- borrowing;
- memory references;
- error results;
- effect metadata;
- capability metadata;
- asynchronous calls;
- cancellation;
- streaming calls;
- variadic arguments;
- optional arguments;
- ABI negotiation;
- ABI compatibility;
- architecture-specific lowering;
- register allocation;
- stack usage;
- call frames;
- tail calls;
- callbacks;
- function pointers;
- closures;
- cross-language invocation;
- cross-process invocation;
- distributed invocation.

The calling convention is one of the fundamental components of ULABI.

It must remain independent of:

- any programming language;
- any compiler;
- any runtime;
- any operating system;
- any CPU architecture;
- any hardware vendor;
- any single memory-management model.

---

# 2. Relationship to ULABI

ULABI defines the interoperability contract.

The calling convention defines how a function contract is physically and semantically invoked.

The architecture is:

```text
Programming Language
        |
        v
Language Adapter
        |
        v
ULABI Function Contract
        |
        v
ULABI Calling Convention
        |
        v
Platform/Architecture Lowering
        |
        v
Native ABI / Runtime / Transport

The ULABI calling convention therefore exists at the semantic boundary.

It does not attempt to replace every native ABI.

Instead, implementations translate between:

ULABI Calling Convention
        |
        +---- Native CPU ABI
        |
        +---- Runtime ABI
        |
        +---- IPC ABI
        |
        +---- Distributed transport
        |
        +---- Accelerator ABI


---

3. Fundamental Rule

> A ULABI function contract must have one stable semantic calling convention even when its physical implementation differs between platforms.



For example:

ULABI Function
     |
     +---- x86-64 implementation
     |
     +---- AArch64 implementation
     |
     +---- RISC-V implementation
     |
     +---- WebAssembly implementation
     |
     +---- GPU implementation
     |
     +---- IPC implementation
     |
     +---- Distributed implementation

The physical mechanism may differ.

The semantic contract must remain compatible.


---

4. Calling Convention Layers

The ULABI calling convention consists of multiple layers.

Layer 0 — Function Identity
Layer 1 — Function Contract
Layer 2 — Argument Semantics
Layer 3 — Return Semantics
Layer 4 — Ownership and Lifetime
Layer 5 — Effects and Capabilities
Layer 6 — Execution Semantics
Layer 7 — Error Semantics
Layer 8 — Physical ABI Lowering
Layer 9 — Transport Lowering

An implementation may omit unnecessary physical layers when the call is local.


---

5. Function Identity

Every externally callable ULABI function MUST have a stable interface identity.

A function identity SHOULD contain:

interface_id
function_id
version
signature_id
profile_set

Example:

interface_id:
    550e8400-e29b-41d4-a716-446655440000

function_id:
    calculate

version:
    1.0

signature_id:
    <canonical signature identifier>

The function name alone MUST NOT be considered sufficient for binary identity.


---

6. Function Contract

A ULABI function is defined by a contract.

Example:

calculate(
    input: Int64
) -> Result<Int64, CalculationError>

The complete contract may include:

FunctionIdentity
Parameters
ReturnType
ErrorType
Ownership
Effects
Capabilities
ExecutionMode
Determinism
Cancellation
Streaming
Version
Compatibility

The implementation MUST NOT infer important semantics that are absent from the contract.


---

7. Argument Ordering

ULABI arguments have a deterministic logical ordering.

For:

function add(
    a: Int32,
    b: Int32,
    c: Int32
)

the logical order is:

1. a
2. b
3. c

An implementation MAY lower these arguments differently for a target architecture.

The semantic order MUST remain unchanged.


---

8. Argument Classification

Arguments are classified according to semantic representation.

Initial classes:

Scalar
Aggregate
Reference
Handle
Capability
Option
Result
FunctionReference
Stream
Future
Resource

Future classes may be introduced through versioned profiles.

Unknown argument classes MUST NOT be silently interpreted as another class.


---

9. Scalar Arguments

Scalar arguments include:

Bool
Int
UInt
Float
Char

Their exact width and representation MUST be specified by the function contract or associated type descriptor.

For example:

Int32
UInt32
Int64
UInt64
Float32
Float64

The implementation MUST NOT assume that:

int

means a particular width.


---

10. Structured Arguments

Structured arguments include:

Record
Tuple
List
Map
Set
Enum
Variant
Option
Result

Structured arguments MUST have deterministic semantic layouts.

Physical layouts MAY differ between implementations.

Example:

Person {
    id: UInt64
    name: String
    active: Bool
}

The ULABI contract defines the semantic fields.

The platform adapter determines the physical representation.


---

11. Argument Passing Modes

ULABI supports explicit argument passing modes.

Initial modes:

In
Out
InOut
Move
Borrow
Shared
Consume

In

The callee receives a value for reading.

Out

The callee produces a value for the caller.

InOut

The callee may read and modify the supplied value.

Move

Ownership transfers to the callee.

Borrow

The callee receives temporary access without ownership transfer.

Shared

Multiple parties may access the value subject to the memory contract.

Consume

The callee consumes the resource/value according to its declared lifetime rules.


---

12. Ownership Rules

Ownership MUST be explicit whenever a value represents a resource or managed memory.

The calling convention MUST never assume that two languages use the same memory-management system.

Possible implementation models include:

Manual memory
Reference counting
Tracing garbage collection
Ownership/borrowing
Region allocation
Arena allocation
Automatic runtime management
Capability-managed memory

ULABI abstracts these differences.


---

13. Borrowed Arguments

A borrowed argument does not transfer ownership.

Example:

inspect(
    value: Borrow<Record>
)

The callee MUST NOT retain the borrowed reference beyond its permitted lifetime.

The lifetime MUST be defined by the applicable memory contract.

If an implementation cannot safely represent the borrow, it MUST use a safe alternative such as copying or ownership transfer.

It MUST NOT create an invalid reference.


---

14. Move Arguments

For:

process(
    resource: Move<Resource>
)

ownership transfers to the callee.

After successful transfer, the caller MUST treat the resource as unavailable unless the contract explicitly defines otherwise.


---

15. Handle Arguments

Handles represent externally managed resources.

Examples:

FileHandle
SocketHandle
ProcessHandle
DeviceHandle
GPUBufferHandle
SharedMemoryHandle

A handle MUST NOT be assumed to be a raw memory address.

A handle SHOULD contain or reference:

handle_id
resource_type
authority
lifetime
capabilities


---

16. Capability Arguments

Capabilities represent authorized access to resources or operations.

Example:

read_file(
    capability: FileReadCapability,
    path: String
)

Possession of a capability SHOULD be sufficient to authorize the associated operation.

Capabilities MUST NOT automatically grant unrelated permissions.


---

17. Return Values

A ULABI function MAY return:

Unit
Scalar
Aggregate
Reference
Handle
Option<T>
Result<T,E>
Future<T>
Stream<T>

Example:

function get_value() -> Result<Int64, Error>

The return contract MUST specify ownership semantics.


---

18. Multiple Return Values

ULABI SHOULD represent multiple semantic return values as a structured result.

Example:

function divide(
    a: Int64,
    b: Int64
) -> Result<DivisionResult, DivisionError>

Where:

DivisionResult {
    quotient: Int64
    remainder: Int64
}

This avoids architecture-specific assumptions about multiple hardware return registers.


---

19. Error Returns

ULABI strongly prefers explicit error contracts.

Preferred form:

Result<T, E>

Example:

Result<Data, IOError>

The calling convention MUST preserve the distinction between:

successful return
error return
panic/fatal termination
cancellation
timeout
transport failure
capability denial


---

20. Exceptions

ULABI does not require a universal exception mechanism.

A language may use:

exceptions
results
status codes
error objects
algebraic effects
traps

The adapter MUST translate the language-specific mechanism into the ULABI error contract.

An exception MUST NOT silently cross the ULABI boundary unless the applicable profile explicitly defines exception interoperability.


---

21. Function Effects

Functions MAY declare effects.

Examples:

Pure
ReadsMemory
WritesMemory
ReadsFilesystem
WritesFilesystem
Network
GPU
Device
Process
Time
Randomness
NonDeterministic

Effects form part of the function metadata.

They may be used for:

sandboxing;

security;

static analysis;

scheduling;

capability validation;

conformance testing.



---

22. Capability Requirements

A function MAY declare required capabilities.

Example:

calculate()
    requires:
        Compute

or:

write_file()
    requires:
        FileWrite

The caller MUST NOT assume that the callee possesses capabilities that were not declared or explicitly delegated.


---

23. Execution Modes

ULABI functions MAY declare:

Synchronous
Asynchronous
Blocking
NonBlocking
Streaming
LongRunning
Cancellable
Idempotent
NonIdempotent

The declaration affects how the caller interacts with the function.


---

24. Synchronous Calls

A synchronous call:

call()

does not complete until the function has returned a defined result or failure.

A synchronous function MUST NOT silently become a remote or asynchronous operation.


---

25. Asynchronous Calls

An asynchronous function MAY return:

Future<T>

Example:

fetch_data()
    -> Future<Result<Data, Error>>

The future contract MUST define:

completion
failure
cancellation
timeout
ownership
resource lifetime


---

26. Cancellation

Cancellable operations MUST define cancellation semantics.

Cancellation may be:

BestEffort
Cooperative
Immediate
Deferred
NonCancellable

A cancelled operation MUST produce an explicit observable state.

The implementation MUST NOT claim successful completion when cancellation has definitively prevented completion.


---

27. Streaming Calls

Streaming functions MAY use:

Stream<T>

Example:

read_records()
    -> Stream<Record>

The calling convention MUST define:

next
end-of-stream
error
cancel
backpressure
ownership
resource release


---

28. Variadic Functions

ULABI MAY support explicitly declared variadic functions.

Example:

log(
    level: LogLevel,
    ...messages: String
)

Variadic parameters MUST include sufficient type information for safe decoding.

Untyped variadic binary arguments MUST NOT be permitted in the Core.

A language adapter MAY provide a language-specific compatibility layer.


---

29. Default Arguments

Default arguments are a source-language feature.

ULABI SHOULD NOT encode hidden compiler-specific default argument behavior.

Instead, the exported ULABI function contract SHOULD expose explicit parameters.

For example:

source:
    connect(host)

ULABI:
    connect(
        host,
        port
    )

The language adapter supplies the default.


---

30. Named Arguments

Named arguments are also primarily a source-language feature.

ULABI function identity is based on stable parameter identifiers and types, not source-language syntax.

Example:

create_user(
    name: String,
    age: UInt32
)

Parameter identifiers SHOULD be represented in the interface metadata.


---

31. Register Model

ULABI defines semantic argument positions.

It does not mandate a universal physical register set.

A target adapter maps:

ULABI argument
      |
      v
Target ABI
      |
      +--> register
      +--> stack
      +--> memory
      +--> descriptor

This separation is mandatory for architecture neutrality.


---

32. Stack Model

A target implementation MAY use a stack-based calling convention.

ULABI itself does not require a stack.

The physical stack frame is an implementation detail unless a profile explicitly exposes it.

The semantic contract remains independent of stack layout.


---

33. Register Allocation

Register allocation is target-specific.

A ULABI implementation MUST NOT make a function incompatible merely because another implementation uses different physical registers.

Example:

ULABI:
    add(Int64, Int64)

x86-64:
    native registers

AArch64:
    different registers

RISC-V:
    different registers

All remain semantically compatible.


---

34. Stack Overflow and Resource Limits

Calling implementations MUST account for resource exhaustion.

Possible failures include:

StackOverflow
OutOfMemory
ArgumentTooLarge
ResourceLimit
CallDepthExceeded

Such failures MUST be represented according to the applicable error and runtime profiles.


---

35. Call Frames

A physical call frame MAY contain:

return information
saved state
arguments
temporary values
metadata
security context
capability context

ULABI does not mandate the physical layout.

Debugging and observability profiles MAY define standardized metadata for inspecting call frames.


---

36. Tail Calls

ULABI MAY support tail-call semantics.

A tail call SHOULD be explicitly declared or safely inferred by the implementation.

Tail-call optimization MUST NOT change observable semantics.


---

37. Callbacks

ULABI supports callbacks through function references.

Example:

register_callback(
    callback: FunctionRef<Event -> Unit>
)

A callback contract MUST define:

signature
lifetime
ownership
threading
execution context
cancellation
reentrancy
capabilities


---

38. Function References

A function reference is not necessarily a raw address.

It may represent:

FunctionID
CodeReference
ClosureReference
RemoteFunctionReference
CapabilityBoundFunction

Function references MUST be validated before invocation when required by the security profile.


---

39. Closures

Closures contain executable behavior plus captured state.

ULABI MUST NOT assume a universal closure memory layout.

A closure crossing a ULABI boundary SHOULD be represented through a standardized callable descriptor.

Conceptually:

CallableDescriptor {
    interface_id
    function_id
    environment
    capabilities
    lifetime
}


---

40. Reentrancy

Functions MUST declare reentrancy requirements where relevant.

Possible classifications:

Reentrant
NonReentrant
ThreadSafe
ThreadConfined
SingleExecutor

A caller MUST NOT assume thread safety unless the contract declares it.


---

41. Concurrency

ULABI calling semantics MUST remain independent of a particular threading model.

A function MAY specify:

ThreadSafe
ThreadConfined
ActorBound
ExecutorBound
Serialized
Concurrent

The runtime profile defines the implementation mechanism.


---

42. Determinism

A function MAY declare:

Deterministic
ConditionallyDeterministic
NonDeterministic

If deterministic, the implementation SHOULD produce equivalent results for equivalent inputs under equivalent declared environments.


---

43. Time and Deadlines

Time-sensitive functions SHOULD use explicit deadline or timeout parameters.

Example:

operation(
    input: Data,
    deadline: Timestamp
)

ULABI SHOULD avoid hidden timeout assumptions.


---

44. Resource Lifetime

Resources passed across a call boundary MUST have explicit lifetime semantics.

Possible lifetime policies:

CallerOwned
CalleeOwned
Shared
Borrowed
Transferred
SessionBound
ProcessBound
ConnectionBound

The applicable resource profile defines detailed release semantics.


---

45. Zero-Copy Calls

ULABI MAY support zero-copy data exchange.

Zero-copy is permitted only when:

ownership
lifetime
alignment
access permissions
memory visibility
security

are explicitly satisfied.

If these requirements cannot be safely satisfied, the implementation MUST copy or use another safe representation.


---

46. Cross-Process Calls

A ULABI function MAY be implemented across a process boundary.

The semantic function contract remains the same.

However, the implementation MUST account for:

serialization
transport failure
process failure
timeouts
capabilities
authentication
authorization
resource limits

A local call and remote call MUST NOT be treated as identical with respect to failure semantics.


---

47. Distributed Calls

Distributed calls MAY use a transport profile.

The function contract MUST explicitly indicate remote-capable behavior.

A local-only function MUST NOT silently become remotely executable.


---

48. ABI Negotiation

Before invoking a function, an implementation MAY negotiate:

ULABI version
interface version
function version
type versions
profiles
optional capabilities
transport
encoding
compression
security requirements

Negotiation MUST produce a deterministic compatibility decision.

Possible outcomes:

Compatible
CompatibleWithDowngrade
Incompatible
Unsupported
SecurityRejected
PolicyRejected


---

49. Version Compatibility

A function contract SHOULD include:

major_version
minor_version
patch_version

Major changes MAY be incompatible.

Minor changes SHOULD preserve compatibility where possible.

Patch changes SHOULD preserve binary and semantic compatibility.

Exact rules are defined by:

ULABI-VERSIONING.md


---

50. Signature Compatibility

A signature change MUST be classified.

Examples:

Compatible
ConditionallyCompatible
Breaking

Potential breaking changes include:

changing parameter meaning;

changing parameter type;

changing ownership;

changing return semantics;

changing error semantics;

removing required capabilities;

adding mandatory parameters;

changing execution semantics.



---

51. ABI Adapters

Language implementations SHOULD use an adapter layer.

Example:

Language
   |
   v
Language ABI Adapter
   |
   v
ULABI Calling Convention
   |
   v
Platform ABI Adapter

This prevents language-specific assumptions from entering the universal specification.


---

52. Native ABI Lowering

A native implementation MAY lower ULABI calls to:

x86-64 ABI
AArch64 ABI
RISC-V ABI
ARM ABI
PowerPC ABI
WebAssembly ABI
GPU ABI
embedded ABI

ULABI remains the semantic source of truth.


---

53. Transport Lowering

The same ULABI function MAY be lowered to:

DirectCall
SharedMemory
IPC
Socket
MessageQueue
RPC
QUIC
WebAssemblyHostCall
DeviceInterface
AcceleratorInterface

The transport adapter MUST preserve the declared semantics.


---

54. Security Context

A call MAY carry a security context.

Conceptually:

CallContext {
    caller_identity
    capabilities
    authorization
    security_policy
    isolation_domain
}

Security context MUST NOT be silently elevated during a call.


---

55. Call Context

A standardized logical call context SHOULD be capable of representing:

call_id
interface_id
function_id
version
caller
deadline
cancellation
capabilities
security_context
trace_context
locale
encoding
transport

Not every field is mandatory for every profile.


---

56. Observability

A call SHOULD be traceable without changing its semantic behavior.

The observability profile MAY provide:

trace_id
span_id
call_id
parent_call_id
timestamp
duration
status
error

Observability metadata MUST NOT alter function semantics.


---

57. Diagnostics

When a call fails, implementations SHOULD provide machine-readable diagnostics.

Example:

CallFailure {
    call_id
    function_id
    failure_class
    error_code
    message
    retryability
    recovery_hint
}


---

58. Retry Semantics

Retry behavior MUST NOT be assumed.

Functions MAY declare:

Retryable
NonRetryable
Idempotent
NonIdempotent
RetryAfter

A caller MUST NOT automatically retry a non-idempotent operation unless the contract explicitly permits it.


---

59. Transaction Semantics

ULABI does not require universal transactions.

A function MAY expose transaction semantics through a profile.

Possible classifications:

Atomic
Transactional
BestEffort
NonAtomic


---

60. Failure Atomicity

The function contract SHOULD define whether partial state changes are possible.

Example:

Atomic
PartialFailurePossible
RollbackSupported
CompensationRequired

This is particularly important for remote and resource-managing calls.


---

61. Self-Healing Interaction

The calling convention MUST support the ULABI Reliability and Self-Healing profiles without allowing arbitrary self-modification.

A failed call may produce:

Failure
   |
   v
Evidence
   |
   v
Diagnosis
   |
   v
Policy Check
   |
   +---- Known Recovery
   |
   +---- Retry
   |
   +---- Rollback
   |
   +---- Failover
   |
   +---- Escalate

The calling convention itself MUST NOT authorize arbitrary code modification.


---

62. Recovery Safety

Recovery operations MUST preserve:

ownership
security
authorization
call identity
data integrity
transaction semantics

A recovery action that cannot preserve these guarantees MUST be rejected or escalated.


---

63. Memory Safety

The calling convention MUST prevent:

use-after-free
double-free
invalid ownership transfer
dangling references
buffer overflow
type confusion
capability confusion

where the applicable memory and security profiles provide the necessary mechanisms.


---

64. Alignment

Physical alignment is architecture-specific.

The ULABI type system MUST provide sufficient metadata for an implementation to determine safe alignment.

The semantic contract MUST NOT depend on undocumented native alignment assumptions.


---

65. Endianness

ULABI semantic values MUST be independent of host endianness.

Canonical serialized representations MUST define byte order explicitly.

Local native calls MAY use native endianness internally.

Cross-boundary encodings MUST be deterministic.


---

66. Large Arguments

Implementations MUST support explicit handling for arguments that are too large for ordinary registers.

Possible mechanisms:

Stack
IndirectReference
Descriptor
SharedMemory
Streaming
Serialization

The choice is implementation-specific.


---

67. Argument Limits

Every implementation SHOULD define resource limits for:

argument count
argument size
nesting depth
string length
collection length
call depth
call duration
memory use

Limits MUST produce explicit failures.


---

68. Security Against Malformed Calls

Implementations MUST validate externally supplied calls before executing them.

Validation SHOULD include:

interface identity
function identity
version
signature
argument count
argument types
argument lengths
ownership
capabilities
security context
resource limits

Malformed input MUST NOT result in undefined behavior.


---

69. ABI Validation

A ULABI validator SHOULD be able to verify:

function identity
signature
parameter types
return types
ownership
effects
capabilities
execution semantics
version compatibility
profile requirements


---

70. Canonical Function Description

The standard SHOULD define a machine-readable function description.

Conceptual structure:

FunctionDescriptor {
    interface_id
    function_id
    version
    parameters[]
    return_type
    error_type
    ownership[]
    effects[]
    capabilities[]
    execution_mode
    determinism
    cancellation
    streaming
    compatibility
}

The exact serialization format belongs to the schema/encoding specifications.


---

71. Generic Functions

Generic functions MAY be exposed through type descriptors.

Example:

identity<T>(
    value: T
) -> T

The runtime representation MUST be explicitly defined.

Possible strategies:

monomorphization
type descriptors
dynamic dispatch
erased representation
specialized ABI

The Core MUST not assume one strategy.


---

72. Generic Type Safety

A generic boundary MUST preserve sufficient type information to prevent type confusion.

A caller MUST NOT substitute an incompatible type merely because the physical representation happens to have the same size.


---

73. Interface Evolution

New optional functions MAY be added without breaking existing callers when versioning rules permit.

Existing function semantics MUST NOT be silently changed.


---

74. Reserved Parameters

Future extensibility MAY be provided through explicitly defined optional metadata.

Implementations MUST NOT invent undocumented positional parameters.


---

75. Extension Mechanism

Calling-convention extensions SHOULD use:

Profile ID
Extension ID
Version
Capability declaration
Negotiation

An implementation MUST ignore unsupported optional extensions only when the contract explicitly permits doing so.


---

76. Unknown Features

Unknown mandatory calling-convention features MUST cause compatibility failure.

Unknown optional features MAY be ignored when safe.

This rule prevents accidental semantic corruption.


---

77. Safety Rule

> An implementation MUST prefer explicit incompatibility over unsafe or ambiguous interpretation.



This is one of the core ULABI calling-convention invariants.


---

78. Core Requirements

The ULABI Core Calling Convention MUST define:

stable function identity;

deterministic parameter ordering;

scalar argument semantics;

structured argument semantics;

return semantics;

explicit error semantics;

ownership semantics;

function effects;

capability requirements;

execution mode;

version compatibility;

ABI validation;

architecture-neutral lowering.



---

79. Optional Profiles

The following profiles MAY extend the calling convention:

Memory Profile
Async Profile
Streaming Profile
Zero-Copy Profile
Shared Memory Profile
Security Profile
Capability Profile
Sandbox Profile
Distributed Profile
Transport Profile
Hardware Profile
GPU Profile
Accelerator Profile
Real-Time Profile
Embedded Profile
Observability Profile
Debugging Profile
Safety-Critical Profile
Verification Profile
Reliability Profile
Self-Healing Profile


---

80. Implementation Architecture

The reference implementation SHOULD be divided into independent modules.

Recommended implementation tree:

implementations/
└── reference/
    ├── core/
    │   ├── abi/
    │   │   ├── calling_convention/
    │   │   │   ├── mod.*
    │   │   │   ├── function.*
    │   │   │   ├── argument.*
    │   │   │   ├── return_value.*
    │   │   │   ├── ownership.*
    │   │   │   ├── effects.*
    │   │   │   ├── capabilities.*
    │   │   │   ├── execution.*
    │   │   │   ├── validation.*
    │   │   │   ├── compatibility.*
    │   │   │   └── context.*
    │   │   │
    │   │   ├── types/
    │   │   ├── encoding/
    │   │   ├── errors/
    │   │   └── identity/
    │   │
    │   └── profiles/
    │
    ├── adapters/
    │   ├── native/
    │   ├── ipc/
    │   ├── distributed/
    │   ├── wasm/
    │   └── accelerator/
    │
    ├── validation/
    └── runtime/

The * extension represents the implementation language chosen for the reference implementation.

The specification itself is language-independent.


---

81. Required Code Modules

The following modules are required for a complete calling-convention implementation.

81.1 function

Responsibilities:

function identity;

signature;

version;

function descriptor;

compatibility metadata.


Required types:

FunctionId
FunctionSignature
FunctionDescriptor
InterfaceId
SignatureId


---

81.2 argument

Responsibilities:

argument representation;

argument ordering;

argument classification;

passing mode;

validation.


Required types:

Argument
ArgumentKind
ArgumentMode
ArgumentDescriptor
ArgumentList


---

81.3 return_value

Responsibilities:

return representation;

multiple return values;

ownership;

result handling.


Required types:

ReturnValue
ReturnDescriptor
ReturnKind


---

81.4 ownership

Responsibilities:

ownership transfer;

borrow;

share;

move;

consume;

lifetime metadata.


Required types:

OwnershipMode
Lifetime
BorrowContract
TransferContract


---

81.5 effects

Responsibilities:

function effects;

effect validation;

effect declarations.


Required types:

Effect
EffectSet
EffectDeclaration


---

81.6 capabilities

Responsibilities:

required capabilities;

delegated capabilities;

capability validation.


Required types:

CapabilityId
CapabilitySet
CapabilityRequirement
CapabilityContext


---

81.7 execution

Responsibilities:

synchronous calls;

asynchronous calls;

blocking;

streaming;

cancellation;

retryability;

determinism.


Required types:

ExecutionMode
CallMode
CancellationMode
RetryPolicy
Determinism


---

81.8 context

Responsibilities:

call context;

caller identity;

deadlines;

cancellation;

security context;

tracing metadata.


Required types:

CallContext
CallerIdentity
Deadline
CancellationToken
TraceContext


---

81.9 validation

Responsibilities:

function validation;

argument validation;

signature validation;

capability validation;

resource-limit validation.


Required interfaces:

FunctionValidator
ArgumentValidator
SignatureValidator
CapabilityValidator


---

81.10 compatibility

Responsibilities:

version compatibility;

signature compatibility;

profile compatibility;

feature negotiation.


Required types:

CompatibilityResult
CompatibilityClass
NegotiationRequest
NegotiationResponse


---

81.11 native

Responsibilities:

mapping ULABI calls to native platform ABIs.


Required components:

NativeAbiAdapter
RegisterMapper
StackMapper
CallingConventionLowerer


---

81.12 ipc

Responsibilities:

process-boundary invocation;

serialization;

transport;

process failure.


Required components:

IpcAdapter
IpcCall
IpcResponse
TransportAdapter


---

81.13 distributed

Responsibilities:

remote invocation;

timeout;

retry;

remote failure;

authentication.


Required components:

RemoteCallAdapter
RemoteCallContext
RemoteFailure
RetryController


---

81.14 wasm

Responsibilities:

WebAssembly host interoperability.


Required components:

WasmAbiAdapter
WasmTypeMapper
WasmCallBridge


---

81.15 accelerator

Responsibilities:

accelerator invocation;

buffer passing;

synchronization;

device capabilities.


Required components:

AcceleratorAbiAdapter
DeviceContext
BufferDescriptor
SynchronizationContract


---

82. Required Schemas

Machine-readable schemas SHOULD be created under:

schemas/

Recommended files:

schemas/
├── function-descriptor.schema.*
├── signature.schema.*
├── argument.schema.*
├── return-value.schema.*
├── ownership.schema.*
├── effect.schema.*
├── capability.schema.*
├── execution.schema.*
├── call-context.schema.*
├── compatibility.schema.*
└── negotiation.schema.*

The actual schema format SHOULD be selected once for the ULABI ecosystem and documented centrally.


---

83. Required Tests

Calling-convention tests MUST be organized independently.

Recommended structure:

tests/
└── abi/
    └── calling_convention/
        ├── function_identity.*
        ├── argument_order.*
        ├── scalar_arguments.*
        ├── aggregate_arguments.*
        ├── ownership.*
        ├── borrowing.*
        ├── moving.*
        ├── handles.*
        ├── capabilities.*
        ├── return_values.*
        ├── errors.*
        ├── effects.*
        ├── async.*
        ├── cancellation.*
        ├── streaming.*
        ├── callbacks.*
        ├── compatibility.*
        ├── malformed_calls.*
        ├── resource_limits.*
        ├── determinism.*
        ├── security.*
        └── cross_boundary.*


---

84. Conformance Tests

A conforming implementation MUST demonstrate at minimum:

Function identity              PASS
Argument ordering              PASS
Type classification             PASS
Ownership semantics             PASS
Return semantics                PASS
Error semantics                 PASS
Capability validation           PASS
Effect validation               PASS
Execution semantics             PASS
Version negotiation             PASS
Malformed input rejection      PASS
Resource limit enforcement     PASS
Architecture independence      PASS


---

85. Negative Tests

Conformance testing MUST include invalid cases.

Examples:

wrong function ID
wrong version
wrong signature
wrong argument count
wrong argument type
invalid ownership
invalid capability
invalid lifetime
oversized argument
malformed descriptor
unsupported mandatory feature
invalid cancellation
unauthorized call

The implementation MUST fail safely.


---

86. Fuzz Testing

Calling-convention parsers and validators SHOULD be fuzz tested against:

function descriptors
argument descriptors
type descriptors
call contexts
negotiation messages
serialized calls

The expected property is:

> Invalid input must never produce undefined behavior.




---

87. Cross-Language Testing

The conformance suite SHOULD eventually test multiple independent language adapters.

Example:

Language A
    |
    v
  ULABI
    ^
    |
Language B

Then:

Language A -> Language B
Language B -> Language A

The test suite SHOULD verify semantic equivalence.


---

88. Cross-Architecture Testing

The same ULABI function SHOULD be tested across:

x86-64
AArch64
RISC-V
WebAssembly

where implementations are available.

The semantic result MUST remain compatible.


---

89. ABI Difference Analysis

Tooling SHOULD compare two function contracts.

Example:

ULABI ABI Version A
        |
        v
    ABI Diff
        |
        v
ULABI ABI Version B

The tool SHOULD identify:

compatible changes
potentially breaking changes
definitely breaking changes
security changes
ownership changes
effect changes
capability changes


---

90. Formal Invariants

The calling convention MUST preserve the following invariants:

Invariant 1 — Deterministic Ordering

Arguments have deterministic semantic ordering.

Invariant 2 — Type Safety

Arguments MUST be interpreted according to their declared types.

Invariant 3 — Ownership Safety

Ownership MUST NOT silently change.

Invariant 4 — Capability Safety

Capabilities MUST NOT silently escalate.

Invariant 5 — Error Transparency

Errors MUST NOT silently become successful results.

Invariant 6 — Version Safety

Unsupported mandatory versions/features MUST fail explicitly.

Invariant 7 — Architecture Independence

Physical ABI differences MUST NOT change semantic function meaning.

Invariant 8 — Transport Independence

Changing transport MUST NOT silently change declared semantics.

Invariant 9 — Lifetime Safety

References MUST NOT outlive their permitted lifetime.

Invariant 10 — Explicit Failure

Ambiguous calls MUST fail rather than being guessed.


---

91. Security Requirements

Calling-convention implementations MUST defend against:

type confusion
argument confusion
function confusion
capability escalation
memory corruption
integer overflow
length confusion
use-after-free
double release
invalid function references
malformed descriptors
replay where applicable
unauthorized callbacks

Security-sensitive implementations SHOULD integrate with:

Security Profile
Capability Profile
Sandbox Profile
Memory Profile
Supply-Chain Security
Verification Profile


---

92. Safety-Critical Requirements

Safety-critical implementations MAY require additional constraints.

Examples:

bounded execution
deterministic behavior
static verification
restricted dynamic allocation
restricted recursion
certified adapters
traceable requirements
reproducible builds

These belong in a Safety-Critical Profile rather than making every ULABI implementation satisfy them.


---

93. Real-Time Requirements

Real-time profiles MAY define:

maximum call latency
maximum argument-copy cost
bounded allocation
priority
deadline
interruptibility
jitter

The base calling convention remains real-time-neutral.


---

94. Embedded Requirements

Embedded implementations MAY use:

static allocation
fixed-size descriptors
minimal metadata
no dynamic runtime
interrupt-safe calls
bounded stack usage

The semantic calling convention remains unchanged.


---

95. High-Performance Requirements

High-performance implementations MAY use:

register passing
zero-copy
shared memory
vectorized arguments
batched calls
asynchronous execution
accelerators

Optimization MUST NOT violate ULABI semantics.


---

96. Batched Calls

A profile MAY support:

Batch<Call>

This can reduce invocation overhead.

The profile MUST define:

ordering
partial failure
atomicity
individual results
cancellation


---

97. Vectorized Calls

A profile MAY expose vectorized operations.

Example:

add_vector(
    a: Vector<Float32>,
    b: Vector<Float32>
) -> Vector<Float32>

Vectorization MUST remain semantically independent of the underlying SIMD architecture.


---

98. Accelerator Calls

Accelerator calls MAY use:

GPU
NPU
FPGA
DSP
TPU-like accelerator
future accelerator

The calling convention SHOULD expose:

device
operation
input buffers
output buffers
synchronization
capabilities


---

99. Quantum and Future Execution Models

ULABI SHOULD permit future execution models without changing the Core.

A future execution profile MAY represent:

quantum computation
neuromorphic execution
specialized hardware
distributed accelerators
future architectures

The calling convention remains semantic.


---

100. Reference Call Lifecycle

A conforming call follows:

Caller
  |
  v
Resolve Function
  |
  v
Validate Version
  |
  v
Validate Signature
  |
  v
Validate Arguments
  |
  v
Validate Ownership
  |
  v
Validate Capabilities
  |
  v
Create Call Context
  |
  v
Lower to Execution Mechanism
  |
  v
Execute
  |
  +------ Failure ------+
  |                     |
  v                     v
Return Result       Recovery Policy
  |                     |
  +----------+----------+
             |
             v
          Validate
             |
             v
          Return


---

101. Required Integration With Other ULABI Documents

This document intentionally defines the calling-convention contract without duplicating the complete specifications of other components.

The following integrations are predefined.

docs/abi/core-abi.md

Defines:

Core ABI boundaries;

fundamental ABI concepts;

stable Core requirements.


Calling convention depends on the Core ABI.

The Core ABI MUST NOT require a specific physical register or stack layout.


---

docs/abi/data-types.md

Defines:

universal primitive types;

aggregate types;

type representations;

type identity.


Calling convention uses those type definitions for parameter and return classification.


---

docs/abi/memory-model.md

Defines:

ownership;

lifetime;

borrowing;

memory visibility;

transfer.


Calling convention consumes those rules when arguments cross a boundary.


---

docs/abi/register-model.md

Defines:

abstract register semantics;

physical lowering;

register classes;

architecture-specific mappings.


Calling convention uses it for physical lowering.


---

docs/abi/stack-model.md

Defines:

stack semantics;

frame rules;

stack resources.


Calling convention uses it when a target requires stack-based arguments or frames.


---

docs/abi/return-values.md

Defines:

return value representation;

result transport;

multiple return values.


Calling convention references these rules.


---

docs/abi/exception-model.md

Defines:

exception boundaries;

traps;

language-specific exception translation.


Calling convention uses the exception model for failure translation.


---

docs/interoperability/foreign-function-interface.md

Defines:

language adapters;

FFI boundaries;

language-specific translation.


Calling convention provides the universal target contract.


---

docs/interoperability/language-interoperability.md

Defines:

cross-language semantic interoperability.


Calling convention provides the invocation semantics.


---

docs/runtime/runtime-interface.md

Defines:

runtime services;

execution context;

resource management.


Calling convention may request runtime services but must not depend on one runtime.


---

docs/security/security-model.md

Defines:

global security requirements.


Calling convention must enforce relevant call-level security.


---

docs/security/capability-security.md

Defines:

capability semantics.


Calling convention carries and validates capability requirements.


---

docs/reliability/self-healing.md

Defines:

failure detection;

diagnosis;

recovery;

rollback.


Calling convention exposes failures to the reliability subsystem.


---

docs/compatibility/feature-negotiation.md

Defines:

feature negotiation;

capability discovery;

compatibility decisions.


Calling convention uses those rules before invocation.


---

docs/standards/conformance.md

Defines:

what constitutes conformance.


Calling convention supplies the required conformance behaviors.


---

docs/standards/test-suite.md

Defines:

standard test organization.


Calling convention tests MUST be included there.


---

102. Integration Contract — No Future Re-Editing Requirement

This document is designed so that the calling convention can be implemented independently.

Later documents MUST NOT require changing the fundamental calling-convention semantics merely to integrate with them.

Future components MUST integrate through:

interfaces
profiles
type descriptors
capability descriptors
extension identifiers
version negotiation

rather than modifying existing calling-convention rules unnecessarily.

If a future feature cannot integrate through these mechanisms, it MUST be treated as a new versioned extension or profile.


---

103. Required Code File Plan

The implementation work should proceed in this order.

Phase 1 — Independent Core Calling Components

Implement first:

function.*
argument.*
return_value.*
ownership.*
effects.*
capabilities.*
execution.*
context.*

These should not depend on transport or platform-specific code.


---

Phase 2 — Validation

Implement:

validation/function_validator.*
validation/argument_validator.*
validation/signature_validator.*
validation/capability_validator.*

These consume Phase 1 contracts.


---

Phase 3 — Compatibility

Implement:

compatibility/version.*
compatibility/signature.*
compatibility/profile.*
compatibility/negotiation.*

These consume the Core descriptors.


---

Phase 4 — Platform Lowering

Implement:

native/abi_adapter.*
native/register_mapper.*
native/stack_mapper.*
native/calling_convention_lowerer.*

These integrate with:

register-model.md
stack-model.md


---

Phase 5 — Process Boundary

Implement:

ipc/ipc_adapter.*
ipc/ipc_call.*
ipc/ipc_response.*
ipc/transport_adapter.*


---

Phase 6 — Distributed Boundary

Implement:

distributed/remote_call_adapter.*
distributed/remote_call_context.*
distributed/remote_failure.*
distributed/retry_controller.*


---

Phase 7 — Specialized Execution

Implement:

wasm/wasm_abi_adapter.*
accelerator/accelerator_abi_adapter.*
accelerator/device_context.*
accelerator/buffer_descriptor.*


---

104. Recommended Reference Implementation Language

The ULABI specification remains language-independent.

For the first reference implementation, a systems language SHOULD be selected.

The implementation should prioritize:

memory safety
explicit data representation
portable binaries
cross-platform support
low runtime overhead
strong tooling
FFI capability

Rust is a strong candidate for the reference implementation.

However:

> Rust is an implementation choice, not part of the ULABI standard.



ULABI MUST remain implementable in other languages.


---

105. Example Cross-Language Invocation

Conceptual source language A:

result = calculate(42)

ULABI contract:

Function:
    calculate

Input:
    Int64

Output:
    Result<Int64, CalculationError>

Language A adapter:

Language A
    |
    v
ULABI Adapter

Language B implementation:

ULABI
    |
    v
Language B Adapter
    |
    v
calculate()

The two languages do not need to know each other's source syntax.


---

106. Example Ownership Transfer

Caller:

send_buffer(
    buffer: Move<Buffer>
)

Contract:

ownership = transfer

After successful transfer:

Caller
   |
   | ownership transferred
   v
Callee

The caller MUST NOT continue using the buffer unless the contract explicitly permits shared access.


---

107. Example Borrow

hash(
    data: Borrow<Bytes>
) -> Hash

The callee may read the data during the permitted lifetime.

The callee MUST NOT retain the reference beyond that lifetime.


---

108. Example Capability

read_secret(
    capability: SecretReadCapability
) -> Result<Bytes, AccessError>

The capability is explicit.

The function does not receive unrestricted access to unrelated resources.


---

109. Example Async Call

download(
    url: String
) -> Future<Result<Bytes, NetworkError>>

The contract defines:

asynchronous
network effect
cancellable
failure-producing


---

110. Example Streaming Call

stream_records()
    -> Stream<Result<Record, Error>>

The stream defines:

next
complete
error
cancel
backpressure


---

111. Example Remote Call

lookup_user(
    id: UserId
) -> Result<User, UserError>

If marked remote-capable:

Caller
  |
ULABI
  |
Transport
  |
Remote ULABI
  |
Implementation

The contract must additionally specify:

timeout
authentication
authorization
retry policy
remote failure


---

112. Implementation Independence

A conforming implementation MAY use:

C
C++
Rust
Go
Java
Python
Swift
Kotlin
Ada
Fortran
Zig
D
other languages

No language is privileged by the specification.

In particular:

Zamani != Sankofa
Zamani != ULABI
Sankofa != ULABI

Both may independently implement ULABI.


---

113. Conformance Levels

The calling convention MAY eventually define:

Level 0 — Descriptor Conformance
Level 1 — Local Call Conformance
Level 2 — Memory-Safe Call Conformance
Level 3 — Cross-Process Conformance
Level 4 — Distributed Call Conformance
Level 5 — Advanced Profile Conformance

An implementation MUST clearly state which level it supports.


---

114. Certification

Certification SHOULD verify:

function identity
signature handling
argument handling
return handling
ownership
capabilities
errors
versioning
validation
security
failure behavior

Certification MUST be based on reproducible tests rather than implementation claims.


---

115. Interoperability Guarantee

A ULABI-conforming implementation SHOULD be interoperable with another conforming implementation when:

same interface
compatible function version
compatible signature
compatible types
compatible profiles
compatible security policy
compatible transport

are established.


---

116. Final Architectural Rule

The calling convention MUST preserve the central ULABI principle:

> ULABI defines what interoperating components mean by a call; implementations define how that call is physically performed.



This distinction is essential.

The ULABI calling convention is therefore:

Language Neutral
Runtime Neutral
OS Neutral
CPU Neutral
Transport Neutral
Vendor Neutral
Implementation Neutral

while still being precise enough to produce deterministic implementations.


---

117. Definition of Done

docs/abi/calling-convention.md is considered complete for the current specification version when:

[ ] Function identity defined
[ ] Function contract defined
[ ] Argument ordering defined
[ ] Argument classification defined
[ ] Argument modes defined
[ ] Ownership defined
[ ] Borrowing defined
[ ] Moving defined
[ ] Handles defined
[ ] Capabilities defined
[ ] Return semantics defined
[ ] Error semantics defined
[ ] Effects defined
[ ] Execution modes defined
[ ] Async semantics defined
[ ] Cancellation defined
[ ] Streaming defined
[ ] Callback semantics defined
[ ] Function references defined
[ ] Reentrancy defined
[ ] Concurrency defined
[ ] Determinism defined
[ ] Resource lifetime defined
[ ] Zero-copy rules defined
[ ] Process boundary defined
[ ] Distributed boundary defined
[ ] ABI negotiation defined
[ ] Version compatibility defined
[ ] Security requirements defined
[ ] Failure behavior defined
[ ] Recovery integration defined
[ ] Conformance requirements defined
[ ] Test requirements defined
[ ] Code-module plan defined
[ ] Integration contracts defined
[ ] Future-extension strategy defined

No implementation-specific rule should be added to the universal specification merely because a particular language, operating system, CPU, compiler, or runtime prefers it.


---

118. Status

Current status:

Specification:
    Draft

Architecture:
    Defined

Semantic Calling Convention:
    Designed

Physical ABI Lowering:
    Extension/implementation responsibility

Reference Implementation:
    Not yet implemented

Conformance Suite:
    Planned

Certification:
    Planned


---

119. Next Documents

After this document, implementation should proceed against the following specifications without modifying the calling-convention contract unnecessarily:

docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/register-model.md
docs/abi/stack-model.md
docs/abi/return-values.md
docs/abi/exception-model.md
docs/interoperability/foreign-function-interface.md
docs/runtime/runtime-interface.md
docs/security/security-model.md
docs/compatibility/feature-negotiation.md
docs/standards/conformance.md
docs/standards/test-suite.md

The first implementation modules should remain independent from those later documents wherever possible, using the integration contracts already defined above.

### Required code modules — consolidated list

For implementation planning, the calling-convention subsystem should ultimately contain these logical modules:

```text
calling_convention/
├── function
├── argument
├── return_value
├── ownership
├── effects
├── capabilities
├── execution
├── context
├── validation
├── compatibility
│
├── native
│   ├── abi_adapter
│   ├── register_mapper
│   ├── stack_mapper
│   └── calling_convention_lowerer
│
├── ipc
│   ├── ipc_adapter
│   ├── ipc_call
│   ├── ipc_response
│   └── transport_adapter
│
├── distributed
│   ├── remote_call_adapter
│   ├── remote_call_context
│   ├── remote_failure
│   └── retry_controller
│
├── wasm
│   ├── wasm_abi_adapter
│   ├── wasm_type_mapper
│   └── wasm_call_bridge
│
└── accelerator
    ├── accelerator_abi_adapter
    ├── device_context
    ├── buffer_descriptor
    └── synchronization_contract

Important: I have deliberately separated the semantic calling convention from the later physical register/stack lowering. That prevents ULABI from becoming accidentally tied to x86, ARM, Rust, C, or any other implementation choice. This follows the architecture already established in your repository.

And the current repository is still at the documentation/pre-implementation stage; the README is currently only a minimal project introduction.