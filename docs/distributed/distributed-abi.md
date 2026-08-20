I read the repository first, rather than treating the directory tree in your message as authoritative.

The current repository confirms that ULABI-DESIGN.md is the architectural authority and is currently 0.2.0-draft. It already establishes the key principles we need here: language/runtime/platform neutrality, Minimal Core + Standard Profiles + Extensible Ecosystem, explicit locality, transport independence, interface identity, negotiation/discovery, compatibility, and distributed interoperability.

I also checked the existing compatibility documents and ABI material. backwards-compatibility.md already owns preservation of older contracts, while feature-negotiation.md owns selecting mutually supported options and capability-discovery.md owns determining what is available.

Most importantly, the requested docs/distributed/distributed-abi.md is currently only a placeholder, so this is the correct file to establish the distributed ABI contract rather than modifying another document.

Below is the complete document to copy into:

docs/distributed/distributed-abi.md


---

ULABI Distributed ABI

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: docs/distributed/distributed-abi.md
Specification Family: Distributed Interoperability
Status: Normative Specification
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Normative Authority: ULABI-DESIGN.md, ULABI-SPEC.md, ULABI-VERSIONING.md
License: Apache-2.0


---

1. Purpose

This document defines the distributed ABI contract for ULABI.

The Distributed ABI defines how a ULABI interface may operate across an execution boundary where the caller and callee do not share the same process or execution environment.

The distributed boundary may exist between:

processes;

containers;

virtual machines;

hosts;

machines;

embedded devices;

accelerators;

trusted execution environments;

edge systems;

cloud systems;

geographically separated systems;

heterogeneous architectures.


The Distributed ABI preserves the semantic ULABI contract while allowing the underlying execution location and transport to differ.

The fundamental model is:

Local ABI
    |
    | distributed boundary
    v
Distributed ABI
    |
    +-- interface identity
    +-- type contract
    +-- call contract
    +-- serialization contract
    +-- error contract
    +-- security contract
    +-- locality contract
    +-- resource contract
    +-- consistency contract
    +-- failure contract

The Distributed ABI MUST NOT make a remote operation appear identical to a local operation when the difference affects observable behavior.

In particular:

> Location transparency MUST NOT become semantic transparency.




---

2. Scope

This specification defines:

1. distributed interface identity;


2. distributed invocation;


3. distributed execution boundaries;


4. remote contract establishment;


5. locality semantics;


6. remote call semantics;


7. distributed data boundaries;


8. distributed ownership;


9. distributed lifetime;


10. distributed errors;


11. distributed cancellation;


12. distributed timeouts;


13. distributed resource limits;


14. distributed security boundaries;


15. distributed authorization;


16. distributed capability requirements;


17. distributed consistency declarations;


18. idempotency;


19. retry semantics;


20. duplicate execution semantics;


21. ordering semantics;


22. delivery semantics;


23. availability semantics;


24. failure classification;


25. partial failure;


26. partition handling;


27. distributed observability;


28. distributed contract negotiation;


29. distributed ABI version compatibility;


30. transport independence;


31. conformance requirements.



This specification does not define:

a specific network protocol;

TCP;

QUIC;

HTTP;

RPC;

a specific serialization format;

a specific consensus algorithm;

a specific service-discovery system;

authentication cryptographic primitives;

authorization policy;

application-level business logic.


Those mechanisms belong to their respective specifications or implementation profiles.


---

3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



The Distributed ABI is a profile layered above the ULABI Core.

ULABI Core
    |
    +-- Local execution
    |
    +-- Process execution
    |
    +-- Distributed ABI
             |
             +-- Remote Calls
             +-- Serialization
             +-- Service Discovery
             +-- Distributed Errors
             +-- Consensus Boundaries

The Distributed ABI MUST consume the contracts defined by:

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md

docs/runtime/runtime-interface.md

docs/security/security-model.md
docs/security/capability-security.md
docs/security/authentication.md
docs/security/authorization.md

docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

docs/standards/conformance.md
docs/standards/test-suite.md

This document defines the distributed boundary itself.

It MUST NOT duplicate the detailed rules of those specifications.


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

5. Fundamental Distributed ABI Principle

A distributed call is not semantically identical to an in-process call.

The following properties MAY change across a distributed boundary:

latency;

availability;

failure probability;

authentication;

authorization;

resource consumption;

consistency;

ordering;

delivery;

cancellation;

retry behavior;

ownership;

lifetime;

observability.


Therefore:

Local Call
    !=
Remote Call

at the execution-semantics level.

ULABI may provide a common interface model for both.

It MUST NOT hide meaningful distributed behavior.


---

6. Distributed Execution Boundary

A distributed boundary exists whenever the caller and callee do not share the same execution context required by the applicable local ABI.

Examples include:

Process A
    |
    +---- Process B

Host A
    |
    +---- Host B

Device A
    |
    +---- Device B

Edge
    |
    +---- Cloud

Application
    |
    +---- Accelerator

The implementation MUST explicitly identify whether an operation crosses a distributed boundary.


---

7. Locality Classes

ULABI defines the conceptual locality classes:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A distributed interface MUST declare the applicable locality.

A LocalOnly operation MUST NOT silently become remote.

A ProcessLocal operation MUST NOT silently cross a host boundary.

A RemoteCapable operation MAY operate remotely if the contract permits it.


---

8. Distributed Interface Identity

Every distributed interface MUST have a stable interface identity.

The identity MUST NOT depend solely on:

hostname;

IP address;

process ID;

memory address;

source-language name;

compiler-generated symbol;

temporary service location.


Conceptually:

DistributedInterfaceIdentity {
    namespace
    interface_id
    major_version
}

The interface identity remains stable even when the implementation moves between:

hosts;

processes;

containers;

regions;

devices;

runtime implementations.



---

9. Endpoint Identity

An endpoint identifies a currently reachable implementation of an interface.

Conceptually:

Endpoint {
    interface_identity
    endpoint_identity
    location
    transport
    protocol_version
    profiles
    capabilities
    security_requirements
}

Endpoint identity MUST NOT be confused with interface identity.

Interface
    =
what contract exists

Endpoint
    =
where an implementation is currently reachable

An implementation MAY expose the same interface through multiple endpoints.


---

10. Location Independence

A distributed interface MAY move between endpoints without changing its interface identity.

For example:

ULABI Interface X

       |
       +-- Host A
       |
       +-- Host B
       |
       +-- Host C
       |
       +-- Accelerator

Changing endpoint location MUST NOT automatically imply changing interface semantics.

However, any changed observable property MUST be represented through the appropriate contract metadata.


---

11. Remote Invocation

A remote invocation MUST contain sufficient information to establish the intended operation.

Conceptually:

RemoteInvocation {
    invocation_id
    interface_identity
    interface_version
    function_id
    arguments
    execution_mode
    deadline
    cancellation_context
    capability_context
    security_context
    locality
    idempotency_policy
}

The exact wire representation belongs to the remote-call and serialization specifications.


---

12. Invocation Identity

Every distributed invocation SHOULD have a globally unique invocation identity within its relevant scope.

Conceptually:

InvocationID {
    namespace
    identifier
}

The identifier allows implementations to correlate:

requests;

responses;

retries;

failures;

diagnostics;

traces.


An invocation identifier MUST NOT be treated as proof of authorization.


---

13. Idempotency

Distributed calls MUST declare their idempotency semantics where retries are possible.

An operation MAY be:

Idempotent
NonIdempotent
ConditionallyIdempotent
Unknown

A retry mechanism MUST NOT assume idempotency when the contract does not establish it.

For example:

read()

may be idempotent.

Whereas:

transfer_funds()

may be non-idempotent.

The Distributed ABI MUST preserve that distinction.


---

14. Retry Semantics

Retries MUST be controlled by the operation contract.

A client MUST NOT automatically retry an operation merely because the transport failed.

Retry policy MUST consider:

idempotency;

invocation state;

server acknowledgment;

deadline;

authorization;

resource constraints;

duplicate execution risk.


The implementation MUST distinguish:

Request definitely not executed

from:

Request execution unknown

and:

Request definitely executed

when such knowledge is available.


---

15. At-Least-Once, At-Most-Once, and Exactly-Once Claims

ULABI MUST distinguish delivery/execution guarantees.

Possible semantics include:

AtMostOnce
AtLeastOnce
BestEffort
ExactlyOnceSemantic
Unknown

ExactlyOnceSemantic MUST NOT be claimed merely because a transport returns one response.

A conforming implementation claiming exactly-once semantics MUST provide the required mechanism for preventing or neutralizing duplicate externally observable effects.


---

16. Duplicate Detection

When an operation supports duplicate detection, the implementation SHOULD use an invocation identity or equivalent mechanism.

Conceptually:

InvocationID
      |
      v
Duplicate Detection
      |
   +--+--+
   |     |
 New   Duplicate
   |     |
 Execute Return/Replay

Duplicate handling MUST be explicitly defined.


---

17. Ordering

Distributed interfaces MUST declare ordering semantics where ordering affects correctness.

Possible models include:

Unordered
PerInvocation
PerStream
PerSession
PerInterface
GloballyOrdered

A system MUST NOT infer global ordering from network arrival order.


---

18. Delivery Semantics

Distributed operations MAY use:

best effort;

at most once;

at least once;

acknowledged delivery;

application-confirmed delivery.


The selected semantics MUST be explicit.

Transport reliability alone MUST NOT automatically imply application-level delivery semantics.


