I read the repository first rather than treating the proposed tree as the current state.

The current repository is Benwellonedge28/ULABI; ULABI-DESIGN.md is the architectural authority and is currently 0.2.0-draft. It already establishes language/runtime/platform neutrality, the Minimal Core + Standard Profiles + Extensible Ecosystem architecture, explicit locality, transport independence, stable interface identity, negotiation/discovery, and the distinction between local and distributed execution.

I also checked the distributed area. docs/distributed/remote-calls.md is currently only a placeholder, so this is the correct file to define remote invocation rather than duplicating an existing specification.  serialization.md is also still a placeholder, so remote-calls must define what invocation metadata is required, while leaving the actual wire encoding to serialization.

The existing distributed-abi.md already establishes the higher-level distributed boundary, including invocation identity, idempotency, retries, delivery, deadlines, cancellation, ownership, resource limits, and security. Therefore the document below deliberately defines the remote invocation protocol contract without recreating the entire Distributed ABI.

Likewise, distributed-errors.md owns detailed distributed error semantics, while consensus-boundaries.md owns distributed agreement/commit semantics.

1. docs/distributed/remote-calls.md

Copy the following entire document into that file:

# ULABI Remote Calls

**Document:** `docs/distributed/remote-calls.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Specification Family:** Distributed Interoperability  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the normative remote-call contract for ULABI.

A remote call is a ULABI invocation in which the caller and callee do not share the execution context required by the applicable local ABI.

A remote call may cross:

- process boundaries;
- container boundaries;
- virtual-machine boundaries;
- host boundaries;
- machine boundaries;
- device boundaries;
- accelerator boundaries;
- trusted-execution boundaries;
- network boundaries;
- administrative boundaries.

The remote-call specification defines how an invocation is represented semantically.

It does not define one mandatory transport protocol.

The fundamental model is:

```text
Caller
   |
   | ULABI Remote Invocation
   v
Invocation Boundary
   |
   +-- Interface Identity
   +-- Function Identity
   +-- Version
   +-- Arguments
   +-- Execution Contract
   +-- Security Context
   +-- Capability Context
   +-- Deadline
   +-- Cancellation
   +-- Idempotency
   +-- Correlation
   |
   v
Remote Callee

The central rule is:

> A remote call MUST preserve the ULABI semantic contract without pretending that remote execution has local execution semantics.




---

2. Scope

This specification defines:

1. remote invocation identity;


2. interface identification;


3. function identification;


4. invocation lifecycle;


5. request semantics;


6. response semantics;


7. invocation correlation;


8. argument boundaries;


9. result boundaries;


10. execution mode;


11. locality declaration;


12. deadlines;


13. cancellation;


14. idempotency;


15. retry interaction;


16. delivery semantics;


17. duplicate detection;


18. request ordering;


19. response ordering;


20. invocation status;


21. acknowledgement semantics;


22. remote references;


23. ownership declarations;


24. capability requirements;


25. security context;


26. resource limits;


27. streaming interaction;


28. version compatibility;


29. feature negotiation interaction;


30. failure interaction;


31. observability;


32. conformance requirements.



This specification does NOT define:

a universal network protocol;

TCP;

QUIC;

HTTP;

gRPC;

a mandatory RPC framework;

a specific serialization format;

a specific authentication algorithm;

a specific authorization system;

a consensus algorithm;

a service-discovery algorithm.


Those mechanisms belong to other ULABI specifications or implementation profiles.


---

3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



Remote calls are part of the Distributed Interoperability profile.

The responsibility boundaries are:

ULABI Core
    |
    +-- Function Contract
    +-- Types
    +-- Errors
    +-- Identity
    +-- Versioning
    |
    v
Distributed ABI
    |
    +-- Remote Calls
    |       |
    |       +-- Invocation
    |       +-- Response
    |       +-- Cancellation
    |       +-- Deadlines
    |
    +-- Serialization
    +-- Service Discovery
    +-- Distributed Errors
    +-- Consensus Boundaries

This specification consumes:

ULABI-DESIGN.md

ULABI-SPEC.md

ULABI-VERSIONING.md

docs/abi/core-abi.md

docs/abi/calling-convention.md

docs/abi/data-types.md

docs/abi/exception-model.md

docs/runtime/runtime-interface.md

docs/security/security-model.md

docs/security/authentication.md

docs/security/authorization.md

docs/security/capability-security.md

docs/distributed/distributed-abi.md

docs/distributed/serialization.md

docs/distributed/service-discovery.md

docs/distributed/distributed-errors.md

docs/distributed/consensus-boundaries.md

docs/compatibility/backwards-compatibility.md

docs/compatibility/forwards-compatibility.md

docs/compatibility/feature-negotiation.md

docs/compatibility/capability-discovery.md

docs/compatibility/graceful-degradation.md

docs/observability/tracing.md

docs/observability/diagnostics.md

docs/standards/conformance.md

docs/standards/test-suite.md


This document defines remote invocation semantics.

It MUST NOT redefine the detailed contracts owned by those specifications.


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

5. Remote Call Definition

A remote call is an invocation of a ULABI interface across a distributed execution boundary.

Conceptually:

Local:

Caller
  |
  v
Function


Remote:

Caller
  |
  v
Remote Invocation
  |
  v
Remote Function

The semantic function contract remains the same.

The execution semantics do not necessarily remain the same.

Remote execution may introduce:

latency;

transport failure;

partial failure;

timeout;

cancellation uncertainty;

unknown execution outcome;

authorization failure;

capability failure;

resource limits;

serialization failure;

service unavailability.


These conditions MUST remain observable through the applicable ULABI contracts.


---

6. Remote Call Contract

Every remotely invocable function MUST have a contract containing, directly or through referenced metadata:

RemoteCallContract {
    interface_identity
    interface_version
    function_identity
    parameter_contract
    result_contract
    error_contract
    locality
    execution_mode
    idempotency
    delivery_semantics
    ordering_semantics
    cancellation_semantics
    deadline_semantics
    capability_requirements
    security_requirements
    resource_limits
}

Not every field must be transmitted on every invocation if it is already established by the interface contract.

The semantic meaning MUST nevertheless be determinable.


---

7. Interface Identity

A remote call MUST identify the interface being invoked.

Conceptually:

InterfaceIdentity {
    namespace
    interface_id
    major_version
}

