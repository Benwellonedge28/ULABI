I read the repository first rather than treating the architecture tree in your message as the current state. The repository is Benwellonedge28/ULABI, with ULABI-DESIGN.md currently at architecture/specification version 0.2.0-draft. The master design already establishes language/runtime/platform neutrality, a minimal Core plus profiles, explicit locality, transport independence, stable interface identity, negotiation/discovery, compatibility, and distributed interoperability.

I also checked the existing distributed and compatibility specifications. In particular, distributed-abi.md already owns the general distributed ABI boundary; consensus-boundaries.md owns the boundary between interoperability and distributed agreement; backward compatibility owns preservation of older contracts; and feature negotiation owns selecting mutually supported features.

The important finding is that docs/distributed/distributed-errors.md is currently only a placeholder, so it is the correct independent file to establish the distributed-error contract rather than modifying an already-defined specification.

Below is the complete specification to copy into:

docs/distributed/distributed-errors.md


---

ULABI Distributed Errors

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: docs/distributed/distributed-errors.md
Specification Family: Distributed Interoperability / Error Model
Status: Normative Specification
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Normative Authority: ULABI-DESIGN.md, ULABI-SPEC.md, ULABI-VERSIONING.md
License: Apache-2.0


---

1. Purpose

This document defines the ULABI distributed-error contract.

Distributed execution introduces failure modes that do not exist, or do not have the same meaning, in purely local execution.

A distributed operation may fail because of:

invalid input;

incompatible interfaces;

unavailable capabilities;

authentication failure;

authorization failure;

serialization failure;

transport failure;

timeout;

cancellation;

resource exhaustion;

service unavailability;

process failure;

host failure;

network partition;

consensus failure;

stale authority;

duplicate execution;

unknown execution outcome;

remote application failure;

protocol violation;

integrity failure.


ULABI MUST represent these conditions without falsely converting fundamentally different failures into one generic error.

The central principle is:

> A distributed error MUST communicate what is known about the operation, not merely that communication failed.



In particular:

Transport Failure
        !=
Remote Execution Failure
        !=
Unknown Execution Outcome
        !=
Remote Rejection

These distinctions are part of the distributed interoperability contract.


---

2. Scope

This specification defines:

1. distributed error identity;


2. error categories;


3. error codes;


4. error severity;


5. error provenance;


6. execution outcome;


7. retryability;


8. idempotency interaction;


9. transient failures;


10. permanent failures;


11. unknown outcomes;


12. partial completion;


13. cancellation errors;


14. timeout errors;


15. transport errors;


16. protocol errors;


17. serialization errors;


18. authentication errors;


19. authorization errors;


20. capability errors;


21. resource errors;


22. availability errors;


23. consistency errors;


24. consensus-related outcomes;


25. stale-authority errors;


26. duplicate execution;


27. integrity failures;


28. compatibility errors;


29. downgrade errors;


30. error propagation;


31. error translation;


32. error preservation;


33. error metadata;


34. retry guidance;


35. recovery guidance;


36. security requirements;


37. observability requirements;


38. conformance requirements.



This document does not define:

a universal transport protocol;

a universal RPC protocol;

a universal consensus algorithm;

authentication algorithms;

cryptographic primitives;

application-specific error taxonomies;

a specific serialization format.


Those belong to their respective specifications or implementation profiles.


---

3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



The distributed-error model is a profile-level contract.

The responsibility boundaries are:

ULABI Core
    |
    +-- Core Errors
    |
    +-- Distributed ABI
             |
             +-- Remote Calls
             |
             +-- Serialization
             |
             +-- Service Discovery
             |
             +-- Distributed Errors
             |
             +-- Consensus Boundaries

The distributed-error specification consumes the contracts defined by:

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md

docs/abi/exception-model.md
docs/abi/core-abi.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/consensus-boundaries.md

docs/security/security-model.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/capability-security.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

docs/observability/diagnostics.md
docs/observability/tracing.md

docs/standards/conformance.md
docs/standards/test-suite.md

This document defines distributed error semantics.

It MUST NOT redefine:

the general ABI exception model;

distributed invocation;

serialization;

service discovery;

consensus;

authentication;

authorization;

compatibility;

graceful degradation.



---

4. Normative Language

The following terms are normative:

MUST

MUST NOT

REQUIRED

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY

OPTIONAL


A conforming implementation MUST satisfy all applicable MUST and MUST NOT requirements.


---

5. Fundamental Principle

A distributed error describes an observable outcome at an interoperability boundary.

The implementation MUST distinguish:

Operation succeeded
Operation failed
Operation was rejected
Operation was cancelled
Operation timed out
Operation was not attempted
Operation execution is unknown
Operation partially completed

The following MUST NOT be conflated:

Request Not Sent
        !=
Request Rejected
        !=
Request Executed and Failed
        !=
Request Executed Successfully
        !=
Execution Unknown

This distinction is especially important for non-idempotent operations.


---

6. Distributed Error Model

A distributed error conceptually consists of:

DistributedError {
    error_id
    category
    code
    severity
    provenance
    operation_state
    retryability
    retry_after
    invocation_id
    interface_identity
    interface_version
    function_id
    capability_context
    security_context
    resource_context
    causal_context
    details
}

Not every field is mandatory in every environment.

However, an implementation MUST provide enough information to preserve the semantics required by the applicable contract.


---

7. Stable Error Identity

Every standardized distributed error MUST have a stable identity.

Conceptually:

