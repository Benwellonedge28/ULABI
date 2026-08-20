I inspected the actual Benwellonedge28/ULABI repository before drafting this. The master architecture establishes language/runtime/platform neutrality, Minimal Core + Profiles, semantic contracts, explicit memory boundaries, transport independence, compatibility, security, and independent implementations.

I also checked the existing ABI/interoperability structure. In particular, calling-convention.md already defines semantic calls, argument/return semantics, ownership, capabilities, effects, callbacks, function references, and architecture-specific lowering, so the FFI document should consume those contracts rather than redefine them.  The repository also already separates core ABI, data types, memory model, distributed serialization/remote calls, and compatibility negotiation.

The current docs/interoperability/foreign-function-interface.md itself is only a placeholder, so this is an appropriate independent specification to complete now.

1. Design dependency decision

foreign-function-interface.md should not become another copy of:

core-abi.md → universal ABI contract

calling-convention.md → call mechanics

data-types.md → universal types

memory-model.md → ownership/lifetime boundary

cross-language-data.md → data conversion between languages

object-model.md → object interoperability

name-mangling.md → symbol naming

symbol-resolution.md → locating exported symbols

serialization.md → canonical serialization

remote-calls.md → remote invocation


Instead, FFI is the integration contract that explains how a language implementation exposes/imports ULABI interfaces through foreign functions.

That gives us:

Language
   │
   ▼
Language Adapter
   │
   ├── Type Mapping ─────────── cross-language-data.md
   ├── Object Mapping ───────── object-model.md
   ├── Symbol Mapping ───────── name-mangling.md
   ├── Symbol Lookup ─────────── symbol-resolution.md
   │
   ▼
ULABI FFI
   │
   ├── Core ABI ─────────────── core-abi.md
   ├── Calling Convention ───── calling-convention.md
   ├── Memory Contract ──────── memory-model.md
   └── Error Contract ──────── exception-model.md
   │
   ▼
Native ABI / Runtime / IPC / Transport

That separation should prevent the later files from forcing this document to be rewritten.


---

2. docs/interoperability/foreign-function-interface.md

Copy the following into the file.

ULABI Foreign Function Interface Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: "docs/interoperability/foreign-function-interface.md"
Specification Family: Interoperability
Component: Foreign Function Interface
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Status: Normative Design Specification / Pre-Implementation
License: Apache-2.0

---

1. Purpose

This specification defines the ULABI Foreign Function Interface (FFI).

The ULABI FFI defines how a programming language, compiler, runtime, library, or application exposes and consumes functions belonging to an external implementation through the ULABI contract.

The FFI provides the bridge between:

Source Language
      |
      v
Compiler / Runtime
      |
      v
Language-Specific Foreign Function Mechanism
      |
      v
ULABI FFI Boundary
      |
      v
ULABI Interface
      |
      v
Foreign Implementation

The purpose is to allow independently implemented languages to call one another without requiring either language to adopt the other's:

- syntax;
- compiler;
- runtime;
- object model;
- memory manager;
- garbage collector;
- ownership model;
- exception model;
- ABI;
- operating system;
- processor architecture;
- vendor technology.

ULABI is the common contract.

The FFI is the mechanism by which a language implementation binds itself to that contract.

---

2. Fundamental Principle

The fundamental rule is:

«The ULABI FFI binds language-specific foreign-function mechanisms to language-neutral ULABI contracts.»

A language adapter MUST translate:

Language-specific function
        |
        v
Language ABI
        |
        v
ULABI Function Contract

and, for imports:

ULABI Function Contract
        |
        v
ULABI FFI Binding
        |
        v
Language-specific callable

No programming language is the reference language.

C, C++, Rust, Go, Java, Python, Swift, Kotlin, Fortran, Ada, Zamani, Sankofa, and future languages MUST be able to implement ULABI independently.

ULABI MUST NOT define its semantics in terms of any one of them.

---

3. Scope

This specification defines:

1. FFI providers;
2. FFI consumers;
3. exported functions;
4. imported functions;
5. interface binding;
6. function binding;
7. type binding;
8. ABI adapters;
9. language adapters;
10. calling-boundary integration;
11. symbol binding;
12. function identity;
13. signature validation;
14. ownership integration;
15. lifetime integration;
16. error translation;
17. exception translation;
18. capability propagation;
19. effect propagation;
20. synchronous calls;
21. asynchronous calls;
22. callbacks;
23. function references;
24. closures;
25. variadic functions;
26. generic functions;
27. opaque foreign values;
28. resource handles;
29. foreign object boundaries;
30. ABI version negotiation;
31. compatibility validation;
32. loading;
33. unloading;
34. lifecycle management;
35. security validation;
36. failure behavior;
37. diagnostics;
38. conformance requirements;
39. reference adapter architecture.

This specification does NOT redefine:

- universal data types;
- general calling convention;
- physical register allocation;
- physical stack layout;
- general memory management;
- canonical serialization;
- network transport;
- distributed service discovery;
- object-model semantics;
- symbol naming rules.

Those are defined by their respective specifications.

---

4. Architectural Position

The FFI sits between language implementations and ULABI contracts.

+--------------------------------------------------+
|                 Programming Language             |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|            Compiler / Runtime / Toolchain        |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|                 Language Adapter                 |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|                     ULABI FFI                    |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|                  ULABI Contract                  |
| Types | Calls | Errors | Memory | Capabilities  |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|       Native ABI / Runtime / IPC / Transport     |
+--------------------------------------------------+

The FFI therefore acts as an integration layer.

---

5. Provider and Consumer