Interface identity MUST NOT depend solely on:

hostname;

IP address;

process ID;

memory address;

source-language symbol;

compiler-generated name;

temporary endpoint.


Interface identity identifies what contract exists.

It does not identify where the implementation currently runs.


---

8. Function Identity

Each remotely callable function MUST have a stable function identity within its interface.

Conceptually:

FunctionIdentity {
    interface_identity
    function_id
}

A function identity MUST NOT rely solely on:

source-language spelling;

memory address;

compiler-generated symbol;

source-file location;

implementation-specific numeric position.


Renaming a source-language function MUST NOT change its ULABI function identity unless the ULABI contract itself is intentionally changed.


---

9. Endpoint Independence

The remote-call contract MUST distinguish:

Interface
    =
What is callable

Endpoint
    =
Where it is currently reachable

An interface MAY have multiple endpoints.

For example:

Interface X
   |
   +-- Endpoint A
   +-- Endpoint B
   +-- Endpoint C

Moving an implementation between endpoints MUST NOT inherently change interface identity.

Endpoint changes that alter observable semantics MUST be represented through the applicable contract metadata.


---

10. Invocation Identity

Every remote invocation SHOULD have an invocation identity.

Conceptually:

InvocationID {
    namespace
    identifier
}

Invocation identity is used for:

correlation;

tracing;

duplicate detection;

retries;

cancellation;

diagnostics;

auditing;

status queries.


Invocation identity MUST NOT be treated as proof of authorization.


---

11. Correlation Identity

An implementation MAY additionally use:

CorrelationID
ParentInvocationID
TraceID
SpanID

These identities support distributed observability.

Correlation identity MUST NOT change the semantic identity of the operation.


---

12. Invocation Request

A remote invocation conceptually contains:

RemoteInvocation {
    invocation_id
    interface_identity
    interface_version
    function_id
    arguments
    execution_mode
    locality
    deadline
    cancellation_context
    capability_context
    security_context
    idempotency_context
    ordering_context
    resource_limits
}

The exact binary representation belongs to:

docs/distributed/serialization.md

The exact transport belongs to the selected transport profile.


---

13. Request Header and Payload Separation

Implementations SHOULD logically distinguish:

Invocation Metadata
        +
Invocation Arguments

Metadata may include:

invocation ID;

interface identity;

version;

function ID;

deadline;

security context;

capability context;

idempotency key;

tracing context.


Arguments contain the values required by the function contract.

Application arguments MUST NOT be confused with protocol metadata.


---

14. Argument Contract

Arguments MUST conform to the ULABI function contract.

The implementation MUST validate:

argument count;

argument identity;

argument type;

required fields;

value constraints;

ownership semantics;

capability requirements;

resource limits.


Invalid arguments MUST be rejected before unsafe execution.


---

15. Argument Ordering

Where the function contract uses positional arguments, their order MUST be stable.

Where named arguments are supported, names MUST have stable semantic identities.

An implementation MUST NOT silently reinterpret an argument merely because the source language uses a different calling convention.

The local language's calling convention is an implementation detail.

The ULABI function contract is authoritative at the interoperability boundary.


---

16. Result Contract

A remote response conceptually contains:

RemoteResponse {
    invocation_id
    outcome
    result
    error
    execution_metadata
}

The response MUST identify the invocation to which it belongs.

The response MUST distinguish at least:

Success
Failure
Rejected
Cancelled
TimedOut
UnknownOutcome
PartialCompletion

where applicable.


---

17. Response Correlation

A response MUST be correlatable to its invocation.

At minimum:

response.invocation_id
==
request.invocation_id

must hold when invocation identity is used.

An implementation MUST reject or isolate responses that cannot safely be associated with the correct invocation.


---

18. Invocation Lifecycle

A remote invocation MAY progress through:

Created
    |
Submitted
    |
Accepted
    |
Executing
    |
Completed

Alternative terminal states include:

Rejected
Failed
Cancelled
TimedOut
Unknown
PartiallyCompleted

The implementation MUST NOT imply that all states are always observable.


---

19. Request Acceptance

Acceptance means that the remote execution environment has accepted the invocation according to the applicable contract.

Acceptance MUST NOT automatically mean:

execution completed;

state committed;

result produced;

side effects completed;

durable persistence occurred.


Therefore:

Accepted != Completed
Accepted != Committed
Accepted != Applied


---

20. Execution State

If execution state is exposed, the implementation SHOULD distinguish:

NotStarted
Queued
Executing
Completed
Failed
Cancelled
Unknown

The exact lifecycle MAY be profile-specific.


---

21. Acknowledgements

An acknowledgement MUST identify what it acknowledges.

Possible acknowledgement meanings include:

Received
Validated
Accepted
Queued
Executing
Persisted
Committed
Completed

A generic success = true field MUST NOT be interpreted as all of these states.


---

22. Idempotency

Every remotely callable operation whose retry semantics matter MUST declare idempotency.

Possible classifications:

Idempotent
NonIdempotent
ConditionallyIdempotent
IdempotencyKeyRequired
Unknown

A caller MUST NOT assume idempotency when the contract does not establish it.


---

23. Idempotency Keys

For operations requiring safe duplicate suppression, the interface MAY define an idempotency key.

Conceptually:

IdempotencyContext {
    key
    scope
    expiration
}

The key MUST have semantics defined by the interface or profile.

An idempotency key MUST NOT be confused with:

authentication credentials;

authorization tokens;

interface identity;

invocation identity.



---

24. Duplicate Requests

If duplicate detection is supported:

Invocation
     |
     v
Duplicate Detector
   /       \
New       Duplicate
 |           |
Execute    Replay/Reject

Duplicate handling MUST be explicitly defined.

An implementation MUST NOT execute a duplicate request merely because the transport delivered it again.


---

25. Unknown Execution Outcome

A remote call can reach an unknown outcome.

Example:

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

The caller MUST NOT conclude:

response lost == operation failed

The outcome MAY be:

UnknownOutcome

This is particularly important for non-idempotent operations.


---

26. Retry Semantics

Retries MUST follow the remote-call contract.

A retry policy MUST consider:

idempotency;

invocation state;

acknowledgement state;

deadline;

cancellation;

duplicate execution risk;

authorization;

resource limits;

error classification.


A transport failure MUST NOT automatically authorize retry.


---

27. Retry Attempts