ErrorIdentity {
    namespace
    error_code
    major_version
}

The identity MUST NOT depend solely on:

source-language exception names;

class names;

memory addresses;

compiler-generated symbols;

process-local identifiers;

textual error messages.


Error identity is part of the ABI contract.


---

8. Error Codes

Error codes MUST be stable within their specification namespace.

An error code MUST NOT silently change meaning.

For example:

ULABI.DISTRIBUTED.TIMEOUT

MUST NOT later mean:

ULABI.DISTRIBUTED.AUTHORIZATION_FAILURE

If semantics fundamentally change, a new error identity MUST be created.


---

9. Error Categories

ULABI SHOULD provide standardized categories including:

InvalidRequest
InvalidArguments
InterfaceError
VersionError
CompatibilityError

SerializationError
DeserializationError
ProtocolError

AuthenticationError
AuthorizationError
CapabilityError
SecurityError
IntegrityError

TransportError
ConnectionError
TimeoutError
CancellationError

UnavailableError
OverloadedError
ResourceError

ExecutionError
ApplicationError
RemoteException

ConsistencyError
ConsensusError
AuthorityError
StaleAuthorityError

DuplicateExecutionError
UnknownOutcomeError
PartialCompletionError

InternalError
UnknownError

Implementations MAY define additional application-specific errors.

Application-specific errors MUST NOT reuse standardized error identities.


---

10. Error Provenance

Every distributed error SHOULD identify where the error originated.

Possible provenance values include:

Caller
LocalRuntime
LocalTransport
RemoteTransport
RemoteRuntime
RemoteApplication
AuthorizationLayer
SerializationLayer
ConsensusDomain
ResourceManager
SecurityBoundary
Unknown

Example:

provenance = RemoteApplication

means the remote application generated the failure.

This differs from:

provenance = RemoteTransport

where the communication layer failed.


---

11. Operation Outcome

An error MUST preserve the known operation state where applicable.

Possible states include:

NotAttempted
Submitted
Accepted
Executing
Completed
Failed
Cancelled
TimedOut
Rejected
Unknown
PartiallyCompleted

The state:

Unknown

is REQUIRED for situations where the caller cannot establish whether execution occurred.


---

12. Unknown Outcome

Unknown outcome is a first-class distributed state.

For example:

Caller
  |
  | request
  v
Remote Service
  |
  | operation executes
  v
Remote State
  |
  X response lost
  |
Caller

The caller cannot safely conclude:

operation failed

merely because the response was lost.

Therefore:

UnknownOutcome

MUST be available where applicable.

An implementation MUST NOT convert an unknown outcome into a definite failure unless sufficient evidence exists.


---

13. Unknown Outcome Is Not Success

Unknown outcome MUST NOT be interpreted as success.

Likewise:

Unknown

MUST NOT automatically mean:

Failure

The caller MAY need to invoke an operation-status query, reconciliation mechanism, or application-defined recovery procedure.


---

14. Partial Completion

A distributed operation MAY produce externally observable effects before failing.

For example:

Step 1  ✓
Step 2  ✓
Step 3  ✗
Step 4  not executed

The resulting state MUST NOT necessarily be reported as an ordinary failure.

Where applicable, the implementation SHOULD report:

PartialCompletion

with sufficient metadata to identify the known completion state.


---

15. Error Severity

Distributed errors MAY declare severity.

Recommended levels:

Informational
Warning
Recoverable
Error
Critical
Fatal

Severity MUST NOT be confused with retryability.

For example:

Severity: Error
Retryability: Yes

is valid.

Likewise:

Severity: Warning
Retryability: No

is also valid.


---

16. Retryability

Errors SHOULD explicitly declare retryability.

Possible values:

RetryAllowed
RetryRecommended
RetryAfterDelay
RetryWithModification
RetryNotAllowed
RetryUnknown

The implementation MUST NOT automatically retry an operation merely because an error occurred.

Retryability depends on:

idempotency;

operation state;

authorization;

resource availability;

deadline;

duplicate execution risk;

error semantics.



---

17. Idempotency Interaction

The error contract MUST preserve the interaction between failure and idempotency.

For an idempotent operation:

UnknownOutcome

may permit a safe retry.

For a non-idempotent operation:

UnknownOutcome

MUST NOT automatically authorize retry.

A safe retry MAY require:

an idempotency key;

operation-status lookup;

transaction reconciliation;

application-defined deduplication.



---

18. Retry-After

An implementation MAY provide:

retry_after

The value MUST NOT be interpreted as a guarantee that a retry will succeed.

Possible forms include:

absolute_time
duration
policy_token

The exact representation belongs to the applicable profile.


---

19. Retry Budget

Implementations SHOULD enforce bounded retry behavior.

A retry policy SHOULD define:

maximum_attempts
maximum_elapsed_time
maximum_backoff
maximum_resource_cost

Retries MUST NOT create an uncontrolled retry storm.


---

20. Transport Errors

Transport errors represent failures in communication.

Examples:

ConnectionRefused
ConnectionLost
ConnectionReset
RouteUnavailable
TransportTimeout
TransportProtocolFailure

A transport error MUST NOT automatically imply:

remote operation was not executed

The execution outcome may remain unknown.


---

21. Remote Execution Errors

A remote execution error indicates that the remote environment received and processed the operation sufficiently to produce a known execution result.

Examples:

InvalidArguments
ApplicationFailure
ResourceExhausted
PermissionDenied
UnsupportedOperation
InternalFailure