A ULABI FFI implementation has two primary roles.

5.1 FFI Provider

The provider exports a ULABI interface.

Implementation
     |
     v
Language Adapter
     |
     v
ULABI Export

The provider MUST expose enough metadata for a consumer to validate the interface.

5.2 FFI Consumer

The consumer imports a ULABI interface.

Consumer
   |
   v
ULABI Import
   |
   v
Language Binding

The consumer MUST validate the imported interface before making calls whenever validation is required by the applicable security/conformance profile.

---

6. Export Contract

An exported ULABI function MUST have:

- interface identity;
- function identity;
- signature;
- parameter contracts;
- result contract;
- error contract;
- ownership semantics;
- lifetime semantics;
- effect metadata;
- capability requirements;
- execution semantics;
- compatibility metadata.

Example:

Interface:
    example.math

Function:
    add

Parameters:
    a: I64
    b: I64

Result:
    Result<I64, MathError>

Execution:
    Synchronous

Effects:
    Pure

Capabilities:
    None

The FFI MUST NOT expose undocumented semantic behavior.

---

7. Import Contract

An imported function MUST be bound to a ULABI interface identity and function identity.

An import declaration conceptually contains:

Import {
    interface_id
    function_id
    required_version
    expected_signature
    required_profiles
    binding_policy
}

The implementation MUST verify that the provider satisfies the requested contract.

A function MUST NOT be invoked solely because a symbol with a matching source-language name exists.

---

8. Function Identity

Function identity is governed by the ULABI Core ABI.

The FFI MUST use stable interface/function identities rather than relying exclusively upon:

- source names;
- linker names;
- memory addresses;
- object names;
- compiler-generated names;
- library filenames.

The FFI therefore separates:

Semantic Identity
       |
       v
ULABI Function ID
       |
       v
Native Symbol
       |
       v
Machine Address

The native symbol and machine address are implementation details.

---

9. Signature Binding

Before binding a function, the FFI MUST validate its signature.

Validation MUST cover:

- parameter count;
- parameter identity;
- parameter type;
- parameter passing mode;
- ownership;
- lifetime;
- result type;
- error type;
- execution mode;
- effects;
- capability requirements;
- compatibility version.

A signature mismatch MUST cause binding failure.

The FFI MUST NOT silently reinterpret incompatible signatures.

---

10. Type Binding

Type conversion is delegated to:

"docs/interoperability/cross-language-data.md"

The FFI consumes the resulting type mappings.

Conceptually:

Language Type
      |
      v
Cross-Language Type Mapping
      |
      v
ULABI Type
      |
      v
Foreign Language Type

The FFI MUST NOT invent independent type equivalence rules.

A language-specific type is interoperable only when its adapter establishes a valid ULABI mapping.

---

11. Primitive Type Binding

Primitive types MAY be bound directly when their semantics are compatible.

Examples:

Bool
I32
U32
I64
U64
F32
F64
Char
Bytes
String

The adapter MUST verify width, signedness, range, encoding, and semantic compatibility where applicable.

A native type named "int", "long", "char", or equivalent MUST NOT be assumed to have a universal representation.

---

12. Aggregate Type Binding

Aggregates such as:

Record
Enum
Variant
Option
Result
List
Map

MUST be bound according to their ULABI semantic contracts.

The FFI MUST NOT assume that the foreign language has the same:

- object layout;
- field order;
- padding;
- alignment;
- vtable;
- inheritance;
- garbage-collection metadata.

---

13. Opaque Foreign Types

A foreign value that cannot safely be represented directly SHOULD be exposed as an opaque ULABI handle.

Example:

ForeignDatabaseConnection
        |
        v
OpaqueHandle<DatabaseConnection>

The handle MUST NOT expose private implementation addresses as its semantic identity.

The handle MUST obey the applicable ownership, lifetime, security, and capability contracts.

---

14. Ownership Integration

Ownership is governed by:

"docs/abi/memory-model.md"

and the applicable memory specifications.

The FFI MUST preserve the declared ownership transition.

Supported semantic modes MAY include:

Borrow
Shared
Owned
Move
Transfer
Consume

Example:

ULABI Function

process(
    resource: Move<Resource>
)

The language adapter MUST translate this into the safest equivalent mechanism available in the target language.

If no safe representation exists, the FFI MUST reject the binding or use an explicitly defined copying/handle strategy.

---

15. Lifetime Integration

The FFI MUST preserve lifetime semantics.

Possible lifetimes include:

DuringCall
UntilReturn
UntilRelease
Pinned
InterfaceLifetime
Explicit

A language adapter MUST NOT allow a foreign function to retain a reference beyond its permitted lifetime.

If the source language cannot guarantee the required lifetime, the adapter MUST use a safe alternative or reject the binding.

---

16. Error Translation

ULABI error semantics are language-neutral.

A language adapter MAY translate:

Exception
Status Code
Error Object
Result
Tagged Union
Trap

into:

ULABI Result<T,E>

The reverse translation is also permitted.

Example:

ULABI:
Result<Data, IOError>

        |
        v

Rust:
Result<Data, IOError>

        OR

Python:
Value / Exception

        OR

Java:
Value / Exception

The semantic error identity MUST remain unchanged.

---

17. Exception Boundary

A language-specific exception MUST NOT automatically cross an FFI boundary as a language-specific exception object.

Instead:

Language Exception
       |
       v
ULABI Error Contract
       |
       v
Target Language Error

The adapter MUST prevent private runtime metadata from leaking across the boundary unless an explicit profile defines such behavior.

---

18. Panic and Fatal Failure

