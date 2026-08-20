I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI before preparing this. The repository currently has the master design at specification version 0.2.0-draft, and its architecture explicitly establishes ULABI as language-, runtime-, OS-, CPU-, and vendor-neutral, with a Minimal Core + Standard Profiles + Extensible Ecosystem model.

I also checked the existing docs/abi/ tree. The ABI documents already exist, including core-abi.md, calling-convention.md, data-types.md, memory-model.md, register-model.md, exception-model.md, return-values.md, and stack-model.md.

Because you asked to rewrite doc/abi/core-abi.md, note that the repository path is actually docs/abi/core-abi.md (plural docs). I would use the existing repository path rather than create a second doc/ tree.

1. Dependency order for the ULABI implementation

The important change I recommend is that we do not implement files according to the order in which they happen to appear in the documentation tree.

We should build the standard in dependency order:

PHASE 0 — GOVERNANCE / IDENTITY
        │
        ├── ULABI-GOVERNANCE.md
        ├── ULABI-VERSIONING.md
        └── specification identifiers
        │
        ▼
PHASE 1 — FOUNDATIONAL CONTRACTS
        │
        ├── ULABI-SPEC.md
        ├── schemas/interface-id.schema.*
        ├── schemas/type-id.schema.*
        ├── schemas/version.schema.*
        └── schemas/error.schema.*
        │
        ▼
PHASE 2 — CORE ABI
        │
        ├── docs/abi/core-abi.md
        ├── docs/abi/data-types.md
        ├── docs/abi/calling-convention.md
        ├── docs/abi/register-model.md
        ├── docs/abi/stack-model.md
        ├── docs/abi/memory-model.md
        ├── docs/abi/exception-model.md
        └── docs/abi/return-values.md
        │
        ▼
PHASE 3 — INTEROPERABILITY
        │
        ├── language-interoperability.md
        ├── foreign-function-interface.md
        ├── object-model.md
        ├── name-mangling.md
        ├── symbol-resolution.md
        └── cross-language-data.md
        │
        ▼
PHASE 4 — RUNTIME
        │
        ├── runtime-interface.md
        ├── process-model.md
        ├── threading.md
        ├── async-model.md
        ├── concurrency.md
        └── resource-management.md
        │
        ▼
PHASE 5 — MEMORY
        │
        ├── memory-safety.md
        ├── ownership.md
        ├── lifetimes.md
        ├── allocation.md
        ├── virtual-memory.md
        └── shared-memory.md
        │
        ▼
PHASE 6 — SECURITY
        │
        ├── security-model.md
        ├── capability-security.md
        ├── sandboxing.md
        ├── cryptography.md
        ├── authentication.md
        ├── authorization.md
        ├── zero-trust.md
        ├── secure-loading.md
        └── supply-chain-security.md
        │
        ▼
PHASE 7 — DISTRIBUTED / RELIABILITY
        │
        ├── distributed-abi.md
        ├── remote-calls.md
        ├── serialization.md
        ├── self-healing.md
        ├── fault-detection.md
        ├── fault-isolation.md
        ├── recovery.md
        └── rollback.md
        │
        ▼
PHASE 8 — COMPATIBILITY
        │
        ├── backwards-compatibility.md
        ├── forwards-compatibility.md
        ├── feature-negotiation.md
        ├── capability-discovery.md
        └── graceful-degradation.md
        │
        ▼
PHASE 9 — CONFORMANCE
        │
        ├── conformance.md
        ├── compliance-levels.md
        ├── test-suite.md
        ├── certification.md
        └── reference-implementations.md

The key principle is:

> A document may depend on another document, but it must not require a future rewrite merely because that dependent document is implemented later.



That is why the contract below defines its integration points explicitly now.


---

2. docs/abi/core-abi.md

Below is the replacement document.

Copy everything from the next heading through the end into:

docs/abi/core-abi.md


---

ULABI Core ABI Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: Core ABI Specification
Path: docs/abi/core-abi.md
Specification Family: ABI
Status: Draft Normative Specification
Specification Version: 0.3.0-draft
Core Profile: ULABI-Core
Normative Language: RFC 2119 / RFC 8174 terminology
License: Apache-2.0


---

1. Purpose

This document defines the ULABI Core ABI.

The Core ABI is the smallest stable contract required for independently implemented software components to interoperate through ULABI.

ULABI Core defines:

interface identity;

callable interface contracts;

parameter and result boundaries;

universal ABI types;

representation boundaries;

ownership boundaries;

lifetime boundaries;

execution semantics;

error propagation;

capability declarations;

deterministic behavior requirements;

compatibility requirements;

validation requirements;

ABI discovery requirements.


ULABI Core does not define:

a programming language;

a compiler;

a CPU instruction set;

an operating system;

a specific runtime;

a garbage collector;

a particular object model;

a particular transport;

a particular network protocol;

a particular vendor implementation.


An implementation MAY use any internal architecture provided that its externally observable ULABI behavior conforms to this specification.


---

2. Architectural Position

ULABI follows:

ULABI-DESIGN.md
                       │
                       ▼
                 ULABI Core ABI
                       │
        ┌──────────────┼──────────────┐
        │              │              │
      Types          Calls          Errors
        │              │              │
        └──────────────┼──────────────┘
                       │
                Extension Profiles
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    Security        Async          Distributed
       │               │                │
       └───────────────┼────────────────┘
                       │
                 Implementations

The Core is intentionally smaller than the complete ULABI ecosystem.

Advanced behavior belongs in profiles.


---

3. Normative Terminology

The following terms are normative.

MUST

The requirement is mandatory for conformance.

MUST NOT

The behavior is prohibited.

SHOULD

The behavior is recommended unless a documented implementation constraint exists.

SHOULD NOT

The behavior is discouraged unless a documented reason exists.

MAY

The behavior is optional.

Core

The mandatory ULABI interoperability contract.

Profile

A standardized extension of ULABI Core.

Implementation

A compiler, runtime, library, operating-system component, language binding, service, device, or other system implementing ULABI.

Provider

The component implementing an exported ULABI interface.

Consumer

The component invoking or consuming a ULABI interface.

Boundary

The point at which ULABI-defined semantics cross between independently implemented components.


---

4. Core Design Principles

ULABI Core MUST satisfy the following principles.

4.1 Language neutrality

No language-specific ABI assumption may be required.

For example:

C ABI        ┐
Rust ABI     │
Python ABI   │
Java ABI     ├──> ULABI Core
Go ABI       │
Zamani ABI   │
Sankofa ABI  ┘

ULABI is the common contract.

It is not an abstraction of any one language.


---

4.2 Runtime neutrality

ULABI Core MUST NOT require:

reference counting;

tracing garbage collection;

ownership checking;

borrow checking;

manual allocation;

automatic allocation;

a specific scheduler;

a specific exception runtime.


Implementations translate their internal runtime semantics into the ULABI contract.