Where execution is known, the implementation SHOULD distinguish remote execution errors from transport errors.


---

22. Request Rejection

A rejected request means execution did not proceed according to the applicable rejection semantics.

Examples:

UnsupportedInterface
UnsupportedVersion
Unauthorized
CapabilityDenied
ResourceLimit
PolicyDenied
MalformedRequest

A rejection SHOULD be distinguishable from:

ExecutionFailed


---

23. Authentication Errors

Authentication errors indicate failure to establish the required identity.

Examples:

MissingCredentials
InvalidCredentials
ExpiredCredentials
InvalidIdentity
AuthenticationProtocolFailure

Authentication failures MUST NOT be reported as generic transport failures when doing so would hide a security-relevant distinction.


---

24. Authorization Errors

Authorization errors indicate that an identified caller is not permitted to perform the requested operation.

Examples:

PermissionDenied
CapabilityDenied
PolicyDenied
ResourceAccessDenied

Authorization failure MUST NOT be silently converted into:

NotFound

unless the security profile explicitly requires information-hiding behavior.


---

25. Capability Errors

A capability error indicates that the requested operation requires a capability that is:

unavailable;

unauthorized;

expired;

revoked;

incompatible;

insufficient.


Example:

Required Capability:
    filesystem.write

Provided:
    filesystem.read

The capability-security specification remains authoritative for capability semantics.

This document only defines how capability-related failure is reported.


---

26. Version and Compatibility Errors

A distributed invocation MAY fail because the parties cannot establish a compatible contract.

Examples:

UnsupportedInterfaceVersion
IncompatibleCoreVersion
UnsupportedProfile
UnsupportedFeature
IncompatibleEncoding
IncompatibleCallingConvention

These MUST be distinguished from application-level failures.

The implementation SHOULD provide sufficient metadata to allow negotiation or graceful degradation where possible.


---

27. Serialization Errors

Serialization errors include:

MalformedEncoding
UnsupportedEncoding
InvalidSchema
InvalidType
InvalidLength
InvalidField
UnknownRequiredField
InvalidVariant
ResourceLimitExceeded

Malformed external data MUST be rejected before unsafe interpretation.

Serialization errors MUST NOT automatically be treated as application failures.


---

28. Protocol Errors

Protocol errors indicate violation of the applicable ULABI communication contract.

Examples:

InvalidMessageSequence
InvalidStateTransition
MissingRequiredField
UnexpectedMessage
InvalidNegotiationState
UnsupportedProtocolVersion

Protocol violations SHOULD terminate or isolate the affected protocol session when continuing would be unsafe.


---

29. Timeout Errors

Timeouts MUST distinguish at least:

ConnectTimeout
RequestTimeout
ResponseTimeout
ExecutionTimeout
DeadlineExceeded

A timeout does NOT necessarily establish that the remote operation did not execute.

Therefore:

Timeout

MAY result in:

UnknownOutcome


---

30. Deadline Exceeded

A deadline-exceeded error means that the relevant deadline condition was reached.

It MUST specify, where applicable, whether:

the caller stopped waiting;

remote execution was requested to stop;

remote execution actually stopped;

the result became invalid;

the operation outcome became unknown.



---

31. Cancellation Errors

Cancellation SHOULD distinguish:

CancellationRequested
CancellationAccepted
CancellationCompleted
CancellationRejected
CancellationTooLate
CancellationUnknown

Cancellation MUST NOT imply rollback of completed side effects unless the operation contract explicitly provides transactional rollback semantics.


---

32. Resource Errors

Resource errors include:

MemoryLimitExceeded
CPUQuotaExceeded
StorageLimitExceeded
BandwidthLimitExceeded
ConcurrencyLimitExceeded
MessageSizeExceeded
RateLimitExceeded
ExecutionQuotaExceeded

The implementation SHOULD provide enough information to determine whether retrying later may be meaningful.


---

33. Availability Errors

Availability errors include:

ServiceUnavailable
EndpointUnavailable
DependencyUnavailable
Overloaded
Maintenance
QuorumUnavailable
LeaderUnavailable

Availability errors MUST NOT be used to hide a known authorization or integrity failure.


---

34. Consensus Errors

Consensus-related errors MAY include:

ConsensusUnavailable
QuorumLost
CommitRejected
CommitUnknown
LeadershipChanged
MembershipChanged
ConfigurationChanged

These errors MUST NOT imply a particular consensus algorithm.

The consensus-boundaries specification remains authoritative for consensus semantics.


---

35. Authority Errors

Authority-related errors MAY include:

NotAuthoritative
StaleAuthority
InvalidEpoch
FencingRejected
LeaseExpired
LeaseRevoked

A stale authority MUST NOT be accepted where the applicable distributed contract requires current authority.


---

36. Duplicate Execution

An operation MAY be detected as a duplicate.

The implementation MAY report:

DuplicateExecution

or:

DuplicateRequest

Where safe, the implementation SHOULD return the original result rather than execute the operation again.

Duplicate detection semantics MUST be defined by the applicable remote-call or distributed profile.


---

37. Integrity Errors

Integrity errors indicate that the implementation cannot trust the received data or execution context.

Examples:

MessageIntegrityFailure
SignatureInvalid
AuthenticationTagInvalid
CorruptPayload
InvalidProof
StateIntegrityFailure

Integrity errors MUST be treated as security-sensitive.

An implementation MUST NOT continue processing corrupted or unauthenticated data when the applicable security profile requires integrity protection.