Implementations SHOULD expose retry metadata where relevant:

RetryContext {
    attempt
    maximum_attempts
    retry_budget
    reason
}

Retry loops MUST be bounded.

Implementations MUST prevent uncontrolled retry storms.


---

28. Delivery Semantics

A remote interface MUST declare delivery semantics where they affect correctness.

Possible semantics include:

BestEffort
AtMostOnce
AtLeastOnce
Acknowledged
ApplicationConfirmed
ExactlyOnceSemantic

ExactlyOnceSemantic MUST NOT be claimed merely because the transport returned one response.


---

29. Exactly-Once Semantics

An implementation claiming exactly-once semantic behavior MUST provide a mechanism that prevents or neutralizes duplicate externally observable effects.

Possible mechanisms include:

durable idempotency keys;

transaction identifiers;

deduplication records;

atomic commit;

equivalent application-level mechanisms.


The mechanism MUST be specified by the applicable profile.


---

30. Ordering

A remote interface MUST declare ordering semantics where ordering affects correctness.

Possible scopes include:

Unordered
PerInvocation
PerConnection
PerSession
PerStream
PerResource
PerPartition
PerInterface
Global

A caller MUST NOT infer global ordering from message arrival order.


---

31. Request Ordering

If ordered requests are required, the implementation MUST provide an explicit ordering mechanism.

Possible mechanisms include:

sequence numbers;

stream ordering;

session ordering;

protocol-level ordering.


The implementation MUST document what happens when messages arrive out of order.


---

32. Response Ordering

Responses MAY arrive in a different order from requests unless the interface explicitly requires ordered responses.

Therefore:

Request A
Request B

Response B
Response A

MUST be considered valid unless ordering is part of the contract.

Invocation identity MUST allow correct correlation.


---

33. Deadlines

Remote calls SHOULD support deadlines.

Conceptually:

Deadline {
    absolute_time
}

A deadline MUST define what it means.

Possible meanings include:

caller stops waiting;

remote execution should stop;

resource reservation expires;

result becomes invalid.


A deadline MUST NOT automatically imply remote cancellation.


---

34. Timeout

A timeout is distinct from a deadline.

Timeout
=
maximum waiting interval

Deadline
=
absolute time boundary

Implementations MUST NOT silently change one semantic into the other where behavior changes.


---

35. Cancellation

Remote cancellation MUST be explicit.

Conceptually:

CancellationContext {
    invocation_id
    cancellation_id
    reason
}

Cancellation states include:

CancellationRequested
CancellationAccepted
CancellationCompleted
CancellationRejected
CancellationUnknown

A cancellation request MUST NOT guarantee that already-completed effects are undone.


---

36. Cancellation Propagation

Cancellation MAY propagate across nested remote calls.

Example:

Caller
  |
  | Call A
  v
Service A
  |
  | Call B
  v
Service B

If Call A is cancelled, Service A MAY propagate cancellation to Call B if:

the contract permits propagation;

authorization permits propagation;

the operation is cancellable;

doing so is semantically safe.


Cancellation propagation MUST NOT be assumed automatically.


---

37. Security Context

Remote calls MUST operate within the applicable ULABI security model.

Conceptually:

SecurityContext {
    caller_identity
    authentication_state
    authorization_context
    policy_context
}

The security context MUST NOT be treated as ordinary application data.

Authentication is authoritative to:

docs/security/authentication.md

Authorization is authoritative to:

docs/security/authorization.md


---

38. Capability Context

A remote invocation MAY carry capability requirements or capability references.

Conceptually:

CapabilityContext {
    required_capabilities
    delegated_capabilities
    expiration
}

A remote endpoint MUST NOT receive capabilities merely because the caller can reach it.

Capability semantics are governed by:

docs/security/capability-security.md


---

39. Capability Delegation

Capability delegation across a remote boundary MUST be explicit.

A caller MUST NOT automatically delegate all of its capabilities to a callee.

Delegated capabilities MUST be:

scoped;

authorized;

bounded;

revocable where applicable;

auditable where required.



---

40. Ownership

Arguments and results crossing the remote boundary MUST have explicit ownership semantics.

Possible modes include:

Copy
Borrow
Transfer
Shared
Lease
Reference

A process-local pointer MUST NOT be transmitted as a portable remote pointer.


---

41. Remote References

A remote reference MUST use an explicitly defined representation.

Conceptually:

RemoteReference {
    resource_namespace
    resource_id
    version
    authority
}

A raw memory address MUST NOT be used as a portable distributed reference.


---

42. Resource Limits

Remote calls SHOULD support explicit resource limits.

Examples:

maximum_message_size
maximum_argument_size
maximum_result_size
maximum_execution_time
maximum_memory
maximum_cpu
maximum_concurrent_calls
maximum_streams
maximum_bandwidth
maximum_retry_count

Resource limits MUST be enforced before unsafe resource consumption occurs.


---

43. Backpressure

Remote calls involving streaming or large data MUST respect the applicable streaming contract.

The caller MUST NOT assume unlimited remote capacity.

The remote endpoint MUST be able to reject or throttle requests according to its declared resource policy.

Detailed streaming semantics belong to the applicable streaming profile.


---

44. Serialization Boundary

Remote-call metadata and arguments MUST have a defined serialization representation.

This document specifies the semantic fields.

It does NOT define their wire encoding.

The serialization contract belongs to:

docs/distributed/serialization.md

The serializer MUST preserve:

interface identity;

function identity;

version;

argument identity;

argument values;

ownership metadata;

invocation identity;

required protocol metadata.



---

45. Serialization Validation

A remote endpoint MUST validate serialized input before execution.

Validation MUST include, as applicable:

message structure;

schema identity;

type identity;

lengths;

bounds;

required fields;

encoding validity;

resource limits;

version compatibility.


Malformed input MUST NOT be interpreted as valid executable arguments.


---

46. Version Compatibility

A remote invocation MUST identify or inherit a compatible interface version.

Compatibility MUST follow:

ULABI-VERSIONING.md;

docs/compatibility/backwards-compatibility.md;

docs/compatibility/forwards-compatibility.md;

docs/compatibility/feature-negotiation.md.


Remote calls MUST NOT silently downgrade to incompatible semantics.


---

47. Feature Negotiation

Optional remote-call capabilities MAY be negotiated.

Examples:

Cancellation
Streaming
Compression
Encryption
Multiplexing
ExactlyOnceSemantic
StatusQuery
LargeMessageSupport
ZeroCopy

Negotiation MUST occur before relying on an optional feature.

The feature-negotiation specification owns the negotiation protocol.

Remote-calls owns the requirement that negotiated features affect invocation semantics correctly.


---

48. Capability Discovery

A remote endpoint MAY advertise supported remote-call capabilities.

Discovery MUST distinguish:

Supported
Enabled
Authorized
CurrentlyAvailable

These states MUST NOT be conflated.

Capability discovery belongs to:

docs/compatibility/capability-discovery.md


---

49. Graceful Degradation

If an optional remote-call feature is unavailable, the implementation MAY use an explicitly defined fallback.

Example:

Streaming supported
       |
       v
Use streaming

Streaming unsupported
       |
       v
Bounded chunked transfer

A fallback MUST NOT silently change the semantic meaning of the operation.

If no semantically safe fallback exists, the operation MUST fail explicitly.

Graceful-degradation rules belong to:

docs/compatibility/graceful-degradation.md


---

50. Remote Error Handling

Remote-call failures MUST use the distributed error model.

This document does not define a second error taxonomy.

The authoritative specification is:

docs/distributed/distributed-errors.md

The remote-call layer MUST preserve the distinction between:

Request Not Sent
Request Rejected
Remote Execution Failed
Remote Execution Succeeded
Execution Unknown
Partial Completion
Cancellation
Timeout


---

51. Error Translation

A local implementation MAY translate a ULABI distributed error into its native language error model.

However, translation MUST preserve the machine-readable ULABI error identity and semantic outcome.

For example:

ULABI.DISTRIBUTED.UNKNOWN_OUTCOME

MUST NOT become a generic local error such as:

RuntimeError

if doing so destroys the ability to distinguish unknown outcome from definite failure.


---

52. Consensus Interaction

Remote calls MAY invoke operations whose completion depends on distributed consensus.

The remote-call layer MUST NOT define the consensus algorithm.

It MUST preserve the outcome semantics supplied by:

docs/distributed/consensus-boundaries.md

For example:

Accepted
!=
Committed
!=
Applied
!=
Acknowledged


---

53. Commit Semantics

If a remote operation returns a commit-related state, the meaning MUST be explicitly defined.

Possible states include:

Submitted
Accepted
Prepared
Committed
Applied
Acknowledged
Unknown

A response MUST NOT claim Committed merely because the remote process accepted the request.


---

54. Service Discovery Interaction

Remote calls MAY resolve an endpoint through service discovery.

The service-discovery system is responsible for determining:

available endpoints;

endpoint identity;

interface availability;

supported profiles;

protocol versions.


Remote-calls is responsible for invoking the selected endpoint according to the established contract.

An endpoint selected by discovery MUST still undergo required compatibility and security checks.


---

55. Endpoint Selection

An implementation MAY select among multiple endpoints.

Selection MAY consider:

locality;

capability;

version;

security policy;

availability;

resource capacity;

latency;

consistency requirements.


Endpoint selection MUST NOT violate the interface contract.


---

56. Failover

A client MAY fail over to another endpoint when permitted.

Failover MUST consider:

invocation state;

idempotency;

duplicate execution risk;

consistency requirements;

authority;

deadline;

security context.


A failed connection MUST NOT automatically imply that another endpoint may safely execute the same operation.


---

57. Observability

Remote calls SHOULD expose sufficient metadata for distributed diagnostics.

Recommended metadata:

invocation_id
correlation_id
trace_id
span_id
interface_identity
function_id
endpoint_identity
attempt
start_time
completion_time
outcome
error_identity

Observability metadata MUST NOT change application semantics.

Sensitive metadata MUST follow the applicable security and privacy policies.


---

58. Auditability

Security-sensitive remote calls SHOULD support audit correlation.

Audit records SHOULD identify:

caller identity;

interface;

function;

invocation;

authorization decision;

outcome;

relevant security policy;

timestamp.


Audit mechanisms MUST NOT expose secrets merely because they are useful for debugging.


---

59. Resource Exhaustion Protection

Remote-call implementations MUST protect against:

oversized requests;

oversized responses;

excessive concurrency;

recursive invocation;

retry storms;

request amplification;

connection exhaustion;

stream exhaustion;

memory exhaustion.


Limits MUST be applied before unbounded allocation where possible.


---

60. Recursive Calls

Remote implementations MAY invoke other remote services.

Example:

A
 |
 +--> B
       |
       +--> C
             |
             +--> D

Implementations SHOULD enforce bounded:

call depth;

total execution time;

resource consumption;

cancellation propagation;

retry propagation.


A remote call MUST NOT create an uncontrolled recursive invocation chain.


---

61. Reentrancy

A service MAY receive another remote call while processing an existing invocation.

The interface MUST declare reentrancy requirements where reentrancy affects correctness.

An implementation MUST NOT assume that remote callers are always single-threaded.


---

62. Concurrency

Remote calls MAY execute concurrently.

Unless the interface explicitly defines serialization, callers MUST assume independent invocations may execute concurrently.

Concurrency semantics belong to the runtime and distributed profiles.


---

63. Streaming Calls

A remote operation MAY be:

Unary
ClientStreaming
ServerStreaming
BidirectionalStreaming

Streaming semantics are governed by the applicable streaming profile.

Remote-calls MUST preserve:

stream identity;

invocation identity;

cancellation;

ordering;

backpressure;

completion state;

error state.



---

64. Long-Running Operations

A remote function MAY execute longer than the caller's connection lifetime.

Such an interface SHOULD support a durable operation identity and status query.

Conceptually:

Submit Operation
       |
       v
OperationID
       |
       +--> Query Status
       |
       +--> Cancel
       |
       +--> Retrieve Result

The exact long-running-operation mechanism is profile-specific.


---

65. Detached Operations

A remote operation MAY be detached from the original connection.

Detached execution MUST have explicit semantics for:

ownership;

authorization;

lifetime;

cancellation;

result retention;

resource limits;

status;

expiration.


A disconnected client MUST NOT automatically imply that the remote operation has stopped.


---

66. Connection Independence

An invocation MUST NOT necessarily be bound to one transport connection unless the contract requires it.

An implementation MAY reconnect or resume an invocation where the profile supports it.