---

4.3 Architecture neutrality

ULABI Core MUST NOT assume:

32-bit CPUs;

64-bit CPUs;

little-endian CPUs;

big-endian CPUs;

a fixed register count;

a fixed stack architecture;

a fixed pointer width.


Architecture-specific mappings belong to platform profiles.


---

4.4 Transport neutrality

The same ULABI interface contract MAY be realized through:

direct in-process calls;

shared memory;

operating-system IPC;

process messaging;

local sockets;

network transport;

WebAssembly host calls;

accelerator interfaces.


Transport behavior MUST NOT silently alter the semantic contract.


---

5. Core ABI Object Model

A ULABI interface consists of:

Interface
 ├── Interface Identity
 ├── Version
 ├── Capabilities
 ├── Functions
 │    ├── Function Identity
 │    ├── Parameters
 │    ├── Result
 │    ├── Errors
 │    ├── Effects
 │    └── Execution Semantics
 ├── Types
 ├── Ownership Rules
 ├── Lifetime Rules
 └── Compatibility Rules


---

6. Interface Identity

Every externally visible ULABI interface MUST have a globally stable identity.

An interface identifier MUST NOT depend solely on:

source filename;

memory address;

compiler-generated local number;

process ID;

executable location.


The identity SHOULD be represented as a canonical identifier.

Conceptually:

ULABI Interface ID
    |
    +-- Namespace
    +-- Name
    +-- Major Version
    +-- Identity Digest

The exact wire representation is defined by the ULABI identity schema.


---

7. Function Identity

Every externally callable function MUST have a stable function identity within its interface.

A function identity MUST remain stable across compatible implementation changes.

A function identity MUST NOT be derived exclusively from:

source line;

linker address;

register address;

generated machine-code address.


Conceptually:

InterfaceID
    +
FunctionID
    +
Signature
    +
Semantic Contract

defines a callable ULABI operation.


---

8. Function Contract

A ULABI function contract contains:

Function
├── identity
├── name
├── parameters
├── result
├── errors
├── ownership
├── lifetime
├── effects
├── capabilities
├── execution mode
├── determinism
├── cancellation semantics
└── compatibility metadata

Example:

Function:
    calculate

Parameters:
    input: Int64

Result:
    Result<Int64, CalculationError>

Execution:
    synchronous

Effects:
    Pure

Capabilities:
    None

The source language may represent this completely differently.


---

9. Parameter Contract

Each parameter MUST specify:

1. stable parameter identity;


2. semantic type;


3. passing mode;


4. ownership mode;


5. lifetime requirements;


6. nullability/optionality semantics;


7. validation requirements.



Passing modes include:

Value
Reference
Borrowed
Owned
Handle
Slice
Stream
Capability

Only modes defined by the applicable ULABI specification or profile may be used.


---

10. Result Contract

Each function MUST define its result semantics.

A function MAY return:

Unit
Value
Option<T>
Result<T,E>
Tuple
Record
Handle
Stream
Future

The result contract MUST explicitly define:

representation;

ownership;

lifetime;

failure behavior;

validity;

cleanup responsibility.


A function MUST NOT communicate undocumented semantic results through hidden ABI state.


---

11. Universal Boundary Types

ULABI Core defines the foundational semantic type families:

Bool
Int
UInt
Float
Char
String
Bytes
Unit
List
Record
Enum
Variant
Option
Result
Handle

Additional types belong to standardized extensions.

The complete binary representation is defined by the type-system and encoding specifications.


---

12. Type Identity

Every non-primitive externally shared type SHOULD have a stable type identity.

Type identity MUST remain independent of:

source language;

compiler;

implementation memory layout;

local symbol name.


For example:

ULABI.Type.Person

is conceptually different from:

Rust struct Person
C struct Person
Python class Person

The implementations may have completely different layouts.


---

13. Representation Independence

ULABI separates:

Semantic Type
       │
       ▼
ULABI Boundary Representation
       │
       ▼
Implementation Representation

A provider MUST be allowed to translate its internal representation into the canonical ULABI representation.

A consumer MUST NOT assume the provider's internal representation.


---

14. Calling Boundary

The calling boundary is divided into:

Semantic Call
     │
     ▼
ULABI Call Contract
     │
     ▼
ABI Mapping
     │
     ▼
Platform Calling Convention

ULABI Core specifies the semantic contract.

docs/abi/calling-convention.md specifies how that contract maps to concrete ABI mechanisms.

Therefore Core MUST NOT hardcode:

x86-64 registers;

AArch64 registers;

RISC-V registers;

JVM operand conventions;

WebAssembly stack conventions.


Those belong to platform-specific mappings.


---

15. Register Independence

ULABI Core treats registers as an implementation concern.

A ULABI interface MUST NOT require a specific physical register.

The register mapping is defined by:

docs/abi/register-model.md

and platform profiles.

The same function contract MUST therefore be representable on architectures with:

0 directly usable argument registers

through architectures with many argument registers.


---

16. Stack Independence

ULABI Core MUST NOT require a particular stack layout.

Stack layout belongs to:

docs/abi/stack-model.md

An implementation MAY use:

hardware stack;

software stack;

segmented stack;

split stack;

heap-based activation records;

register-only calls;

coroutine frames.


The semantic ULABI call contract remains unchanged.


---

17. Memory Boundary

ULABI Core defines semantic memory boundaries but does not mandate one memory-management strategy.

The boundary MUST explicitly identify:

ownership
lifetime
mutability
aliasing
transfer
release

Detailed memory semantics are specified by:

docs/abi/memory-model.md
docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md


---

18. Ownership

Every resource crossing a ULABI boundary MUST have an ownership rule.

Supported semantic states include:

Borrowed
Owned
Shared
Transferred
HandleOwned
HandleBorrowed
Immutable
Mutable

An implementation MUST NOT leave ownership ambiguous.

If ownership is transferred:

Provider
   │
   │ transfer
   ▼
Consumer

the provider MUST NOT continue using the transferred resource except where the contract explicitly permits shared use.


---

19. Lifetime

Every borrowed or referenced value MUST have a defined lifetime.

The contract MUST specify whether a value is valid:

DuringCall
UntilReturn
UntilRelease
UntilHandleClose
ForInterfaceLifetime
ExplicitlyPinned

An implementation MUST NOT retain a borrowed object beyond its permitted lifetime.


---

20. Handles

Opaque resources SHOULD be represented using ULABI handles when direct representation would violate abstraction or safety.

Examples:

FileHandle
SocketHandle
DeviceHandle
ProcessHandle
GpuBufferHandle
MemoryHandle
CapabilityHandle

A handle MUST NOT expose implementation-specific internal addresses as its semantic identity.


---

21. Nullability

ULABI Core MUST NOT use an undocumented null pointer convention.

Optional values MUST use:

Option<T>