---

38. Internal Errors

An implementation MAY expose:

InternalError

when a failure cannot safely be classified more precisely.

However, implementations SHOULD provide a more specific standardized error whenever doing so does not reveal sensitive information.


---

39. Unknown Errors

A receiver MUST be able to safely process an error it does not recognize.

Unknown errors MUST preserve:

stable identity;

category where available;

machine-readable representation;

human-readable description where appropriate.


An unknown error MUST NOT automatically be interpreted as success.

Unless the contract explicitly says otherwise, an unknown error SHOULD be treated as failure or uncertainty rather than ignored.


---

40. Error Details

An error MAY include structured details.

Conceptually:

ErrorDetails {
    field
    expected
    actual
    limit
    retry_after
    resource
    capability
    required_version
    supported_versions
    required_feature
    operation_state
    diagnostic_reference
}

Sensitive details MUST NOT be disclosed merely because an error occurred.


---

41. Error Chaining

A distributed error MAY contain a causal chain.

Example:

Caller Error
    |
    +-- TransportError
          |
          +-- ConnectionReset

or:

Caller Error
    |
    +-- RemoteExecutionError
          |
          +-- AuthorizationError

Error chains MUST preserve provenance.


---

42. Error Cause

An error MAY reference a causal error.

Conceptually:

cause {
    error_identity
    provenance
    details
}

Causal references MUST NOT create unbounded recursive error structures.

Implementations SHOULD enforce maximum error-chain depth.


---

43. Error Translation

An implementation MAY translate an error between:

transport-specific;

runtime-specific;

language-specific;

ULABI-standardized.


However, translation MUST preserve semantics.

For example:

Remote:
PermissionDenied

MUST NOT become:

Local:
Success

A translation that loses critical semantics MUST be classified as lossy.


---

44. Error Preservation

When forwarding an error through multiple ULABI boundaries, implementations SHOULD preserve:

original error identity;

original provenance;

original operation state;

original retryability;

original invocation identity;

relevant security classification.


Example:

Service C
   |
   | PermissionDenied
   v
Service B
   |
   | translated but preserved
   v
Service A


---

45. Error Translation Registry

Implementations MAY maintain mappings:

Source Error
     |
     v
ULABI Error
     |
     v
Target Error

Mappings SHOULD be deterministic.

A mapping MUST NOT claim equivalence when only partial semantic similarity exists.


---

46. Lossy Translation

If exact translation is impossible, the implementation MUST NOT silently claim exact equivalence.

It MAY use:

UnknownError

or:

TranslatedError {
    original_identity
    mapped_identity
    translation_quality
}

where the applicable profile defines such metadata.


---

47. Error Security Classification

Errors MAY have security classifications such as:

Public
AuthenticatedOnly
Privileged
Internal
Sensitive
Restricted

The classification MUST control which diagnostic information can cross a boundary.

An implementation MUST NOT disclose:

credentials;

private keys;

secrets;

internal memory;

confidential capability metadata;

sensitive infrastructure information;


merely because a remote caller requests detailed diagnostics.


---

48. Information Disclosure

Error messages MUST be designed to avoid unnecessary information disclosure.

For example:

"User does not exist"

may disclose more information than:

"Authentication failed"

where account enumeration is a security concern.

The security profile determines the appropriate information-hiding behavior.


---

49. Error Rate Limiting

Implementations SHOULD rate-limit repeated error responses where necessary.

This prevents:

error amplification;

resource exhaustion;

diagnostic flooding;

retry storms.


Rate limiting MUST NOT falsely alter the underlying operation semantics.


---

50. Distributed Error Propagation

An error propagating through multiple systems MUST preserve the distinction between:

Originating Failure
Propagation Failure
Translation Failure
Local Handling Failure

Example:

Service C
   |
   | ApplicationError
   v
Service B
   |
   | propagation succeeds
   v
Service A

The final receiver SHOULD be able to determine that the error originated at C.


---

51. Error Aggregation

A distributed system MAY aggregate multiple failures.

Example:

AggregateError {
    failures: [
        NodeA unavailable,
        NodeB timeout,
        NodeC unauthorized
    ]
}

Aggregation MUST preserve individual error identities.

An aggregate error MUST NOT erase important distinctions.


---

52. Partial Failure

A distributed system MAY have:

Node A  ✓
Node B  ✗
Node C  ✓

A result MUST distinguish:

CompleteSuccess
CompleteFailure
PartialFailure

when the application contract requires that distinction.


---

53. Dependency Failure

A service MAY fail because a dependency failed.

Example:

Service A
   |
   v
Service B
   |
   v
Service C
   X

The error SHOULD preserve the dependency relationship where safe.

Possible error:

DependencyUnavailable {
    dependency_identity
    dependency_error
}


---

54. Circuit-Breaker Interaction

ULABI MAY expose circuit-breaker-related errors such as:

CircuitOpen
DependencySuppressed
RecoveryProbe

The actual circuit-breaker algorithm remains implementation-specific.

ULABI defines only the interoperability semantics when such states cross an ABI boundary.


---

55. Error Recovery

Error recovery MUST be governed by the applicable operation contract.

Possible actions include:

Retry
RetryAfterDelay
Reconnect
Renegotiate
RefreshCapability
Reauthenticate
Failover
Reconcile
Rollback
Escalate
Abort

An implementation MUST NOT automatically perform a recovery action merely because the error appears recoverable.

Recovery authorization belongs to the applicable runtime, reliability, security, or distributed profile.