---

19. Deadlines

Distributed operations SHOULD support deadlines.

Conceptually:

Deadline {
    absolute_time
}

A deadline MUST NOT be interpreted as a guarantee that the remote operation stops executing at that exact time.

Instead, deadline semantics MUST specify whether it means:

caller no longer waits;

remote execution should be cancelled;

resource reservation expires;

response becomes invalid.



---

20. Timeouts

Timeouts and deadlines MUST be distinguished.

Timeout
    =
maximum waiting interval

Deadline
    =
absolute execution/wait boundary

Implementations MUST NOT silently convert one into the other when doing so changes semantics.


---

21. Cancellation

Distributed cancellation MUST be explicit.

A cancellation request SHOULD contain:

CancellationContext {
    invocation_id
    cancellation_id
    reason
}

Cancellation MUST distinguish:

CancellationRequested
CancellationAccepted
CancellationCompleted
CancellationRejected
CancellationUnknown

A cancellation request MUST NOT guarantee that already-completed side effects are undone.


---

22. Partial Completion

A distributed operation MAY partially complete before communication fails.

The implementation MUST provide an appropriate outcome such as:

Success
Failure
Cancelled
TimedOut
Rejected
UnknownOutcome
PartialCompletion

An UnknownOutcome state is essential where the client cannot determine whether the remote operation executed.


---

23. Distributed Ownership

Ownership crossing a distributed boundary MUST be explicit.

Possible semantics include:

Borrow
Copy
Transfer
Shared
Move
Lease
Reference

A pointer from one process or host MUST NOT be treated as a valid remote pointer unless an explicitly defined shared-memory mechanism establishes that semantic.


---

24. Distributed References

A local memory address MUST NOT be transmitted as a portable distributed reference.

For example:

0x7FFF1234

MUST NOT be assumed meaningful on another process or host.

Distributed references MUST use an explicitly defined representation.

Conceptually:

RemoteReference {
    resource_namespace
    resource_id
    version
    authority
}


---

25. Resource Leases

Distributed resources MAY use leases.

A lease SHOULD define:

resource identity;

owner;

expiration;

renewal rules;

revocation;

failure behavior.


An expired lease MUST NOT be treated as permanently valid.


---

26. Serialization Boundary

Values crossing a distributed boundary MUST use a defined serialization contract.

The Distributed ABI relies on:

docs/distributed/serialization.md

for:

canonical encoding;

decoding;

schema identity;

versioning;

validation;

limits.


Distributed ABI MUST NOT define a second serialization system.


---

27. Zero-Copy Restrictions

Zero-copy behavior across a distributed boundary MUST NOT be assumed.

Zero-copy MAY exist when a defined shared-memory or hardware mechanism provides the necessary semantics.

The implementation MUST establish:

ownership;

lifetime;

access rights;

synchronization;

memory visibility;

security.


Otherwise values MUST use an appropriate transfer representation.


---

28. Distributed Type Compatibility

A distributed type is compatible when:

1. its semantic identity is compatible;


2. its serialized representation is compatible;


3. its ownership semantics are compatible;


4. its validation requirements are compatible;


5. its security requirements are compatible.



A source-language type name alone is insufficient evidence of compatibility.


---

29. Resource Limits

Every distributed implementation SHOULD support explicit limits.

Examples include:

maximum_message_size
maximum_argument_size
maximum_result_size
maximum_stream_size
maximum_execution_time
maximum_concurrent_calls
maximum_memory
maximum_bandwidth
maximum_retry_count

Limits MUST be enforced before unsafe resource consumption occurs.


---

30. Backpressure

Streaming and large-data distributed operations MUST support backpressure where required.

A sender MUST NOT assume unlimited receiver capacity.

The applicable streaming profile defines the detailed mechanism.

Distributed ABI defines the requirement that capacity and flow-control semantics remain explicit.


---

31. Security Boundary

A distributed boundary is a security boundary unless explicitly declared otherwise.

A remote caller MUST NOT receive capabilities merely because it can reach an endpoint.

Security MUST be evaluated independently from:

network reachability;

interface discovery;

feature support;

protocol compatibility.



---

32. Authentication

Authentication MUST be handled through the applicable ULABI security specification or security profile.

Distributed ABI MUST record the security context relevant to the invocation.

Authentication MUST NOT be inferred from:

IP address;

hostname;

interface identity;

transport connection alone.



---

33. Authorization

Authorization MUST be evaluated independently of capability discovery.

The following are distinct:

Supported
    !=
Authorized

and:

Reachable
    !=
Authorized

A discovered remote capability MUST NOT automatically grant permission to use it.


---

34. Capability Context

A distributed invocation MAY carry an explicitly scoped capability context.

The capability MUST define:

authority;

scope;