unless a specialized profile explicitly defines another representation.

This prevents ambiguity between:

missing
invalid
zero
null
empty


---

22. Errors

ULABI Core recognizes errors as explicit contract values.

The preferred semantic model is:

Result<T,E>

where:

T = success
E = failure

Exception-based language runtimes MAY map their exceptions into ULABI error semantics.

They MUST NOT require the consumer to understand the provider's private exception runtime.

Detailed rules are defined by:

docs/abi/exception-model.md


---

23. Error Identity

Errors crossing a ULABI boundary MUST have stable identities.

An error SHOULD contain:

ErrorID
Category
Severity
Message
StructuredData
Recoverability
Retryability
Origin

Human-readable messages MUST NOT be the only machine-readable error identity.


---

24. Determinism

Where a function is declared deterministic, identical valid inputs and equivalent execution context MUST produce semantically equivalent outputs.

A deterministic contract MUST NOT silently depend on:

wall-clock time;

process identity;

random state;

memory addresses;

unspecified iteration order;

environment-specific behavior.


Nondeterministic functions MUST declare the relevant effect.


---

25. Effects

ULABI functions MAY declare effects such as:

Pure
ReadsMemory
WritesMemory
ReadsResource
WritesResource
Filesystem
Network
Process
Time
Randomness
GPU
ExternalDevice
NonDeterministic

Effect declarations provide machine-readable metadata for:

validators;

security systems;

sandboxing;

static analysis;

conformance testing.


An implementation MUST NOT claim a stronger effect guarantee than its behavior provides.


---

26. Capabilities

A ULABI function MAY require capabilities.

Examples:

Filesystem.Read
Filesystem.Write
Network.Connect
Process.Spawn
GPU.Execute
Device.Access
Memory.Shared

Capabilities MUST be explicit.

A function MUST NOT acquire an undeclared privileged capability merely because its implementation requires it.


---

27. Execution Semantics

Functions MUST declare relevant execution properties.

Possible properties include:

Synchronous
Asynchronous
Blocking
NonBlocking
Streaming
OneShot
LongRunning
Cancellable
Idempotent
NonIdempotent

A function advertised as non-blocking MUST NOT perform undocumented unbounded blocking.


---

28. Reentrancy

A function MUST declare reentrancy restrictions where relevant.

Possible states:

Reentrant
NonReentrant
ThreadCompatible
ThreadRestricted
Serialized

If an interface is non-reentrant, the contract MUST define the behavior of concurrent invocation attempts.


---

29. Thread Safety

Thread safety MUST be explicit.

An interface MAY specify:

ThreadSafe
ThreadCompatible
ThreadAffine
SingleThreadOnly
ExternallySerialized

A consumer MUST NOT infer thread safety merely from successful local testing.


---

30. Concurrency Independence

ULABI Core does not mandate a particular concurrency model.

Implementations MAY use:

OS threads;

green threads;

fibers;

actors;

event loops;

async tasks;

processes;

hardware execution units.


The ULABI contract describes observable semantics rather than implementation strategy.


---

31. Serialization Boundary

A ULABI interface that crosses a process or machine boundary MUST have an explicit representation.

The serialization mechanism MUST preserve:

type identity;

field identity;

value semantics;

error semantics;

ownership semantics where representable;

version information.


Serialization is specified further by:

docs/distributed/serialization.md


---

32. Local Versus Remote Execution

ULABI MUST distinguish execution locality.

Supported semantic classifications include:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A function declared LocalOnly MUST NOT silently become a remote operation.

Remote-capable interfaces MUST define relevant:

latency;

timeout;

retry;

cancellation;

partial failure;

authentication;

authorization;

consistency semantics.



---

33. Interface Discovery

A ULABI implementation MAY expose interface discovery metadata.

Discovery SHOULD provide:

InterfaceID
Version
Profiles
Functions
Types
Capabilities
SecurityRequirements
CompatibilityInformation

Discovery MUST NOT grant capabilities by itself.

Discovery answers:

> What does this component support?



It does not answer:

> What is this caller authorized to do?




---

34. Versioning

Every ULABI interface MUST have a version.

Version compatibility is governed by:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

A compatible implementation MUST NOT silently reinterpret an existing function's semantic contract.


---

35. ABI Stability

The following MUST remain stable within a compatible interface version:

interface identity;

function identity;

semantic parameter types;

result semantics;

ownership rules;

lifetime rules;

error identities;

required capability semantics.


Implementation internals MAY change freely.


---

36. Extension Profiles

ULABI Core MUST remain extensible without continuously expanding the Core.

Profiles MAY define:

Memory
Security
Async
Streaming
Zero-Copy
Shared-Memory
Distributed
Hardware
GPU
Tensor
Real-Time
Embedded
Observability
Debugging
Safety-Critical
Verification
Reliability
Self-Healing

A profile MUST:

1. have a stable identity;


2. declare its required Core version;


3. define additional interfaces;


4. define additional types;


5. define compatibility rules;


6. define conformance tests;


7. define failure behavior.




---

37. Profile Isolation

An implementation claiming:

ULABI Core

MUST NOT automatically claim support for optional profiles.

For example:

ULABI Core       ✓
ULABI Security   ✓
ULABI Async      ✗
ULABI Distributed ✓

is valid.

Conformance MUST therefore be capability/profile based rather than a single binary "ULABI compatible" claim.


---

38. Validation

A ULABI implementation MUST validate externally supplied interface metadata before trusting it.

Validation MUST include, as applicable:

identifier validity;

version validity;

type validity;

function signature validity;

capability declarations;

ownership declarations;

profile requirements;

compatibility constraints.


Malformed contracts MUST be rejected deterministically.


---

39. Security Boundary

ULABI Core treats every external boundary as potentially untrusted unless the applicable security profile establishes trust.

An implementation MUST NOT assume that:

same machine

means:

trusted component

Likewise:

same process

does not automatically imply that all ULABI data is valid.

Detailed security requirements belong to:

docs/security/security-model.md


---

40. Memory Safety

Implementations MUST prevent invalid boundary behavior such as:

use-after-free;

double release;

invalid ownership transfer;

out-of-bounds access;

incompatible type interpretation;

lifetime violations.


The Core defines the contract.

The implementation chooses the mechanism.


---

41. Representation Validation

Before interpreting boundary data, an implementation MUST establish that the representation satisfies the applicable ULABI schema.

An implementation MUST NOT reinterpret arbitrary bytes as a valid ULABI object without validation where validation is required.


---

42. Canonical Representation

Where ULABI defines a canonical representation, equivalent semantic values MUST have the same canonical representation.

Canonical representation is necessary for:

hashing;

signing;

comparison;

caching;

ABI verification;

deterministic serialization;

reproducible builds.



---

43. ABI Hashing

A ULABI interface MAY have a canonical ABI digest.

The digest SHOULD be derived from canonical semantic metadata rather than implementation machine code.