---

56. Unknown Outcome Recovery

For:

UnknownOutcome

the preferred recovery order SHOULD be:

Determine Status
       |
       v
Known?
  /    \
Yes     No
 |       |
Use      Reconcile
Result     |
           v
       Safe Recovery

Blind retry MUST be avoided for non-idempotent operations.


---

57. Error and Graceful Degradation

Distributed errors MAY trigger graceful degradation.

However, the distributed-error specification does not define degradation policy.

Instead:

Distributed Error
       |
       v
Graceful-Degradation Policy
       |
       +-- Alternative feature
       +-- Reduced capability
       +-- Local fallback
       +-- Cached result
       +-- Retry
       +-- Fail

docs/compatibility/graceful-degradation.md remains authoritative for degradation behavior.


---

58. Error and Feature Negotiation

An error MAY indicate that a feature could not be negotiated.

Examples:

RequiredFeatureUnavailable
RequiredProfileUnavailable
IncompatibleFeature
NegotiationRejected

Feature negotiation remains authoritative for the negotiation process.

This document only defines the resulting error semantics.


---

59. Error and Capability Discovery

Capability discovery may establish that a feature is unavailable.

The distinction is:

Capability Discovery
    =
What exists?

Feature Negotiation
    =
What will we use?

Distributed Error
    =
What failed or was rejected?

These concepts MUST remain separate.


---

60. Error and Backward Compatibility

A compatibility failure MAY result in:

IncompatibleVersion
IncompatibleInterface
IncompatibleType
IncompatibleProfile
IncompatibleEncoding

The backward-compatibility specification remains authoritative for determining compatibility.


---

61. Error and Forward Compatibility

A forward-compatibility failure MAY occur when an older implementation encounters a newer feature that cannot safely be processed.

Possible errors include:

UnsupportedExtension
UnknownRequiredFeature
UnknownRequiredField
UnsupportedVersion
UnsupportedVariant

Unknown optional information SHOULD be ignored or preserved where the applicable contract permits.


---

62. Error and Serialization

Serialization failures MUST occur before unsafe execution.

Conceptually:

Receive
  |
Validate Encoding
  |
Validate Schema
  |
Validate Types
  |
Validate Limits
  |
Decode
  |
Execute

A receiver MUST NOT execute an operation merely because a message can be partially decoded.


---

63. Error and Security

Security errors MUST be distinguishable from ordinary application failures where doing so is necessary for security policy.

Security-sensitive failures SHOULD terminate the relevant session or isolate the relevant operation when continued processing would be unsafe.


---

64. Error and Observability

Distributed errors SHOULD integrate with:

Tracing
Diagnostics
Metrics
Audit
Health Monitoring

Where an invocation has an:

InvocationID

the same identifier SHOULD be usable for correlating:

request;

response;

retry;

error;

trace;

diagnostic event.



---

65. Error Correlation

Conceptually:

InvocationID
     |
     +-- Request
     |
     +-- Remote Execution
     |
     +-- Retry
     |
     +-- Error
     |
     +-- Recovery

Correlation identifiers MUST NOT be treated as authorization credentials.


---

66. Error Determinism

For the same protocol violation under the same defined conditions, an implementation SHOULD produce semantically equivalent error classifications.

However, implementations MAY vary in human-readable diagnostic text.

Machine-readable error identity MUST remain stable.


---

67. Human-Readable Messages

Human-readable error messages are supplementary.

They MUST NOT be the sole basis for programmatic error handling.

Applications MUST use standardized error identities and structured metadata where available.


---

68. Localization

Human-readable error messages MAY be localized.

Machine-readable error identity MUST remain language-independent.

For example:

Error ID:
ULABI.DISTRIBUTED.TIMEOUT

may have different localized descriptions without changing semantics.


---

69. Error Ordering

When multiple errors are possible, an implementation SHOULD report the earliest semantically authoritative failure where practical.

However, security policies MAY intentionally suppress more detailed errors.


---

70. Error Precedence

A general precedence model MAY be:

Integrity/Security Violation
        >
Protocol Violation
        >
Compatibility Failure
        >
Authorization Failure
        >
Resource Failure
        >
Execution Failure
        >
Transport Failure

This is not an unconditional global ordering.

The applicable protocol determines which error is authoritative.

An implementation MUST NOT fabricate an error solely to follow this illustrative ordering.


---

71. Error State Machine

A distributed operation MAY follow:

Created
   |
Submitted
   |
Accepted
   |
Executing
   |
 +-------------------+
 |                   |
 v                   v
Completed          Failed
                     |
              +------+------+
              |             |
              v             v
          Recoverable    Permanent
              |
              v
            Retry

Alternative path:

Executing
   |
   X response lost
   |
   v
UnknownOutcome

Another path:

Executing
   |
Cancellation
   |
   v
Cancelled / CancellationUnknown


---

72. Error Immutability

Once an error has been externally committed as the definitive result of an operation, its identity and core semantics MUST NOT be silently changed.

Additional diagnostic information MAY be attached later.


---

73. Error Replay

An implementation MAY replay a previously generated error for duplicate requests.

The replay MUST preserve:

original error identity;

relevant operation state;

invocation correlation;

applicable retry semantics.



---

74. Error Caching

Error responses MAY be cached only when the applicable operation semantics permit caching.

Transient errors SHOULD generally have bounded cache lifetimes.

Authorization and security failures MUST be cached cautiously because authorization state may change.