Resumption MUST preserve invocation identity.


---

67. Transport Independence

Remote calls MUST remain transport-neutral.

Possible transports include:

shared memory;

operating-system IPC;

pipes;

Unix sockets;

TCP;

QUIC;

message queues;

device buses;

WebAssembly host mechanisms;

future transports.


Changing transport MUST NOT require changing the semantic ULABI function contract.

Transport-specific guarantees MUST be represented by the relevant transport profile.


---

68. Local vs Remote Semantic Boundary

ULABI MAY expose one interface that can operate locally or remotely.

However:

Same Interface
      |
      +-- Local Execution
      |
      +-- Remote Execution

does NOT mean:

Same Observable Semantics

Remote execution MUST preserve explicit differences in:

latency;

failure;

availability;

security;

consistency;

cancellation;

resource usage;

ownership.



---

69. Security Requirements

A conforming implementation MUST:

1. authenticate where required;


2. authorize every protected operation;


3. validate remote input;


4. enforce capabilities;


5. enforce resource limits;


6. protect integrity;


7. prevent unauthorized capability escalation;


8. avoid treating network reachability as authorization;


9. protect sensitive invocation metadata;


10. prevent protocol confusion.



Remote-call security MUST use the authoritative ULABI security specifications.


---

70. Replay Protection

Security-sensitive remote calls SHOULD support replay protection.

Possible mechanisms include:

nonce;

invocation identity;

expiration;

sequence number;

timestamp with policy;

cryptographic freshness mechanism.


Replay protection MUST be appropriate to the security profile.

An invocation identity alone MUST NOT automatically provide cryptographic replay protection.


---

71. Confidentiality

Where confidentiality is required, the transport/security profile MUST provide it.

Remote-calls MUST NOT assume that all transports provide confidentiality.

Sensitive argument values MUST be protected according to the applicable security contract.


---

72. Integrity

A remote endpoint MUST detect malformed or integrity-invalid protocol messages where integrity protection is required.

Integrity failures MUST be distinguishable from ordinary application failures.


---

73. Availability

A remote interface MUST NOT claim availability guarantees that are not supported by its deployment and distributed-consistency model.

Failover MAY improve availability but MUST respect:

authority;

consistency;

idempotency;

security;

resource limits.



---

74. Failure Containment

A remote-call failure SHOULD remain isolated to the affected invocation where possible.

An invalid remote request MUST NOT corrupt unrelated invocation state.

A malformed request MUST NOT be allowed to compromise the host process.


---

75. Protocol State Machine

A conforming implementation MAY model remote invocation using:

+-----------+
                  |  Created  |
                  +-----+-----+
                        |
                        v
                  +-----------+
                  | Submitted |
                  +-----+-----+
                        |
             +----------+----------+
             |                     |
             v                     v
        +----------+          +----------+
        | Accepted |          | Rejected |
        +----+-----+          +----------+
             |
             v
        +-----------+
        | Executing |
        +-----+-----+
              |
       +------+------+------+------+ 
       |      |      |      |
       v      v      v      v
   Completed Failed Cancelled Timeout

An additional state is:

UnknownOutcome

which MAY arise after communication failure or other uncertainty.


---

76. Terminal States

Terminal states include:

Completed
Failed
Rejected
Cancelled
TimedOut
UnknownOutcome
PartiallyCompleted

An implementation MUST NOT transition from a terminal state to a different terminal state without an explicitly defined reconciliation mechanism.


---

77. Invocation State Query

Long-running or uncertainty-sensitive operations SHOULD support status queries.

Conceptually:

OperationStatusRequest {
    invocation_id
}

Possible response:

OperationStatus {
    invocation_id
    state
    result
    error
    completion_metadata
}

Status queries MUST themselves obey authentication, authorization, and capability requirements.


---

78. Reconciliation

When an invocation has an unknown outcome, an implementation MAY perform reconciliation.

Possible mechanisms:

status query;

durable operation log;

idempotency record;

transaction lookup;

application-defined reconciliation.


Reconciliation MUST NOT falsely report an outcome without sufficient evidence.


---

79. Partial Completion

Remote calls MAY partially complete.

Where partial completion is possible, the interface SHOULD define:

checkpoint semantics;

partial result semantics;

rollback semantics;

retry semantics;

reconciliation semantics.


If partial completion cannot be safely represented, the interface MUST explicitly state the limitation.


---

80. Rollback

Rollback MUST NOT be assumed.

A remote operation MAY define transactional rollback, compensating actions, or no rollback.

The interface contract MUST distinguish:

Rollback Supported
Rollback Requested
Rollback Completed
Rollback Failed
Rollback Not Supported


---

81. Determinism

A remote call declared deterministic MUST produce equivalent results under equivalent declared inputs and execution conditions.

Remote transport differences MUST NOT silently change deterministic function semantics.

Non-deterministic operations SHOULD declare relevant effects.


---

82. Side Effects

Remote functions with side effects SHOULD declare their effects.

Examples:

WritesResource
WritesMemory
Network
Filesystem
ExternalDevice
Time
Randomness
Process

Effect semantics are inherited from the ULABI function contract.


---

83. Time and Clock Semantics

Remote systems MUST NOT assume that independently operated clocks are identical.

Time-sensitive operations SHOULD use explicit deadline semantics rather than relying on remote wall-clock equality.

Distributed timestamp semantics belong to the applicable distributed profile.


---

84. Maximum Message Size

Every transport/profile SHOULD define a maximum message size.

The remote-call layer MUST expose or otherwise make discoverable any limit that affects interoperability.

Oversized messages MUST be rejected safely.


---

85. Capability Mismatch

If a remote endpoint lacks a required capability:

Required Capability
        |
        v
Available?
    /       \
  Yes        No
   |          |
 Execute    Reject

The invocation MUST NOT silently execute with weaker authority if doing so changes semantics or security.

A safe fallback MAY be used only when explicitly defined.


---

86. Version Mismatch

If the endpoint cannot satisfy the required interface version, the call MUST fail with an appropriate compatibility error or perform an explicitly negotiated compatible alternative.

The implementation MUST NOT silently reinterpret incompatible fields or operations.


---

87. Unknown Fields

Forward-compatible remote protocols MAY encounter fields unknown to an implementation.

Unknown fields MAY be ignored only when the governing schema/compatibility contract permits this.