Conceptually:

Interface
   ↓
Canonical Contract
   ↓
Canonical Encoding
   ↓
Hash
   ↓
ABI Identity / Verification

Changing a semantically significant field MUST change the digest.

Changes that do not alter the contract SHOULD NOT alter the semantic ABI identity.


---

44. Symbol Resolution

The mapping from source-language symbols to ULABI identities MUST be deterministic.

ULABI Core does not mandate a particular source-level name mangling algorithm.

Instead:

Source Symbol
      ↓
Language Adapter
      ↓
ULABI Symbol Identity
      ↓
Provider

Detailed rules belong to:

docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md


---

45. Language Adapters

Every language integration MUST have a translation layer.

Conceptually:

Language
   │
   ▼
Language Adapter
   │
   ▼
ULABI Contract
   │
   ▼
ULABI Implementation

The adapter is responsible for mapping:

types;

calls;

errors;

ownership;

lifetimes;

effects;

capabilities;

execution semantics.


ULABI Core remains independent of the language.


---

46. Foreign Function Interface

A language MAY expose ULABI functions through its native FFI.

The FFI implementation MUST preserve the ULABI contract.

It MUST NOT silently change:

ownership;

lifetime;

error semantics;

integer width;

string encoding;

calling semantics.


Detailed requirements belong to:

docs/interoperability/foreign-function-interface.md


---

47. ABI Boundary Safety

An implementation MUST treat boundary conversion as a security and correctness boundary.

Conversion code MUST validate:

Type
Size
Alignment
Lifetime
Ownership
Range
Encoding
Capability

where applicable.


---

48. Alignment

ULABI Core does not require a universal native-memory alignment.

Boundary formats MUST define their own alignment requirements where applicable.

A platform adapter MUST translate between:

ULABI alignment

and:

native alignment

without producing undefined behavior.


---

49. Integer Semantics

ULABI integer types MUST have explicit:

signedness;

width;

range;

overflow semantics;

encoding;

byte order where serialized.


An implementation MUST NOT substitute architecture-dependent int semantics for a fixed-width ULABI type without an explicit compatible mapping.


---

50. Floating-Point Semantics

ULABI floating-point contracts MUST define supported representation and exceptional values.

Where floating-point behavior is exposed, the implementation MUST document handling of:

NaN
+Infinity
-Infinity
+0
-0

The canonical encoding MUST be specified by the type/encoding specification.


---

51. String Semantics

ULABI Core treats strings as semantic Unicode text.

The canonical interchange representation SHOULD use UTF-8.

A string boundary MUST define:

encoding;

validity;

length semantics;

maximum size;

invalid sequence behavior.


A byte buffer MUST NOT be silently interpreted as a string.


---

52. Bytes

Bytes represents arbitrary binary data.

It MUST remain semantically distinct from:

String

because byte sequences do not necessarily represent valid text.


---

53. Collections

Collection contracts MUST specify:

element type;

cardinality;

ordering;

ownership;

lifetime;

mutation;

maximum supported size.


A collection MUST NOT expose an implementation-specific internal iterator as the universal ABI contract.


---

54. Records

Records MUST define stable field identities.

A record contract MUST specify:

FieldID
FieldType
Required/Optional
Default semantics
Unknown-field behavior

Field order MUST NOT be semantically significant unless the specific type contract declares it significant.


---

55. Variants

Variants MUST define stable variant identities.

Unknown variants MUST have explicitly defined compatibility behavior.

An implementation MUST NOT reinterpret an unknown variant as a different known variant.


---

56. Option

Option<T> has exactly two semantic states:

None
Some(T)

None MUST NOT be represented through an undocumented sentinel value.


---

57. Result

Result<T,E> has exactly two semantic states:

Ok(T)
Err(E)

The provider MUST NOT return an Err value while simultaneously claiming successful Ok semantics.


---

58. Resource Semantics

Resources that cannot safely be represented as ordinary values SHOULD cross the boundary using handles or explicit resource profiles.

Examples include:

File
Socket
GPU Buffer
Device
Process
Shared Memory
Capability
Transaction

Resource cleanup MUST be deterministic where required by the resource contract.


---

59. Cancellation

Cancellable operations MUST explicitly define cancellation behavior.

Cancellation MAY occur:

Before execution
During execution
After partial progress
During cleanup

A cancelled operation MUST NOT falsely report successful completion.

Partial effects MUST be specified.


---

60. Idempotency

Functions MAY declare themselves:

Idempotent

or:

NonIdempotent

An implementation MUST NOT claim idempotency if repeating the operation can alter observable semantics.

This becomes especially important for distributed profiles.


---

61. Blocking

Blocking behavior MUST be explicit.

An operation declared:

NonBlocking

MUST NOT perform unbounded blocking on behalf of the consumer.

Blocking behavior belongs in execution metadata.


---

62. Panic / Fatal Failure

An implementation MUST NOT terminate the entire host process merely because a ULABI consumer supplied invalid ordinary input unless the contract explicitly defines process-fatal behavior.

Recoverable boundary failures SHOULD be represented using ULABI errors.

Memory corruption and unrecoverable runtime failures MAY require stronger isolation or process termination.


---

63. Process Isolation

ULABI Core does not require process isolation.

However, an implementation MAY place ULABI components into separate processes to provide:

fault isolation;

security isolation;

runtime isolation;

resource limits.


The semantic contract MUST remain unchanged.


---

64. Deterministic Failure

Invalid ULABI requests MUST produce deterministic failure classes where practical.

Examples:

InvalidInterface
UnsupportedVersion
InvalidType
InvalidArgument
InvalidOwnership
InvalidLifetime
MissingCapability
UnsupportedProfile
MalformedEncoding
ResourceUnavailable
ExecutionFailure

The exact standardized error registry belongs to the error specification.


---

65. Compatibility

Compatibility MUST be evaluated at the semantic contract level.

Two implementations are compatible only if they agree on all contract elements required for the operation.

Compatibility MUST NOT be inferred solely from:

matching source names;

matching machine architecture;

matching compiler;

matching memory layout;

successful linkage.



---

66. Backward Compatibility

A newer provider MAY add:

new optional functions;

new optional fields;

new optional profiles;

new capabilities;


provided existing valid consumers continue to operate according to the prior contract.

Removing or changing mandatory semantics requires a new incompatible interface version.


---

67. Forward Compatibility

Consumers SHOULD ignore explicitly extensible metadata they do not understand where the relevant contract permits it.

Unknown mandatory semantics MUST cause rejection rather than silent reinterpretation.


---

68. Capability Negotiation

Before using an optional feature, the consumer MAY negotiate:

Core Version
Profile
Feature
Capability
Encoding
Execution Mode

Negotiation MUST NOT grant authorization.

Capability negotiation determines support.

Security authorization determines permission.


---

69. ABI Negotiation Model