---

75. Error Expiration

Errors containing temporary state MAY include:

valid_until

or equivalent metadata.

After expiration, the receiver MUST NOT assume that the same failure still exists.


---

76. Error Extensions

ULABI permits extensible error metadata.

Unknown optional fields SHOULD be safely ignored.

Unknown mandatory fields MUST cause a defined compatibility or protocol failure.

Extensions MUST NOT redefine existing standardized fields.


---

77. Application-Specific Errors

Applications MAY define:

application:<namespace>:<error>

or an equivalent namespaced identity.

Application errors MUST NOT collide with standardized ULABI error identifiers.

Application errors SHOULD declare:

retryability;

operation state;

provenance;

severity;

security classification.



---

78. Language Mapping

Different languages MAY map ULABI distributed errors to:

exceptions;

result values;

error objects;

status codes;

tagged unions;

futures/promises;

callbacks.


The mapping MUST preserve ULABI semantics.

For example:

ULABI UnknownOutcome

MUST NOT be mapped to:

ordinary Success

without losing semantics.


---

79. FFI Mapping

The FFI layer MAY translate:

ULABI DistributedError
        |
        v
Language-specific error representation

The reverse mapping MUST also be possible for errors crossing back into ULABI where required.


---

80. No Exception-System Requirement

ULABI MUST NOT require all languages to implement exceptions.

A language using:

Result<T,E>

MUST be equally capable of conforming as a language using exceptions.

The distributed-error contract is semantic, not syntax-dependent.


---

81. Security Requirements

A conforming implementation:

1. MUST validate error metadata before trusting it.


2. MUST NOT treat an error as authorization.


3. MUST NOT expose secrets in error details.


4. MUST protect security-sensitive diagnostic information.


5. MUST preserve integrity of machine-readable error identity.


6. MUST prevent malformed error messages from causing unsafe resource consumption.


7. MUST bound recursive error chains.


8. MUST bound aggregate error size.


9. MUST prevent attacker-controlled errors from causing unbounded retries.


10. MUST NOT interpret unknown errors as successful execution.




---

82. Resource Limits

Implementations MUST impose reasonable limits on:

maximum_error_size
maximum_error_chain_depth
maximum_error_details
maximum_nested_errors
maximum_retry_hint
maximum_diagnostic_payload

Limits MUST be enforced before excessive allocation.


---

83. Failure of Error Handling

If an implementation cannot safely process a received error, it MUST fail closed.

It MAY report:

ULABI.DISTRIBUTED.UNKNOWN_ERROR

or an equivalent safe failure.

It MUST NOT silently convert an unprocessable error into success.


---

84. Conformance Requirements

A conforming Distributed Error implementation MUST:

provide stable error identities;

distinguish known execution outcomes;

support unknown outcomes where applicable;

distinguish transport failure from remote execution failure;

represent retryability;

preserve idempotency implications;

support compatibility failures;

preserve security-sensitive error semantics;

support structured machine-readable errors;

safely process unknown optional error extensions;

enforce resource limits;

prevent uncontrolled retries;

integrate invocation correlation where applicable.



---

85. Mandatory Conformance Cases

The ULABI conformance suite MUST eventually test at least:

DE-001 — Stable Error Identity

Same standardized error retains its identity across compatible versions.

DE-002 — Transport Failure

Transport failure does not automatically become OperationNotExecuted.

DE-003 — Unknown Outcome

Lost response produces an explicit unknown outcome when execution cannot be determined.

DE-004 — Retry Safety

Non-idempotent unknown outcome cannot automatically trigger retry.

DE-005 — Authorization

Authorization failure remains distinguishable from transport failure.

DE-006 — Serialization

Malformed serialized input produces a serialization/protocol error before execution.

DE-007 — Compatibility

Unsupported mandatory feature produces a compatibility/negotiation error.

DE-008 — Error Extension

Unknown optional error metadata is safely handled.

DE-009 — Security

Sensitive diagnostic data is not exposed without authorization.

DE-010 — Error Translation

Translation preserves standardized semantics.

DE-011 — Partial Completion

Partial distributed execution remains distinguishable from complete failure.

DE-012 — Duplicate Request

Duplicate detection does not silently create additional externally observable effects when the contract requires deduplication.

DE-013 — Timeout

Timeout does not automatically prove non-execution.

DE-014 — Cancellation

Cancellation does not falsely imply rollback of completed side effects.

DE-015 — Resource Limit

Oversized error payloads are rejected safely.


---

86. Reference Error Schema

The repository SHOULD eventually define a machine-readable schema corresponding to:

DistributedError {
    error_id
    category
    code
    severity
    provenance
    operation_state
    retryability
    retry_after
    invocation_id
    interface_identity
    interface_version
    function_id
    capability_context
    security_context
    resource_context
    causal_context
    details
}

The schema MUST remain language-neutral.


---

87. Reference State Model

The distributed error subsystem SHOULD implement:

Operation State
        |
        +-- Success
        +-- Failed
        +-- Rejected
        +-- Cancelled
        +-- TimedOut
        +-- Unknown
        +-- PartiallyCompleted

Error classification then supplies the reason.

This prevents the common architectural mistake of using one field to represent both:

what happened

and:

why it happened


---

88. Separation of Outcome and Error

ULABI explicitly separates:

Outcome

from:

Error

For example:

Outcome = Unknown
Error   = Timeout

is valid.

Likewise:

Outcome = Failed
Error   = PermissionDenied