Unknown required semantics MUST cause explicit rejection.


---

88. Unknown Operations

An unknown function identity MUST NOT be interpreted as another function.

The endpoint SHOULD return a machine-readable unsupported-operation error.


---

89. Unknown Profiles

An endpoint MAY reject an invocation requiring a profile it does not support.

It MUST NOT claim support for a profile merely because it recognizes its name.


---

90. Conformance Requirements

A conforming remote-call implementation MUST demonstrate:

Identity

stable interface identity;

stable function identity;

invocation correlation.


Invocation

valid argument handling;

valid result handling;

explicit lifecycle semantics.


Reliability

timeout handling;

cancellation semantics;

unknown outcome handling;

retry safety.


Compatibility

version validation;

feature negotiation interaction;

unsupported-feature handling.


Security

authentication integration;

authorization integration;

capability enforcement;

replay protection where required;

resource limits.


Distributed Semantics

ordering;

idempotency;

delivery semantics;

endpoint independence;

failure classification.


Observability

invocation correlation;

diagnostics;

tracing integration.



---

91. Conformance Test Categories

The ULABI test suite SHOULD contain tests for:

RemoteCall.Identity
RemoteCall.FunctionIdentity
RemoteCall.Correlation
RemoteCall.ArgumentValidation
RemoteCall.ResultValidation
RemoteCall.Acceptance
RemoteCall.Rejection
RemoteCall.Idempotency
RemoteCall.DuplicateDetection
RemoteCall.Retry
RemoteCall.UnknownOutcome
RemoteCall.Timeout
RemoteCall.Deadline
RemoteCall.Cancellation
RemoteCall.Ordering
RemoteCall.Delivery
RemoteCall.VersionNegotiation
RemoteCall.FeatureNegotiation
RemoteCall.CapabilityValidation
RemoteCall.Authorization
RemoteCall.ResourceLimits
RemoteCall.Serialization
RemoteCall.ErrorPropagation
RemoteCall.Failover
RemoteCall.Reconciliation
RemoteCall.Streaming
RemoteCall.LongRunning
RemoteCall.Security
RemoteCall.Observability


---

92. Reference Implementation Requirements

A reference implementation SHOULD provide at minimum:

RemoteCallClient
RemoteCallServer
InvocationContext
InvocationID
CorrelationContext
RemoteRequest
RemoteResponse
Deadline
CancellationContext
IdempotencyContext
RemoteReference
RetryPolicy
InvocationStatus

The reference implementation MUST remain transport-independent.


---

93. Implementation Independence

A remote-call implementation MAY be written in any language.

Examples include:

C;

C++;

Rust;

Go;

Java;

Kotlin;

Swift;

Python;

Fortran;

Ada;

Zamani;

Sankofa;

future languages.


ULABI MUST NOT require any one language runtime.


---

94. Language Adapter Boundary

The recommended architecture is:

Language A
    |
Language Adapter
    |
ULABI Remote Call API
    |
Transport Adapter
    |
================ Distributed Boundary ================
    |
Transport Adapter
    |
ULABI Remote Call API
    |
Language Adapter
    |
Language B

Language-specific calling conventions terminate at the language adapter.

The ULABI remote contract begins at the ULABI boundary.


---

95. Required Invariants

A conforming implementation MUST preserve these invariants:

Invariant 1

Interface identity uniquely identifies the applicable contract.

Invariant 2

Function identity uniquely identifies the operation within the interface.

Invariant 3

Invocation identity allows safe correlation.

Invariant 4

Unknown execution outcome is never silently converted into definite failure.

Invariant 5

Unknown execution outcome is never silently converted into success.

Invariant 6

Non-idempotent operations are never automatically retried without an explicitly safe retry mechanism.

Invariant 7

Remote memory addresses are never treated as portable references.

Invariant 8

Network reachability never implies authorization.

Invariant 9

Transport failure never automatically proves remote non-execution.

Invariant 10

Accepted never automatically means completed.

Invariant 11

A negotiated feature MUST NOT be assumed before successful negotiation.

Invariant 12

An unsupported feature MUST NOT silently change operation semantics.

Invariant 13

Resource limits MUST remain enforceable.

Invariant 14

Remote errors MUST preserve their ULABI machine-readable identity.

Invariant 15

Remote invocation MUST remain independent of source programming language.


---

96. Failure Model

Remote calls MUST explicitly account for:

InvalidRequest
InvalidArguments
UnsupportedOperation
VersionMismatch
FeatureMismatch
AuthenticationFailure
AuthorizationFailure
CapabilityFailure
SerializationFailure
TransportFailure
Timeout
DeadlineExceeded
Cancellation
ResourceExhaustion
RemoteExecutionFailure
UnknownOutcome
PartialCompletion
ProtocolViolation
IntegrityFailure
UnavailableEndpoint

Detailed distributed error semantics belong to:

docs/distributed/distributed-errors.md


---

97. Recovery Model

The recommended recovery sequence is:

Failure
   |
   v
Classify
   |
   v
Determine execution outcome
   |
   +---- Known Not Executed
   |          |
   |          v
   |       Retry if safe
   |
   +---- Known Executed
   |          |
   |          v
   |       Reconcile
   |
   +---- Unknown
              |
              v
         Status/Reconcile
              |
        +-----+-----+
        |           |
     Resolved     Unknown
        |           |
        v           v
     Continue    Escalate

Recovery MUST respect:

idempotency;

authorization;

resource limits;

deadlines;

consistency;

security policy.



---

98. No Implicit Semantic Downgrade

A remote implementation MUST NOT silently transform:

Secure -> insecure
Authenticated -> unauthenticated
Authorized -> unauthorized
Exactly-once -> at-least-once
Ordered -> unordered
Transactional -> non-transactional
Cancellable -> non-cancellable
Durable -> non-durable
Strong consistency -> weaker consistency

unless the interface explicitly permits that degradation and the caller accepts it.


---

99. Transport Adapter Contract

A transport adapter MUST provide enough functionality for the selected remote-call profile.

Conceptually:

TransportAdapter {
    connect()
    send()
    receive()
    close()
    cancel()
    capabilities()
}

The exact interface belongs to the transport profile.

Transport adapters MUST NOT redefine ULABI function semantics.


---

100. Client Contract

A conforming client implementation SHOULD provide:

RemoteClient {
    discover()
    negotiate()
    invoke()
    cancel()
    status()
    close()
}

The client MUST:

validate the local contract;

negotiate required capabilities;

construct valid invocation metadata;

enforce local resource limits;

correlate responses;

classify outcomes;

preserve distributed errors.



---

101. Server Contract

A conforming server implementation SHOULD provide:

RemoteServer {
    advertise()
    negotiate()
    accept()
    validate()
    authorize()
    execute()
    respond()
    cancel()
    status()
}

The server MUST:

validate invocation identity;

validate interface/function identity;

validate arguments;

authenticate where required;

authorize;

enforce capabilities;

enforce resource limits;

execute only supported operations;

return semantically correct outcomes.



---

102. Middleware

Implementations MAY provide middleware for:

authentication;

authorization;

tracing;

logging;

rate limiting;

retry;

circuit breaking;

metrics;

compression;

encryption;

validation.


Middleware MUST NOT alter the semantic contract without explicit specification.


---

103. Circuit Breaking

Circuit breakers MAY be implemented above the remote-call layer.

A circuit breaker MUST NOT falsely report a remote operation as executed when it was never attempted.

Possible local states:

Closed
Open
HalfOpen

These are implementation states and MUST NOT be confused with remote execution states.


---

104. Rate Limiting

Rate limiting MAY reject a call before execution.

Such rejection MUST be distinguishable from:

Remote execution failed

where the error model requires the distinction.


---

105. Privacy

Remote invocation metadata MAY contain sensitive information.

Implementations SHOULD minimize unnecessary disclosure of:

caller identity;

resource names;

topology;

invocation arguments;

tracing metadata;

endpoint metadata.


Privacy requirements MUST be compatible with observability and audit requirements.


---

106. Deterministic Invocation Metadata

Where deterministic protocol behavior is required, canonical metadata ordering and encoding MUST be supplied by the serialization specification.

Remote-calls MUST NOT define a competing canonical encoding.


---

107. Compatibility With Local Calls

A local implementation MAY expose the same semantic function through:

Local Call
Remote Call

The function identity and type contract MAY remain identical.

However, locality and distributed effects MUST remain explicit.


---

108. Required Documentation Cross-References

Implementations claiming conformance MUST document:

supported locality classes;

supported delivery semantics;

supported ordering semantics;

supported idempotency classes;

cancellation behavior;

deadline behavior;

retry behavior;

unknown-outcome behavior;

serialization profile;

transport profiles;

security profiles;

capability profiles;

resource limits;

version negotiation;

feature negotiation;

observability support.



---

109. Implementation Checklist

A remote-call implementation is complete when:

[ ] Interface identity implemented
[ ] Function identity implemented
[ ] Invocation identity implemented
[ ] Request validation implemented
[ ] Response correlation implemented
[ ] Argument/result contracts implemented
[ ] Deadline semantics implemented
[ ] Timeout semantics implemented
[ ] Cancellation semantics implemented
[ ] Idempotency semantics implemented
[ ] Retry policy implemented
[ ] Duplicate handling implemented
[ ] Unknown outcome implemented
[ ] Ordering semantics implemented
[ ] Delivery semantics implemented
[ ] Serialization integration implemented
[ ] Security integration implemented
[ ] Capability integration implemented
[ ] Resource limits implemented
[ ] Version negotiation implemented
[ ] Feature negotiation implemented
[ ] Error propagation implemented
[ ] Status/reconciliation implemented where required
[ ] Observability implemented
[ ] Conformance tests implemented


---

110. Final Architectural Rule

ULABI remote calls provide a universal invocation contract.

They do not define:

one language;

one runtime;

one transport;

one network;

one operating system;

one consensus system;

one serialization format;

one cloud;

one vendor.


The fundamental rule is:

> ULABI standardizes the semantics of interoperable remote invocation while allowing implementations to choose their own language, runtime, transport, deployment model, and internal execution strategy.



A conforming implementation may therefore be implemented independently by different organizations without sharing source code or internal architecture.


---

111. Conformance Claim

An implementation MUST NOT claim:

> "ULABI Remote Calls compatible"



merely because it can send or receive RPC messages.

A valid conformance claim MUST identify:

ULABI architecture version;

Remote Calls specification version;

supported profiles;

supported transports;

supported serialization formats;

supported security profiles;

supported compatibility features;

conformance-test version;

known limitations.


Example:

ULABI Remote Calls

Core:
    Supported

Remote Calls:
    0.1.0-draft

Serialization:
    Profile X

Security:
    Profile Y

Streaming:
    Supported

Cancellation:
    Supported

Unknown Outcome:
    Supported

Exactly-Once:
    Not Supported

Conformance:
    Test Suite Version X

This prevents vague compatibility claims and preserves the independence of implementations.

## 2. What must be done in the repository

The important architectural point is that **`remote-calls.md` can now be completed independently**. It does not require `serialization.md` or `service-discovery.md` to be rewritten first. Those files are dependencies by contract, not dependencies by textual content.

### Phase A — independent specifications

These should be completed first because each establishes its own contract without requiring the others to be rewritten:

| Order | File | Responsibility |
|---|---|---|
| 1 | `docs/distributed/serialization.md` | Canonical representation, schemas, encoding/decoding, validation |
| 2 | `docs/distributed/service-discovery.md` | Finding interfaces/endpoints and discovering capabilities |
| 3 | `docs/distributed/remote-calls.md` | **Remote invocation contract — the document above** |
| 4 | `docs/distributed/distributed-errors.md` | Distributed error taxonomy and outcome semantics |
| 5 | `docs/distributed/consensus-boundaries.md` | Consensus/authority/commit boundary |
| 6 | `docs/compatibility/feature-negotiation.md` | Negotiation protocol |
| 7 | `docs/compatibility/capability-discovery.md` | Capability discovery |
| 8 | `docs/compatibility/graceful-degradation.md` | Safe fallback/degradation |
| 9 | `docs/compatibility/backwards-compatibility.md` | Old-version compatibility |
| 10 | `docs/compatibility/forwards-compatibility.md` | Newer-version interaction |

This separation is important because the repository currently contains placeholders in parts of the distributed subsystem; for example, `remote-calls.md` and `serialization.md` are placeholders rather than already-established competing specifications.  

### Phase B — ABI/runtime dependencies

Then establish the contracts that implementations will consume:

```text
docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/exception-model.md
docs/abi/memory-model.md

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/concurrency.md
docs/runtime/async-model.md
docs/runtime/resource-management.md

These must define the local semantic foundations. Remote Calls must not redefine them.

Phase C — security dependencies

docs/security/security-model.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/capability-security.md
docs/security/zero-trust.md
docs/security/secure-loading.md

Remote calls consume these contracts.

Phase D — observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md

Remote Calls already reserves correlation/trace metadata so these later documents can integrate without forcing a redesign.

Phase E — standards/conformance

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md
docs/standards/reference-implementations.md

The remote-call specification above already defines its conformance categories, so the standards files can consume those requirements rather than requiring this file to be rewritten.


---

3. Required schemas

The following should eventually exist under schemas/:

schemas/
├── interface-identity.schema.json
├── function-identity.schema.json
├── endpoint.schema.json
├── invocation.schema.json
├── invocation-context.schema.json
├── invocation-response.schema.json
├── invocation-status.schema.json
├── cancellation-context.schema.json
├── deadline.schema.json
├── idempotency-context.schema.json
├── retry-context.schema.json
├── capability-context.schema.json
├── security-context.schema.json
├── remote-reference.schema.json
├── resource-limits.schema.json
├── distributed-error.schema.json
├── feature-negotiation.schema.json
└── conformance-result.schema.json

These schemas should define machine-readable structure only. They should not become a second normative specification.

The Markdown specifications define semantics; schemas define machine-readable representations.


---

4. Required code/modules

ULABI is language-independent, so these are reference architecture module names, not requirements that every implementation use the same programming language or source tree.

Core

ulabi_core/
├── identity
├── types
├── functions
├── contracts
├── versioning
├── errors
└── validation

Remote-call layer

ulabi_remote/
├── client
├── server
├── invocation
├── invocation_context
├── request
├── response
├── status
├── cancellation
├── deadlines
├── idempotency
├── retry
├── ordering
├── delivery
├── reconciliation
├── remote_reference
└── resource_limits

Serialization

ulabi_serialization/
├── encoder
├── decoder
├── schema
├── validator
├── canonical
└── limits

Discovery

ulabi_discovery/
├── interface_registry
├── endpoint_registry
├── capability_registry
├── resolver
└── health

Compatibility

ulabi_compatibility/
├── version
├── feature_negotiation
├── capability_discovery
├── compatibility_check
└── graceful_degradation

Security

ulabi_security/
├── authentication
├── authorization
├── capabilities
├── policy
├── integrity
├── confidentiality
└── replay_protection

Distributed reliability

ulabi_distributed/
├── distributed_errors
├── outcome
├── retry_policy
├── failure_classification
├── reconciliation
├── consensus_boundary
└── authority

Observability

ulabi_observability/
├── tracing
├── correlation
├── diagnostics
├── metrics
└── audit


---

5. Required integration tests

The remote-call implementation should eventually have dedicated tests such as:

tests/
├── remote_calls/
│   ├── identity/
│   ├── invocation/
│   ├── arguments/
│   ├── responses/
│   ├── cancellation/
│   ├── deadlines/
│   ├── idempotency/
│   ├── retries/
│   ├── duplicates/
│   ├── unknown_outcome/
│   ├── ordering/
│   ├── delivery/
│   ├── versioning/
│   ├── negotiation/
│   ├── security/
│   ├── capabilities/
│   ├── resource_limits/
│   ├── serialization/
│   ├── failover/
│   ├── reconciliation/
│   └── observability/

And distributed integration scenarios:

tests/integration/distributed/
├── local_vs_remote/
├── process_to_process/
├── host_to_host/
├── heterogeneous_architectures/
├── version_mismatch/
├── capability_mismatch/
├── transport_failure/
├── response_loss/
├── duplicate_delivery/
├── timeout/
├── cancellation/
├── partial_completion/
├── failover/
├── partition/
└── unknown_outcome/


---

6. Required conformance modules

Eventually:

conformance/
├── core/
├── types/
├── abi/
├── serialization/
├── remote_calls/
├── security/
├── compatibility/
├── distributed/
├── observability/
└── profiles/

conformance/remote_calls/ should directly map to the requirements in sections 90–91 of the document above.


---

7. Integration graph

The clean dependency direction should be:

ULABI-DESIGN.md
                           |
                      ULABI-SPEC.md
                           |
                    ULABI-VERSIONING.md
                           |
              +------------+------------+
              |            |            |
             ABI        Types       Runtime
              |            |            |
              +------------+------------+
                           |
                    Distributed ABI
                           |
        +------------------+------------------+
        |                  |                  |
 Remote Calls        Serialization      Service Discovery
        |                  |                  |
        +------------------+------------------+
                           |
                Distributed Errors
                           |
                Consensus Boundaries
                           |
              Compatibility Profiles
                           |
                    Security Profiles
                           |
                  Observability
                           |
                    Conformance

The crucial rule is:

remote-calls.md must not depend on implementation-specific code.

Instead:

Implementation
      |
      v
ULABI Remote-Call Contract
      |
      +---- ABI
      +---- Types
      +---- Serialization
      +---- Security
      +---- Compatibility
      +---- Errors
      +---- Discovery
      +---- Runtime

That preserves ULABI's independence from Zamani, Sankofa, C, Rust, Go, Python, Java, or any other implementation language.

What is now "owned" by remote-calls.md

To prevent future repetition, this file owns:

invocation lifecycle;

request/response semantics;

invocation identity;

function identity at invocation;

request correlation;

remote arguments/results;

deadlines;

timeout semantics;

cancellation;

idempotency interaction;

retries;

duplicate invocation handling;

delivery semantics;

ordering;

long-running operations;

remote status/reconciliation;

remote-call resource limits;

remote-call security integration;

remote-call compatibility integration;

transport abstraction;

client/server remote-call contracts.


It does not own:

canonical serialization;

service discovery;

general distributed error taxonomy;

consensus algorithms;

authentication algorithms;

authorization policy;

general ABI types;

general memory model;

language-specific calling conventions.


That boundary is what prevents the documentation set from becoming repetitive.

So the file you can now copy/paste is docs/distributed/remote-calls.md above, and it is deliberately written so the later serialization, discovery, error, consensus, compatibility, and conformance documents integrate into its predefined interfaces rather than requiring this file to be rewritten.