permitted operation;

resource;

expiration;

delegation rules.


Capability delegation MUST NOT exceed the authority of the delegating party.


---

35. Distributed Effects

Distributed effects MUST remain explicit.

Examples include:

Network
RemoteProcess
ExternalDevice
RemoteFilesystem
DistributedState
RemoteResource

A local-looking operation MUST NOT silently acquire distributed effects that materially change its security or execution semantics.


---

36. Consistency Semantics

Distributed interfaces that access shared state MUST declare relevant consistency semantics.

Possible profiles include:

LocalConsistency
SessionConsistency
EventualConsistency
CausalConsistency
StrongConsistency
ApplicationDefinedConsistency

The Distributed ABI MUST NOT assume strong consistency by default.

The selected consistency model MUST be part of the applicable interface contract.


---

37. Consensus Boundaries

Consensus MUST NOT be implicitly introduced into every distributed ABI operation.

A distributed interface MAY declare that an operation requires consensus.

For example:

Read
    -> no consensus required

Write
    -> application-defined consistency

Commit
    -> consensus required

The detailed consensus rules belong to:

docs/distributed/consensus-boundaries.md

The Distributed ABI only carries the declared requirement.


---

38. Partial Failure

Distributed systems MUST assume partial failure.

Possible failure locations include:

Caller
Callee
Network
Transport
Serialization
Authentication
Authorization
Resource
Storage
Service Discovery
Consensus
Hardware

A failure in one component MUST NOT automatically be interpreted as total system failure.


---

39. Failure Classification

Distributed errors SHOULD distinguish at least:

TransportFailure
EndpointUnavailable
AuthenticationFailure
AuthorizationFailure
VersionMismatch
CapabilityMismatch
SerializationFailure
ValidationFailure
ResourceExhaustion
Timeout
Cancellation
RemoteExecutionFailure
RemoteExecutionUnknown
ConsistencyFailure
ProtocolViolation
SecurityViolation

Detailed error identities belong to:

docs/distributed/distributed-errors.md


---

40. Unknown Outcome

UnknownOutcome is a first-class distributed state.

It occurs when:

Client sends request
        |
        v
Communication fails
        |
        v
Did server execute?
      Unknown

The implementation MUST NOT automatically classify this as failure or success unless sufficient evidence exists.

This distinction is essential for non-idempotent operations.


---

41. Failure Containment

A failed remote component SHOULD be isolated from unrelated local components.

The implementation SHOULD support:

Failure
   |
   v
Detect
   |
   v
Contain
   |
   v
Classify
   |
   +---- Recover
   |
   +---- Retry
   |
   +---- Degrade
   |
   +---- Escalate

Recovery behavior is governed by the reliability specifications.


---

42. Graceful Degradation

Distributed functionality MAY degrade when optional remote capabilities are unavailable.

For example:

Requested:
Core + Streaming + GPU

Available:
Core + Streaming

Selected:
Core + Streaming

The Distributed ABI MUST NOT silently degrade a mandatory semantic requirement.

Graceful degradation follows:

docs/compatibility/graceful-degradation.md.


---

43. Capability Discovery

Distributed endpoints MAY advertise their supported capabilities.

Capability discovery MUST answer:

> What can this endpoint provide?



It MUST NOT be treated as authorization.

Capability discovery follows:

docs/compatibility/capability-discovery.md.


---

44. Feature Negotiation

After capabilities are known, participants MAY negotiate:

interface versions;

profiles;

encodings;

transports;

execution modes;

optional features;

security mechanisms;

resource configurations.


Feature negotiation follows:

docs/compatibility/feature-negotiation.md.

The selected contract MUST be established before negotiated behavior is used.


---

45. Version Compatibility

Distributed participants MUST explicitly establish compatible versions.

Version compatibility follows:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

A remote endpoint MUST NOT claim compatibility merely because it shares an interface identifier.


---

46. Transport Independence

The Distributed ABI MUST remain transport-independent.

Possible transports include:

TCP;

QUIC;

Unix sockets;

operating-system IPC;

message queues;

shared memory;

device buses;

WebAssembly host calls;

future transports.


Changing the transport MUST NOT require changing the semantic interface contract.

Transport-specific behavior belongs to the transport profile.


---

47. Transport Properties

A transport adapter SHOULD expose properties such as:

reliable
ordered
encrypted
authenticated
multiplexed
streaming
datagram
connection_oriented
message_oriented

The Distributed ABI MUST NOT assume these properties unless declared.


---

48. Service Discovery

Distributed endpoints MAY be discovered dynamically.

Service discovery MUST return information sufficient to establish:

interface identity;

endpoint identity;

supported versions;

profiles;

capabilities;

security requirements;

transport information.


Detailed discovery behavior belongs to:

docs/distributed/service-discovery.md.


