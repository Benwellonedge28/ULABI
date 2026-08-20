I read the repository first, including the current ULABI-DESIGN.md, ULABI-SPEC.md, the architecture/compatibility/distributed areas, and searched the repository for existing consensus-related material so this specification does not duplicate an existing implementation.

The important finding is that docs/distributed/consensus-boundaries.md is currently only a placeholder, as are the other distributed documents I inspected.  The master design already establishes the critical principles we need: ULABI is language/runtime/platform neutral, uses a minimal core plus profiles, distinguishes local/process/distributed execution, and explicitly says distributed operation must be defined through profiles rather than being treated as local execution.

The normative specification also already requires distributed calls to preserve distinctions in latency, failure, serialization, availability, and security.

So consensus-boundaries.md should not become another distributed-ABI document. Its unique responsibility should be:

> Define exactly where ULABI interoperability ends and distributed consensus/coordination begins, what ULABI may standardize at that boundary, and what it must deliberately leave to a distributed system's own protocol.




---

docs/distributed/consensus-boundaries.md

Copy everything below into that file.

# ULABI Distributed Consensus Boundaries

**Document:** `docs/distributed/consensus-boundaries.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Status:** Normative Design Specification  
**Specification Family:** Distributed Interoperability  
**Version:** 0.1.0-draft  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the boundary between the ULABI interoperability contract and distributed consensus, coordination, replication, and consistency mechanisms.

ULABI provides a universal interoperability boundary.

ULABI does not define one universal distributed-consensus algorithm.

ULABI MUST NOT require every implementation to use:

- Raft;
- Paxos;
- Byzantine Fault Tolerance;
- Proof of Stake;
- Proof of Work;
- blockchain consensus;
- quorum replication;
- leader election;
- CRDTs;
- transactional replication;
- a particular database;
- a particular message broker;
- a particular transport;
- or any other single distributed coordination mechanism.

Instead, ULABI defines the interoperability contract at the boundary where distributed components communicate.

The central rule is:

> ULABI standardizes what components must agree about at an interoperability boundary without standardizing how an entire distributed system reaches agreement internally.

---

# 2. Scope

This document defines:

1. consensus boundaries;
2. authority boundaries;
3. consistency boundaries;
4. coordination boundaries;
5. replication boundaries;
6. distributed state ownership;
7. agreement metadata;
8. quorum metadata where applicable;
9. conflict semantics;
10. consistency declarations;
11. commit and acknowledgement semantics;
12. failure semantics;
13. partition semantics;
14. retry semantics;
15. idempotency requirements;
16. ordering guarantees;
17. causal relationships;
18. version and epoch information;
19. trust boundaries;
20. security requirements;
21. compatibility requirements;
22. conformance requirements.

This document does NOT define:

- a universal consensus algorithm;
- a universal database;
- a universal replication protocol;
- a universal distributed transaction protocol;
- a universal leader-election mechanism;
- a universal failure detector;
- a universal clock;
- a universal global ordering mechanism.

Those mechanisms MAY be implemented by higher-level protocols or profiles.

---

# 3. Relationship to Other ULABI Specifications

This specification is part of the ULABI Distributed Interoperability family.

The responsibilities are separated as follows:

`ULABI-DESIGN.md`

Defines the overall architectural philosophy.

`ULABI-SPEC.md`

Defines normative Core requirements.

`docs/distributed/distributed-abi.md`

Defines distributed ABI architecture.

`docs/distributed/remote-calls.md`

Defines remote invocation semantics.

`docs/distributed/serialization.md`

Defines distributed representation and encoding.

`docs/distributed/service-discovery.md`

Defines discovery of distributed services.

`docs/distributed/distributed-errors.md`

Defines distributed error semantics.

`docs/distributed/consensus-boundaries.md`

Defines where consensus and distributed agreement semantics begin and end.

`docs/compatibility/feature-negotiation.md`

Defines negotiation of optional distributed capabilities.

`docs/compatibility/capability-discovery.md`

Defines discovery of supported capabilities.

`docs/security/security-model.md`

Defines security boundaries.

`docs/security/capability-security.md`

Defines capability-based authorization.

`docs/runtime/concurrency.md`

Defines runtime concurrency semantics.

`docs/standards/conformance.md`

Defines conformance requirements.

No document in this family should redefine the responsibilities of another document.

---

# 4. Fundamental Principle

ULABI is an interoperability standard, not a distributed operating system.

A distributed system may contain:

```text
Node A
   |
Node B
   |
Node C
   |
Node D

ULABI defines how these nodes can expose and consume interoperable interfaces.

It does not determine how those nodes internally agree on:

ownership;

leadership;

state;

membership;

transactions;

replicated logs;

configuration;

or application-level truth.


Therefore:

Distributed System
        |
        +----------------------+
        |                      |
   Coordination            ULABI Boundary
        |                      |
   Consensus                  Interface
        |                      |
   Replication              Data Contract
        |                      |
   State Machine            Error Contract
        |                      |
   Application State       Capability Contract

The left side is implementation/protocol specific.

The right side is where ULABI applies.


---

5. Consensus Boundary

A consensus boundary is the point at which multiple independent participants must establish agreement over shared state or a shared decision.

Examples include:

selecting a leader;

committing a replicated operation;

agreeing on configuration;

assigning ownership;

deciding transaction outcome;

agreeing on membership;

ordering conflicting operations;

determining whether a state transition is committed.


ULABI MUST represent the outcome of such coordination when that outcome affects an interoperable contract.

ULABI MUST NOT require all participants to expose the internal consensus algorithm.

For example:

Implementation A
      |
      | internal Raft
      v
 Consensus Cluster
      |
      | ULABI boundary
      v
 External Component

The external component needs the resulting contract.

It does not necessarily need to know that Raft was used.


---

6. Consensus Is Not the Same as Interoperability

The following concepts MUST remain distinct:

Interoperability
        !=
Consensus

Interoperability answers:

> "How can independently implemented components understand and invoke one another?"



Consensus answers:

> "How do distributed participants establish agreement over shared state or decisions?"



A system MAY have:

ULABI without consensus;

consensus without ULABI;

ULABI plus consensus;

multiple independent consensus domains connected through ULABI.



---

7. Consensus Domain

A consensus domain is a set of participants that share a defined agreement protocol.

