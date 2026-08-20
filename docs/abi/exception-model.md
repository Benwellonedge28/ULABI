I read the repository’s current ULABI-DESIGN.md before preparing this. It is currently a 0.2.0-draft, and its architecture already establishes the key principles we need: language/runtime/platform neutrality, a minimal stable Core, extension profiles, explicit execution/locality semantics, semantic types, function contracts, effects, execution semantics, and explicit memory-boundary semantics.

The repository currently contains the master design, ULABI-SPEC.md, versioning/governance placeholders, and docs/; so the next documents should build on those rather than redefining them.

Below is docs/abi/exception-model.md, designed so it can be implemented without needing to rewrite this document when the later ABI documents are added.


---

docs/abi/exception-model.md

# ULABI Exception Model

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/abi/exception-model.md`  
**Specification Area:** Core ABI / Error and Exception Semantics  
**Status:** Architecture / Normative Design  
**Specification Version:** 0.2.0-draft  
**Repository:** `Benwellonedge28/ULABI`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI exception and failure model.

The purpose of the ULABI Exception Model is to provide a language-neutral,
runtime-neutral, platform-neutral and transport-neutral mechanism for
representing, propagating, containing, translating, observing and recovering
from exceptional execution conditions across ABI boundaries.

ULABI must allow independently implemented systems to communicate failure
semantics without requiring them to share:

- a programming language;
- an exception mechanism;
- a runtime;
- a compiler;
- an operating system;
- a CPU architecture;
- a memory-management model;
- a transport;
- an object model;
- or a vendor.

ULABI therefore defines the **semantic contract of failure**, rather than
requiring every implementation to use the same internal exception mechanism.

---

# 2. Relationship to ULABI Architecture

This document is subordinate to:

- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

The master architecture establishes that ULABI is an interoperability contract
rather than a programming language, runtime or operating system.

The ULABI architecture also defines:

- language neutrality;
- runtime neutrality;
- explicit execution semantics;
- explicit failure semantics;
- `Result<T,E>`;
- capability declarations;
- effects;
- deterministic behavior;
- process and sandbox isolation;
- compatibility;
- fault containment;
- recovery;
- extension profiles.

This document specializes those principles for exceptional execution.

It does not redefine them.

---

# 3. Fundamental Principle

ULABI does not standardize the internal exception mechanism of an implementation.

An implementation may internally use:

- hardware exceptions;
- language exceptions;
- status codes;
- `Result<T,E>`;
- tagged unions;
- error objects;
- traps;
- signals;
- futures;
- promises;
- asynchronous error channels;
- supervisor messages;
- or another mechanism.

At a ULABI boundary, however, the implementation MUST expose failure
semantics through the ULABI exception model.

Therefore:

```text
Implementation-specific failure mechanism
              |
              v
       ULABI Error Contract
              |
              v
Implementation-specific failure mechanism

The boundary is standardized.

The implementation is not.


---

4. Exception vs Error

ULABI distinguishes between:

1. expected operation failure;


2. exceptional execution failure;


3. cancellation;


4. resource exhaustion;


5. security rejection;


6. contract violation;


7. process failure;


8. transport failure;


9. infrastructure failure;


10. unrecoverable termination.



These categories MUST NOT be collapsed into one generic "exception" value.

A caller must be able to determine the semantic class of a failure.


---

5. Primary Failure Categories

ULABI defines the following top-level failure classes.

5.1 ApplicationError

The requested operation executed according to its contract but could not produce the requested result.

Examples:

NotFound
InvalidInput
AlreadyExists
Conflict
Unavailable
Rejected

Application errors are normally recoverable by the caller.


---

5.2 ContractError

The caller or callee violated the declared ULABI interface contract.

Examples:

InvalidArgument
InvalidState
UnsupportedOperation
TypeMismatch
ProtocolViolation
OwnershipViolation
CapabilityViolation

Contract errors indicate a programming, integration, protocol or validation failure.


---

5.3 ResourceError

The operation could not complete because a required resource was unavailable or exhausted.

Examples:

OutOfMemory
ResourceLimitExceeded
HandleLimitExceeded
StackLimitExceeded
TimeLimitExceeded
QuotaExceeded
StorageUnavailable

Resource failures MUST include sufficient metadata to distinguish temporary resource exhaustion from permanent resource unavailability when known.


---

5.4 SecurityError

The operation was rejected because execution would violate a security policy.

Examples:

Unauthorized
Forbidden
CapabilityDenied
SandboxViolation
IntegrityFailure
AuthenticationFailure
PolicyViolation
SecureLoadFailure

Security failures MUST NOT be silently converted into ordinary application errors.


---