Language-level panic, abort, fatal exception, segmentation failure, or equivalent termination MUST NOT automatically be interpreted as a normal ULABI error.

The implementation MUST distinguish:

Normal Result
Recoverable Error
Cancellation
Timeout
Capability Denial
Resource Exhaustion
Contract Violation
Fatal Failure
Process Termination

The applicable error/reliability specifications define detailed behavior.

---

19. Calling Convention Integration

Physical invocation is governed by:

"docs/abi/calling-convention.md"

The FFI MUST NOT define a second calling convention.

The relationship is:

ULABI FFI
    |
    v
ULABI Calling Convention
    |
    v
Native ABI

The FFI implementation performs the necessary lowering.

For example:

ULABI argument
      |
      +--> native register
      +--> native stack
      +--> descriptor
      +--> memory
      +--> IPC message

The semantic function contract remains unchanged.

---

20. Register and Stack Independence

The FFI MUST NOT assume a universal register or stack model.

Architecture-specific behavior belongs to:

- "docs/abi/register-model.md"
- "docs/abi/stack-model.md"
- platform architecture specifications.

An FFI implementation MAY use:

- registers;
- stack;
- heap frames;
- descriptors;
- trampolines;
- generated wrappers;
- runtime invocation mechanisms.

---

21. Symbol Binding

The FFI MUST distinguish:

ULABI Function Identity
        |
        v
Language Binding
        |
        v
Native Symbol

Symbol naming is governed by:

"docs/interoperability/name-mangling.md"

Symbol resolution is governed by:

"docs/interoperability/symbol-resolution.md"

The FFI MUST consume those specifications rather than duplicate them.

---

22. Dynamic Loading

An implementation MAY dynamically load foreign modules.

Conceptually:

Discover Module
      |
      v
Load Module
      |
      v
Validate Metadata
      |
      v
Resolve Interface
      |
      v
Validate Version
      |
      v
Validate Signature
      |
      v
Bind Functions
      |
      v
Ready

A module MUST NOT be treated as trusted merely because it loaded successfully.

Security validation MUST occur according to the applicable security profile.

---

23. Static Linking

A ULABI implementation MAY use static linking.

The same semantic validation requirements apply.

Static linking MUST NOT alter the ULABI contract.

The resulting binary may contain:

Language A
   +
ULABI Adapter
   +
Foreign Implementation

without exposing implementation-specific details as part of the ULABI semantic contract.

---

24. Binding Lifecycle

The standard FFI lifecycle is:

Declare
   |
Discover
   |
Load / Locate
   |
Validate Identity
   |
Validate Version
   |
Validate Profiles
   |
Validate Signature
   |
Validate Types
   |
Validate Capabilities
   |
Bind
   |
Invoke
   |
Release
   |
Unload

An implementation MAY optimize these steps but MUST preserve their required semantics.

---

25. Binding Failure

Binding MUST fail safely if:

- interface identity is invalid;
- function identity is invalid;
- version is incompatible;
- signature is incompatible;
- type mapping is unsupported;
- ownership cannot be represented safely;
- lifetime cannot be represented safely;
- required capability is unavailable;
- required profile is unavailable;
- security validation fails;
- integrity validation fails.

The implementation MUST NOT silently fall back to an incompatible interpretation.

---

26. ABI Versioning

The FFI MUST use ULABI versioning rules.

A binding MUST identify:

Required Core Version
Required Interface Version
Required Function Version
Required Profile Versions

Version compatibility is governed by:

- "ULABI-VERSIONING.md"
- "docs/compatibility/backwards-compatibility.md"
- "docs/compatibility/forwards-compatibility.md"
- "docs/compatibility/feature-negotiation.md"

The FFI MUST NOT invent a competing versioning system.

---

27. Profile Negotiation

A foreign function MAY require profiles.

Example:

Required Profiles:

ULABI-Core
ULABI-Memory
ULABI-Async
ULABI-Security

The FFI MUST verify profile availability before binding.

Optional profiles MUST NOT be treated as mandatory unless the contract requires them.

---

28. Capability Requirements

A function MAY declare required capabilities.

Example:

write_file()
requires:
    FileWrite

The FFI MUST preserve capability requirements.

An adapter MUST NOT automatically grant capabilities merely because a function is imported.

Capability enforcement is governed by:

"docs/security/capability-security.md"

---

29. Effect Metadata

Foreign functions MAY declare effects such as:

Pure
ReadsMemory
WritesMemory
Filesystem
Network
GPU
Device
Process
Time
Randomness
NonDeterministic

The FFI MUST preserve effect metadata.

Effects MAY be consumed by:

- security systems;
- static analyzers;
- schedulers;
- sandboxes;
- validators;
- conformance tooling.

---

30. Synchronous Functions

A synchronous ULABI function MUST provide a defined completion result.

The FFI MUST NOT silently transform a synchronous call into an asynchronous call.

If a transport or implementation requires asynchronous machinery internally, the externally visible semantic contract MUST remain synchronous.

---

31. Asynchronous Functions

An asynchronous foreign function MAY expose:

Future<T>

The FFI MUST preserve:

- completion;
- failure;
- cancellation;
- timeout;
- ownership;
- lifetime;
- resource cleanup.

The asynchronous model is governed by:

"docs/runtime/async-model.md"

The FFI integrates with that model but does not redefine it.

---

32. Cancellation

If a function is declared cancellable, the FFI MUST preserve the cancellation semantics.

Possible semantics include:

BestEffort
Cooperative
Immediate
Deferred
NonCancellable

Cancellation MUST NOT be confused with successful completion.