is valid.

And:

Outcome = Rejected
Error   = UnsupportedVersion

is valid.

This separation is REQUIRED for robust distributed semantics.


---

89. Integration Contract

This specification integrates with the repository as follows:

ULABI-DESIGN.md
        |
        v
ULABI-SPEC.md
        |
        +------------------------------+
        |                              |
        v                              v
Core ABI                         Distributed ABI
                                       |
                    +------------------+------------------+
                    |                  |                  |
                    v                  v                  v
               Remote Calls      Serialization      Consensus
                    |
                    +------------------+
                    |
                    v
             Distributed Errors
                    |
       +------------+-------------+
       |            |             |
       v            v             v
 Compatibility   Security   Observability
       |
       v
 Graceful Degradation
       |
       v
 Conformance Suite

The file owns:

> distributed error semantics and propagation.



It does not own:

compatibility determination;

negotiation;

capability discovery;

degradation policy;

consensus algorithms;

authentication;

serialization format;

transport protocol.



---

90. Required Repository Artifacts

The specification above establishes the contract. The following artifacts should subsequently implement and verify it.

A. Normative schema

File:

schemas/distributed-error.schema.json

Purpose:

Machine-readable representation of DistributedError.

Must define:

error identity;

category;

severity;

provenance;

operation state;

retryability;

retry-after;

invocation identity;

interface identity;

structured details;

causal error;

security classification.


This should be independent of any implementation language.


---

B. Error code registry

File:

schemas/distributed-error-codes.json

Purpose:

Canonical registry of standardized ULABI distributed-error identities.

Each entry should contain:

namespace
error_id
category
description
severity
retryability
operation_state
security_classification
introduced_version
deprecated_version

This prevents error-code reuse and semantic collisions.


---

C. Error registry specification

File:

schemas/error-registry.schema.json

Purpose:

Schema validating the error registry itself.

This prevents the registry from becoming an informal list.


---

91. Required Runtime Modules

Because ULABI is implementation-neutral, these are logical modules, not mandated programming-language files.

An implementation MAY use Rust, C++, Go, Java, Python, Sankofa, Zamani, or another language.

Required logical modules:

distributed_errors
distributed_error_code_registry
distributed_error_classifier
distributed_error_builder
distributed_error_validator
distributed_error_serializer
distributed_error_deserializer
distributed_error_translator
distributed_error_propagator
distributed_error_chain
distributed_error_aggregator
distributed_error_retry_policy
distributed_error_outcome_tracker
distributed_error_correlation
distributed_error_security_filter
distributed_error_resource_guard
distributed_error_extension_handler


---

92. Module Responsibilities

distributed_errors

Canonical error data structures.

distributed_error_code_registry

Standard error identity lookup and validation.

distributed_error_classifier

Maps failures into ULABI error categories.

distributed_error_builder

Constructs valid structured errors.

distributed_error_validator

Validates incoming/outgoing error structures.

distributed_error_serializer

Serializes distributed errors according to the selected encoding.

distributed_error_deserializer

Safely decodes errors.

distributed_error_translator

Maps implementation-specific errors into ULABI errors and back where supported.

distributed_error_propagator

Preserves errors across ULABI boundaries.

distributed_error_chain

Maintains bounded causal error chains.

distributed_error_aggregator

Combines multiple distributed failures without losing individual identities.

distributed_error_retry_policy

Evaluates retryability without independently deciding whether retry is authorized.

distributed_error_outcome_tracker

Tracks:

Submitted
Accepted
Executing
Completed
Failed
Cancelled
TimedOut
Rejected
Unknown
PartiallyCompleted

distributed_error_correlation

Connects errors with invocation and tracing identities.

distributed_error_security_filter

Removes or masks unauthorized diagnostic information.

distributed_error_resource_guard

Enforces size/depth/complexity limits.

distributed_error_extension_handler

Handles unknown optional extensions safely.


---

93. Required Test Modules

The following logical test modules should exist:

distributed_error_identity_tests
distributed_error_classification_tests
distributed_error_outcome_tests
distributed_error_unknown_outcome_tests
distributed_error_retry_tests
distributed_error_idempotency_tests
distributed_error_timeout_tests
distributed_error_cancellation_tests
distributed_error_serialization_tests
distributed_error_security_tests
distributed_error_compatibility_tests
distributed_error_translation_tests
distributed_error_propagation_tests
distributed_error_aggregation_tests
distributed_error_extension_tests
distributed_error_resource_limit_tests
distributed_error_fuzz_tests
distributed_error_conformance_tests


---

94. Required Examples

The repository should eventually contain:

examples/distributed-errors/
├── timeout.md
├── unknown-outcome.md
├── authorization-failure.md
├── serialization-failure.md
├── compatibility-failure.md
├── retry-safe-error.md
├── non-idempotent-error.md
├── partial-completion.md
├── cancellation.md
├── duplicate-execution.md
└── error-propagation.md

These are explanatory examples, not normative definitions.


---

95. Required Conformance Files

The distributed-error conformance implementation should eventually contain:

conformance/distributed-errors/
├── error-identity/
├── classification/
├── outcomes/
├── retry/
├── idempotency/
├── timeout/
├── cancellation/
├── serialization/
├── security/
├── compatibility/
├── translation/
├── propagation/
├── aggregation/
├── extensions/
└── resource-limits/


---

96. Integration Dependencies — Already Defined Files

The implementation of this document will integrate with:

docs/abi/exception-model.md
docs/abi/core-abi.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/consensus-boundaries.md

docs/security/security-model.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/capability-security.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

docs/observability/diagnostics.md
docs/observability/tracing.md

docs/standards/conformance.md
docs/standards/test-suite.md

Important: none of those files needs to be edited merely to make this document internally complete. Their existing responsibilities remain authoritative.


---

97. Integration Dependencies — Future Files

The following should eventually be added where the repository architecture requires them:

schemas/distributed-error.schema.json
schemas/distributed-error-codes.json
schemas/error-registry.schema.json

reference/distributed-errors/

tests/distributed-errors/

conformance/distributed-errors/

implementations/<language>/distributed_errors/

The final implementation directory layout should remain language-neutral.

For example:

implementations/
├── c/
├── cpp/
├── rust/
├── go/
├── java/
├── python/
├── swift/
├── kotlin/
├── ada/
├── fortran/
├── zamani/
└── sankofa/

The last two are optional implementations, not dependencies of ULABI.

ULABI itself remains independent of Zamani and Sankofa.


---

98. Implementation Order

To satisfy your requirement that each file be completed independently before integration, the recommended order is:

Phase 1 — Independent normative artifacts

1. docs/distributed/distributed-errors.md ← this document


2. schemas/distributed-error.schema.json


3. schemas/distributed-error-codes.json


4. schemas/error-registry.schema.json



These establish the contract before implementation code exists.

Phase 2 — Independent reference logic

5. reference/distributed-errors/


6. distributed_error_code_registry


7. distributed_error_validator


8. distributed_error_classifier


9. distributed_error_outcome_tracker


10. distributed_error_builder



Phase 3 — Boundary implementation

11. distributed_error_serializer


12. distributed_error_deserializer


13. distributed_error_translator


14. distributed_error_propagator


15. distributed_error_chain


16. distributed_error_aggregator



Phase 4 — Reliability/security

17. distributed_error_retry_policy


18. distributed_error_security_filter


19. distributed_error_resource_guard


20. distributed_error_extension_handler


21. distributed_error_correlation



Phase 5 — Tests

22. identity tests


23. classification tests


24. outcome tests


25. unknown-outcome tests


26. retry/idempotency tests


27. timeout/cancellation tests


28. serialization tests


29. security tests


30. compatibility tests


31. translation/propagation tests


32. aggregation tests


33. fuzz tests


34. conformance tests



Phase 6 — Language implementations

Only after the language-neutral contract is stable:

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

can independently implement the same distributed-error contract.


---

99. Definition of Done

docs/distributed/distributed-errors.md is considered complete when:

distributed outcomes are separated from error causes;

unknown execution outcome is explicitly defined;

partial completion is defined;

transport and execution errors are distinguishable;

retryability is defined;

idempotency interaction is defined;

timeout semantics are defined;

cancellation semantics are defined;

security/error-information boundaries are defined;

compatibility errors are defined;

serialization errors are defined;

consensus-related errors are bounded to their proper scope;

error propagation is defined;

error translation is defined;

error aggregation is defined;

extension behavior is defined;

resource limits are defined;

conformance requirements are defined;

logical implementation modules are identified;

schema integration is specified;

test integration is specified;

reference implementation integration is specified;

no other specification has to be rewritten merely to understand this file.



---

100. Final Architectural Rule

ULABI distributed errors MUST follow this principle:

> An error reports the strongest conclusion that the implementation can establish from available evidence; it must never manufacture certainty that the distributed system does not possess.



Therefore:

Response Lost
     |
     v
Unknown Outcome

is preferable to incorrectly claiming:

Operation Failed

and:

Transport Failed

must not automatically become:

Operation Did Not Execute

This is essential for a genuinely universal distributed ABI.


---

Required files/modules — consolidated list

Normative documentation

docs/distributed/distributed-errors.md

Schemas

schemas/distributed-error.schema.json
schemas/distributed-error-codes.json
schemas/error-registry.schema.json

Reference implementation

reference/distributed-errors/

Logical implementation modules

distributed_errors
distributed_error_code_registry
distributed_error_classifier
distributed_error_builder
distributed_error_validator
distributed_error_serializer
distributed_error_deserializer
distributed_error_translator
distributed_error_propagator
distributed_error_chain
distributed_error_aggregator
distributed_error_retry_policy
distributed_error_outcome_tracker
distributed_error_correlation
distributed_error_security_filter
distributed_error_resource_guard
distributed_error_extension_handler

Test modules

distributed_error_identity_tests
distributed_error_classification_tests
distributed_error_outcome_tests
distributed_error_unknown_outcome_tests
distributed_error_retry_tests
distributed_error_idempotency_tests
distributed_error_timeout_tests
distributed_error_cancellation_tests
distributed_error_serialization_tests
distributed_error_security_tests
distributed_error_compatibility_tests
distributed_error_translation_tests
distributed_error_propagation_tests
distributed_error_aggregation_tests
distributed_error_extension_tests
distributed_error_resource_limit_tests
distributed_error_fuzz_tests
distributed_error_conformance_tests

Conformance tree

conformance/distributed-errors/

Examples

examples/distributed-errors/

Later language adapters

implementations/<language>/distributed_errors/

This keeps the ownership clean: distributed-errors.md defines what a distributed error means; the schema defines its machine representation; the reference implementation defines one implementation; language modules implement it independently; and the conformance suite proves interoperability. It does not turn ULABI into a dependency of Zamani, Sankofa, or any other language.