---

49. Endpoint Changes

An endpoint MAY disappear or move during the lifetime of an interface.

The interface identity MAY remain stable.

An implementation MUST distinguish:

Interface unavailable

from:

Interface identity invalid

and:

Endpoint temporarily unavailable


---

50. Distributed Sessions

A distributed interface MAY establish a session.

A session SHOULD have:

session_id
interface_identity
participant_identity
security_context
negotiated_contract
expiration

Session state MUST have explicitly defined lifetime semantics.


---

51. Stateless Operation

Where possible, distributed operations SHOULD support stateless invocation.

Stateless interfaces improve:

failover;

scaling;

recovery;

endpoint migration;

load balancing.


Stateful interfaces MAY be used when required.

Their state semantics MUST be explicit.


---

52. Session Migration

If a stateful session can migrate between endpoints, the implementation MUST preserve all state required by the contract.

A session MUST NOT silently migrate to an endpoint lacking:

required interface version;

required security properties;

required capabilities;

required profile support.



---

53. Distributed Concurrency

Concurrent remote calls MUST follow the concurrency semantics declared by the interface.

The implementation MUST NOT assume:

request order = execution order

unless explicitly guaranteed.

Race-sensitive operations MUST define appropriate synchronization or consistency semantics.


---

54. Distributed Atomicity

An operation MUST NOT claim atomicity across multiple distributed resources unless the applicable contract provides a defined mechanism.

For example:

A
|
+-- update
|
B
|
+-- update

does not automatically constitute one atomic transaction.

Distributed transactions, if supported, MUST be defined by a separate profile.


---

55. Transactions

A transaction profile MAY define:

transaction identity;

participant registration;

prepare;

commit;

abort;

timeout;

recovery;

durability.


The base Distributed ABI MUST remain usable without distributed transactions.


---

56. Resource Accounting

Distributed resource usage SHOULD be attributable to the appropriate:

invocation;

session;

capability;

participant;

tenant;

interface.


This supports:

quotas;

billing;

security;

diagnostics;

fault isolation.



---

57. Determinism

Where an interface declares deterministic behavior, crossing a distributed boundary MUST NOT silently remove that guarantee.

Sources of nondeterminism MUST be explicitly declared.

Examples include:

remote time;

random number generation;

concurrent execution;

external state;

network timing.



---

58. Observability

Distributed implementations SHOULD propagate sufficient metadata to correlate an operation across boundaries.

This MAY include:

trace_id
span_id
invocation_id
interface_id
endpoint_id
session_id

Observability metadata MUST NOT automatically grant authorization.

Sensitive metadata MUST follow the applicable security and privacy requirements.


---

59. Distributed Diagnostics

A distributed failure SHOULD provide machine-readable diagnostic information.

Diagnostics SHOULD distinguish:

where failure occurred

from:

why failure occurred

and:

whether retry is safe


---

60. Distributed Contract Commitment

A negotiated distributed contract SHOULD be committed atomically.

The implementation MUST NOT expose a partially established contract as active.

Conceptually:

Discovery
   |
Negotiation
   |
Validation
   |
Security Verification
   |
Atomic Commit
   |
Active Distributed Contract


---

61. Contract Invariants

A conforming implementation MUST preserve these invariants:

Invariant 1 — Interface identity

An interface identifier MUST NOT be reused for unrelated semantics.

Invariant 2 — Explicit locality

Remote execution MUST NOT be hidden when it changes observable semantics.

Invariant 3 — Explicit ownership

Distributed ownership MUST be defined.

Invariant 4 — Explicit failure

Unknown remote outcomes MUST remain distinguishable from confirmed failures.

Invariant 5 — Explicit security

Reachability MUST NOT imply authorization.

Invariant 6 — Explicit compatibility

Version compatibility MUST be established before incompatible functionality is used.

Invariant 7 — Explicit negotiation

Optional features MUST NOT be assumed to be active.

Invariant 8 — Transport independence

Changing transport MUST NOT redefine interface semantics.

Invariant 9 — No unsafe retries

Non-idempotent operations MUST NOT be blindly retried.

Invariant 10 — No implicit consensus

Distributed operations MUST NOT acquire consensus semantics without an explicit contract.


---

62. Security Requirements

A conforming Distributed ABI implementation:

MUST authenticate where required by the interface security policy.

MUST authorize every protected operation.

MUST validate remote input.

MUST enforce resource limits.

MUST prevent capability escalation.

MUST prevent unauthorized endpoint substitution.

MUST protect integrity of negotiated contracts.

MUST prevent downgrade attacks where applicable.

MUST distinguish discovery from authorization.

MUST NOT treat network location as sufficient identity.

MUST NOT expose raw local memory addresses as remote references.


---

63. Failure Modes