---

33. Callbacks

The FFI MUST support callbacks where the applicable profile permits them.

Conceptually:

Foreign Function
      |
      v
Callback Reference
      |
      v
ULABI Function Reference
      |
      v
Language Callback

A callback contract MUST define:

- signature;
- lifetime;
- ownership;
- execution context;
- thread requirements;
- reentrancy;
- cancellation;
- capabilities.

The FFI MUST NOT assume that callbacks can safely execute on arbitrary threads.

---

34. Function References

A function reference MAY represent:

FunctionID
Closure
Callable Object
Remote Function
Capability-Bound Function

It MUST NOT necessarily be represented as a raw machine address.

The target implementation MUST validate function references according to the security profile.

---

35. Closures

Language closures MAY be exposed through a ULABI callable descriptor.

Conceptually:

CallableDescriptor {
    interface_id
    function_id
    environment
    capabilities
    lifetime
}

The FFI MUST NOT require all languages to use the same closure memory layout.

Captured environments MUST obey the applicable ownership and lifetime rules.

---

36. Variadic Functions

The ULABI Core MUST NOT depend upon untyped native variadic calling conventions.

A variadic ULABI function MUST provide explicit type information.

Example:

log(
    level: LogLevel,
    messages: List<String>
)

is preferred over exposing an untyped native "..." boundary.

If a profile permits variadic ABI support, the FFI MUST validate argument descriptors before invocation.

---

37. Generic Functions

Generic functions MUST be represented through explicit ULABI type metadata or monomorphized concrete contracts.

The FFI MUST NOT require another language to understand the source language's generic implementation mechanism.

For example:

Language A:
    generic sort<T>()

ULABI:
    sort(
        sequence: SequenceDescriptor,
        comparator: FunctionRef
    )

may be used when the semantic contract is preserved.

---

38. Object-Oriented Languages

The FFI MUST NOT require foreign implementations to share:

- class layouts;
- vtables;
- inheritance;
- RTTI;
- method dispatch;
- garbage collectors.

Object interoperability is governed by:

"docs/interoperability/object-model.md"

The FFI consumes the object-model contract.

Opaque object handles SHOULD be used where direct object representation is unsafe.

---

39. Foreign Resources

Foreign resources SHOULD be exposed using ULABI handles where direct representation would expose implementation details.

Examples:

File
Socket
Database
GPU Buffer
Device
Window
Thread
Process
Shared Memory

A handle MUST have explicit:

- type;
- ownership;
- lifetime;
- authority;
- capability requirements;
- release semantics.

---

40. Resource Release

If the FFI creates or receives an owned resource, its release contract MUST be explicit.

Possible mechanisms include:

release(handle)
close(handle)
destroy(handle)
drop(handle)
finalize(handle)

The semantic release operation is authoritative.

A language's garbage collector MUST NOT be assumed to provide timely resource release.

---

41. Threading

The FFI MUST respect the threading contract of the imported function.

A function MAY declare:

ThreadSafe
ThreadConfined
SingleThread
ExecutorBound
Reentrant
NonReentrant

The FFI MUST NOT automatically move execution to another thread if doing so violates the function contract.

Threading semantics are governed by:

"docs/runtime/threading.md"

---

42. Reentrancy

A foreign function MUST declare reentrancy requirements where relevant.

A language adapter MUST preserve the distinction between:

Reentrant
NonReentrant
ThreadSafe
ThreadConfined

Callbacks that re-enter a provider MUST obey the provider's reentrancy contract.

---

43. Zero-Copy FFI

Zero-copy interoperability MAY be used when safe.

A zero-copy boundary MUST validate:

- ownership;
- lifetime;
- mutability;
- aliasing;
- alignment;
- representation;
- synchronization;
- security.

If these cannot be proven safe, the implementation MUST use copying or another safe representation.

Zero-copy MUST be an optimization, not a requirement for semantic compatibility.

---

44. Shared Memory FFI

Shared-memory interoperability MAY be provided through a profile.

Shared memory MUST define:

- ownership;
- synchronization;
- visibility;
- lifetime;
- access permissions;
- mutation;
- consistency;
- cleanup.

The FFI MUST NOT assume that a pointer into one process is meaningful in another process.

---

45. Out-of-Process FFI

A ULABI FFI binding MAY be implemented through IPC.

Conceptually:

Language A
    |
ULABI FFI
    |
IPC Adapter
    |
Process B
    |
ULABI FFI
    |
Language B

The semantic contract remains ULABI.

IPC-specific behavior belongs to the applicable distributed/transport specifications.

---

46. Distributed FFI

A distributed call MUST NOT be treated as identical to an in-process call.

The FFI MUST preserve explicit distinctions involving:

- latency;
- failure;
- timeout;
- retry;
- cancellation;
- serialization;
- authentication;
- authorization;
- partial failure.

Distributed behavior is governed by the distributed ULABI specifications.

---

47. Security Boundary

An FFI boundary is a security boundary whenever the foreign implementation is not fully trusted.

The implementation MUST support applicable validation for:

- interface identity;
- integrity;
- provenance;
- permissions;
- capabilities;
- signature compatibility;
- memory safety;
- resource limits;
- sandbox policy.

The FFI MUST NOT equate successful loading with authorization.

---

48. ABI Validation

A conforming implementation SHOULD validate an imported ABI before exposing it to application code.

Validation SHOULD include:

Interface Identity
Version
Profile Set
Function Identity
Signature
Types
Ownership
Lifetime
Capabilities
Effects
Execution Semantics
Security Metadata

Validation failure MUST prevent unsafe invocation.