5.5 Cancellation

Cancellation indicates that execution was intentionally terminated by an authorized cancellation mechanism.

Cancellation is semantically distinct from failure.

Cancelled

An implementation MUST NOT report cancellation as success.

An implementation MUST NOT reinterpret unauthorized termination as cancellation.


---

5.6 Timeout

A timeout indicates that a defined execution deadline expired.

DeadlineExceeded

Timeouts MUST carry enough information to determine which deadline was violated when the implementation has that information.


---

5.7 TransportError

Transport errors occur when a ULABI operation crosses an execution or transport boundary and communication fails.

Examples:

ConnectionFailure
PeerUnavailable
MessageCorruption
TransportTimeout
TransportClosed
ProtocolMismatch

Transport errors are especially important for process-local and distributed ULABI execution.

A local call MUST NOT be silently converted into a remote call merely to recover from a transport failure.


---

5.8 RuntimeError

Runtime errors indicate failure in the execution environment.

Examples:

RuntimeUnavailable
RuntimeInvariantViolation
LoaderFailure
SchedulerFailure
RuntimeResourceFailure

Runtime errors SHOULD normally be treated as infrastructure failures rather than application-level failures.


---

5.9 HardwareError

Hardware errors indicate failure of an underlying device or execution unit.

Examples:

DeviceUnavailable
DeviceFailure
MemoryHardwareFailure
AcceleratorFailure
UncorrectableHardwareFault

Hardware-specific details MAY be included through structured diagnostic metadata.


---

5.10 InternalError

An internal error indicates that an implementation encountered an unexpected condition for which no more specific public failure category is appropriate.

Example:

InternalFailure

Implementations SHOULD avoid exposing internal implementation details that could create a security or information-disclosure risk.


---

5.11 FatalError

A fatal error indicates that the execution context cannot safely continue.

Examples:

CorruptedExecutionState
UnrecoverableIntegrityFailure
FatalRuntimeFailure
FatalHardwareFailure

Fatal failures MAY terminate:

the current operation;

the current ULABI context;

the current process;

the current execution environment.


The scope MUST be explicit.


---

6. Exception Severity

Every ULABI failure SHOULD expose a severity classification.

Recommended severity levels:

Info
Recoverable
Degraded
Critical
Fatal

Severity does not determine whether an error is recoverable.

Recovery semantics are determined by the contract and policy.


---

7. Exception Identity

Every externally visible ULABI error MUST have a stable error identity.

The identity MUST NOT depend on:

memory address;

process ID;

pointer value;

compiler-generated class name;

source-code location;

implementation-specific symbol;

stack address.


A recommended conceptual representation is:

ErrorIdentity {
    namespace
    code
    version
}

Example:

ulabi.memory.OUT_OF_MEMORY@1

The exact wire encoding is defined by the canonical encoding specification.


---

8. Error Code

Each standardized error MUST have a stable numeric or symbolic code.

The code MUST:

be unique within its namespace;

remain stable across compatible versions;

not be reused for a different semantic meaning;

be independently identifiable;

be machine-readable.


Human-readable text MUST NOT be the sole representation of an error identity.


---

9. Error Namespace

ULABI errors are namespaced.

Example namespaces:

ulabi.core
ulabi.type
ulabi.memory
ulabi.call
ulabi.runtime
ulabi.security
ulabi.transport
ulabi.resource
ulabi.hardware
ulabi.async
ulabi.distributed

Extensions MUST use their own registered namespaces.

Implementations MUST NOT impersonate standardized ULABI namespaces.


---

10. Error Descriptor

A ULABI error SHOULD conceptually contain:

Error {
    identity
    category
    severity
    message?
    details?
    cause?
    source?
    operation?
    retry_policy?
    recovery_hint?
    trace_context?
    timestamp?
    correlation_id?
}

Optional fields MUST NOT become mandatory merely because one implementation supports them.


---

11. Human-Readable Messages

Human-readable error messages are informational.

They MUST NOT be used as the stable basis for:

program control flow;

compatibility;

error classification;

retry decisions;

security decisions.


Programs MUST use the machine-readable error identity and structured metadata.


---

12. Error Cause Chains

ULABI supports causal error relationships.

Example:

ApplicationError
    |
    +-- caused by ResourceError
             |
             +-- caused by RuntimeError

A cause chain SHOULD preserve:

error identity;

category;

relevant structured details;

correlation information.


Implementations MUST place limits on:

chain depth;

total serialized size;

recursive nesting.


This prevents denial-of-service through maliciously deep error chains.


---

13. Error Context

An error MAY include contextual metadata.

Examples:

operation
parameter
resource
capability
interface_id
function_id
request_id
correlation_id
deadline