Conceptually:

Consumer
   │
   ├── Core version?
   │
   ├── Profile support?
   │
   ├── Feature support?
   │
   ├── Encoding support?
   │
   └── Capability requirements?
          │
          ▼
       Provider
          │
          ▼
     Negotiated Contract

The negotiated contract MUST be explicit and reproducible.


---

70. Conformance

An implementation claiming ULABI Core conformance MUST implement all mandatory Core requirements applicable to its declared execution mode.

Conformance MUST be testable.

A conformance result SHOULD identify:

ULABI Core
Specification Version
Supported Profiles
Supported Execution Modes
Supported Types
Supported Features
Known Restrictions
Test Suite Version


---

71. Conformance Levels

The future ULABI conformance system SHOULD distinguish at least:

Core-Minimal
Core-Standard
Core-Extended

Profiles are reported independently.

Example:

ULABI Core              PASS
ULABI Types             PASS
ULABI Memory            PASS
ULABI Security          PASS
ULABI Async             PARTIAL
ULABI Distributed       NOT IMPLEMENTED


---

72. Reference Test Categories

The Core conformance suite MUST eventually test:

Identity

interface identity;

function identity;

type identity.


Types

primitive values;

structured values;

option;

result;

variants;

records.


Calls

parameters;

results;

invalid arguments;

ownership.


Memory

ownership transfer;

borrowing;

lifetime;

release.


Errors

error identity;

error propagation;

malformed errors.


Compatibility

compatible versions;

incompatible versions;

unknown optional fields;

unknown mandatory fields.


Security

invalid capability;

unauthorized invocation;

malformed boundary input.


Determinism

canonical representations;

deterministic metadata;

ABI hashing.



---

73. Required Conformance Properties

A conforming implementation MUST demonstrate:

No undocumented ABI dependency
No ambiguous ownership
No ambiguous lifetime
No silent type reinterpretation
No silent version downgrade
No silent capability escalation
No undefined boundary representation
No hidden remote execution

where the property applies to the declared profile.


---

74. Reference Implementation Requirements

A reference implementation SHOULD be:

independent of any one language;

independently testable;

deterministic;

instrumentable;

capable of running the conformance suite;

suitable for interoperability testing.


The reference implementation is informative unless a future governance decision explicitly makes a particular artifact normative.


---

75. Required Machine-Readable Schemas

The Core ABI requires machine-readable definitions for at least:

Interface
Function
Parameter
Type
Field
Variant
Error
Capability
Profile
Version
ExecutionSemantics
Effect
Ownership
Lifetime
ConformanceResult

Recommended location:

schemas/


---

76. Required Core Schema Files

The following schemas SHOULD exist:

schemas/interface.schema.json
schemas/function.schema.json
schemas/parameter.schema.json
schemas/type.schema.json
schemas/field.schema.json
schemas/variant.schema.json
schemas/error.schema.json
schemas/capability.schema.json
schemas/profile.schema.json
schemas/version.schema.json
schemas/execution-semantics.schema.json
schemas/effect.schema.json
schemas/ownership.schema.json
schemas/lifetime.schema.json
schemas/conformance-result.schema.json

Schema evolution MUST follow ULABI versioning rules.


---

77. Required Core Code Modules

The standard itself is language-neutral, but reference implementations require concrete modules.

The reference implementation SHOULD be divided into independent modules:

core/
├── identity
├── version
├── interface
├── function
├── parameter
├── type
├── value
├── record
├── variant
├── option
├── result
├── handle
├── ownership
├── lifetime
├── call
├── result-return
├── error
├── effect
├── capability
├── execution
├── validation
├── compatibility
├── negotiation
├── encoding
├── decoding
├── canonicalization
├── hashing
└── conformance

The exact programming language is deliberately unspecified.


---

78. Recommended Rust Reference Module Layout

If the first reference implementation is Rust, it SHOULD use:

reference/
└── rust/
    └── ulabi-core/
        ├── Cargo.toml
        └── src/
            ├── lib.rs
            ├── identity.rs
            ├── version.rs
            ├── interface.rs
            ├── function.rs
            ├── parameter.rs
            ├── types.rs
            ├── values.rs
            ├── records.rs
            ├── variants.rs
            ├── option.rs
            ├── result.rs
            ├── handles.rs
            ├── ownership.rs
            ├── lifetimes.rs
            ├── call.rs
            ├── errors.rs
            ├── effects.rs
            ├── capabilities.rs
            ├── execution.rs
            ├── validation.rs
            ├── compatibility.rs
            ├── negotiation.rs
            ├── encoding.rs
            ├── decoding.rs
            ├── canonical.rs
            ├── hashing.rs
            └── conformance.rs

This is a reference implementation, not part of the normative ULABI language specification.


---

79. Adapter Module Requirements

Language adapters SHOULD be separate from Core.

Recommended conceptual structure:

implementations/
├── c/
├── cpp/
├── rust/
├── go/
├── python/
├── java/
├── swift/
├── kotlin/
├── fortran/
├── ada/
├── zamani/
└── sankofa/

The last two are implementation targets only.

They MUST NOT influence the Core specification.


---

80. Adapter Responsibilities

Each adapter MUST handle:

native types
      ↓
ULABI types

native calling convention
      ↓
ULABI call

native errors
      ↓
ULABI errors

native ownership
      ↓
ULABI ownership

native lifetime
      ↓
ULABI lifetime

native capabilities
      ↓
ULABI capabilities

The adapter MUST NOT redefine ULABI semantics.


---

81. Testing Architecture

Tests SHOULD be separated into:

tests/
├── core/
├── identity/
├── types/
├── calls/
├── ownership/
├── lifetimes/
├── errors/
├── compatibility/
├── negotiation/
├── encoding/
├── security/
├── determinism/
├── fuzz/
└── interoperability/

Conformance tests SHOULD be independent of any particular implementation language.


---

82. Cross-Language Interoperability Tests

The conformance system SHOULD eventually test combinations such as:

C ↔ Rust
C ↔ Go
Rust ↔ C++
Go ↔ Python
Java ↔ C
Swift ↔ Rust
Kotlin ↔ C
Fortran ↔ C
Ada ↔ C
Zamani ↔ Rust
Sankofa ↔ C

These are validation targets.

They are not dependencies of the standard.


---

83. ABI Golden Tests

The repository SHOULD contain canonical ABI fixtures.

Example:

tests/golden/
├── interfaces/
├── functions/
├── types/
├── errors/
├── capabilities/
└── versions/

A conforming implementation MUST reproduce the expected canonical representation where the test is normative.


---

84. Fuzz Testing

ULABI implementations SHOULD fuzz:

type decoders;

interface metadata;

function signatures;

malformed values;

version negotiation;

capability declarations;

canonical encodings.


Fuzzing MUST NOT replace deterministic conformance tests.


---

85. Security Testing