---

49. Sandboxing

A foreign implementation MAY execute in a sandbox.

The FFI MUST preserve the declared security boundary.

If a foreign function requires capabilities unavailable to the sandbox, binding or invocation MUST fail according to the applicable policy.

Sandbox semantics are governed by:

"docs/security/sandboxing.md"

---

50. ABI Integrity

Implementations MAY cryptographically authenticate interface metadata and foreign modules.

If integrity metadata is supplied, the FFI MUST validate it before trusting the associated contract according to the security profile.

Cryptographic mechanisms are defined by:

"docs/security/cryptography.md"

The FFI does not mandate one cryptographic algorithm.

---

51. Diagnostics

FFI failures SHOULD provide structured diagnostics.

Conceptually:

FFIError {
    code
    interface_id
    function_id
    expected_version
    actual_version
    expected_signature
    actual_signature
    reason
}

Diagnostics MUST NOT leak secrets or private implementation state contrary to the applicable security policy.

---

52. Failure Categories

At minimum, implementations SHOULD distinguish:

InterfaceNotFound
FunctionNotFound
VersionMismatch
ProfileMismatch
SignatureMismatch
TypeMismatch
OwnershipMismatch
LifetimeMismatch
CapabilityDenied
SecurityValidationFailed
ModuleLoadFailed
SymbolResolutionFailed
UnsupportedFeature
ResourceLimit
InvocationFailed
ForeignError
ForeignFatalFailure
Cancellation
Timeout

Implementations MAY add implementation-specific diagnostic details.

---

53. Recovery

The FFI MUST NOT automatically retry an operation unless the contract permits retry.

In particular, retrying a non-idempotent foreign function may create duplicate side effects.

Retry behavior MUST consider:

Idempotency
Side Effects
Execution Mode
Transport
Failure Class
Capability State
Resource State

Recovery policies belong to the reliability specifications.

---

54. Module Unloading

A foreign module MUST NOT be unloaded while active ULABI references remain unless the applicable lifecycle contract explicitly permits it.

Before unloading:

Stop New Calls
      |
Wait / Cancel Active Calls
      |
Release Resources
      |
Invalidate References
      |
Unload

Function references, callbacks, handles, and borrowed values MUST no longer reference the unloaded implementation.

---

55. FFI Adapter Architecture

A reference FFI implementation SHOULD be divided into:

FFI Layer
│
├── Interface Loader
├── Interface Validator
├── Function Binder
├── Signature Validator
├── Type Mapper
├── ABI Lowering
├── Calling Adapter
├── Error Adapter
├── Ownership Adapter
├── Lifetime Adapter
├── Capability Adapter
├── Effect Adapter
├── Callback Adapter
├── Resource Adapter
├── Module Manager
├── Symbol Resolver
├── Security Validator
└── Diagnostics

These modules MUST remain independently testable.

---

56. Adapter Isolation

The language adapter SHOULD isolate language-specific implementation details from the ULABI contract.

Recommended architecture:

Language Runtime
       |
       v
Language Adapter
       |
       +---- Type Adapter
       +---- Call Adapter
       +---- Error Adapter
       +---- Memory Adapter
       +---- Resource Adapter
       |
       v
ULABI FFI

The ULABI layer MUST NOT contain language-specific semantics.

---

57. No Universal Native ABI

ULABI MUST NOT define one universal physical ABI.

Instead:

                 ULABI
                   |
       +-----------+-----------+
       |           |           |
    x86-64       ARM64       RISC-V
       |           |           |
   Native ABI   Native ABI   Native ABI

The FFI translates between the ULABI semantic contract and the native mechanism.

This preserves architecture neutrality.

---

58. Compatibility

An FFI binding is compatible only when the imported interface satisfies the applicable compatibility rules.

Compatibility MUST consider more than binary layout.

It MUST include semantic compatibility for:

- types;
- ownership;
- errors;
- effects;
- capabilities;
- execution behavior;
- version;
- optional features.

A binary-compatible but semantically incompatible function MUST be rejected.

---

59. Forward Compatibility

An FFI implementation SHOULD tolerate compatible additions according to the applicable ULABI compatibility profile.

It MUST NOT assume that unknown features have safe defaults.

Unknown mandatory features MUST cause negotiation or binding failure.

---

60. Backward Compatibility

Existing valid bindings SHOULD continue to work when a provider releases a compatible implementation.

Providers MUST NOT silently change:

- function meaning;
- parameter meaning;
- ownership;
- lifetime;
- error semantics;
- capability requirements;

while retaining the same incompatible contract identity.

---

61. Conformance Requirements

A conforming ULABI FFI implementation MUST:

1. bind stable ULABI interface identities;
2. validate function signatures;
3. preserve ULABI type semantics;
4. preserve ownership semantics;
5. preserve lifetime semantics;
6. preserve error semantics;
7. preserve capability requirements;
8. preserve effect metadata;
9. respect execution semantics;
10. reject incompatible bindings;
11. avoid undocumented native ABI assumptions;
12. expose structured diagnostics;
13. prevent unsafe lifetime extension;
14. prevent unauthorized capability escalation;
15. support the applicable versioning rules.

---

62. Conformance Test Categories

The FFI conformance suite MUST eventually test:

Interface Discovery
Function Discovery
Signature Validation
Primitive Types
Aggregate Types
Type Conversion
Ownership
Borrowing
Lifetime
Errors
Exceptions
Capabilities
Effects
Callbacks
Function References
Closures
Async Calls
Cancellation
Threading
Reentrancy
Dynamic Loading
Static Linking
Symbol Resolution
Version Negotiation
Security Validation
Module Unloading
Failure Handling
Diagnostics