A ULABI implementation SHOULD identify the consensus domain associated with an operation when consensus semantics affect the operation.

A consensus domain MAY be:

a cluster;

a replicated service;

a storage group;

a transaction coordinator;

a control plane;

a device group;

an embedded redundant system;

a distributed application.


A consensus domain MUST NOT be assumed merely because two components communicate remotely.


---

8. Consensus Domain Identity

When exposed through a distributed ULABI profile, a consensus domain MAY have:

Domain Identifier
Protocol Identifier
Protocol Version
Membership Epoch
Configuration Version
Leader/Elected Authority Information
Consistency Model
Commit Semantics
Failure Model

The exact fields depend on the applicable distributed profile.

An implementation MUST NOT claim consensus guarantees that it cannot establish.


---

9. Authority Boundary

A distributed interface MUST distinguish between:

1. component identity;


2. execution authority;


3. state authority;


4. administrative authority;


5. consensus authority.



These are not necessarily the same entity.

For example:

Client
  |
  v
ULABI Service
  |
  +---- Leader
  |
  +---- Replica
  |
  +---- Observer

The client MUST NOT infer that every replica has authority to commit state.


---

10. State Authority

A component that exposes distributed state MUST identify the authority responsible for authoritative state where practical.

Possible authority models include:

Single Authority
Leader Authority
Quorum Authority
Replicated Authority
Partitioned Authority
Application-Defined Authority

ULABI MUST NOT assume one model universally.

The authority model MUST be declared when it materially affects observable behavior.


---

11. State Ownership

Distributed state SHOULD have an explicit ownership model.

Possible models include:

exclusive ownership;

replicated ownership;

partitioned ownership;

shared ownership;

delegated ownership;

temporary ownership;

lease-based ownership.


A ULABI component MUST NOT silently change ownership semantics across compatible interface versions.


---

12. Consistency Model

Distributed interfaces SHOULD declare the consistency guarantees relevant to operations.

Possible consistency levels include:

Local
Eventual
Causal
Session
Read-Your-Writes
Monotonic-Read
Monotonic-Write
Linearizable
Serializable
Application-Defined

The exact supported consistency levels are profile-specific.

An implementation MUST NOT advertise a stronger consistency guarantee than it provides.


---

13. Consistency Is an Observable Contract

Consistency is part of the semantic contract when callers can observe it.

For example:

write(x = 5)
read(x)

The result of read(x) depends on the consistency model.

Therefore:

Data Type
+
Function Signature
+
Error Model
+
Consistency Contract

may collectively define the interoperable meaning of an operation.

A remote interface MUST NOT describe a distributed operation as equivalent to a local operation when their consistency semantics differ.


---

14. Linearizability

Linearizability MUST be explicitly declared.

An implementation MUST NOT imply linearizable behavior merely because operations are exposed through a ULABI interface.

If linearizability is guaranteed, the implementation MUST identify:

the operation scope;

the state scope;

the consistency domain;

failure conditions;

membership assumptions;

commit semantics.



---

15. Eventual Consistency

If an interface provides eventual consistency, the interface SHOULD declare:

convergence expectation;

conflict behavior;

synchronization model;

visibility delay characteristics where relevant;

failure behavior;

stale-read possibility.


A caller MUST NOT assume immediate visibility of a successful write unless the interface contract guarantees it.


---

16. Causal Consistency

If causal guarantees are supported, the interface MAY expose causal metadata.

Possible metadata includes:

Causal Context
Parent Operation
Logical Timestamp
Vector Metadata
Dependency Set

Causal metadata MUST be treated as protocol metadata rather than ordinary application data unless explicitly defined otherwise.


---

17. Global Ordering

ULABI MUST NOT require a global total ordering for all distributed operations.

A distributed system MAY use:

total ordering;

partial ordering;

causal ordering;

per-key ordering;

per-partition ordering;

application-defined ordering;

no ordering guarantee.


The ordering contract MUST be explicit.


---

18. Operation Ordering

An interface that requires ordered operations MUST identify the ordering scope.

Examples:

Global
Domain
Service
Session
Connection
Resource
Key
Partition

A caller MUST NOT infer global ordering from local or per-resource ordering.


---

19. Operation Identity

Distributed operations SHOULD have stable operation identifiers.

An operation identifier MAY contain:

Operation ID
Interface ID
Function ID
Caller Identity
Request Sequence
Correlation ID

Operation identity is especially important for:

retries;

deduplication;

idempotency;

auditing;

tracing;

recovery.



---

20. Idempotency

Distributed operations MUST declare their idempotency semantics when retries are possible.

Possible classifications:

Idempotent
NonIdempotent
ConditionallyIdempotent
Idempotency-Key Required
Unknown

A caller MUST NOT automatically retry a non-idempotent operation unless the interface explicitly defines safe retry semantics.


---

21. Commit Semantics

A distributed operation MAY transition through:

Submitted
Accepted
Prepared
Committed
Applied
Acknowledged
Rejected
Aborted
Unknown

These states MUST NOT be conflated.

For example:

Accepted != Committed
Committed != Applied
Applied != Acknowledged

A successful network response MUST NOT automatically mean that the distributed state has been durably committed.


---

22. Commit Evidence

Where practical, a distributed implementation SHOULD expose sufficient metadata to establish the meaning of a successful commit.

Possible evidence includes:

Commit ID
Log Position
Epoch
Version
Quorum Confirmation
Durability Level
Consistency Level

The exact representation is profile-specific.


---

23. Acknowledgement Semantics

Acknowledgements MUST specify what they acknowledge.

Possible meanings include:

Receipt
Validation
Admission
Persistence
Replication
Quorum
Commit
Application
Delivery

A field named merely success MUST NOT be interpreted as all of the above.


---

24. Quorum Semantics

ULABI MAY expose quorum-related metadata when an implementation uses quorum-based coordination.

Possible metadata includes:

Required Votes
Received Votes
Membership Size
Quorum Status
Commit Status
Epoch

ULABI MUST NOT require quorum-based consensus.

If quorum semantics are exposed, their meaning MUST be defined by the applicable profile.


---

25. Failure Model

Distributed interfaces MUST treat failures as first-class conditions.

Possible failure classes include:

node failure;

process failure;

network failure;

partition;

timeout;

overload;

stale membership;

leader change;

quorum loss;