Security tests MUST include:

Malformed interface
Malformed type
Malformed length
Invalid ownership
Expired lifetime
Unauthorized capability
Unknown mandatory feature
Invalid version
Resource exhaustion
Oversized input
Recursive type exhaustion


---

86. Resource Limits

ULABI Core implementations SHOULD expose configurable limits for:

maximum message size;

maximum nesting depth;

maximum collection length;

maximum string length;

maximum allocation;

maximum execution duration;

maximum recursion depth.


A resource limit failure MUST produce an explicit failure rather than undefined behavior.


---

87. No Hidden ABI State

A ULABI function MUST NOT require undocumented state such as:

global mutable variables
hidden thread-local contracts
undocumented registers
implicit process-global handles
undocumented environment variables

unless the applicable profile explicitly defines such state.


---

88. ABI Observability

A conforming implementation SHOULD be able to expose diagnostic information including:

Interface ID
Function ID
Version
Execution Mode
Profile
Capability Requirements
Failure Code

Observability MUST NOT expose sensitive data unless authorized.


---

89. Debugging

Debuggers MAY map:

ULABI Interface
ULABI Function
ULABI Type
ULABI Error

to native debugging information.

Debugging metadata MUST NOT change runtime ABI semantics.

Detailed debugging requirements belong to the observability and tooling specifications.


---

90. Tooling Integration

The Core ABI MUST be consumable by:

Compiler
Linker
Loader
Validator
Debugger
Profiler
ABI-diff tool
Conformance tool
Schema generator
Binding generator

These tools MUST consume the normative contract rather than reverse-engineering implementation details.


---

91. ABI Difference Analysis

A future ABI-diff tool SHOULD classify changes as:

Compatible
Conditionally Compatible
Profile Extension
Breaking
Security Significant
Representation Significant
Semantic Significant

The classification MUST be based on the contract.


---

92. Loader Requirements

A ULABI-aware loader MAY verify:

Interface ID
Version
Required Profiles
ABI Digest
Required Capabilities
Platform Requirements

The loader MUST reject incompatible mandatory requirements.


---

93. Linker Requirements

A ULABI-aware linker MAY resolve:

InterfaceID + FunctionID

rather than relying solely on native symbol names.

Native linkage remains an implementation mechanism.


---

94. Compiler Requirements

A compiler targeting ULABI MUST preserve:

semantic types;

calling contracts;

ownership;

lifetimes;

error semantics;

effect declarations;

capability declarations.


Compiler optimization MUST NOT alter observable ULABI semantics.


---

95. Runtime Requirements

A runtime implementing ULABI MUST provide the mechanisms necessary to honor the declared contract.

It MAY implement those mechanisms using:

native runtime facilities;

operating-system facilities;

custom runtime facilities;

hardware facilities.


The choice is implementation-specific.


---

96. Core Invariants

The following invariants are mandatory:

Invariant 1 — Identity

Every externally visible interface has a stable identity.

Invariant 2 — Type

Every boundary value has a known semantic type.

Invariant 3 — Ownership

Every transferable resource has explicit ownership semantics.

Invariant 4 — Lifetime

Every reference has a defined lifetime.

Invariant 5 — Capability

Privileged operations have explicit capability requirements.

Invariant 6 — Version

Every interface has version semantics.

Invariant 7 — Error

Failure is represented by an explicit contract.

Invariant 8 — Compatibility

Compatible interfaces preserve existing semantics.

Invariant 9 — Validation

Untrusted boundary data is validated before unsafe interpretation.

Invariant 10 — Independence

ULABI semantics do not depend on a particular language or implementation.


---

97. Failure Model

Core failures include:

InvalidInterface
UnsupportedVersion
InvalidFunction
InvalidType
InvalidValue
InvalidArgument
InvalidOwnership
InvalidLifetime
MissingCapability
UnsupportedProfile
MalformedEncoding
ResourceLimitExceeded
ExecutionFailure
Cancelled
Timeout
Unavailable

The standardized error registry MUST assign stable identities to these classes before declaring Core 1.0 final.


---

98. Recovery

ULABI Core itself does not define autonomous recovery.

An implementation MAY recover from a failure where the applicable contract permits recovery.

Recovery MUST NOT silently alter the semantic contract.

Advanced recovery belongs to:

docs/reliability/


---

99. Self-Healing Boundary

Self-healing MUST remain an extension profile.

ULABI Core MUST NOT permit arbitrary self-modification.

A self-healing implementation MUST operate under an explicit policy:

Failure
   ↓
Evidence
   ↓
Known Policy?
 ┌─┴─┐
Yes  No
 │    │
Recover Escalate
 │
Verify
 │
Healthy?
 ┌─┴─┐
Yes  No
 │    │
Done Rollback/Escalate

This preserves Core determinism and security.


---

100. Distributed Boundary

ULABI Core remains usable locally without distributed functionality.

Distributed behavior belongs to:

docs/distributed/distributed-abi.md

A local implementation MUST NOT be required to implement network functionality merely to claim Core conformance.


---

101. Hardware Boundary

ULABI Core MUST NOT depend on:

CPU
GPU
NPU
FPGA
Quantum processor

Hardware-specific functionality belongs to hardware profiles.


---

102. Embedded Boundary

Embedded systems MAY implement a reduced ULABI Core profile where explicitly permitted.

They MUST document:

Unsupported Features
Resource Limits
Supported Types
Supported Execution Modes

They MUST NOT silently claim unsupported Core features.


---

103. Real-Time Boundary

Real-time guarantees MUST be declared by a real-time profile.

ULABI Core does not itself guarantee:

bounded latency
deadline completion
priority inheritance
hard real-time scheduling


---

104. Safety-Critical Boundary

Safety-critical use requires an explicit safety profile.

ULABI Core does not by itself constitute certification for:

aviation;

automotive;

medical;

industrial;

nuclear;

railway;

other safety-critical domains.



---

105. Cryptographic Boundary

Cryptographic algorithms MUST NOT be hardcoded into Core unnecessarily.

Cryptographic agility belongs to security profiles.

Any cryptographic identity used for ABI verification MUST be versioned and algorithm-identified.


---

106. Post-Quantum Readiness

ULABI identity and verification mechanisms SHOULD permit cryptographic algorithm migration without redesigning the ABI contract.


---

107. Implementation Independence

Two conforming implementations MUST be allowed to differ internally in:

memory management
object representation
register allocation
stack layout
compiler architecture
runtime
operating system
CPU
transport
optimization

provided observable ULABI semantics remain conformant.


---

108. Integration Contract

This document integrates with the following specifications.

Mandatory foundational integration

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md
ULABI-GOVERNANCE.md

ABI integration

docs/abi/data-types.md
docs/abi/calling-convention.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

Type-system integration

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md

Interoperability integration

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Runtime integration

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Memory integration

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

Security integration

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md