---

63. Negative Tests

The conformance suite MUST include invalid cases.

Examples:

Wrong Function ID
Wrong Signature
Wrong Type
Wrong Version
Missing Profile
Missing Capability
Invalid Lifetime
Invalid Ownership
Invalid Handle
Unsupported Conversion
Malformed Metadata
Invalid Symbol
Unauthorized Module
Unsafe Callback
Use-After-Unload
Illegal Retry

Each case MUST produce a defined failure rather than undefined behavior.

---

64. Reference Execution Flow

A conforming FFI implementation should follow:

                FFI IMPORT
                    |
                    v
             Locate Interface
                    |
                    v
            Validate Identity
                    |
                    v
             Validate Version
                    |
                    v
             Validate Profiles
                    |
                    v
            Validate Signature
                    |
                    v
              Map Types
                    |
                    v
          Validate Ownership
                    |
                    v
           Validate Lifetimes
                    |
                    v
         Validate Capabilities
                    |
                    v
               Bind Call
                    |
                    v
              Invoke Function
                    |
                    v
             Adapt Result
                    |
                    v
             Return to Caller

Any failed mandatory validation terminates the flow before unsafe invocation.

---

65. Integration Contract

This document is complete independently of the implementation of later documents.

The following integration points are fixed in advance.

Core ABI

"docs/abi/core-abi.md"

Defines:

- interface identity;
- function identity;
- universal ABI boundary;
- core contract semantics.

The FFI consumes these definitions.

Calling Convention

"docs/abi/calling-convention.md"

Defines:

- argument semantics;
- return semantics;
- physical ABI lowering;
- callbacks;
- function references.

The FFI invokes through these rules.

Data Types

"docs/abi/data-types.md"

Defines universal data types.

The FFI does not redefine them.

Memory Model

"docs/abi/memory-model.md"

Defines boundary memory semantics.

The FFI consumes ownership/lifetime rules.

Cross-Language Data

"docs/interoperability/cross-language-data.md"

Defines type mapping and conversion.

The FFI invokes those mappings.

Object Model

"docs/interoperability/object-model.md"

Defines object interoperability.

The FFI uses object descriptors or opaque handles defined there.

Name Mangling

"docs/interoperability/name-mangling.md"

Defines mapping between semantic identities and language/native symbols.

Symbol Resolution

"docs/interoperability/symbol-resolution.md"

Defines how exported symbols are located.

Exception Model

"docs/abi/exception-model.md"

Defines exception/error boundary behavior.

Runtime

"docs/runtime/runtime-interface.md"

Defines runtime integration.

Async

"docs/runtime/async-model.md"

Defines asynchronous semantics.

Threading

"docs/runtime/threading.md"

Defines threading requirements.

Security

"docs/security/security-model.md"

Defines security boundaries.

Capability Security

"docs/security/capability-security.md"

Defines capability authorization.

Sandboxing

"docs/security/sandboxing.md"

Defines sandbox behavior.

Serialization

"docs/distributed/serialization.md"

Defines serialization where a serialized boundary is used.

Remote Calls

"docs/distributed/remote-calls.md"

Defines distributed invocation.

Compatibility

The FFI consumes:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/graceful-degradation.md

Conformance

The FFI contributes tests to:

docs/standards/conformance.md
docs/standards/test-suite.md
docs/standards/certification.md

---

66. Required FFI Implementation Modules

A reference implementation SHOULD contain the following modules.

implementations/
└── ffi/
    ├── mod
    ├── interface_loader
    ├── interface_validator
    ├── function_binder
    ├── signature_validator
    ├── type_mapper
    ├── call_adapter
    ├── abi_lowering
    ├── return_adapter
    ├── error_adapter
    ├── exception_adapter
    ├── ownership_adapter
    ├── lifetime_adapter
    ├── capability_adapter
    ├── effect_adapter
    ├── callback_adapter
    ├── closure_adapter
    ├── resource_adapter
    ├── handle_manager
    ├── symbol_resolver
    ├── module_manager
    ├── version_validator
    ├── profile_validator
    ├── security_validator
    ├── invocation_context
    ├── diagnostics
    └── errors

The implementation language is intentionally unspecified.

---

67. Required Language Adapter Modules

Each language implementation SHOULD provide a language-specific adapter:

implementations/
└── languages/
    └── <language>/
        ├── mod
        ├── types
        ├── imports
        ├── exports
        ├── calls
        ├── results
        ├── errors
        ├── ownership
        ├── lifetimes
        ├── callbacks
        ├── resources
        └── runtime

"<language>" is an implementation choice.

ULABI does not privilege any language.

---

68. Required Schemas

The FFI implementation should consume or eventually add schemas for:

schemas/
├── interface.schema
├── function.schema
├── signature.schema
├── parameter.schema
├── result.schema
├── type-binding.schema
├── ownership.schema
├── lifetime.schema
├── capability.schema
├── effect.schema
├── profile.schema
├── ffi-import.schema
├── ffi-export.schema
├── ffi-binding.schema
└── ffi-error.schema

These schemas define metadata structure rather than language-specific implementation behavior.

---

69. Required Test Modules

The FFI conformance suite SHOULD contain:

tests/
└── ffi/
    ├── interface_discovery
    ├── function_binding
    ├── signature_validation
    ├── primitive_types
    ├── aggregate_types
    ├── ownership
    ├── borrowing
    ├── lifetimes
    ├── errors
    ├── exceptions
    ├── callbacks
    ├── closures
    ├── async
    ├── cancellation
    ├── threading
    ├── capabilities
    ├── effects
    ├── dynamic_loading
    ├── static_linking
    ├── versioning
    ├── compatibility
    ├── security
    ├── module_unloading
    └── negative