integrity failure;

authorization failure;

protocol incompatibility;

unknown commit status.


A distributed operation MUST NOT treat communication failure as proof that the operation was not committed.


---

26. Unknown Outcome

A distributed operation MAY reach an unknown outcome.

For example:

Client
  |
  | request
  v
Server
  |
  | commit
  v
Consensus Domain
  |
  X response lost
  |
Client

The client may not know whether the operation committed.

Therefore ULABI MUST support an explicit:

OutcomeUnknown

or equivalent machine-readable state when applicable.


---

27. Retry After Unknown Outcome

A caller MUST NOT blindly retry an operation with unknown outcome unless:

1. the operation is idempotent; or


2. an idempotency key is supplied; or


3. the interface defines a safe retry mechanism.



A distributed interface SHOULD provide an operation-status query where practical.


---

28. Partition Semantics

A network partition MUST be treated separately from ordinary unavailability.

Possible states include:

Reachable
TemporarilyUnavailable
PartitionSuspected
Partitioned
QuorumLost
Degraded
Recovering

Implementations MAY collapse these states internally, but MUST expose enough information to preserve the semantics required by the interface contract.


---

29. Split-Brain Protection

A distributed system claiming exclusive authority MUST define how split-brain conditions are prevented or detected.

ULABI does not prescribe the mechanism.

Possible mechanisms include:

quorum;

fencing;

leases;

epochs;

fencing tokens;

external authority;

hardware arbitration;

application-specific mechanisms.


A system MUST NOT expose two simultaneously authoritative writers when its contract promises exclusive authority.


---

30. Epochs

Distributed authorities SHOULD use epochs, terms, generations, or equivalent monotonic identifiers where stale authority is possible.

An epoch MAY identify:

Membership Configuration
Leadership Term
Ownership Generation
Protocol Generation
Configuration Generation

A stale epoch MUST NOT be accepted as current authority where the protocol requires fencing.


---

31. Fencing

Where stale participants can cause unsafe writes, the distributed profile SHOULD support fencing.

A fencing token MUST be distinguishable from an ordinary identifier.

A component MUST NOT assume that possession of an old lease or authority token grants current authority.


---

32. Membership Changes

Membership changes MAY affect consensus semantics.

A distributed interface SHOULD identify:

membership epoch;

configuration version;

active authority;

transition state.


Membership changes MUST NOT silently alter the meaning of a compatible interface.


---

33. Consensus Protocol Independence

A ULABI implementation MAY use any internally appropriate consensus mechanism.

For example:

ULABI Interface
      |
      +---- Raft
      |
      +---- Paxos
      |
      +---- BFT
      |
      +---- CRDT-based coordination
      |
      +---- Database transaction protocol
      |
      +---- Hardware redundancy
      |
      +---- Application-defined protocol

The external ULABI contract remains independent of the internal mechanism.


---

34. Consensus Protocol Identification

A distributed implementation MAY expose protocol identity when useful for diagnostics, auditing, certification, or interoperability.

If exposed, protocol metadata MUST NOT be interpreted as a universal guarantee.

For example:

protocol = "example-consensus"

does not itself prove:

safety;

liveness;

Byzantine tolerance;

durability;

linearizability.


Those properties MUST be declared separately.


---

35. Safety and Liveness

Distributed profiles SHOULD distinguish:

Safety

Something bad does not happen.

Examples:

conflicting commits are not both accepted;

stale authority cannot write;

invalid state transitions are rejected.


Liveness

Something good eventually happens under defined assumptions.

Examples:

requests eventually commit;

leadership can eventually be established;

replicas eventually converge.


An implementation MUST NOT claim unconditional liveness.


---

36. Failure Assumptions

Consensus guarantees depend on assumptions.

A distributed profile SHOULD declare assumptions such as:

Crash Faults
Network Partitions
Message Loss
Message Reordering
Message Duplication
Byzantine Behaviour
Clock Failure
Storage Failure
Hardware Failure
Membership Churn

A guarantee MUST be interpreted only within its declared failure model.


---

37. Time and Clocks

ULABI MUST NOT assume synchronized physical clocks for consensus correctness.

Distributed systems MAY use:

physical clocks;

logical clocks;

hybrid clocks;

monotonic clocks;

vector clocks;

application-defined clocks.


Clock metadata MUST NOT be treated as proof of consensus unless the applicable protocol explicitly establishes that relationship.


---

38. Transactions

ULABI MUST NOT assume that every distributed operation is transactional.

If transactional semantics are exposed, the applicable profile MUST define:

transaction identity;

participants;

isolation;

commit semantics;

abort semantics;

timeout;

retry;

recovery;

durability;

failure behavior.


Distributed transaction protocols belong to higher-level profiles.


---

39. Atomicity

Atomicity MUST be explicitly declared.

A multi-resource operation MUST NOT be assumed atomic merely because it is represented as one ULABI function call.

For example:

function transfer(A, B, amount)

does not automatically guarantee:

A changed
AND
B changed

atomically.

Such semantics must be part of the contract.


---

40. Consistency Boundaries Across Services

Two independent ULABI services MUST NOT be assumed to share one consistency domain.

For example:

Service A
   |
   | ULABI
   v
Service B
   |
   | ULABI
   v
Service C

does not imply:

A + B + C = one consensus domain

Each service MAY have an independent consistency model.


---

41. Cross-Domain Consensus

If an operation crosses multiple consensus domains, the interface MUST identify the resulting semantics.

Possible models include:

Independent Commit
Best Effort
Saga
Two-Phase Commit
Three-Phase Commit
Application Coordination
Atomic Cross-Domain Protocol

ULABI MUST NOT silently upgrade independent commits into atomic distributed consensus.


---

42. Consensus Boundary Diagram

The canonical architectural model is:

+-------------------------------------------------------+
|                 Distributed Application               |
+-------------------------------------------------------+
|                                                       |
|   +----------------+       +----------------+         |
|   | Consensus      |       | Consensus      |         |
|   | Domain A       |       | Domain B       |         |
|   +-------+--------+       +-------+--------+         |
|           |                        |                  |
|           | Internal Protocol      |                  |
|           |                        |                  |
+-----------+------------------------+------------------+
            |                        |
            |      ULABI Boundary    |
            v                        v
       +---------+              +---------+
       | ULABI   |              | ULABI   |
       | Contract|              | Contract|
       +---------+              +---------+
            |                        |
            +-----------+------------+
                        |
                  External Client