Compatibility integration

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Conformance integration

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md
docs/standards/reference-implementations.md


---

109. Integration Rule

The documents above MUST NOT redefine Core semantics inconsistently.

The dependency direction is:

ULABI-DESIGN
      ↓
ULABI-SPEC
      ↓
CORE ABI
      ↓
Profiles / Extensions
      ↓
Implementations
      ↓
Conformance

A lower-level implementation document MUST NOT redefine a higher-level normative contract.


---

110. Code Integration Contract

The reference implementation MUST map directly to the normative concepts:

Normative Concept              Reference Module

Interface identity        →    identity
Version                   →    version
Interface                 →    interface
Function                  →    function
Parameter                 →    parameter
Type                      →    types
Value                     →    values
Record                    →    records
Variant                   →    variants
Option                    →    option
Result                    →    result
Handle                    →    handles
Ownership                 →    ownership
Lifetime                  →    lifetimes
Call                      →    call
Error                     →    errors
Effect                    →    effects
Capability                →    capabilities
Execution                 →    execution
Validation                →    validation
Compatibility             →    compatibility
Negotiation               →    negotiation
Encoding                  →    encoding
Decoding                  →    decoding
Canonicalization          →    canonical
ABI Hash                  →    hashing
Conformance               →    conformance

This mapping is intentionally defined now so later documents do not require architectural rewrites.


---

111. Required Repository Artifacts

The Core ABI is considered structurally complete only when the repository provides:

Normative specification
Machine-readable schemas
Canonical examples
Conformance tests
Reference implementation
ABI fixtures
Compatibility tests
Security tests
Fuzz tests
Documentation


---

112. Core ABI Completion Criteria

docs/abi/core-abi.md is complete when:

all Core concepts have normative definitions;

every Core interface concept has a stable identity;

type boundaries are defined;

function contracts are defined;

ownership is explicit;

lifetimes are explicit;

errors are explicit;

capabilities are explicit;

execution semantics are explicit;

compatibility rules are explicit;

validation requirements are explicit;

extension boundaries are explicit;

conformance requirements are testable;

reference implementation mappings are established.



---

113. Core ABI Non-Goals

This specification deliberately does not standardize:

A programming language
A compiler implementation
A garbage collector
A CPU ISA
An operating system
A specific network protocol
A single serialization technology
A single object model
A single scheduler
A single memory allocator
A single cryptographic algorithm
A single runtime


---

114. Architectural Guarantee

ULABI Core MUST remain independent of any specific implementation.

In particular:

ULABI ≠ Zamani
ULABI ≠ Sankofa
ULABI ≠ C
ULABI ≠ Rust
ULABI ≠ Linux
ULABI ≠ x86
ULABI ≠ ARM
ULABI ≠ JVM
ULABI ≠ WebAssembly

All of these MAY implement, consume, host, compile to, or interoperate through ULABI.

None owns the specification.


---

115. Final Core Contract

The ULABI Core contract can be summarized as:

ULABI CORE
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    Identity       Types         Calls
       │             │             │
       ├─────────────┼─────────────┤
       │             │             │
   Ownership      Errors      Execution
       │             │             │
       ├─────────────┼─────────────┤
       │             │             │
   Lifetimes     Capability    Compatibility
       │             │             │
       └─────────────┼─────────────┘
                     │
              Validation
                     │
              Conformance
                     │
             Implementations

The central rule is:

> ULABI Core defines what interoperating components must mean at their boundary, not how their internals must implement that meaning.




---

116. Normative Status

This document is a draft normative specification.

Until ULABI Core reaches its first finalized major release:

identifiers remain subject to governance;

canonical encodings remain subject to schema finalization;

conformance requirements remain subject to test-suite validation;

profiles may evolve independently;

implementations MUST clearly identify the ULABI specification version they target.


No implementation should claim final ULABI Core 1.0 conformance until the corresponding conformance suite and certification rules are finalized.


---

3. What must be created around core-abi.md

The most important point for your workflow is that core-abi.md should now be treated as a contract anchor. The other files should implement the integration points already defined above rather than later changing this document.

A. Foundational files — do these first

These are the files that should be completed before the remaining ABI documents.

1. ULABI-SPEC.md

Must define:

normative specification hierarchy;

RFC terminology;

Core versus profile distinction;

normative document precedence;

identifiers;

canonical representations;

mandatory/optional semantics;

conformance terminology;

specification authority.


Integration:

ULABI-DESIGN
      ↓
ULABI-SPEC
      ↓
core-abi


---

2. ULABI-VERSIONING.md

Must define:

major/minor/patch semantics;

interface versions;

profile versions;

schema versions;

compatibility;

breaking changes;

ABI identity changes;

deprecation;

migration.



---

3. ULABI-GOVERNANCE.md

Must define:

who may propose changes;

review process;

normative change process;

emergency security changes;

version approval;

profile approval;

certification authority;

independent implementation requirements.



---

4. Required ABI documents

These are the exact ABI modules needed.

docs/abi/
├── core-abi.md
├── calling-convention.md
├── data-types.md
├── memory-model.md
├── stack-model.md
├── register-model.md
├── exception-model.md
└── return-values.md

Their dependencies should be:

core-abi
              /     |      \
             /      |       \
        data-types  calls   errors
             |       |        |
             ▼       ▼        ▼
          memory   registers exception
             │       │        │
             ▼       ▼        ▼
           stack  calling    return


---

5. Required type-system modules

docs/type-system/
├── universal-types.md
├── type-descriptors.md
├── generics.md
├── enums.md
├── structs.md
├── unions.md
└── type-compatibility.md

Code/schema modules:

type_id
type_descriptor
primitive_type
composite_type
record_type
enum_type
variant_type
generic_type
type_compatibility
type_validation


---

6. Required interoperability modules

docs/interoperability/
├── language-interoperability.md
├── foreign-function-interface.md
├── object-model.md
├── name-mangling.md
├── symbol-resolution.md
└── cross-language-data.md

Required implementation modules:

language_adapter
ffi
symbol
symbol_resolver
name_registry
binding_generator
type_mapper
error_mapper
ownership_mapper
lifetime_mapper


---

7. Required runtime modules

docs/runtime/
├── runtime-interface.md
├── process-model.md
├── threading.md
├── async-model.md
├── concurrency.md
└── resource-management.md

Required implementation modules:

runtime
process
thread
task
future
scheduler_adapter
async_adapter
resource
resource_registry
cancellation


---

8. Required memory modules

docs/memory/
├── memory-safety.md
├── ownership.md
├── lifetimes.md
├── allocation.md
├── virtual-memory.md
└── shared-memory.md

Required implementation modules:

memory
allocator
ownership
borrow
lifetime
reference
shared_memory
memory_region
memory_handle
release


---

9. Required security modules