---

70. Required Examples

The repository SHOULD eventually contain:

examples/
└── ffi/
    ├── c-provider
    ├── c-consumer
    ├── rust-provider
    ├── rust-consumer
    ├── python-provider
    ├── python-consumer
    ├── java-provider
    ├── java-consumer
    ├── callback
    ├── opaque-handle
    ├── ownership-transfer
    ├── borrowed-data
    ├── async-function
    ├── capability-protected-function
    └── incompatible-binding

These examples demonstrate interoperability without making any one language normative.

---

71. Security Invariants

A conforming FFI implementation MUST maintain these invariants:

1. A foreign symbol MUST NOT automatically become a trusted ULABI function.
2. A signature mismatch MUST NOT be silently accepted.
3. A type mismatch MUST NOT be silently coerced into an unsafe representation.
4. A borrowed value MUST NOT outlive its permitted lifetime.
5. An owned resource MUST have explicit release semantics.
6. A foreign function MUST NOT gain undeclared capabilities.
7. An unloaded module MUST NOT remain callable.
8. A callback MUST NOT outlive its declared environment.
9. A non-idempotent operation MUST NOT be blindly retried.
10. A language-specific exception MUST NOT leak across the boundary without an explicit contract.
11. Native memory addresses MUST NOT become semantic identities.
12. Unsupported semantics MUST fail explicitly.

---

72. Failure-Oriented FFI

The FFI MUST be designed around explicit failure.

The preferred model is:

Discover
   |
   v
Validate
   |
   +---- failure ----> Reject Binding
   |
   v
Bind
   |
   +---- failure ----> Unbind / Cleanup
   |
   v
Invoke
   |
   +---- failure ----> Adapt Error
   |
   v
Validate Result
   |
   +---- failure ----> Reject Invalid Result
   |
   v
Return

Undefined behavior MUST NOT be used as an interoperability strategy.

---

73. Implementation Independence

Two independent organizations MUST be able to implement the ULABI FFI independently and still interoperate.

They may use:

- different programming languages;
- different compilers;
- different runtimes;
- different operating systems;
- different CPUs;
- different memory-management systems;
- different native ABIs.

Only the ULABI contract is shared.

---

74. Relationship to Zamani and Sankofa

Zamani and Sankofa may independently implement ULABI FFI adapters.

For example:

Zamani
   |
Zamani ULABI Adapter
   |
ULABI FFI
   |
Foreign Implementation

and:

Sankofa
   |
Sankofa ULABI Adapter
   |
ULABI FFI
   |
Foreign Implementation

These are independent implementations.

ULABI MUST NOT import, embed, depend upon, or define either language.

---

75. Non-Goals

The FFI is not:

- a programming language;
- a compiler;
- a universal runtime;
- a universal object system;
- a universal garbage collector;
- a universal native ABI;
- a package manager;
- a network protocol;
- a replacement for existing language FFIs.

ULABI standardizes the interoperability contract between them.

---

76. Final Contract

The ULABI FFI can be summarized as:

Language
   |
   v
Language Adapter
   |
   v
      +----------------------+
      |      ULABI FFI      |
      +----------------------+
       |    |    |    |    |
       v    v    v    v    v
     Types Calls Errors Memory Security
       |    |    |    |    |
       +----+----+----+----+
                |
                v
         ULABI Contract
                |
                v
       Native / Runtime / IPC

The governing principle is:

«ULABI FFI standardizes how foreign functions bind to a universal language-neutral contract; it does not standardize the internal implementation of any participating language.»

This document is therefore considered complete as an independent FFI contract once its referenced contracts exist at their declared paths. Later implementation of those referenced components MUST integrate with these interfaces rather than requiring this document to be rewritten.3. What should be implemented under which files

The important part is that we do not immediately scatter FFI code everywhere. The dependency order should be explicit.

Phase A — independent FFI foundation

These can be implemented without waiting for the entire ULABI ecosystem:

implementations/ffi/mod.*
implementations/ffi/errors.*
implementations/ffi/invocation_context.*
implementations/ffi/diagnostics.*
implementations/ffi/interface_validator.*
implementations/ffi/signature_validator.*
implementations/ffi/version_validator.*
implementations/ffi/profile_validator.*

These establish the FFI's internal model and validation boundary.

Phase B — binding machinery

implementations/ffi/interface_loader.*
implementations/ffi/function_binder.*
implementations/ffi/call_adapter.*
implementations/ffi/abi_lowering.*
implementations/ffi/return_adapter.*
implementations/ffi/symbol_resolver.*
implementations/ffi/module_manager.*

These integrate the validated ULABI contract with an actual callable implementation.

Phase C — semantic adapters

implementations/ffi/type_mapper.*
implementations/ffi/error_adapter.*
implementations/ffi/exception_adapter.*
implementations/ffi/ownership_adapter.*
implementations/ffi/lifetime_adapter.*
implementations/ffi/capability_adapter.*
implementations/ffi/effect_adapter.*
implementations/ffi/resource_adapter.*
implementations/ffi/handle_manager.*

These should consume the already-defined ABI/type/memory/security contracts rather than creating parallel semantics.

Phase D — advanced invocation

implementations/ffi/callback_adapter.*
implementations/ffi/closure_adapter.*
implementations/ffi/async_adapter.*
implementations/ffi/cancellation_adapter.*
implementations/ffi/thread_adapter.*