The consensus mechanisms remain behind the boundary.

Observable guarantees cross the boundary.


---

43. Distributed State Machine Boundary

A consensus domain may internally implement:

Input
  |
  v
Validation
  |
  v
Consensus
  |
  v
Ordered Log
  |
  v
State Machine
  |
  v
Committed State

ULABI begins at the interoperable interface:

Committed State
       |
       v
ULABI Representation
       |
       v
External Consumer

ULABI MUST NOT require the internal state machine architecture.


---

44. Read Semantics

Distributed reads SHOULD identify whether they are:

Local
Leader
Quorum
Linearizable
Snapshot
Stale-Allowed
Eventually-Consistent
Causal

A read operation MAY return metadata describing the version or consistency point observed.


---

45. Write Semantics

Distributed writes SHOULD identify:

Accepted
Persisted
Replicated
Committed
Applied
Visible

These states MAY occur at different times.

An implementation MUST NOT collapse them when doing so would change the observable contract.


---

46. Versioned State

Distributed state SHOULD have a version or equivalent monotonic identifier where version comparison is meaningful.

Possible forms include:

Revision
Sequence
Epoch
Log Position
Generation
Version Vector
Application Version

Version identifiers MUST have explicitly defined comparison semantics.


---

47. Conflict Detection

If concurrent updates can conflict, the interface MUST define conflict semantics.

Possible outcomes:

Reject
Last-Writer-Wins
Merge
Application Callback
Conflict Object
Retry
Manual Resolution

A ULABI implementation MUST NOT silently discard conflicting state unless the contract explicitly permits it.


---

48. Conflict Representation

A structured conflict SHOULD contain:

Conflict ID
Resource ID
Versions
Participants
Cause
Resolution State
Resolution Policy

Human-readable text MUST NOT be the only representation.


---

49. Replication

Replication is not equivalent to consensus.

A system MAY replicate data without reaching consensus.

Therefore:

Replication != Consensus

Similarly:

Consensus != Replication

A distributed interface MUST distinguish them when their semantics affect callers.


---

50. Durability

Durability MUST be explicitly defined where promised.

Possible durability levels include:

MemoryOnly
LocalPersistent
Replicated
QuorumDurable
ConsensusCommitted
ApplicationDefined

A successful acknowledgement MUST NOT imply durable persistence unless specified.


---

51. Availability

Availability guarantees MUST be bounded by the declared failure model.

A service SHOULD identify whether an operation is available during:

node failure;

replica loss;

leader failure;

network partition;

quorum loss;

degraded operation.


An implementation MUST NOT claim universal availability.


---

52. CAP-Related Semantics

ULABI does not mandate a particular CAP strategy.

A distributed implementation MAY prioritize different properties under partition.

The interface MUST clearly describe what happens during partition.

For example:

Partition
   |
   +---- Reject writes
   |
   +---- Serve stale reads
   |
   +---- Continue one partition
   |
   +---- Degrade capability
   |
   +---- Application-defined behavior

The choice is implementation/profile specific.


---

53. Degraded Operation

A distributed service MAY continue operating with reduced guarantees.

If so, the degraded mode MUST be explicit.

Possible metadata:

Normal
Degraded
ReadOnly
Stale
Partitioned
QuorumLost
Recovering

A caller MUST NOT assume normal guarantees while the service reports degraded semantics.


---

54. Recovery

Recovery MAY include:

replica synchronization;

state replay;

leader re-election;

conflict resolution;

checkpoint restoration;

log recovery;

membership restoration.


Recovery semantics MUST remain consistent with the declared distributed contract.

Recovery MUST NOT silently invalidate previously declared committed state.


---

55. Rollback

Rollback MUST be explicitly defined.

A failed recovery MAY cause:

Rollback
Retry
Abort
Quarantine
Escalation
Manual Recovery

Rollback MUST NOT be inferred from cancellation alone.


---

56. Self-Healing Interaction

Self-healing and consensus MUST remain separate concerns.

A self-healing implementation MUST NOT:

rewrite consensus state without authorization;

change consensus rules without protocol agreement;

bypass quorum requirements;

bypass fencing;

bypass authentication;

silently alter committed state.


Self-healing MAY execute an explicitly authorized recovery procedure.

The relevant recovery process is:

Failure
   |
Evidence
   |
Diagnosis
   |
Known Policy?
 +-----+-----+
 |           |
Yes          No
 |           |
Recovery    Escalate
 |
Verification
 |
Healthy?
 +-----+-----+
 |           |
Yes          No
 |           |
Continue    Rollback /
            Escalate


---

57. Security Boundary

Consensus metadata MUST NOT itself be treated as authorization.

For example:

leader = true

does not automatically mean:

authorized = true

Authorization MUST remain governed by the ULABI security model.

Consensus identity, component identity, and security identity MAY be related but MUST remain conceptually distinct.


---

58. Byzantine Systems

ULABI MAY support Byzantine-resilient distributed profiles.

However, ULABI Core MUST NOT require Byzantine consensus.

A Byzantine profile SHOULD define:

participant authentication;

message integrity;

identity;

fault assumptions;

quorum requirements;

commit semantics;

evidence;

failure handling.


A component MUST NOT claim Byzantine safety without satisfying the applicable profile.


---

59. Evidence and Proof

A distributed profile MAY expose cryptographic or protocol evidence.

Examples include:

Commit Certificate
Quorum Certificate
Membership Proof
State Proof
Integrity Proof
Attestation
Signature Set

Evidence formats MUST be defined by the applicable profile.

A consumer MUST NOT infer security properties from opaque evidence.


---

60. Determinism

Consensus-critical operations SHOULD be deterministic where required by their underlying protocol.

ULABI MUST NOT require a language-specific deterministic runtime.

If deterministic execution is part of a distributed contract, the applicable profile MUST define:

deterministic inputs;

deterministic outputs;

allowed nondeterminism;

time semantics;

randomness semantics;

external effects.



---

61. Randomness

Consensus-critical randomness MUST be explicitly defined.

An implementation MUST NOT assume that a source-language random-number generator is suitable for distributed agreement.

Where verifiable randomness is required, the applicable profile MUST define it.


---

62. External Side Effects