Sensitive information MUST NOT be included unless the receiving security context is authorized to access it.


---

14. Stack Traces

Stack traces are implementation diagnostics.

A stack trace is NOT part of the stable semantic identity of an error.

Implementations MAY attach stack traces when:

diagnostics are enabled;

policy permits disclosure;

the receiving environment is authorized.


Stack traces MUST NOT be required for ULABI compatibility.


---

15. Exception Propagation

An error crossing a ULABI boundary MUST be converted into a ULABI-compatible error representation.

Conceptually:

Language A
   |
   | internal exception
   v
ULABI Error
   |
   v
Language B
   |
   | native error representation

The receiving implementation MAY convert the ULABI error into its own native error mechanism.

The semantic identity MUST remain recoverable.


---

16. Propagation Rules

An implementation MUST NOT silently discard a failure returned by a ULABI operation.

If an operation returns:

Result<T,E>

the implementation MUST preserve the distinction between:

Success(T)
Failure(E)

If an implementation maps E into a native exception, the original ULABI error identity MUST remain recoverable.

If an implementation maps a native exception into ULABI, it MUST choose the most specific valid ULABI category available.


---

17. Exception Translation

Exception translation is allowed.

Example:

NativeException
      |
      v
ULABI Error
      |
      v
NativeException

Translation MUST preserve, where possible:

error identity;

semantic category;

severity;

retry semantics;

cancellation semantics;

security semantics;

causal relationships.


Translation MUST NOT invent a successful result.


---

18. Unknown Errors

A receiving implementation MUST be able to represent an error whose identity it does not recognize.

Example:

UnknownError {
    namespace
    code
    version
    opaque_details?
}

Unknown errors MUST NOT automatically be treated as success.

Unless the interface explicitly declares otherwise, an unknown failure MUST remain a failure.


---

19. Unknown Error Extensions

An implementation encountering an unknown extension field SHOULD preserve it when safe and feasible.

Unknown fields MUST NOT cause failure solely because they are unknown unless the relevant contract declares strict interpretation.

This supports forward compatibility.


---

20. Retry Semantics

An error MAY declare retry semantics.

Possible classifications:

NotRetryable
RetryImmediately
RetryAfterDelay
RetryAfterCondition
RetryWithModifiedRequest
Unknown

Retry metadata is advisory unless the interface contract makes it normative.

A non-idempotent operation MUST NOT automatically be retried merely because an error appears transient.


---

21. Idempotency

Exception handling MUST respect the function's execution semantics.

For example:

Operation:
    CreatePayment
Idempotent:
    false

A transport timeout does not prove that the operation was never executed.

Therefore:

Timeout != NotExecuted

The caller MUST NOT assume rollback merely because a response was not received.


---

22. Cancellation Semantics

Cancellation MUST be represented independently from failure.

Conceptually:

Running
   |
   +--> Cancel Requested
            |
            v
       Cancellation

A cancellation request MUST be subject to the relevant capability and authorization policy.

An implementation MAY refuse cancellation when the operation contract does not permit it.


---

23. Cancellation Race

Cancellation may race with completion.

Possible outcomes include:

CompletedBeforeCancellation
CancelledBeforeCompletion
CompletionAndCancellationRace

The interface MUST define the observable result.

The implementation MUST NOT fabricate a deterministic ordering where the underlying contract cannot guarantee one.


---

24. Deadline Semantics

A ULABI operation MAY specify:

deadline
timeout
budget

A deadline is an execution constraint.

When the deadline is exceeded, the implementation SHOULD return:

ulabi.core.DEADLINE_EXCEEDED

unless a more specific contractually defined failure is appropriate.


---

25. Partial Failure

Operations that may produce partial results MUST explicitly define partial failure semantics.

Example:

Result {
    completed_items
    failed_items
    final_error
}

An implementation MUST NOT assume that a failed operation produced no side effects.

The contract MUST state:

whether partial effects are possible;

whether partial results are observable;

whether rollback is available;

whether retry is safe.



---

26. Atomicity

An operation MAY declare:

Atomic
BestEffort
Partial
Transactional

If an operation is declared atomic, a successful return MUST indicate that the defined atomicity conditions were satisfied.

If atomicity cannot be guaranteed, the implementation MUST NOT advertise the operation as atomic.


---

27. Exception Boundaries

ULABI defines explicit exception boundaries.

A boundary may exist between:

Function
Thread
Task
Process
Runtime
Host
Device
Machine
Distributed Service

An error MUST NOT cross a boundary unless the boundary contract permits it.

For example, an internal implementation exception MUST NOT automatically escape a process sandbox.


---

28. Process Isolation