docs/security/
├── security-model.md
├── capability-security.md
├── sandboxing.md
├── cryptography.md
├── authentication.md
├── authorization.md
├── zero-trust.md
├── secure-loading.md
└── supply-chain-security.md

Required implementation modules:

security
capability
capability_registry
authorization
authentication
sandbox
policy
secure_loader
signature
trust
audit


---

10. Required reliability modules

docs/reliability/
├── self-healing.md
├── fault-detection.md
├── fault-isolation.md
├── recovery.md
├── rollback.md
└── health-monitoring.md

Required modules:

fault_detector
diagnostics
fault_isolator
recovery_engine
recovery_policy
health
health_monitor
rollback
checkpoint
escalation

Important: these are profiles/extensions and must not become hidden dependencies of Core.


---

11. Required distributed modules

docs/distributed/
├── distributed-abi.md
├── remote-calls.md
├── serialization.md
├── service-discovery.md
├── distributed-errors.md
└── consensus-boundaries.md

Required modules:

transport
serializer
deserializer
remote_call
endpoint
service_discovery
distributed_error
timeout
retry
consistency


---

12. Required compatibility modules

docs/compatibility/
├── backwards-compatibility.md
├── forwards-compatibility.md
├── feature-negotiation.md
├── capability-discovery.md
└── graceful-degradation.md

Required modules:

compatibility
version_negotiator
feature_negotiator
capability_discovery
compatibility_checker
abi_diff
deprecation
fallback


---

13. Required schemas

I recommend creating the schema tree before implementing large amounts of runtime code:

schemas/
├── interface.schema.json
├── function.schema.json
├── parameter.schema.json
├── type.schema.json
├── field.schema.json
├── variant.schema.json
├── error.schema.json
├── capability.schema.json
├── profile.schema.json
├── version.schema.json
├── execution-semantics.schema.json
├── effect.schema.json
├── ownership.schema.json
├── lifetime.schema.json
├── handle.schema.json
├── encoding.schema.json
├── conformance-result.schema.json
└── abi-manifest.schema.json

These schemas become the machine-readable counterpart of the Markdown specifications.


---

14. Required examples

examples/
├── minimal/
├── functions/
├── types/
├── records/
├── variants/
├── option-result/
├── ownership/
├── handles/
├── errors/
├── capabilities/
├── versioning/
├── negotiation/
├── cross-language/
├── local/
├── process-local/
└── distributed/

Every important normative rule should eventually have an executable example.


---

15. Required tests

tests/
├── core/
│   ├── identity/
│   ├── functions/
│   ├── parameters/
│   ├── results/
│   ├── types/
│   ├── records/
│   ├── variants/
│   ├── option/
│   ├── result/
│   ├── handles/
│   ├── ownership/
│   ├── lifetimes/
│   ├── errors/
│   ├── effects/
│   ├── capabilities/
│   ├── execution/
│   └── validation/
│
├── compatibility/
├── negotiation/
├── encoding/
├── decoding/
├── canonicalization/
├── hashing/
├── security/
├── determinism/
├── interoperability/
├── fuzz/
└── regression/


---

16. Required conformance tree

conformance/
├── core/
├── types/
├── calls/
├── memory/
├── errors/
├── security/
├── async/
├── distributed/
├── reliability/
└── profiles/

The conformance system should produce machine-readable results such as:

ULABI Core       PASS
ULABI Types      PASS
ULABI Memory     PASS
ULABI FFI        PASS
ULABI Security   PASS
ULABI Async      PASS
ULABI Self-Heal  NOT CLAIMED
ULABI Distributed PASS


---

17. Required reference implementation tree

I recommend not putting the implementation directly inside core/ at the repository root.

Use:

reference/
└── rust/
    └── ulabi-core/

and later:

reference/
├── rust/
├── c/
├── cpp/
├── go/
├── python/
├── java/
├── swift/
├── kotlin/
├── fortran/
└── ada/

Again, this is an implementation ecosystem, not the specification.


---

18. Complete required code/module inventory

For the first complete ULABI implementation, I would lock the module inventory to:

IDENTITY
├── interface_id
├── function_id
├── type_id
├── error_id
├── capability_id
└── profile_id

VERSIONING
├── version
├── compatibility
├── negotiation
└── deprecation

CORE
├── interface
├── function
├── parameter
├── result
├── type
├── value
├── record
├── enum
├── variant
├── option
├── result_value
└── handle

CALL
├── call
├── argument
├── return_value
├── calling_boundary
├── invocation
└── invocation_context

MEMORY
├── memory
├── memory_region
├── allocator
├── ownership
├── borrow
├── lifetime
├── reference
├── shared_memory
└── release

ERRORS
├── error
├── error_code
├── error_data
├── error_chain
└── error_conversion

SECURITY
├── capability
├── capability_registry
├── policy
├── authorization
├── authentication
├── sandbox
├── trust
└── secure_loader

EXECUTION
├── execution
├── effects
├── execution_mode
├── cancellation
├── timeout
├── determinism
└── reentrancy

ENCODING
├── encoder
├── decoder
├── canonical
├── schema
├── validation
└── hashing

INTEROPERABILITY
├── language_adapter
├── type_mapper
├── ffi
├── symbol
├── symbol_resolver
├── name_registry
├── error_mapper
├── ownership_mapper
└── lifetime_mapper

RUNTIME
├── runtime
├── process
├── thread
├── task
├── future
├── resource
└── resource_registry

DISTRIBUTED
├── transport
├── endpoint
├── serializer
├── remote_call
├── service_discovery
├── retry
├── consistency
└── distributed_error

RELIABILITY
├── fault_detector
├── fault_isolator
├── diagnosis
├── recovery_engine
├── recovery_policy
├── rollback
├── checkpoint
├── health_monitor
└── escalation

TOOLING
├── abi_parser
├── abi_validator
├── abi_diff
├── binding_generator
├── manifest_generator
└── conformance_runner


---

19. The critical integration rule going forward

From this point on, I recommend we do not rewrite core-abi.md again merely because another document is being created.

The dependency contract is now:

ULABI-DESIGN
                               │
                               ▼
                           ULABI-SPEC
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
               VERSIONING            GOVERNANCE
                    │
                    ▼
                CORE ABI
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
      TYPES       CALLS        ERRORS
        │           │            │
        └───────────┼────────────┘
                    ▼
                  MEMORY
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   RUNTIME      SECURITY    INTEROP
       │            │            │
       └────────────┼────────────┘
                    ▼
             EXTENSION PROFILES
                    │
                    ▼
              CONFORMANCE
                    │
                    ▼
            IMPLEMENTATIONS

Every subsequent document should consume these contracts, not redefine them.

The current repository structure already contains the ABI documents as separate files, so this approach also fits the architecture already established in the repository rather than creating another parallel specification tree.

One important correction for the workflow: the file you asked for is docs/abi/core-abi.md, not doc/abi/core-abi.md. The repository confirms that exact existing path.