The implementation MUST be prepared for:

EndpointUnavailable
TransportFailure
NetworkPartition
Timeout
RemoteCrash
LocalCrash
SerializationFailure
VersionMismatch
CapabilityMismatch
AuthenticationFailure
AuthorizationFailure
ResourceExhaustion
DuplicateInvocation
UnknownOutcome
StaleEndpoint
SessionExpiration
ConsistencyViolation
ProtocolViolation
SecurityViolation


---

64. Recovery Behavior

Recovery behavior MUST depend on the operation's declared semantics.

The implementation MAY:

retry;

reconnect;

fail over;

resume;

re-negotiate;

degrade;

rollback;

escalate.


It MUST NOT perform unsafe recovery merely because recovery is technically possible.

For example:

Non-idempotent UnknownOutcome
        |
        +-- DO NOT blindly retry
        |
        +-- determine outcome
        |
        +-- reconcile
        |
        +-- escalate if necessary


---

65. Distributed ABI State Machine

A conforming implementation SHOULD model the distributed lifecycle explicitly:

Unresolved
    |
    v
Discovered
    |
    v
Authenticated
    |
    v
Authorized
    |
    v
Compatible
    |
    v
Negotiated
    |
    v
Committed
    |
    v
Active
    |
    +--------+
    |        |
    v        v
Failed    Expired
    |
    v
Recovering
    |
    +------+
    |      |
    v      v
Active  Terminated

A transition MUST NOT skip required security or compatibility validation.


---

66. Reference Distributed Contract

A conceptual distributed interface may be represented as:

Interface: example.compute
Version: 1.x

Function:
    calculate

Arguments:
    Input

Return:
    Result

Locality:
    RemoteCapable

Execution:
    Synchronous

Idempotency:
    Idempotent

Cancellation:
    Supported

Deadline:
    Supported

Required Profiles:
    Core
    Security
    Distributed

Optional Profiles:
    Streaming
    Accelerator

Consistency:
    ApplicationDefined

This is a contract.

It is not a programming-language declaration.


---

67. Implementation Independence

The Distributed ABI MUST NOT require a particular:

programming language;

compiler;

runtime;

operating system;

processor;

network stack;

serialization library;

RPC framework;

cloud provider.


Implementations MAY be written in:

C;

C++;

Rust;

Go;

Java;

Python;

Swift;

Kotlin;

Ada;

Fortran;

Zamani;

Sankofa;

or any future language.


ULABI remains independent of all of them.


---

68. Conformance Requirements

A Distributed ABI implementation is conforming only if it demonstrates:

stable distributed interface identity;

explicit locality;

explicit remote invocation semantics;

version compatibility;

capability-aware operation;

feature negotiation where applicable;

secure authorization;

explicit failure semantics;

retry safety;

timeout/deadline handling;

cancellation semantics;

distributed ownership semantics;

serialization compatibility;

resource limits;

transport independence;

distributed error classification.



---

69. Required Conformance Tests

The ULABI test suite SHOULD contain tests for:

Interface

interface identity stability;

endpoint identity separation;

interface migration.


Invocation

valid remote invocation;

invalid function ID;

invalid argument;

duplicate invocation.


Compatibility

compatible version;

incompatible major version;

compatible minor version;

capability mismatch;

profile mismatch.


Security

unauthorized caller;

invalid capability;

expired capability;

endpoint substitution;

downgrade attempt.


Reliability

timeout;

cancellation;

remote crash;

network partition;

unknown outcome;

safe retry;

unsafe retry rejection.


Serialization

canonical encoding;

malformed input;

oversized message;

unknown optional field;

incompatible type.


Ownership

transfer;

borrow;

release;

expired lease;

invalid remote reference.


Consistency

declared consistency model;

ordering;

concurrent invocation;

atomicity claims.


Transport

transport substitution;

connection loss;

reconnection;

transport capability mismatch.



---

70. Reference Implementation Requirements

A reference implementation SHOULD provide:

DistributedInterface
Endpoint
RemoteInvocation
InvocationTracker
DeadlineManager
CancellationManager
RetryPolicy
IdempotencyManager
RemoteReferenceManager
CapabilityContext
SecurityContext
ContractNegotiator
DistributedErrorMapper
ResourceLimiter
ConsistencyContext
TransportAdapter
SessionManager
ObservabilityContext

The reference implementation MUST NOT become the specification itself.

The specification remains authoritative.


---

71. Integration Contract

This document integrates with the repository as follows:

ULABI-DESIGN.md
        |
        v
ULABI-SPEC.md
        |
        +----------------------+
        |                      |
        v                      v
Core ABI                Versioning
        |                      |
        +----------+-----------+
                   |
                   v
          Distributed ABI
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
 Remote Calls  Serialization  Discovery
       |           |           |
       +-----------+-----------+
                   |
                   v
             Negotiation
                   |
                   v
              Security
                   |
                   v
              Execution