An out-of-process ULABI implementation MUST treat process termination as a distinct failure condition.

Example:

PeerProcessExited
PeerProcessCrashed
PeerProcessKilled
PeerProcessUnresponsive

These MUST NOT be represented merely as generic application exceptions.


---

29. Distributed Failure

Distributed execution introduces failures that do not exist in purely local execution.

Examples:

NetworkPartition
PeerUnavailable
DuplicateDelivery
LostResponse
DelayedResponse
UnknownExecutionOutcome

These belong to the distributed execution profile.

The exception model MUST preserve the distinction between:

operation failed

and:

operation outcome unknown

This distinction is critical for non-idempotent operations.


---

30. Unknown Execution Outcome

The following state is explicitly supported:

Request Sent
     |
     v
Response Lost
     |
     v
Outcome Unknown

This MUST NOT automatically become:

Failure

or:

Success

Instead, implementations SHOULD expose an explicit state such as:

UnknownOutcome

when the contract permits it.


---

31. Error Ownership

Errors are data.

The memory ownership of an error object MUST follow the ULABI memory boundary contract.

An implementation MUST NOT:

return a pointer to invalid memory;

expose a private allocator object without an agreed contract;

require the receiver to free memory using an incompatible allocator.


Errors MUST be transferable, borrowed, or otherwise managed according to the declared ownership semantics.


---

32. Error Lifetime

Error lifetime MUST be explicitly defined at an ABI boundary.

A returned error MUST remain valid for the lifetime declared by the interface.

Possible models include:

Borrowed
Owned
Transferred
ReferenceCounted
RegionBound
Serialized

The implementation MUST NOT assume that all languages share one lifetime model.


---

33. Error Serialization

Errors crossing process or machine boundaries MUST use the canonical ULABI encoding rules.

Serialization MUST preserve:

error identity;

category;

severity;

supported structured metadata;

causal relationships;

compatibility information.


Implementation-specific exception objects MUST NOT be serialized directly as the portable ULABI representation.


---

34. Error Size Limits

Implementations MUST enforce configurable limits on:

error message size;

detail size;

cause-chain depth;

metadata count;

serialized error size.


This protects against denial-of-service conditions.


---

35. Recursive Errors

Error structures MUST NOT allow unbounded recursive expansion.

An implementation MUST detect or limit:

Error -> Cause -> Cause -> Cause -> ...

and equivalent recursive structures.


---

36. Security Requirements

Error handling is part of the security boundary.

Implementations MUST prevent:

information leakage;

credential leakage;

secret disclosure;

memory disclosure;

path disclosure where policy forbids it;

stack disclosure where policy forbids it;

capability disclosure;

internal topology disclosure;

cryptographic secret disclosure.


Error messages MUST be treated as potentially observable by an untrusted party.


---

37. Capability Security

An error MUST NOT grant a capability.

An error may identify that a capability was required:

CapabilityDenied {
    capability_id
}

but MUST NOT return an authority token unless the relevant security contract explicitly permits such behavior.


---

38. Error Authenticity

When an error crosses an untrusted boundary, implementations SHOULD provide integrity and authenticity protection where required by the security profile.

This is particularly important for:

distributed execution;

remote calls;

secure loading;

capability systems;

privileged operations.



---

39. Exception Injection

Implementations MUST assume that externally supplied error objects may be malicious.

Therefore, received errors MUST be validated before:

deserialization;

recursive traversal;

logging;

rendering;

retry processing;

policy evaluation.



---

40. Logging Safety

Implementations MUST NOT blindly log untrusted error strings or metadata.

Logging systems SHOULD protect against:

log injection;

control characters;

unbounded messages;

sensitive-data leakage.



---

41. Determinism

For deterministic ULABI operations, equivalent failures MUST produce deterministically comparable machine-readable identities.

Human-readable diagnostic text may differ between implementations.


---

42. Error Ordering

If multiple failures occur, the contract MUST define whether the implementation returns:

FirstFailure
PrimaryFailure
AggregateFailure
AllFailures

An implementation MUST NOT claim deterministic error ordering when the execution model cannot guarantee it.


---

43. Aggregate Errors

ULABI MAY support:

AggregateError {
    errors[]
}

Aggregate errors are useful for:

batch operations;

parallel execution;

validation;

distributed systems.


Aggregate size MUST be bounded.


---

44. Error Suppression

An implementation MAY suppress an internal error only when:

1. the interface explicitly permits suppression;


2. suppression does not change the externally observable contract;


3. security policy permits suppression;


4. required diagnostics remain available.



Silent suppression of contract violations is prohibited.


---

45. Recovery Semantics

The exception model does not itself authorize recovery.