These depend on the runtime specifications.


---

4. Files that this FFI specification integrates with

The integration matrix should be treated as fixed now:

Existing/future file	FFI responsibility

ULABI-DESIGN.md	Overall architectural authority
ULABI-SPEC.md	Normative global rules
ULABI-VERSIONING.md	Version semantics
docs/abi/core-abi.md	Interface/function identity
docs/abi/calling-convention.md	Invocation semantics
docs/abi/data-types.md	Universal types
docs/abi/memory-model.md	Memory boundary
docs/abi/exception-model.md	Exception/error boundary
docs/abi/return-values.md	Return semantics
docs/abi/register-model.md	Physical register lowering
docs/abi/stack-model.md	Physical stack lowering
docs/interoperability/cross-language-data.md	Type conversion
docs/interoperability/object-model.md	Object adaptation
docs/interoperability/name-mangling.md	Native symbol naming
docs/interoperability/symbol-resolution.md	Symbol lookup
docs/runtime/runtime-interface.md	Runtime integration
docs/runtime/threading.md	Thread semantics
docs/runtime/async-model.md	Async calls
docs/runtime/resource-management.md	Resource lifecycle
docs/security/security-model.md	Security boundary
docs/security/capability-security.md	Capability enforcement
docs/security/sandboxing.md	Isolation
docs/security/secure-loading.md	Module loading security
docs/distributed/serialization.md	Serialized boundaries
docs/distributed/remote-calls.md	Remote invocation
docs/compatibility/backwards-compatibility.md	Backward compatibility
docs/compatibility/forwards-compatibility.md	Forward compatibility
docs/compatibility/feature-negotiation.md	Profile negotiation
docs/compatibility/capability-discovery.md	Capability discovery
docs/standards/conformance.md	Conformance rules
docs/standards/test-suite.md	FFI tests
docs/standards/certification.md	Certification


This is deliberate because the repository already has separate documents for many of these concerns. For example, the repository explicitly has core-abi.md, data-types.md, memory-model.md, register/stack specifications, and distributed/compatibility documents.


---

5. Required code/module tree

I recommend making the eventual implementation tree explicit now:

implementations/
└── ffi/
    ├── mod.*
    │
    ├── interface_loader.*
    ├── interface_validator.*
    ├── function_binder.*
    ├── signature_validator.*
    ├── type_mapper.*
    │
    ├── call_adapter.*
    ├── abi_lowering.*
    ├── return_adapter.*
    │
    ├── error_adapter.*
    ├── exception_adapter.*
    │
    ├── ownership_adapter.*
    ├── lifetime_adapter.*
    ├── resource_adapter.*
    ├── handle_manager.*
    │
    ├── capability_adapter.*
    ├── effect_adapter.*
    │
    ├── callback_adapter.*
    ├── closure_adapter.*
    ├── async_adapter.*
    ├── cancellation_adapter.*
    ├── thread_adapter.*
    │
    ├── symbol_resolver.*
    ├── module_manager.*
    ├── version_validator.*
    ├── profile_validator.*
    ├── security_validator.*
    │
    ├── invocation_context.*
    ├── diagnostics.*
    └── errors.*

The * is intentional: ULABI must not prescribe Rust, C++, Go, Python, or any other implementation language.

6. Schemas

The FFI metadata layer should eventually contain:

schemas/
├── interface.schema.*
├── function.schema.*
├── signature.schema.*
├── parameter.schema.*
├── result.schema.*
├── type-binding.schema.*
├── ffi-import.schema.*
├── ffi-export.schema.*
├── ffi-binding.schema.*
├── ownership.schema.*
├── lifetime.schema.*
├── capability.schema.*
├── effect.schema.*
├── profile.schema.*
└── ffi-error.schema.*

These are independent from the implementation language.

7. Tests

The corresponding tests should be:

tests/
└── ffi/
    ├── interface_discovery.*
    ├── function_binding.*
    ├── signature_validation.*
    ├── primitive_types.*
    ├── aggregate_types.*
    ├── ownership.*
    ├── borrowing.*
    ├── lifetimes.*
    ├── errors.*
    ├── exceptions.*
    ├── callbacks.*
    ├── closures.*
    ├── async.*
    ├── cancellation.*
    ├── threading.*
    ├── capabilities.*
    ├── effects.*
    ├── dynamic_loading.*
    ├── static_linking.*
    ├── versioning.*
    ├── compatibility.*
    ├── security.*
    ├── module_unloading.*
    └── negative.*

The negative tests are particularly important. A serious ABI standard must prove that invalid foreign interfaces are rejected, not merely that valid examples work.

8. What we should not add to the FFI module

To prevent future repetition, I would explicitly prohibit these from being implemented inside ffi/:

❌ universal type definitions
❌ independent serialization format
❌ independent memory-management system
❌ independent versioning system
❌ independent object model
❌ independent capability model
❌ independent transport protocol
❌ independent distributed RPC protocol
❌ language-specific semantics in the ULABI core
❌ Zamani-specific implementation
❌ Sankofa-specific implementation

The FFI is the adapter/integration layer, not another competing ABI.

Recommended completion boundary

Once docs/interoperability/foreign-function-interface.md is copied in, we should consider the FFI contract itself frozen for the current draft. Subsequent implementation of cross-language-data.md, object-model.md, name-mangling.md, symbol-resolution.md, or the runtime/security documents should implement the integration points already declared here rather than forcing this document to be redesigned.

That directly satisfies your requirement that when one file is finished, later files should integrate with it rather than making us reopen it.