The ownership boundaries are:

Concern	Authoritative file

Overall architecture	ULABI-DESIGN.md
Normative Core	ULABI-SPEC.md
Version policy	ULABI-VERSIONING.md
Core ABI	docs/abi/core-abi.md
Types	docs/abi/data-types.md
Calling convention	docs/abi/calling-convention.md
Memory	docs/abi/memory-model.md
Compatibility	docs/compatibility/*
Remote invocation	docs/distributed/remote-calls.md
Serialization	docs/distributed/serialization.md
Discovery	docs/distributed/service-discovery.md
Distributed errors	docs/distributed/distributed-errors.md
Consensus boundary	docs/distributed/consensus-boundaries.md
Distributed ABI boundary	this document
Security	docs/security/*
Runtime	docs/runtime/*
Conformance	docs/standards/*


No later document needs to rewrite the fundamental distributed ABI rules established here.

Later documents refine their own domains and reference this contract.


---

72. Explicit Non-Duplication Rule

The following rules are intentionally not redefined here:

Serialization

Defined by:

docs/distributed/serialization.md

Remote-call protocol

Defined by:

docs/distributed/remote-calls.md

Service discovery

Defined by:

docs/distributed/service-discovery.md

Distributed error taxonomy

Defined by:

docs/distributed/distributed-errors.md

Consensus

Defined by:

docs/distributed/consensus-boundaries.md

Backward compatibility

Defined by:

docs/compatibility/backwards-compatibility.md

Forward compatibility

Defined by:

docs/compatibility/forwards-compatibility.md

Feature negotiation

Defined by:

docs/compatibility/feature-negotiation.md

Capability discovery

Defined by:

docs/compatibility/capability-discovery.md

Graceful degradation

Defined by:

docs/compatibility/graceful-degradation.md

This document defines how those components fit together at the distributed ABI boundary.


---

73. Summary

ULABI Distributed ABI provides a universal contract for interoperability across execution boundaries.

It establishes:

Stable Interface
       |
       v
Explicit Locality
       |
       v
Discovery
       |
       v
Compatibility
       |
       v
Negotiation
       |
       v
Security
       |
       v
Remote Invocation
       |
       v
Explicit Failure
       |
       v
Recovery / Degradation / Escalation

The central rule is:

> ULABI makes distributed interoperability possible without pretending that distributed execution is equivalent to local execution.



A conforming implementation must preserve:

semantic correctness;

interface identity;

compatibility;

security;

ownership;

failure visibility;

transport independence;

implementation independence.



---

What should be implemented under which files

I would not start writing distributed code immediately. The repository is still specification-first, and several foundational documents are placeholders. The safest implementation order is therefore:

Phase 1 — Independent normative foundations

These files establish contracts that other components consume.

File	Responsibility	Depends on

ULABI-VERSIONING.md	Version model	ULABI-DESIGN.md
ULABI-GOVERNANCE.md	Governance	ULABI-DESIGN.md
docs/architecture/overview.md	Architecture decomposition	master design
docs/architecture/design-principles.md	Architectural invariants	master design
docs/architecture/universal-model.md	Universal abstraction	master design
docs/architecture/scalability.md	Scaling model	architecture


These should be completed before implementation modules.


---

Phase 2 — Core ABI contracts

docs/abi/core-abi.md
docs/abi/data-types.md
docs/abi/calling-convention.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/return-values.md
docs/abi/exception-model.md

These define the actual ABI foundation.


---

Phase 3 — Type system

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/enums.md
docs/type-system/generics.md

These should define machine-readable type identity and compatibility before serialization implementations are written.


---

Phase 4 — Interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

These define how languages attach to ULABI.


---

Phase 5 — Distributed contracts

Now implement the distributed documentation family:

docs/distributed/distributed-abi.md              # DONE WITH THIS RESPONSE
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

The integration relationship is:

distributed-abi.md
       |
       +--> remote-calls.md
       |
       +--> serialization.md
       |
       +--> service-discovery.md
       |
       +--> distributed-errors.md
       |
       +--> consensus-boundaries.md

distributed-abi.md should remain the boundary contract, while the other five define mechanisms.


---

Phase 6 — Compatibility

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

These should not redefine distributed behavior.

They answer:

Can we interoperate?
What do we support?
What can we negotiate?
What can we discover?
What can we safely omit?

The current repository already reflects this separation.


---

Phase 7 — Runtime

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md


---

Phase 8 — Memory

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md


---

Phase 9 — Security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md


---

Phase 10 — Reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md
docs/reliability/health-monitoring.md

The distributed ABI should consume these rather than implement its own recovery system.


---

Phase 11 — Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md


---

Phase 12 — Standards

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md
docs/standards/reference-implementations.md

These are what eventually turn the specifications into something independently verifiable.


---

Required code/module architecture

The specification should eventually map to language-neutral implementation modules.

I recommend the following canonical module names.

Core

ulabi-core
ulabi-identity
ulabi-version
ulabi-contract
ulabi-type
ulabi-value
ulabi-error
ulabi-effect
ulabi-capability
ulabi-validation

ABI

ulabi-abi
ulabi-calling-convention
ulabi-memory-boundary
ulabi-stack
ulabi-register
ulabi-return
ulabi-exception

Type system

ulabi-types
ulabi-type-descriptor
ulabi-type-registry
ulabi-type-compatibility
ulabi-schema
ulabi-schema-registry

Interoperability

ulabi-interop
ulabi-ffi
ulabi-object-model
ulabi-symbol
ulabi-name-resolution
ulabi-language-adapter

Compatibility

ulabi-compatibility
ulabi-backward-compatibility
ulabi-forward-compatibility
ulabi-feature-negotiation
ulabi-capability-discovery
ulabi-graceful-degradation

Runtime

ulabi-runtime
ulabi-process
ulabi-thread
ulabi-async
ulabi-concurrency
ulabi-resource

Security

ulabi-security
ulabi-authentication
ulabi-authorization
ulabi-capability-security
ulabi-sandbox
ulabi-crypto
ulabi-secure-loader

Distributed

ulabi-distributed
ulabi-distributed-abi
ulabi-remote-call
ulabi-serialization
ulabi-service-discovery
ulabi-distributed-error
ulabi-consensus-boundary
ulabi-endpoint
ulabi-invocation
ulabi-session
ulabi-remote-reference
ulabi-deadline
ulabi-cancellation
ulabi-retry
ulabi-idempotency
ulabi-consistency
ulabi-distributed-resource

Reliability

ulabi-fault
ulabi-fault-detection
ulabi-fault-isolation
ulabi-recovery
ulabi-rollback
ulabi-self-healing
ulabi-health

Observability

ulabi-tracing
ulabi-diagnostics
ulabi-telemetry
ulabi-debug

Validation/conformance

ulabi-validator
ulabi-conformance
ulabi-conformance-runner
ulabi-test-vectors
ulabi-abi-diff
ulabi-schema-validator
ulabi-fuzz


---

Recommended repository implementation tree

When implementation begins, I recommend this rather than mixing implementation code into docs/:

src/
├── core/
│   ├── identity/
│   ├── version/
│   ├── contract/
│   ├── types/
│   ├── values/
│   ├── errors/
│   ├── effects/
│   ├── capabilities/
│   └── validation/
│
├── abi/
│   ├── calling_convention/
│   ├── memory/
│   ├── stack/
│   ├── registers/
│   ├── returns/
│   └── exceptions/
│
├── interop/
│   ├── ffi/
│   ├── language_adapter/
│   ├── object_model/
│   ├── symbols/
│   └── names/
│
├── compatibility/
│   ├── backward/
│   ├── forward/
│   ├── negotiation/
│   ├── discovery/
│   └── degradation/
│
├── runtime/
│   ├── process/
│   ├── threading/
│   ├── async/
│   ├── concurrency/
│   └── resources/
│
├── security/
│   ├── authentication/
│   ├── authorization/
│   ├── capabilities/
│   ├── sandbox/
│   ├── crypto/
│   └── loading/
│
├── distributed/
│   ├── abi/
│   ├── invocation/
│   ├── endpoint/
│   ├── session/
│   ├── serialization/
│   ├── discovery/
│   ├── errors/
│   ├── retry/
│   ├── idempotency/
│   ├── cancellation/
│   ├── deadlines/
│   ├── references/
│   ├── consistency/
│   ├── resources/
│   └── consensus/
│
├── reliability/
│   ├── faults/
│   ├── detection/
│   ├── isolation/
│   ├── recovery/
│   ├── rollback/
│   └── self_healing/
│
├── observability/
│   ├── tracing/
│   ├── diagnostics/
│   ├── telemetry/
│   └── debugging/
│
└── conformance/
    ├── validator/
    ├── runner/
    ├── vectors/
    ├── abi_diff/
    └── fuzz/

Important architectural decision

Do not create a separate implementation module for every .md file automatically.

For example:

docs/distributed/distributed-abi.md

does not necessarily mean there must be:

src/distributed/distributed_abi.rs

The document defines a contract. Several implementation modules may jointly implement that contract.

This preserves the distinction you established:

> ULABI defines the universal interoperability contract; implementations choose their internal architecture.



That is especially important because ULABI must remain independent of Zamani, Sankofa, Rust, C++, or any other implementation language. The repository's current architecture already explicitly establishes that independence.