Recovery is governed by:

the operation contract;

the runtime contract;

the resource policy;

the security policy;

the reliability profile;

the self-healing profile where enabled.


An error MAY provide a recovery hint.

A recovery hint MUST NOT be interpreted as authorization.


---

46. Recovery Hint

Example:

RecoveryHint {
    retryable
    retry_after
    restart_required
    reconnect_required
    capability_required
}

Hints are advisory unless declared normative by the relevant interface.


---

47. Rollback

Rollback MUST NOT be assumed merely because an operation failed.

An operation MUST explicitly define whether it supports:

NoRollback
LocalRollback
TransactionalRollback
CompensatingAction
ExternalRollback

Rollback semantics belong to the relevant transaction/reliability profile.


---

48. Exception Safety Levels

ULABI MAY classify operations according to exception safety.

Recommended levels:

NoGuarantee
BasicGuarantee
StrongGuarantee
NoThrow

Definitions:

NoGuarantee

Failure may leave externally observable partial effects.

BasicGuarantee

The system remains internally valid, but partial effects may remain.

StrongGuarantee

Failure leaves the operation's defined state unchanged.

NoThrow

The operation contract guarantees that the operation does not propagate a ULABI exception.

NoThrow MUST NOT mean that the operation can never fail.

It may instead return an explicit Result failure.


---

49. Result-Based APIs

ULABI strongly supports:

Result<T,E>

for expected operation failures.

Example:

Result<User, UserError>

Expected domain failures SHOULD use Result.

Exceptional runtime conditions MAY use the ULABI exception mechanism.

Implementations MAY map between these models.


---

50. Exceptions and Result Are Not Equivalent

The following distinction MUST be preserved:

Expected domain failure
        !=
Exceptional runtime failure

For example:

UserNotFound

may be a normal Result failure.

Whereas:

RuntimeInvariantViolation

may be an exceptional runtime failure.


---

51. ABI Contract

A ULABI function contract SHOULD declare:

Function {
    interface_id
    function_id
    parameters
    return_type
    error_set
    effects
    execution_mode
    ownership
    capability_requirements
    cancellation
    deadline
    exception_safety
}

The exact representation is defined by the Core ABI and calling-convention specifications.


---

52. Declared Error Sets

An interface MAY declare a closed or open error set.

Closed

Only explicitly declared errors may be returned through the normal contract.

Open

The implementation may return additional errors that conform to the ULABI error model.

Open error sets are recommended for infrastructure-facing interfaces.


---

53. Compatibility

Adding a new error to an open error set SHOULD be backward compatible.

Adding a new error to a closed error set may require a version or capability negotiation change.

Removing or reusing an existing error identity is NOT backward compatible.

Changing the semantic meaning of an existing error code is NOT permitted.


---

54. Forward Compatibility

Receivers MUST be capable of representing unknown ULABI errors.

This is required so that a newer implementation can communicate with an older implementation without forcing either side to reinterpret failure as success.


---

55. ABI Stability

The following MUST remain stable within a compatible ABI version:

error identity;

error category semantics;

machine-readable code;

required fields;

ownership semantics;

propagation semantics;

cancellation semantics;

declared compatibility guarantees.



---

56. Conformance Requirements

An implementation claiming conformance to the ULABI Exception Model MUST:

1. represent machine-readable error identities;


2. distinguish success from failure;


3. preserve declared error semantics;


4. support unknown errors;


5. preserve error identity across supported ABI boundaries;


6. enforce error-size limits;


7. prevent unbounded cause chains;


8. distinguish cancellation from ordinary failure;


9. distinguish timeout from ordinary failure where supported;


10. preserve security failures;


11. respect declared ownership and lifetime rules;


12. avoid treating human-readable messages as error identity;


13. implement the declared propagation semantics;


14. validate received error data;


15. satisfy the applicable Core ABI requirements.




---

57. Conformance Tests

The ULABI test suite MUST eventually include tests for:

Identity

stable error IDs;

namespace isolation;

version handling;

unknown error handling.


Propagation

local propagation;

cross-language propagation;

process-boundary propagation;

distributed propagation.


Translation

native exception -> ULABI;

ULABI -> native exception;

Result -> ULABI;

ULABI -> Result.


Security

oversized errors;

malicious metadata;

recursive cause chains;

sensitive-data leakage;

forged error metadata.


Compatibility

old receiver / new error;

new receiver / old error;

unknown extension fields;

version negotiation.


Cancellation

cancellation before execution;

cancellation during execution;

cancellation after completion;

cancellation races.


Distributed Failure

lost response;

peer termination;

timeout;

unknown execution outcome;

duplicate request;