Consensus decisions and external side effects SHOULD be separated.

For example:

Consensus Commit
       |
       v
Committed Decision
       |
       v
External Effect

A commit does not automatically prove that an external side effect occurred.

Where exactly-once or effectively-once behavior is required, the applicable profile MUST define the mechanism.


---

63. Exactly-Once Semantics

ULABI MUST NOT claim exactly-once execution merely because an operation has a unique identifier.

Exactly-once semantics require explicit protocol guarantees.

Possible mechanisms include:

durable deduplication;

transactional state transition;

idempotent effect;

transactional outbox;

application-specific coordination.



---

64. At-Least-Once and At-Most-Once

Distributed interfaces MAY declare:

AtMostOnce
AtLeastOnce
EffectivelyOnce
ExactlyOnce
BestEffort

The guarantee MUST be defined in terms of observable behavior.


---

65. Retry and Consensus Interaction

Retries MUST preserve the consensus contract.

A retry MUST NOT:

create duplicate committed operations where duplication is forbidden;

bypass ordering;

bypass authorization;

bypass epoch validation;

bypass fencing;

alter transaction identity unexpectedly.



---

66. Client-Side Failover

A client MAY fail over between authorities.

If failover is supported, the interface SHOULD define:

Authority Discovery
Authority Verification
Epoch Validation
Session Continuity
Retry Semantics
Consistency Semantics

A client MUST NOT fail over to an incompatible authority merely because it is reachable.


---

67. Session Semantics

Session-based consistency MAY be supported.

A session SHOULD have a stable identity.

Session metadata MAY include:

Session ID
Session Epoch
Last Observed Version
Causal Context
Authority
Capabilities

Session semantics MUST be explicit.


---

68. Distributed Locks

ULABI MUST NOT assume that a distributed lock is equivalent to consensus.

If distributed locking is exposed, it SHOULD define:

lock identity;

ownership;

expiration;

renewal;

fencing;

failure behavior;

recovery.


Locks without fencing MUST NOT automatically be treated as safe exclusive authority.


---

69. Leases

A lease MUST include an explicit expiration model.

A lease holder MUST NOT assume continued authority after lease expiry.

Where stale holders can cause unsafe writes, fencing SHOULD be used.


---

70. Consensus and Capability Discovery

Capability discovery SHOULD be able to expose distributed guarantees.

Examples:

SupportsLinearizableReads
SupportsQuorumWrites
SupportsCausalReads
SupportsTransactions
SupportsIdempotencyKeys
SupportsOperationStatus
SupportsFencing
SupportsConsensusEvidence

A capability MUST describe a real supported behavior.


---

71. Consensus and Feature Negotiation

Optional consensus-related functionality MUST be negotiated before use.

For example:

Client
  |
  | capabilities?
  v
Service
  |
  | SupportsLinearizableReads
  | SupportsFencing
  | SupportsOperationStatus
  v
Negotiated Contract

Unknown optional features MAY be ignored.

Unknown mandatory features MUST produce safe incompatibility handling.


---

72. Compatibility

Consensus semantics MUST be treated as part of interface compatibility when they affect observable behavior.

The following change MAY be breaking:

Linearizable
      ↓
Eventual

Likewise:

Atomic
      ↓
Best Effort

or:

Idempotent
      ↓
NonIdempotent

or:

Committed
      ↓
Accepted

Therefore semantic compatibility is more important than structural signature compatibility alone.


---

73. Backward Compatibility

A new implementation MUST preserve the distributed guarantees promised by a compatible interface version.

Adding optional metadata SHOULD NOT break existing consumers.

Removing a previously guaranteed consistency property MAY be a breaking change.


---

74. Forward Compatibility

Consumers MUST safely handle unknown optional distributed metadata.

For example:

known fields
+
unknown optional fields

must remain processable when the unknown fields are not required to understand the operation.

Unknown mandatory distributed semantics MUST cause safe rejection or explicit incompatibility.


---

75. Failure Translation

Distributed protocol failures MUST be translated into stable ULABI errors.

Examples:

QuorumLost
LeaderChanged
EpochMismatch
PartitionDetected
CommitUnknown
StaleAuthority
Conflict
ConsistencyUnavailable
TransactionAborted

These MAY map to the general ULABI error model.

A language implementation MAY map them to:

exceptions;

result values;

status codes;

error objects.


The underlying semantic error MUST remain stable.


---

76. Observability

Distributed operations SHOULD expose correlation information where supported.

Useful metadata includes:

Trace ID
Span ID
Operation ID
Consensus Domain ID
Epoch
Commit ID
Replica ID
Authority ID

Observability metadata MUST NOT become a hidden correctness dependency.


---

77. Privacy

Distributed metadata MAY reveal:

topology;

membership;

node identity;

timing;

operation history;

consistency state.


Security profiles SHOULD define which metadata may be exposed.

Sensitive consensus metadata MUST be protected according to the applicable security policy.


---

78. Resource Limits

Distributed consensus operations SHOULD support explicit limits.

Examples:

Timeout
Message Size
Retry Count
Replication Fanout
Quorum Wait
Transaction Duration
Lease Duration
Session Duration

Resource exhaustion MUST produce a defined error.


---

79. No Hidden Consensus

An interface MUST NOT silently introduce distributed consensus merely because its implementation moved from local to remote execution.

For example:

Local Function
      |
      v
Remote Implementation
      |
      X
Hidden Consensus

is prohibited when the added consensus changes observable behavior without being declared.

Location changes MUST preserve declared semantics or produce an explicit compatibility difference.


---

80. No Hidden Network Dependency

A local-only interface MUST NOT silently become dependent on remote consensus.

If an interface requires remote coordination, that requirement MUST be declared.

This preserves the locality semantics defined by ULABI.


---

81. Local-to-Distributed Promotion

An implementation MAY provide a distributed version of a local interface.

However, the distributed version SHOULD explicitly identify additional semantics:

Latency
Failure
Consistency
Availability
Ordering
Retry
Security
Resource Limits

The distributed interface MUST NOT claim equivalence when those semantics materially differ.


---

82. Consensus Boundary Contract

A distributed ULABI interface that exposes consensus-sensitive behavior SHOULD provide the following logical contract:

ConsensusBoundary {
    domain_id
    authority_model
    consistency_model
    ordering_model
    commit_model
    durability_model
    failure_model
    partition_model
    idempotency_model
    epoch
    capability_set
}

This is a semantic model.

The wire representation is defined by the applicable serialization/profile specification.


---

83. Minimum Consensus Boundary Metadata

A distributed profile claiming explicit consensus semantics SHOULD provide, where applicable:

Domain Identity
Authority Model
Consistency Model
Commit Semantics
Failure Semantics
Operation Identity
Idempotency Semantics
Version/Epoch

Optional metadata MAY include:

Quorum
Leader
Commit Certificate
Replica Set
Causal Context
Fencing Token


---

84. Reference State Machine

A canonical conceptual state machine is:

+-------------+
                  |  Submitted  |
                  +------+------+
                         |
                         v
                  +-------------+
                  |  Accepted   |
                  +------+------+
                         |
                         v
                  +-------------+
                  |  Prepared   |
                  +------+------+
                         |
                  +------+------+
                  |             |
                  v             v
            +-----------+  +-----------+
            | Committed |  |  Aborted  |
            +-----+-----+  +-----------+
                  |
                  v
            +-----------+
            |  Applied  |
            +-----+-----+
                  |
                  v
            +-----------+
            |Acknowledged|
            +-----------+

A communication failure may instead result in:

+----------------+
             | OutcomeUnknown |
             +----------------+

OutcomeUnknown MUST NOT be interpreted as Aborted.


---

85. Consensus Boundary Invariants

The following invariants are REQUIRED:

Invariant 1 — No Algorithm Mandate

ULABI MUST NOT require one universal consensus algorithm.

Invariant 2 — No False Guarantees

An implementation MUST NOT advertise guarantees it cannot provide.

Invariant 3 — Explicit Commit

Accepted, committed, applied, and acknowledged MUST remain distinguishable where their distinction is observable.

Invariant 4 — Explicit Consistency

Distributed consistency MUST be declared when it affects observable behavior.

Invariant 5 — Explicit Failure

Unknown distributed outcomes MUST be representable.

Invariant 6 — Retry Safety

Retry semantics MUST preserve operation identity and idempotency requirements.

Invariant 7 — Epoch Safety

Stale authority MUST NOT be treated as current authority where fencing is required.

Invariant 8 — No Hidden Consensus

Consensus MUST NOT be introduced invisibly across a compatibility boundary.

Invariant 9 — Security Separation

Consensus authority MUST NOT automatically imply authorization.

Invariant 10 — Implementation Independence

The external contract MUST NOT depend on the implementation language or internal consensus algorithm.


---

86. Security Requirements

A conformant implementation MUST:

1. authenticate protected consensus participants where required;


2. validate authority metadata;


3. reject stale authority where the contract requires fencing;


4. protect consensus credentials;


5. prevent unauthorized state transitions;


6. validate protocol versions;


7. validate message integrity where required;


8. prevent replay where replay could alter consensus state;


9. enforce capability restrictions;


10. preserve auditability where required.



Consensus metadata MUST NOT bypass normal authorization controls.


---

87. Failure Modes

Implementations SHOULD explicitly handle:

NoConsensus
QuorumUnavailable
LeaderUnavailable
LeaderChanged
EpochMismatch
MembershipChanged
Partition
Timeout
CommitUnknown
DuplicateRequest
Conflict
StaleRead
StaleWrite
AuthorizationFailure
IntegrityFailure
ProtocolMismatch
UnsupportedConsistency
UnsupportedTransaction
ResourceExhausted

Each failure MUST map to a deterministic ULABI error representation.


---

88. Recovery Behaviour

Recovery MUST preserve safety guarantees.

After failure:

Detect
  |
Classify
  |
Determine Consensus State
  |
Recover
  |
Verify
  |
Expose Stable State

An implementation MUST NOT declare recovery successful solely because communication has been restored.

The recovered state MUST satisfy the applicable consistency and integrity guarantees.


---

89. Conformance Requirements

A consensus-boundary implementation is conformant only if it can demonstrate, where applicable:

CB-001

Consensus semantics are explicitly identified.

CB-002

The implementation does not mandate a single consensus algorithm.

CB-003

Consistency guarantees are correctly declared.

CB-004

Commit and acknowledgement semantics are distinguishable.

CB-005

Unknown outcomes are represented.

CB-006

Retry behavior preserves operation identity.

CB-007

Idempotency semantics are enforced.

CB-008

Stale authority is rejected where required.

CB-009

Partition behavior matches the declared contract.

CB-010

Consensus authority does not bypass security authorization.

CB-011

Unknown optional metadata is safely handled.

CB-012

Unknown mandatory semantics result in safe incompatibility.

CB-013

Distributed failures map to stable ULABI errors.

CB-014

The implementation preserves declared guarantees across compatible versions.

CB-015

Consensus implementation details do not leak into the universal ABI contract unless explicitly exposed as metadata.


---

90. Required Conformance Tests

The eventual conformance suite SHOULD include:

consensus_domain_identity
authority_model
consistency_declaration
commit_state_machine
acknowledgement_semantics
unknown_outcome
retry_idempotency
duplicate_request
epoch_validation
stale_authority
fencing
leader_change
membership_change
partition_behavior
quorum_loss
recovery
rollback
conflict_detection
read_consistency
write_consistency
transaction_semantics
cross_domain_commit
capability_negotiation
security_boundary
protocol_version_mismatch
unknown_optional_feature
unknown_mandatory_feature


---

91. Interoperability Test

Two independently implemented systems MUST be able to exchange consensus-boundary metadata without sharing:

source code;

compiler;

programming language;

runtime;

operating system;

CPU;

database;

consensus implementation.


Example:

Implementation A
    |
    | ULABI
    v
Implementation B

Both implementations may use completely different internal architectures.

The observable distributed contract MUST remain compatible.


---

92. Reference Example

Consider:

Client
  |
  | write(value)
  v
Service
  |
  +---- Node A
  +---- Node B
  +---- Node C

Suppose the service uses an internal consensus protocol.

The client does not need to know the internal algorithm.

The ULABI contract may expose:

Consistency = Linearizable
Commit = QuorumCommitted
Durability = Replicated
Idempotency = Required
Ordering = PerResource

The internal implementation may use any compliant mechanism.


---

93. Cross-Language Example

The same distributed contract may be consumed by:

C
Rust
Go
Java
Python
Swift
Kotlin
Fortran
Ada
Zig

The language implementation maps the contract into its native programming model.

ULABI remains unchanged.


---

94. Cross-Project Independence

ULABI MUST remain independent from:

Zamani
Sankofa
Any specific compiler
Any specific runtime
Any specific operating system
Any specific database
Any specific consensus protocol

Zamani MAY implement the consensus-boundary profile.

Sankofa MAY implement the consensus-boundary profile.

Neither project defines the profile.


---

95. Implementation Layering

The eventual implementation SHOULD separate:

ULABI Core
    |
    +-- Identity
    +-- Types
    +-- Calls
    +-- Errors
    +-- Versioning
    |
    +-- Distributed Profile
            |
            +-- Consensus Boundary
            +-- Remote Calls
            +-- Serialization
            +-- Service Discovery
            +-- Distributed Errors
            +-- Consistency
            +-- Operation Tracking
            +-- Recovery

The distributed implementation MUST NOT contaminate the Core with algorithm-specific assumptions.


---

96. Reference Implementation Boundary

A reference implementation MAY implement:

ConsensusBoundary
ConsistencyModel
CommitState
OperationIdentity
Epoch
Authority
QuorumMetadata
Fencing
OutcomeStatus

The reference implementation MUST remain an implementation of the specification.

It MUST NOT become the specification itself.


---

97. Future Extensions

Future ULABI profiles MAY define:

Byzantine consensus;

replicated state machines;

distributed transactions;

CRDT interoperability;

blockchain interoperability;

secure multiparty coordination;

hardware consensus;

safety-critical redundant consensus;

real-time consensus;

cross-domain atomic transactions;

verifiable distributed state;

cryptographic consensus evidence.


Such extensions MUST remain profiles unless their semantics are fundamental to the ULABI Core.


---

98. Non-Goals

This document does not attempt to define:

one consensus algorithm;

one distributed database;

one replication mechanism;

one transaction protocol;

one messaging protocol;

one service mesh;

one cluster manager;

one cloud architecture;

one blockchain;

one distributed operating system.


ULABI interoperates with these systems.

It does not replace them.


---

99. Architectural Rule

The definitive rule is:

> ULABI defines the interoperable meaning of distributed operations at a boundary; it does not define how an entire distributed system internally reaches agreement.



Therefore:

Consensus Algorithm
        |
        v
Internal Distributed System
        |
        v
Consensus Result
        |
        v
ULABI Contract
        |
        v
Independent Consumer

Only the interoperable contract crosses the ULABI boundary unless the applicable profile explicitly exposes additional protocol metadata.


---

100. Summary

ULABI MUST distinguish:

Interoperability
Consensus
Replication
Consistency
Authority
Commit
Acknowledgement
Durability
Availability
Ordering
Failure
Recovery
Security

These concepts interact, but they are not interchangeable.

The resulting architecture is:

ULABI
                    |
        +-----------+-----------+
        |                       |
 Local Interoperability   Distributed Boundary
                                |
                    +-----------+-----------+
                    |           |           |
                Authority   Consistency   Commit
                    |           |           |
                Ordering     Failure     Recovery
                    |           |           |
                    +-----------+-----------+
                                |
                         Internal Protocol
                                |
                    +-----------+-----------+
                    |           |           |
                  Raft        BFT       Other

ULABI remains universal because the lower layer is implementation-specific while the boundary remains standardized.

The goal is not to make every distributed system work the same way.

The goal is to make independently designed distributed systems understand one another without requiring them to share the same internal architecture.

## What this file deliberately does **not** duplicate

I intentionally kept this document from becoming a second copy of the distributed ABI, remote-call, serialization, or service-discovery specifications.

The existing repository already has those dedicated files, although the distributed files are currently placeholders.    

The new document therefore owns **consensus boundary semantics**, while the other documents should eventually own their respective protocols.

---

# Complete implementation/documentation dependency plan

I would **not** start writing implementation code yet. The repository currently has the architecture/specification foundation but no discovered implementation module tree; the root currently consists of the master documents plus `docs/`, and the repository searches did not reveal an existing `.rs` implementation tree. 

The correct order is therefore:

## Phase 1 — Independent normative foundations

These should be completed first because almost everything else depends on them.

| Order | File | Responsibility |
|---|---|---|
| 1 | `ULABI-DESIGN.md` | Overall architecture |
| 2 | `ULABI-SPEC.md` | Normative Core |
| 3 | `ULABI-VERSIONING.md` | Version identity and compatibility |
| 4 | `ULABI-GOVERNANCE.md` | Change/governance process |
| 5 | `docs/architecture/design-principles.md` | Architectural invariants |
| 6 | `docs/architecture/overview.md` | Architecture boundaries |
| 7 | `docs/architecture/universal-model.md` | Universal semantic model |
| 8 | `docs/architecture/scalability.md` | Scale model |

`ULABI-VERSIONING.md` is currently only a placeholder, so this is an important missing foundation. 

---

# Phase 2 — Core ABI contract

These define the stable ABI on which distributed semantics depend.