duplicate response.


Recovery

retryable error;

non-retryable error;

rollback-supported operation;

rollback-unsupported operation.



---

58. Reference State Machine

A generic ULABI operation follows:

+----------------+
                 |    Created     |
                 +-------+--------+
                         |
                         v
                 +----------------+
                 |    Running     |
                 +---+--------+---+
                     |        |
              success|        |failure
                     |        |
                     v        v
              +---------+  +-----------+
              | Success |  |  Failed   |
              +---------+  +-----+-----+
                                  |
                         +--------+--------+
                         |                 |
                     recover             escalate
                         |                 |
                         v                 v
                  +------------+     +-----------+
                  | Recovering |     | Terminate |
                  +-----+------+     +-----------+
                        |
                    verify
                        |
                 +------+------+
                 |             |
              healthy       unhealthy
                 |             |
                 v             v
              Success       Rollback /
                             Escalate


---

59. Failure Processing Pipeline

The normative conceptual pipeline is:

Failure
   |
   v
Classify
   |
   v
Assign Stable Identity
   |
   v
Attach Safe Context
   |
   v
Apply Security Policy
   |
   v
Determine Propagation
   |
   v
Translate if Required
   |
   v
Deliver to Caller
   |
   v
Caller Policy
   |
   +---- Retry
   |
   +---- Recover
   |
   +---- Rollback
   |
   +---- Escalate
   |
   +---- Terminate

The exception model itself does not decide which branch must be taken.

That decision belongs to the operation contract and applicable policy.


---

60. No Hidden Recovery

ULABI implementations MUST NOT silently perform arbitrary recovery actions merely because an exception occurred.

In particular, an implementation MUST NOT automatically:

modify executable code;

modify interface definitions;

modify security policy;

grant capabilities;

alter ownership;

change memory protection;

disable sandboxing;

change ABI semantics;

silently retry non-idempotent operations;

silently contact another service;

silently migrate execution.


Any such behavior requires an explicit contract and authorized policy.


---

61. Self-Healing Integration

The ULABI Self-Healing Profile may consume the exception model.

The relationship is:

Exception Model
      |
      v
Failure Evidence
      |
      v
Self-Healing Policy
      |
      v
Detection
      |
      v
Diagnosis
      |
      v
Isolation
      |
      v
Authorized Recovery
      |
      v
Verification
      |
      +---- Healthy ----> Continue
      |
      +---- Unhealthy --> Rollback / Escalate

The exception model supplies evidence.

It does not authorize autonomous modification.


---

62. Observability Integration

Errors SHOULD integrate with:

tracing;

diagnostics;

telemetry;

deterministic debugging.


Where supported, an error may carry:

trace_id
span_id
correlation_id
request_id

Observability metadata MUST NOT alter error identity.


---

63. Debugging Integration

Debugging information is an extension of the semantic error model.

Debuggers MAY expose:

stack traces;

source locations;

registers;

thread state;

memory state;

causal history.


Such information is not required for ordinary ULABI interoperability.


---

64. Memory Integration

The exception model depends on the memory model for:

ownership;

lifetime;

transfer;

borrowing;

allocation;

serialization buffers.


This document defines the semantic requirements.

docs/abi/memory-model.md defines the corresponding memory contract.


---

65. Calling Convention Integration

The calling convention MUST define how exceptional returns are represented at the ABI boundary.

Possible mechanisms include:

status + return value
Result<T,E>
exception register/state
out-of-band error object
tagged return structure

ULABI does not require one mechanism for all implementations.

The chosen mechanism MUST preserve the semantic contract defined here.


---

66. Data Type Integration

Error objects MUST use only data representations supported by the ULABI type system and canonical encoding rules when crossing a portable boundary.

Implementation-specific exception classes MUST NOT become required ULABI types.


---

67. Runtime Integration

The runtime interface is responsible for:

exception delivery;

task termination;

cancellation;

runtime failure;

process failure;

resource exhaustion.


The runtime MUST preserve the semantic distinctions established by this document.


---

68. Security Integration

The security model governs:

error visibility;

authentication failures;

authorization failures;

capability failures;

sensitive diagnostics;

error authenticity.


Security failures MUST NOT be downgraded merely to make interoperability easier.


---

69. Distributed Integration

The distributed ABI defines:

remote failure;

transport failure;

unknown outcome;

retry;

duplicate execution;

timeout;

peer failure.


The exception model provides the common semantic error representation.


---

70. Versioning Integration

Error identities are versioned independently from implementation-specific exception classes.

Compatible versions MUST preserve existing error semantics.

Breaking changes require the applicable ULABI versioning mechanism.


---

71. Implementation Independence

A conforming implementation may implement this model using:

C exceptions / status objects
C++ exceptions
Rust Result
Python exceptions
Java exceptions
Go error values
Swift Error
Kotlin exceptions
Fortran status values
Ada exceptions
Zamani native errors
Sankofa native errors

No implementation is required to adopt another language's error mechanism.


---

72. Required Invariants

The following invariants are normative.

E-001

A failure MUST NOT be represented as success.

E-002

A machine-readable error identity MUST be stable within a compatible version.

E-003

Human-readable error text MUST NOT be the sole error identity.

E-004

Unknown errors MUST remain representable.

E-005

Cancellation MUST remain distinguishable from ordinary failure.

E-006

Security failures MUST remain distinguishable from ordinary application failures.

E-007

Error propagation MUST respect ownership and lifetime rules.

E-008

Error data MUST be bounded.

E-009

Cause chains MUST be bounded.

E-010

Implementations MUST NOT silently perform unauthorized recovery.

E-011

Non-idempotent operations MUST NOT be silently retried without authorization.

E-012

Unknown distributed execution outcomes MUST NOT be automatically classified as success.

E-013

Error identity MUST NOT depend on implementation-specific memory addresses, symbols or object layouts.

E-014

ABI compatibility MUST preserve the semantics of existing error identities.


---

73. Required Error Metadata

At minimum, a standardized ULABI error identity consists conceptually of:

namespace
code
version

The following metadata may additionally be standardized by the relevant profile:

category
severity
message
cause
details
retry_policy
operation
capability
trace_context
correlation_id


---

74. Minimal Core Requirement

The ULABI Core MUST remain small.

The Core exception contract SHOULD therefore standardize only what is necessary for reliable interoperability:

error identity;

error category;

propagation;

unknown errors;

compatibility;

basic ownership;

basic validation.


Advanced facilities such as:

distributed diagnostics;

rich tracing;

automated recovery;

self-healing;

hardware diagnostics;


belong in extension profiles where appropriate.


---

75. Design Rule

The fundamental rule of this document is:

> ULABI standardizes the meaning and propagation of failure, not the internal mechanism used by an implementation to produce or handle failure.




---

76. Integration Contract

This document is intentionally complete with respect to its required integration points.

Later documents MUST integrate with these defined concepts rather than changing them silently.

The following documents consume this contract:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/return-values.md

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/cross-language-data.md

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/distributed-errors.md

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md

docs/observability/diagnostics.md
docs/observability/tracing.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md

These documents MUST consume the exception semantics defined here.

They MUST NOT redefine the meaning of existing ULABI error identities.


---

77. Required Schemas

The following future machine-readable schemas SHOULD be created:

schemas/error.json
schemas/error-identity.json
schemas/error-category.json
schemas/error-details.json
schemas/error-cause.json
schemas/error-policy.json
schemas/error-compatibility.json

These schemas should correspond to this specification and MUST NOT introduce language-specific object models.


---

78. Required Examples

The repository SHOULD eventually contain:

examples/errors/
├── basic-error/
├── result-error/
├── exception-translation/
├── cancellation/
├── timeout/
├── resource-error/
├── security-error/
├── process-failure/
├── distributed-failure/
├── unknown-error/
└── recovery/

Examples are illustrative and MUST NOT override normative requirements.


---

79. Required Tests

The repository SHOULD eventually contain:

tests/abi/exceptions/
├── error_identity/
├── error_categories/
├── propagation/
├── translation/
├── unknown_errors/
├── cancellation/
├── timeout/
├── ownership/
├── limits/
├── security/
├── compatibility/
└── distributed/


---

80. Required Conformance Tests

The conformance suite SHOULD include language-independent tests such as:

exception_identity_stable
unknown_error_preserved
error_category_preserved
error_translation_preserves_identity
cancellation_is_distinct
timeout_is_distinct
security_error_is_distinct
oversized_error_rejected
deep_cause_chain_rejected
non_idempotent_retry_not_implicit
unknown_distributed_outcome_preserved
error_ownership_respected
error_lifetime_respected
forward_error_compatibility
backward_error_compatibility


---

81. Reference Implementation Requirements

The reference implementation SHOULD provide:

ErrorIdentity
Error
ErrorCategory
ErrorSeverity
ErrorCause
ErrorContext
RetryPolicy
Cancellation
UnknownOutcome
ErrorTranslator
ErrorValidator
ErrorSerializer

The reference implementation MUST demonstrate the specification without becoming the specification itself.


---

82. Final Architectural Rule

ULABI does not require all languages to throw exceptions.

ULABI does not require all languages to return Result.

ULABI does not require all runtimes to share one error object.