```text
docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

Required responsibility

These files must settle:

binary boundary;

call representation;

primitive representation;

structured representation;

ownership;

alignment;

memory;

return values;

errors;

ABI identity.


No distributed module should redefine these.


---

Phase 3 — Type-system contract

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/type-compatibility.md

These define what a distributed consensus result actually means at the type boundary.


---

Phase 4 — Interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

These should define language adapters without making any one language authoritative.


---

Phase 5 — Runtime

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

The consensus document depends particularly on:

async-model
concurrency
resource-management

but should not duplicate them.


---

Phase 6 — Memory

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

These determine whether distributed buffers, shared-memory transports, and zero-copy mechanisms are safe.


---

Phase 7 — Security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Consensus authority must never become authorization authority.

That separation is now explicitly locked into consensus-boundaries.md.


---

Phase 8 — Distributed layer

This is where the new document belongs.

Recommended completion order:

1. docs/distributed/consensus-boundaries.md
2. docs/distributed/distributed-abi.md
3. docs/distributed/serialization.md
4. docs/distributed/service-discovery.md
5. docs/distributed/remote-calls.md
6. docs/distributed/distributed-errors.md

Then:

docs/distributed/consensus-boundaries.md
        |
        +--> distributed-abi.md
        |
        +--> serialization.md
        |
        +--> remote-calls.md
        |
        +--> service-discovery.md
        |
        +--> distributed-errors.md

This order prevents the remote-call document from inventing its own consensus semantics.


---

Phase 9 — Compatibility

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

The dependency should be:

Versioning
   |
   +--> Backward Compatibility
   |
   +--> Forward Compatibility
   |
   +--> Feature Negotiation
             |
             +--> Capability Discovery
             |
             +--> Graceful Degradation

Consensus-boundary semantics feed into this layer rather than duplicating it.


---

Phase 10 — Reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md
docs/reliability/self-healing.md

The dependency becomes:

Distributed Failure
       |
       v
Fault Detection
       |
       v
Fault Isolation
       |
       v
Recovery
       |
       v
Verification
       |
       +---- success ---> continue
       |
       +---- failure ---> rollback/escalate

This keeps self-healing from becoming an uncontrolled consensus-state modification mechanism.


---

Phase 11 — Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md

The validator becomes particularly important because it will eventually verify:

Interface
+
Types
+
Version
+
Capabilities
+
Distributed Guarantees
+
Security


---

Phase 12 — Observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md

These should consume distributed metadata such as:

operation_id
trace_id
consensus_domain_id
epoch
commit_id
authority_id

rather than defining their own distributed semantics.


---

Required schemas

Once the specifications above stabilize, create:

schemas/
├── interface.schema
├── function.schema
├── type.schema
├── capability.schema
├── version.schema
├── error.schema
├── distributed-operation.schema
├── consensus-boundary.schema
├── consistency.schema
├── commit-status.schema
├── authority.schema
├── epoch.schema
├── fencing-token.schema
└── feature-negotiation.schema

The most important new schema from this document is:

schemas/consensus-boundary.schema

It should represent the logical contract defined in section 82.


---

Required examples

Create:

examples/
├── local/
├── process/
├── distributed/
├── consensus/
│   ├── linearizable-service/
│   ├── eventual-service/
│   ├── quorum-service/
│   ├── unknown-outcome/
│   ├── leader-change/
│   └── partition/
└── cross-language/

The examples must demonstrate semantics, not merely syntax.


---

Required tests

Create:

tests/
├── core/
├── types/
├── memory/
├── compatibility/
├── security/
├── distributed/
│   ├── consensus-boundary/
│   │   ├── authority.rs
│   │   ├── consistency.rs
│   │   ├── commit.rs
│   │   ├── unknown-outcome.rs
│   │   ├── retry.rs
│   │   ├── idempotency.rs
│   │   ├── epoch.rs
│   │   ├── fencing.rs
│   │   ├── partition.rs
│   │   ├── quorum.rs
│   │   └── recovery.rs
│   └── integration/
└── conformance/

The language is not prescribed by ULABI. The .rs names above are a proposed reference-test implementation, not an existing repository structure.


---

Required conformance layer

conformance/
├── core/
├── types/
├── memory/
├── security/
├── compatibility/
├── distributed/
│   ├── consensus-boundary/
│   ├── remote-calls/
│   ├── serialization/
│   └── service-discovery/
└── profiles/

The distributed conformance tests should verify the contract rather than test whether a particular implementation uses Raft, Paxos, etc.


---

Required reference implementation modules

Because there is currently no established implementation module tree to preserve, I recommend not creating arbitrary language-specific source files yet.

Once the normative documents stabilize, the reference implementation should have language-neutral module responsibilities approximately equivalent to:

reference/
└── core/
    ├── identity
    ├── version
    ├── types
    ├── encoding
    ├── errors
    ├── calls
    ├── memory
    ├── capabilities
    └── compatibility

and:

reference/
└── distributed/
    ├── consensus_boundary
    ├── authority
    ├── consistency
    ├── ordering
    ├── operation_identity
    ├── commit_status
    ├── epoch
    ├── fencing
    ├── retry
    ├── conflict
    ├── partition
    ├── quorum
    ├── recovery
    └── distributed_errors

These are module responsibilities, not claims that those files currently exist.


---

Required implementation modules

When implementation begins, I recommend these logical modules:

Core

ulabi_core
ulabi_identity
ulabi_version
ulabi_types
ulabi_encoding
ulabi_calls
ulabi_errors
ulabi_memory
ulabi_capabilities
ulabi_compatibility

Compatibility

ulabi_feature_negotiation
ulabi_capability_discovery
ulabi_version_negotiation
ulabi_graceful_degradation

Distributed

ulabi_distributed
ulabi_consensus_boundary
ulabi_authority
ulabi_consistency
ulabi_ordering
ulabi_operation
ulabi_commit
ulabi_epoch
ulabi_fencing
ulabi_retry
ulabi_conflict
ulabi_partition
ulabi_quorum
ulabi_durability
ulabi_recovery
ulabi_distributed_errors

Transport-independent distributed interfaces

ulabi_remote_call
ulabi_serialization
ulabi_service_discovery
ulabi_transport

Security

ulabi_authentication
ulabi_authorization
ulabi_capability_security
ulabi_integrity
ulabi_secure_loading

Validation

ulabi_validator
ulabi_schema_validator
ulabi_compatibility_checker
ulabi_conformance


---

Most important integration rule

The implementation dependency graph should be:

ULABI-SPEC
                        |
                        v
                   ULABI Core
                        |
          +-------------+-------------+
          |             |             |
        Types         Calls        Versioning
          |             |             |
          +-------------+-------------+
                        |
                        v
                 Compatibility
                        |
                        v
                Capability System
                        |
                        v
              Distributed ABI Layer
                        |
             +----------+----------+
             |                     |
             v                     v
      Consensus Boundary      Remote Calls
             |                     |
             +----------+----------+
                        |
                        v
                  Serialization
                        |
                        v
                 Service Discovery
                        |
                        v
                 Distributed Errors
                        |
                        v
                    Recovery
                        |
                        v
                   Conformance

This is the order I would use to prevent the repeated re-editing problem you specifically want to avoid.

Most importantly, consensus-boundaries.md should now be treated as the authoritative definition of the consensus/interoperability boundary. The later distributed-abi.md, remote-calls.md, serialization.md, and service-discovery.md documents should reference and implement this contract rather than independently inventing consensus semantics.