ULABI requires that when execution crosses a ULABI boundary, failure has a stable, machine-readable, secure, versioned and interoperable meaning.

Therefore:

Language-specific failure
        |
        v
   ULABI Error Contract
        |
        v
Language-specific failure

is the universal interoperability boundary.


---

83. Status

This document defines the proposed ULABI Exception Model.

It is normative once incorporated into the applicable ULABI specification version.

Future changes MUST follow the ULABI versioning and governance rules.

End of docs/abi/exception-model.md.

## What must be implemented — and in which files

The important point is that **the Markdown specification is not the implementation**. We should build the implementation around stable, language-neutral contracts.

I would use this implementation architecture:

```text
ULABI/
├── schemas/
│   ├── error.json
│   ├── error-identity.json
│   ├── error-category.json
│   ├── error-details.json
│   ├── error-cause.json
│   ├── error-policy.json
│   └── error-compatibility.json
│
├── reference/
│   └── exception/
│       ├── error_identity.*
│       ├── error.*
│       ├── error_category.*
│       ├── error_severity.*
│       ├── error_cause.*
│       ├── error_context.*
│       ├── retry_policy.*
│       ├── cancellation.*
│       ├── unknown_outcome.*
│       ├── error_validator.*
│       ├── error_translator.*
│       └── error_serializer.*
│
├── implementations/
│   ├── c/
│   ├── cpp/
│   ├── rust/
│   ├── go/
│   ├── python/
│   ├── java/
│   ├── swift/
│   ├── kotlin/
│   ├── fortran/
│   ├── ada/
│   └── ...
│
├── tests/
│   └── abi/
│       └── exceptions/
│           ├── error_identity/
│           ├── error_categories/
│           ├── propagation/
│           ├── translation/
│           ├── unknown_errors/
│           ├── cancellation/
│           ├── timeout/
│           ├── ownership/
│           ├── limits/
│           ├── security/
│           ├── compatibility/
│           └── distributed/
│
└── conformance/
    └── exception/
        ├── identity/
        ├── propagation/
        ├── translation/
        ├── cancellation/
        ├── compatibility/
        └── security/

Core code modules

The canonical conceptual module list should be:

Module	Responsibility	Depends on

error_identity	Stable namespace/code/version identity	Core identifiers
error_category	Application/contract/resource/security/etc. classification	Identity
error_severity	Recoverable/degraded/critical/fatal semantics	Category
error	Complete portable error contract	Identity + category + severity
error_cause	Bounded causal chains	Error
error_context	Safe structured diagnostic context	Error
retry_policy	Retry semantics	Error + execution semantics
cancellation	Cancellation representation	Error + runtime
unknown_outcome	Distributed uncertain execution	Error + distributed ABI
error_validator	Validate untrusted errors	All error structures
error_translator	Native ↔ ULABI error conversion	Error
error_serializer	Canonical cross-boundary encoding	Error + type/encoding system


Integration order

To satisfy your requirement that we finish a file once and do not have to reopen it later, I recommend this dependency order:

1. Core identifiers
        ↓
2. error_identity
        ↓
3. error_category
        ↓
4. error_severity
        ↓
5. error
        ↓
6. error_cause
        ↓
7. error_context
        ↓
8. retry_policy
        ↓
9. cancellation
        ↓
10. unknown_outcome
        ↓
11. error_validator
        ↓
12. error_translator
        ↓
13. error_serializer
        ↓
14. Core ABI integration
        ↓
15. Calling-convention integration
        ↓
16. Return-value integration
        ↓
17. Runtime integration
        ↓
18. Security integration
        ↓
19. Distributed integration
        ↓
20. Conformance suite

That ordering is important. Do not implement error.rs, error.go, error.h, etc. first and then retrofit the calling convention later. The contracts above are deliberately separated so that the later ABI documents consume already-defined semantics.

Files that will eventually consume this document

The exception model has already declared its integration points, so later work can proceed independently:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/return-values.md

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/async-model.md

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/distributed-errors.md

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md

docs/standards/conformance.md
docs/standards/test-suite.md
docs/standards/certification.md

The critical architectural decision is that none of those later documents gets to change the semantic meaning of this exception contract merely because its implementation has a different language, runtime, CPU, or transport.

That is consistent with the master design's central rule that ULABI is the interoperability contract while Zamani, Sankofa, C, Rust, Python, etc. remain independent implementations.

One important correction to the overall implementation plan: the repository is still extremely early-stage—the GitHub tree currently shows the major top-level specification files and docs/, rather than a mature implementation tree.  So we should define the contracts first, then create the implementation modules from those contracts, rather than pretending the currently proposed implementation paths already exist.