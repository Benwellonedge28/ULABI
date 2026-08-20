I read the current ULABI-DESIGN.md first, then checked the existing architecture documents and the repository tree so that scalability.md does not simply repeat the existing architecture/design-principles material.

The important finding is that the repository already contains substantial documents for the architecture, ABI, compatibility, distributed, hardware, security, etc., but several areas are still placeholders. In particular, docs/architecture/scalability.md is currently only a placeholder, while overview.md and design-principles.md are already substantial documents.

The master design establishes Minimal Core + Standard Profiles + Extensible Ecosystem, explicit semantics, platform/language/runtime neutrality, stable compatibility, capability limitation, determinism, and implementation independence.

So the scalability document below is deliberately about how ULABI scales in size, throughput, concurrency, data volume, topology, implementation count, and resource consumption without changing its fundamental contract.


---

PART 1 — docs/architecture/scalability.md

Copy everything inside the following block into:

docs/architecture/scalability.md

# ULABI Scalability Specification

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/architecture/scalability.md`  
**Specification Family:** Architecture  
**Specification Version:** 0.1.0-draft  
**Status:** Draft / Normative Architecture Specification  
**Normative Authority:** `ULABI-DESIGN.md` and `ULABI-SPEC.md`  
**Scope:** Scalability of ULABI interfaces, implementations, data, execution, resources, and interoperability topology  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the scalability requirements of ULABI.

ULABI SHALL remain usable when the size, number, throughput, concurrency, geographical distribution, or resource requirements of participating systems increase.

Scalability applies to:

- interface count;
- type count;
- function count;
- implementation count;
- application count;
- concurrent calls;
- call rate;
- data volume;
- message size;
- stream size;
- memory usage;
- resource usage;
- process count;
- machine count;
- distributed topology;
- hardware diversity;
- accelerator count;
- language diversity;
- version diversity;
- profile diversity.

This document does not redefine the ULABI Core.

It defines how existing ULABI contracts SHALL behave when systems grow.

---

# 2. Architectural Relationship

ULABI scalability follows:

> Stable Contract + Bounded Resources + Explicit Capability + Partitionable Work + Deterministic Semantics.

The scalability model SHALL preserve the existing ULABI architectural principles:

- language neutrality;
- runtime neutrality;
- platform neutrality;
- architecture neutrality;
- vendor neutrality;
- explicit semantics;
- stable interfaces;
- compatibility;
- capability limitation;
- deterministic boundary behavior;
- implementation independence.

Scalability SHALL NOT be achieved by silently changing the meaning of an interface.

---

# 3. Scope

This specification covers seven scalability dimensions.

```text
                    ULABI Scalability
                           |
       +-------------------+-------------------+
       |                   |                   |
   Structural           Execution           Resource
   Scalability          Scalability        Scalability
       |                   |                   |
       +-------------------+-------------------+
                           |
                    Data Scalability
                           |
                    Topology Scalability
                           |
                  Implementation Scalability
                           |
                    Evolution Scalability

The dimensions are:

1. Structural scalability


2. Execution scalability


3. Resource scalability


4. Data scalability


5. Topology scalability


6. Implementation scalability


7. Evolution scalability




---

4. Fundamental Scalability Invariant

An implementation MUST NOT require an unbounded resource merely because a valid ULABI interface is used.

For every potentially growing resource, an implementation SHALL define:

resource type;

unit;

limit;

current usage;

allocation policy;

failure behavior;

reclamation behavior;

observability;

optional scaling strategy.


Examples include:

memory;

stack;

heap;

handles;

concurrent calls;

streams;

queued messages;

serialized payloads;

worker threads;

connections;

transport buffers;

accelerator memory;

shared-memory regions.



---

5. No Implicit Infinite Resources

ULABI SHALL NOT assume:

infinite memory;

infinite CPU;

infinite storage;

infinite bandwidth;

infinite concurrency;

infinite recursion;

infinite message size;

infinite stream length;

infinite number of handles;

infinite number of interfaces;

infinite number of clients.


Every implementation operates under finite resources.

Therefore scalable ULABI behavior SHALL be based on explicit resource limits.


---

6. Resource Limit Model

A resource SHALL conceptually be represented as:

Resource
    |
    +-- Resource ID
    +-- Resource Type
    +-- Capacity
    +-- Current Usage
    +-- Reserved
    +-- Available
    +-- Policy
    +-- Owner
    +-- Capability
    +-- Lifetime
    +-- Failure Behavior

The implementation MAY represent this differently internally.

The externally observable semantics MUST remain compatible.


---

7. Resource Quotas

ULABI implementations SHOULD support quotas for:

memory;

CPU time;

wall-clock time;

stack depth;

concurrent operations;

open handles;

stream count;

message count;

message size;

bandwidth;

storage;

device access;

accelerator resources.


A quota SHALL be associated with an explicit scope.

Possible scopes include:

Interface
Function
Call
Session
Process
Component
Tenant
Device
Host
Profile
Application


---

8. Resource Accounting

Resource consumption SHOULD be measurable.

An implementation SHOULD be able to determine:

requested
allocated
consumed
reserved
released
failed

Resource accounting MUST NOT require exposing implementation-specific internal details that are irrelevant to the ULABI contract.


---

9. Structural Scalability

Structural scalability concerns the number of entities participating in ULABI.

Examples:

1 interface
10 interfaces
1,000 interfaces
1,000,000 interfaces

The same applies to:

functions;

types;

schemas;

profiles;

implementations;

consumers;

providers.


ULABI identifiers MUST remain unique without requiring centralized coordination for every entity.


---

10. Identifier Scalability

Identifiers SHALL be designed so that large ecosystems can avoid practical exhaustion and collision.

Identifiers SHOULD be:

stable;

machine-readable;

globally distinguishable;

version-independent where appropriate;

suitable for indexing;

suitable for caching;

suitable for distributed systems.


Human-readable names SHALL NOT be the sole interoperability identity.


---

11. Interface Registry Scalability

A registry containing ULABI interfaces SHOULD support:

partitioning;

indexing;

caching;

incremental synchronization;

version selection;

capability filtering;

profile filtering;

namespace separation.


A conforming implementation MUST NOT require loading every known interface into memory merely to use one interface.


---

12. Lazy Discovery

Implementations SHOULD support lazy discovery.

Conceptually:

Application
    |
Request Interface
    |
Lookup
    |
Load Metadata
    |
Validate Compatibility
    |
Bind
    |
Execute

Metadata that is not required for the current operation SHOULD NOT be required to be loaded.


---

13. Caching

ULABI metadata MAY be cached.

Cached metadata SHALL have:

identity;

version;

validity information;

provenance;

integrity information;

invalidation rules.


A cache MUST NOT cause an implementation to use an incompatible interface.


---

14. Cache Correctness

A cached interface definition is valid only when:

Identity
+
Version
+
Compatibility
+
Integrity
+
Policy

remain valid.

A cache MUST be invalidated or rejected when required compatibility conditions cease to hold.


---

15. Execution Scalability

Execution scalability concerns:

calls per second;

concurrent calls;

parallel execution;

asynchronous operations;

streaming;

long-running operations;

background operations;

distributed execution.


ULABI semantics MUST remain independent of the number of workers used internally.


---

16. Parallelism Independence

An implementation MAY execute independent ULABI operations:

sequentially;

concurrently;

in parallel;

remotely;

on accelerators.


However, the implementation MUST preserve the contract's declared semantics.

Parallel execution MUST NOT introduce observable behavior that violates:

ordering guarantees;

atomicity;

ownership;

lifetime;

determinism requirements;

synchronization requirements.



---

17. Concurrency Limits

An implementation SHOULD expose concurrency limits.

Examples:

max_concurrent_calls
max_pending_calls
max_streams
max_connections
max_workers
max_inflight_bytes

When a limit is reached, behavior MUST be explicit.

Possible outcomes include:

queue;

reject;

backpressure;

retryable failure;

cancellation;

escalation.


The implementation MUST NOT silently consume unlimited resources.


---

18. Backpressure

Streaming and asynchronous profiles SHALL support backpressure where applicable.

Conceptual model:

Producer
   |
   | data
   v
Buffer
   |
   v
Consumer

When the consumer cannot keep up:

Producer
   |
   v
Backpressure
   |
   +-- slow producer
   +-- bounded queue
   +-- temporary rejection
   +-- cancellation
   +-- spill to permitted storage

The chosen behavior MUST be part of the applicable contract.


---

19. Queue Bounds

Queues MUST have defined bounds or an explicitly authorized elastic policy.

An implementation MUST NOT allow unbounded queue growth merely because a consumer is unavailable.

Exhaustion SHALL result in deterministic failure or backpressure behavior.


---

20. Data Scalability

ULABI SHALL support data whose size exceeds practical single-buffer limits.

Examples include:

large files;

media;

datasets;

tensors;

model data;

database results;

logs;

scientific data;

continuous streams.


Large data SHOULD use:

streaming;

chunking;

segmented buffers;

bounded windows;

shared memory where explicitly permitted;

external resource handles.



---

21. Maximum Message Size

A transport or profile MAY impose a maximum message size.

The maximum SHALL be discoverable or explicitly specified.

Applications MUST NOT assume that every transport can carry arbitrarily large messages.


---

22. Chunking

Large payloads MAY be divided into chunks.

A chunked transfer SHALL preserve:

ordering where required;

completeness;

integrity;

type identity;

stream identity;

cancellation semantics;

ownership semantics.


A receiver MUST detect:

missing chunks;

duplicate chunks;

invalid chunks;

corrupted chunks;

unexpected termination.



---

23. Streaming

Streaming SHALL be preferred over enormous single allocations when data size may exceed safe memory limits.

A stream SHOULD expose:

Stream ID
Element Type
Chunk Size
Flow Control
Cancellation
Completion
Error
Ownership
Lifetime

A stream MAY be:

finite;

infinite;

bounded;

cancellable;

replayable;

non-replayable.


These properties MUST be explicit.


---

24. Infinite and Unbounded Streams

ULABI MAY support logically unbounded streams.

However, an implementation MUST NOT interpret an unbounded stream as permission for unlimited resource consumption.

Resource limits remain applicable.


---

25. Memory Scalability

ULABI memory behavior SHALL remain bounded by implementation policy.

Implementations SHOULD support:

bounded allocations;

incremental decoding;

streaming decoding;

lazy materialization;

memory mapping;

shared-memory profiles;

zero-copy profiles;

resource handles.



---

26. Allocation Safety

An implementation MUST validate externally supplied sizes before allocation.

It MUST defend against:

integer overflow;

multiplication overflow;

addition overflow;

excessive allocation;

recursive allocation;

nested container exhaustion;

malicious length fields.



---

27. Zero-Copy Scalability

Zero-copy operation MAY improve scalability.

However, zero-copy MUST NOT override:

ownership;

lifetime;

mutability;

isolation;

capability;

security;

memory validity.


A zero-copy optimization that violates the contract is non-conforming.


---

28. Shared Memory

Shared memory MAY be used for high-throughput interoperability.

Shared memory SHALL define:

region identity;

size;

ownership;

access rights;

synchronization;

lifetime;

invalidation;

cleanup;

isolation requirements.


Shared memory MUST NOT be assumed to be safe merely because it avoids copying.


---

29. Stack Scalability

Implementations SHOULD avoid requiring excessive stack growth for large or deeply nested ULABI values.

Large data SHOULD be represented through:

heap-backed structures;

streams;

handles;

bounded buffers.


A deeply nested value MUST NOT automatically imply unbounded native stack usage.


---

30. Recursion Limits

Where recursive structures or recursive calls are supported, implementations SHOULD define:

maximum depth;

maximum recursive memory;

detection behavior;

failure behavior.


Exceeding the limit MUST produce an explicit failure.


---

31. Distributed Scalability

Distributed ULABI operation MAY involve:

1 node
10 nodes
1,000 nodes
1,000,000 nodes

ULABI MUST NOT require a single centralized coordinator for every operation.

Distributed profiles SHOULD support:

partitioning;

locality;

discovery;

routing;

load distribution;

failure isolation;

retry policies;

timeout policies.



---

32. Locality-Aware Scaling

Implementations SHOULD prefer local resources when the contract permits.

Possible locality levels:

Same Call Context
Process
Host
Local Network
Remote Network
Region
Global

Changing locality MUST NOT silently change the semantic contract.


---

33. Distributed Failure Amplification

A scalable distributed implementation MUST prevent one failing component from exhausting resources throughout the system.

Protection mechanisms MAY include:

bounded retries;

circuit breakers;

rate limits;

concurrency limits;

deadlines;

bulkheads;

load shedding.


These belong to the appropriate runtime, reliability, security, or distributed profiles.


---

34. Retry Amplification

Retries MUST be bounded.

If:

Client retries
+
Service retries
+
Transport retries
+
Dependency retries

are all unlimited, a single failure can produce exponential load amplification.

Therefore each layer MUST have explicit retry limits or budgets.


---

35. Timeout and Deadline Scaling

Long-running operations SHOULD support deadlines.

A deadline SHOULD be propagated where supported.

Conceptually:

Original Deadline
       |
       +---- Adapter
       |
       +---- Runtime
       |
       +---- Transport
       |
       +---- Remote Service

A child operation MUST NOT silently extend an inherited deadline beyond policy.


---

36. Rate Limiting

ULABI implementations MAY enforce rate limits.

Rate limits MAY apply to:

calls;

bytes;

streams;

connections;

resource creation;

authentication attempts;

discovery;

remote operations.


Rate limiting MUST produce explicit behavior when exceeded.


---

37. Load Shedding

When an implementation cannot safely accept more work, it MAY shed load.

Load shedding SHOULD prefer:

1. reject new work;


2. preserve already accepted work;


3. preserve safety-critical work according to policy;


4. release resources cleanly.



An implementation MUST NOT corrupt accepted operations merely because new work cannot be accepted.


---

38. Multi-Tenant Scalability

A ULABI implementation MAY serve multiple tenants.

Tenant isolation SHOULD include:

resource quotas;

capability isolation;

identity isolation;

memory limits;

concurrency limits;

observability boundaries;

failure isolation.


One tenant MUST NOT automatically consume unlimited resources belonging to another tenant.


---

39. Fairness

Where multiple consumers compete for shared resources, implementations SHOULD support a defined scheduling policy.

Possible policies include:

FIFO;

weighted fairness;

priority;

quota-based fairness;

deadline-aware scheduling.


The policy MAY be implementation-specific unless the ULABI contract requires a specific guarantee.


---

40. Priority

Priority MAY be attached to operations where a relevant profile supports it.

Priority MUST NOT override:

security policy;

capability restrictions;

resource safety;

correctness;

mandatory isolation.


A high-priority operation does not obtain unlimited resources.


---

41. Implementation Scalability

ULABI SHALL support multiple independent implementations.

Scalability MUST NOT require:

one central runtime;

one compiler;

one vendor;

one registry;

one operating system;

one CPU architecture.


Independent implementations SHALL be able to implement compatible subsets of the specification.


---

42. Adapter Scalability

An ecosystem may contain:

Language A
Language B
Language C
Language D
...
Language N

Each implementation SHOULD use a reusable ULABI adapter architecture.

The ecosystem MUST NOT require every language to implement every other language directly.

Preferred topology:

Language A ----\
Language B -----\
Language C ------> ULABI
Language D -----/
Language N ----/

rather than:

A <-> B
A <-> C
A <-> D
B <-> C
B <-> D
C <-> D
...


---

43. Adapter Complexity

For N independently interoperating languages:

Direct pairwise interoperability tends toward:

N × (N - 1) / 2

pairwise relationships.

A shared ULABI contract instead allows each implementation to target the common contract.

ULABI therefore acts as an interoperability complexity reduction layer.


---

44. Profile Scalability

Profiles SHALL be composable where their semantics do not conflict.

Each profile MUST define:

profile ID;

profile version;

dependencies;

required Core version;

capabilities;

incompatibilities;

resource implications;

security requirements;

conformance tests.



---

45. Profile Dependency Graph

Profiles SHOULD form an explicit dependency graph.

Example:

Core
 |
 +-- Memory
 |
 +-- Async
 |     |
 |     +-- Streaming
 |
 +-- Security
 |     |
 |     +-- Capability
 |
 +-- Distributed
       |
       +-- Remote Calls

Cycles SHOULD be prohibited unless explicitly justified by the standard.


---

46. Profile Explosion Prevention

ULABI SHALL avoid creating a separate profile for every minor feature.

A new profile SHOULD only be created when the feature:

has distinct semantics;

has independent conformance requirements;

has meaningful deployment boundaries;

cannot be cleanly represented by an existing profile.


Small related features SHOULD remain within an existing profile.


---

47. Version Scalability

Version negotiation SHALL scale without requiring every implementation to know every future version.

Implementations SHOULD advertise:

supported Core versions;

supported profile versions;

optional capabilities;

required capabilities;

compatibility constraints.


Negotiation SHOULD determine the highest mutually compatible contract.


---

48. Forward Compatibility

Unknown optional fields, capabilities, and extensions SHOULD be safely ignorable when the relevant specification permits.

Unknown mandatory semantics MUST NOT be silently ignored.

The distinction between:

Unknown Optional

and:

Unknown Required

MUST remain explicit.


---

49. Backward Compatibility

New implementations SHOULD continue supporting older compatible contracts according to the compatibility rules.

Compatibility MUST be semantic rather than merely syntactic.

An interface that preserves the same function name but changes:

ownership;

error semantics;

lifetime;

capability requirements;

ordering;

side effects;


is not necessarily backward compatible.


---

50. Evolution Without Global Rebuild

Adding an interface, profile, type, or implementation MUST NOT require rebuilding unrelated implementations.

The architecture SHOULD support incremental evolution.

For example:

ULABI Core 1.x
       |
       +-- Profile A
       +-- Profile B
       +-- Profile C
       |
       +-- New Profile D

Adding Profile D SHOULD NOT invalidate A, B, or C.


---

51. Observability Scalability

Large ULABI deployments require scalable observability.

Implementations SHOULD expose aggregated metrics for:

calls;

failures;

latency;

throughput;

memory;

queue depth;

active streams;

resource exhaustion;

retries;

cancellations.


Telemetry SHOULD be bounded and policy-controlled.


---

52. Trace Scalability

Tracing SHOULD support:

sampling;

aggregation;

correlation IDs;

trace propagation;

bounded storage.


Tracing MUST NOT become an unbounded resource consumer.


---

53. Deterministic Diagnostics

Diagnostic information SHOULD remain structurally consistent across implementations.

Diagnostics SHOULD identify:

interface;

operation;

version;

profile;

failure;

resource condition;

locality;

relevant correlation identifier.


Human-readable text MAY differ between implementations.

Machine-readable diagnostic identity SHOULD remain stable.


---

54. Security and Scalability

Scalability MUST NOT weaken security.

An implementation MUST NOT automatically:

increase capabilities;

disable validation;

remove isolation;

bypass authentication;

bypass authorization;

increase resource limits indefinitely;


merely because load has increased.

Scaling policies MUST remain subject to security policy.


---

55. Capability-Aware Scaling

Scaling a service MAY require additional resources.

However:

More Capacity
    !=
More Capability

A component receiving additional compute capacity MUST NOT automatically receive additional permissions.

Capability grants remain independently controlled.


---

56. Resource Exhaustion

Resource exhaustion is a normal operating condition and MUST be handled explicitly.

Possible exhaustion states include:

MemoryExhausted
StackExhausted
QueueFull
ConcurrencyLimitReached
BandwidthExceeded
StorageExceeded
HandleLimitReached
StreamLimitReached
DeadlineExceeded
CPUQuotaExceeded

The exact error representation belongs to the error model.


---

57. Graceful Failure

When resources are unavailable, an implementation SHOULD:

1. detect the condition;


2. prevent unsafe allocation;


3. return a structured failure;


4. release resources already acquired where safe;


5. preserve unrelated operations;


6. expose diagnostic information;


7. permit recovery where supported.




---

58. Failure Isolation

A resource exhaustion event SHOULD remain within the smallest safe scope.

Preferred isolation:

Operation
   |
   X failure

rather than:

Entire Process
   |
   X failure

rather than:

Entire Host
   |
   X failure

The applicable runtime and reliability profiles define stronger isolation guarantees.


---

59. Elastic Scaling

An implementation MAY dynamically increase or decrease capacity.

Elastic scaling MAY apply to:

worker pools;

memory pools;

connection pools;

storage;

compute;

accelerator resources.


Elastic scaling MUST respect:

quotas;

capabilities;

security policy;

deadlines;

fairness;

cost/resource policy.



---

60. Autoscaling

Autoscaling is implementation or deployment behavior, not a mandatory ULABI Core feature.

An autoscaling system MAY use ULABI telemetry to make scaling decisions.

However, autoscaling MUST NOT alter the semantic meaning of the ULABI contract.


---

61. Horizontal Scaling

An implementation MAY replicate a provider:

ULABI Interface
                    |
        +-----------+-----------+
        |           |           |
    Provider A  Provider B  Provider C

Replicas MUST preserve the same declared semantics unless the interface explicitly permits different behavior.


---

62. Vertical Scaling

An implementation MAY increase:

CPU;

memory;

accelerator capacity;

storage;

bandwidth.


Vertical scaling MUST NOT silently modify interface semantics.


---

63. Sharding

Large datasets MAY be partitioned.

A sharded implementation SHOULD define:

partition identity;

routing;

ownership;

consistency;

failure handling;

rebalancing;

locality.


Sharding is not part of the ULABI Core unless explicitly standardized by a profile.


---

64. Partitioning

Scalable implementations SHOULD allow work to be partitioned when semantics permit.

Possible partitions include:

Data
Requests
Tenants
Interfaces
Functions
Devices
Geographic regions

Partitioning MUST preserve required ordering, atomicity, and consistency semantics.


---

65. Rebalancing

A distributed implementation MAY rebalance work.

Rebalancing MUST NOT violate:

ownership;

lifetime;

transaction semantics;

security boundaries;

capability restrictions.


In-flight operations MUST have explicitly defined migration behavior.


---

66. Accelerator Scalability

ULABI MAY support:

GPU;

NPU;

FPGA;

specialized accelerators;

future computational devices.


An implementation MAY scale across multiple accelerators.

Accelerator resource ownership and synchronization MUST remain explicit.


---

67. Heterogeneous Scaling

A scalable implementation MAY combine:

CPU
GPU
NPU
FPGA
Remote Compute
Embedded Device

The ULABI contract MUST remain independent of the scheduling strategy.


---

68. Embedded Scalability

ULABI SHALL support constrained implementations.

An embedded implementation MAY have:

fixed memory;

fixed stack;

no heap;

fixed concurrency;

no dynamic discovery;

no network;

no operating system.


A constrained implementation MAY implement a conforming subset according to its declared conformance profile.


---

69. Real-Time Scalability

Real-time profiles MAY define:

deadlines;

bounded latency;

priority;

deterministic allocation;

bounded queues;

predictable scheduling.


A general-purpose ULABI implementation MUST NOT claim real-time guarantees unless it satisfies the relevant profile.


---

70. Cloud Scalability

Cloud deployment MAY provide:

horizontal replication;

autoscaling;

distributed storage;

load balancing;

regional deployment.


These deployment properties MUST remain outside the Core unless standardized through profiles.


---

71. WebAssembly Scalability

WebAssembly-based implementations MAY use:

isolated instances;

shared memory where permitted;

host calls;

streaming;

worker pools.


ULABI semantics MUST remain independent of the WebAssembly runtime.


---

72. Persistence Scalability

Persistent resources MAY grow beyond memory capacity.

Implementations SHOULD support:

streaming;

pagination;

bounded reads;

incremental writes;

external storage handles.


A ULABI function MUST NOT imply that an entire persistent dataset must be materialized in memory.


---

73. Pagination

Large collections MAY use pagination.

A paginated interface SHOULD expose:

page identity;

page size;

continuation state;

ordering;

consistency semantics;

termination;

failure behavior.


Continuation state MUST be protected against tampering where security requires it.


---

74. Resource Lifecycle Scaling

Resource lifetimes MUST remain bounded unless explicitly declared otherwise.

Every scalable resource SHOULD have:

Create
Acquire
Use
Pause
Resume
Close
Release

Failure paths MUST define cleanup behavior.


---

75. Cancellation

Long-running operations SHOULD support cancellation when the operation semantics permit.

Cancellation MUST distinguish:

Requested
Accepted
Stopping
Cancelled
Completed
Failed

Cancellation MUST NOT imply that already committed external effects are automatically undone.


---

76. Idempotency and Scaling

Retry and replication depend on idempotency.

An operation SHOULD declare whether it is:

idempotent;

non-idempotent;

conditionally idempotent.


An implementation MUST NOT automatically retry non-idempotent operations unless the applicable policy explicitly permits it.


---

77. Scalability and Transactions

Transactions MAY span multiple operations.

Large-scale distributed transactions SHOULD NOT be assumed to have the same semantics as local transactions.

The relevant transaction or distributed profile MUST define:

scope;

isolation;

consistency;

commit;

rollback;

timeout;

failure behavior.



---

78. Compatibility at Scale

A large ecosystem may contain many versions simultaneously.

Implementations SHOULD support:

Provider
  |
  +-- Core 1.x
  +-- Core 2.x
  +-- Profile A v1
  +-- Profile A v2

without requiring every consumer to upgrade simultaneously.


---

79. Version Retirement

Old versions MAY eventually be retired.

Retirement SHOULD define:

retirement date;

supported replacement;

migration path;

compatibility window;

security status.


Retirement MUST NOT silently invalidate already-deployed systems without following the governance and versioning rules.


---

80. Scalability and Conformance

Conformance testing SHALL test resource boundaries.

Tests SHOULD include:

maximum payload;

minimum payload;

maximum nesting;

maximum concurrency;

queue exhaustion;

stream exhaustion;

cancellation;

timeout;

memory exhaustion;

handle exhaustion;

version negotiation;

profile negotiation;

large interface registries;

large type registries.



---

81. Stress Testing

ULABI implementations SHOULD be stress tested under increasing:

call rate
payload size
concurrency
stream count
interface count
type count
node count
resource pressure

The test suite SHOULD identify the point at which declared limits are reached.


---

82. Soak Testing

Long-running implementations SHOULD undergo soak testing.

Soak tests SHOULD detect:

memory leaks;

handle leaks;

queue growth;

resource fragmentation;

connection leaks;

state accumulation;

degraded latency;

telemetry growth.



---

83. Scalability Test Model

A scalability test SHOULD record:

Workload
Resource Limit
Initial Usage
Peak Usage
Throughput
Latency
Failure Rate
Recovery Time
Resource Reclamation
Final Usage


---

84. Complexity Requirements

Implementations SHOULD document the expected complexity of important operations.

Examples:

Interface lookup: O(log n) or better
Type lookup: implementation-defined
Function dispatch: implementation-defined
Registry synchronization: implementation-defined

ULABI SHALL NOT mandate one algorithm.

However, an implementation claiming scalable deployment SHOULD provide evidence that critical operations do not degrade catastrophically with ecosystem size.


---

85. No Accidental Global State

A scalable implementation SHOULD avoid global mutable state where that state creates:

contention;

single points of failure;

unbounded memory growth;

synchronization bottlenecks.


Global registries MAY exist where necessary, but SHOULD be partitionable or cacheable.


---

86. Hot Path Requirements

Critical execution paths SHOULD avoid:

unnecessary allocations;

global locks;

repeated schema parsing;

repeated capability resolution;

repeated version negotiation;

unnecessary serialization.


Caching and optimization MUST preserve ULABI semantics.


---

87. Cold Path Separation

Operations such as:

discovery;

metadata loading;

validation;

compatibility analysis;

adapter generation;


SHOULD be separable from the hot execution path.

This allows implementations to optimize repeated calls without weakening validation.


---

88. Precomputation

An implementation MAY precompute:

dispatch tables;

schema validators;

type layouts;

compatibility results;

capability checks;

serialization plans.


Precomputation MUST NOT bypass runtime checks whose conditions may change.


---

89. Incremental Processing

Where possible, implementations SHOULD process large inputs incrementally rather than requiring complete materialization.

This applies to:

serialization;

deserialization;

streams;

files;

logs;

datasets;

tensors.



---

90. Bounded Work

An implementation MUST NOT convert a bounded ULABI request into unbounded hidden work.

For example:

Small Request
    |
    X
    |
Unbounded recursive processing

is prohibited unless the interface explicitly declares such behavior.


---

91. Denial-of-Service Resistance

Scalable implementations MUST consider resource exhaustion attacks.

Potential attacks include:

enormous payloads;

deeply nested values;

excessive streams;

excessive connections;

repeated discovery;

registry flooding;

retry amplification;

decompression bombs;

pathological schemas.


Security profiles SHALL define stronger requirements where necessary.


---

92. Compression and Scaling

Compression MAY reduce bandwidth requirements.

However, implementations MUST protect against:

decompression bombs;

excessive expansion;

CPU exhaustion;

memory exhaustion.


Compressed size MUST NOT be treated as equivalent to decompressed resource requirements.


---

93. Serialization Scalability

Serialization SHOULD support:

bounded encoding;

streaming;

incremental decoding;

canonical representation;

schema-aware validation.


A decoder MUST NOT allocate based on untrusted lengths without validation.


---

94. Capability Discovery Scalability

Capability discovery SHOULD be:

incremental;

cacheable;

scoped;

authenticated where required.


A client SHOULD NOT need to retrieve the complete capability set of a large system merely to invoke one operation.


---

95. Multi-Region Scalability

Distributed deployments MAY span regions.

The implementation SHOULD expose locality and consistency semantics when they materially affect behavior.

A remote region MUST NOT be treated as equivalent to local execution merely because both implement ULABI.


---

96. Global Namespace Scalability

A global ULABI ecosystem SHOULD support namespace partitioning.

Possible organization:

Namespace
   |
   +-- Organization
       |
       +-- Interface
           |
           +-- Version

Namespace policy belongs to the governance and identity specifications.


---

97. Governance Scalability

ULABI governance itself MUST scale.

The standard SHOULD permit:

independent contributors;

independent implementations;

independent test suites;

independent registries;

independent profile development.


A single implementation MUST NOT become the de facto authority merely because it is widely used.


---

98. Implementation Diversity

A scalable standard MUST permit implementations optimized for different environments:

Microcontroller
Desktop
Mobile
Server
Cloud
HPC
WebAssembly
GPU
Embedded
Safety-Critical
Distributed

Conformance SHALL be based on contract compliance, not implementation size.


---

99. Minimal Implementation

A minimal ULABI implementation SHOULD be possible.

It SHOULD implement only:

required Core semantics;

required types;

required call model;

required errors;

required validation;

required compatibility behavior.


Advanced scaling features MAY be omitted unless claimed by the corresponding profile.


---

100. Large Implementation

A large implementation MAY provide:

distributed execution;

high concurrency;

zero-copy;

shared memory;

accelerators;

streaming;

autoscaling;

observability;

advanced security.


Additional capabilities MUST NOT change Core semantics.


---

101. Scalability Profiles

ULABI SHOULD eventually define explicit scalability profiles.

Possible profiles:

ULABI-Embedded
ULABI-Minimal
ULABI-General
ULABI-HighThroughput
ULABI-RealTime
ULABI-Streaming
ULABI-Distributed
ULABI-HPC
ULABI-Accelerator
ULABI-SafetyCritical

Profiles SHALL define their own:

limits;

guarantees;

required features;

conformance tests.



---

102. Profile Selection

An implementation SHOULD advertise the profiles it supports.

Example:

Core: 1.x

Profiles:
    General
    Streaming
    Security
    Distributed

Capabilities:
    Async
    Cancellation
    Backpressure
    ZeroCopy

A consumer MUST NOT assume unsupported profiles.


---

103. Graceful Degradation

When a stronger scalability feature is unavailable, an implementation MAY fall back to a compatible weaker mechanism.

Examples:

ZeroCopy
    |
    X unavailable
    |
Copy

SharedMemory
    |
    X unavailable
    |
Serialized Transfer

Fallback MUST preserve semantics.

Performance characteristics may differ.


---

104. Scalability Does Not Mean Unlimited Performance

ULABI SHALL NOT define "scalable" as:

> Unlimited performance.



Instead:

> A scalable implementation increases supported workload while preserving correctness, safety, compatibility, and explicit resource behavior.




---

105. Required Invariants

The following invariants are normative.

INV-SCALE-001

No resource is implicitly infinite.

INV-SCALE-002

Resource limits MUST have explicit failure behavior.

INV-SCALE-003

Scaling MUST NOT silently change semantic behavior.

INV-SCALE-004

Scaling MUST NOT silently increase capabilities.

INV-SCALE-005

Large data MUST be representable without requiring unbounded memory.

INV-SCALE-006

Concurrency MUST remain bounded by policy.

INV-SCALE-007

Retries MUST be bounded.

INV-SCALE-008

Queue growth MUST be bounded or explicitly controlled.

INV-SCALE-009

Remote execution MUST remain distinguishable from local execution where semantics differ.

INV-SCALE-010

Profile composition MUST NOT redefine Core semantics.

INV-SCALE-011

Independent implementations MUST remain possible.

INV-SCALE-012

Resource exhaustion MUST fail safely.

INV-SCALE-013

Optimizations MUST preserve ULABI semantics.

INV-SCALE-014

Security restrictions MUST remain active under load.

INV-SCALE-015

Scalability mechanisms MUST be testable.


---

106. Failure Model

Scalability-related failures include:

ResourceExhausted
ConcurrencyExceeded
QueueFull
PayloadTooLarge
StreamLimitExceeded
MemoryLimitExceeded
StackLimitExceeded
CPUQuotaExceeded
BandwidthLimitExceeded
StorageLimitExceeded
DeadlineExceeded
RateLimitExceeded
CapabilityLimitExceeded
PartitionUnavailable
ServiceOverloaded

These SHALL be represented using the ULABI error model.


---

107. Recovery

Recovery behavior SHALL depend on the failure.

Possible responses include:

Retry
Backoff
ReduceConcurrency
ReduceBatchSize
ResumeStream
RestartOperation
FailOperation
FailPartition
Escalate

Recovery MUST NOT exceed authorized resource or capability limits.


---

108. Observability Requirements

A scalable implementation SHOULD expose:

current resource utilization;

resource limits;

active operations;

queue depth;

stream count;

error rates;

latency;

throughput;

rejected work;

retries;

cancellations.


Sensitive information MUST be protected according to the security model.


---

109. Conformance Requirements

An implementation claiming a scalability profile SHALL demonstrate:

1. bounded resource behavior;


2. explicit limit enforcement;


3. safe exhaustion behavior;


4. large payload handling;


5. concurrency handling;


6. cancellation behavior where supported;


7. compatibility behavior;


8. profile negotiation;


9. security preservation;


10. resource reclamation.




---

110. Required Conformance Test Categories

The conformance suite SHALL eventually include:

SCAL-001 Resource Limits
SCAL-002 Memory Exhaustion
SCAL-003 Queue Exhaustion
SCAL-004 Concurrency Limits
SCAL-005 Large Payload
SCAL-006 Chunked Transfer
SCAL-007 Streaming
SCAL-008 Backpressure
SCAL-009 Cancellation
SCAL-010 Deadline
SCAL-011 Retry Bounds
SCAL-012 Capability Preservation
SCAL-013 Version Negotiation
SCAL-014 Profile Negotiation
SCAL-015 Registry Scaling
SCAL-016 Type Registry Scaling
SCAL-017 Interface Registry Scaling
SCAL-018 Distributed Scaling
SCAL-019 Failure Isolation
SCAL-020 Resource Reclamation
SCAL-021 Security Under Load
SCAL-022 Long-Running Stability
SCAL-023 Fragmentation
SCAL-024 Graceful Degradation
SCAL-025 Implementation Independence


---

111. Reference Implementation Requirements

A reference implementation SHOULD demonstrate at least:

bounded memory;

bounded concurrency;

streaming;

backpressure;

cancellation;

resource accounting;

profile negotiation;

compatibility negotiation;

structured exhaustion errors;

deterministic resource cleanup.


The reference implementation MUST NOT become normative merely because it is the reference implementation.

The specification remains authoritative.


---

112. Integration With Other Specifications

This document intentionally does not redefine other ULABI contracts.

It integrates with:

Core ABI

docs/abi/core-abi.md

Defines the fundamental ABI contract whose scalability is governed here.

Calling Convention

docs/abi/calling-convention.md

Defines concrete call-boundary behavior.

This document adds concurrency, batching, resource and throughput requirements without redefining calling semantics.

Data Types

docs/abi/data-types.md

Defines semantic boundary types.

This document defines how large and repeated values are handled at scale.

Memory Model

docs/abi/memory-model.md

Defines memory ownership and boundary behavior.

This document defines scalable allocation and resource-limit behavior.

Stack Model

docs/abi/stack-model.md

Defines stack semantics.

This document defines scalable stack-depth and recursion requirements.

Register Model

docs/abi/register-model.md

Defines register abstraction.

This document does not redefine registers.

Return Values

docs/abi/return-values.md

Defines result transport.

Scalability-related errors MUST use the defined return/error semantics.

Exception Model

docs/abi/exception-model.md

Defines exception/error propagation.

Resource exhaustion and scalability failures MUST use that model.

Runtime Interface

docs/runtime/runtime-interface.md

Defines runtime integration.

Runtime implementations SHALL enforce scalability policies.

Async Model

docs/runtime/async-model.md

Defines asynchronous execution.

This document adds bounded concurrency, queues, cancellation, and backpressure requirements.

Streaming

docs/distributed/serialization.md and future streaming specification

Large-data transfer SHALL use the appropriate streaming/serialization contracts.

Security

docs/security/security-model.md

Defines security boundaries.

Scalability MUST NOT weaken security.

Capability Security

docs/security/capability-security.md

Defines capability semantics.

Scaling capacity MUST NOT automatically expand capabilities.

Distributed ABI

docs/distributed/distributed-abi.md

Defines distributed execution.

This document defines scalable resource and topology behavior.

Remote Calls

docs/distributed/remote-calls.md

Defines remote invocation.

This document adds bounded retries, deadlines, load handling and locality-aware scaling.

Serialization

docs/distributed/serialization.md

Defines serialization.

This document adds large-payload and incremental-processing requirements.

Feature Negotiation

docs/compatibility/feature-negotiation.md

Defines negotiation.

Scalability profiles SHALL use the same negotiation mechanism.

Capability Discovery

docs/compatibility/capability-discovery.md

Defines discovery.

Scalability requires discovery to remain incremental and cacheable.

Backward Compatibility

docs/compatibility/backwards-compatibility.md

Defines compatibility guarantees.

Scaling mechanisms MUST preserve them.

Graceful Degradation

docs/compatibility/graceful-degradation.md

Defines fallback behavior.

Scalable optimizations MAY fall back where semantics remain compatible.

Self-Healing

docs/reliability/self-healing.md

Defines bounded recovery.

Scalability failures MAY trigger recovery only through authorized policies.

Conformance

docs/standards/conformance.md

Defines conformance methodology.

This document supplies scalability-specific requirements.

Test Suite

docs/standards/test-suite.md

Defines common test infrastructure.

The SCAL test categories defined above SHALL be integrated into that suite.


---

113. Integration Contract

This document is complete without requiring future edits merely because another specification is later implemented.

Other specifications SHALL integrate with the scalability contract by referencing the invariants and requirements defined here.

They MUST NOT redefine:

what resource exhaustion means;

the principle that resources are bounded;

the prohibition on silent capability expansion;

the requirement for explicit failure behavior.


If another specification needs stronger limits or guarantees, it SHALL define them as a profile-specific requirement.


---

114. Implementation Independence

This specification does not require a particular:

programming language;

compiler;

operating system;

CPU;

runtime;

scheduler;

allocator;

transport;

database;

cloud provider;

hardware vendor.


Two implementations may use completely different scaling mechanisms while remaining conformant.


---

115. Security Requirements

Scaling implementations MUST:

enforce capability boundaries;

validate untrusted sizes;

enforce resource quotas;

prevent retry amplification;

prevent unbounded allocation;

prevent queue exhaustion attacks;

preserve authentication and authorization;

preserve isolation;

prevent telemetry from becoming an uncontrolled resource consumer.


Performance optimization MUST NOT bypass security validation.


---

116. Determinism Requirements

Where ULABI declares deterministic behavior, scaling MUST preserve it.

Parallel execution MUST NOT change deterministic results.

Distributed execution MUST NOT silently introduce nondeterminism into an operation declared deterministic.


---

117. Future Extensions

Future scalability work MAY define:

adaptive batching;

distributed load balancing;

federated registries;

accelerator pools;

NUMA-aware execution;

heterogeneous scheduling;

large-scale tensor streaming;

geographically distributed execution;

edge-to-cloud scaling;

energy-aware execution;

carbon-aware execution;

quantum-resource scheduling.


Such features SHOULD be introduced as profiles rather than enlarging the Core unnecessarily.


---

118. Final Principle

ULABI scalability is governed by one central rule:

> ULABI SHALL scale by adding capacity, partitioning work, bounding resources, and composing profiles—not by weakening semantics or creating hidden behavior.



A system may become:

larger;

faster;

more concurrent;

more distributed;

more heterogeneous;

more resource constrained;

more geographically distributed;


while remaining the same ULABI contract.

That is the required meaning of scalability in ULABI.

---

# PART 2 — Why this document does not duplicate the existing files

The current repository already has substantial `overview.md` and `design-principles.md` documents, so I intentionally did **not** make `scalability.md` another 50-page statement of language neutrality, Core/profile architecture, or general design philosophy. The existing architecture already establishes those points. 

The current architecture directory contains:

- `design-principles.md`
- `overview.md`
- `scalability.md`
- `universal-model.md`

with `scalability.md` currently only a placeholder. 

Likewise, the repository already has substantial ABI specifications including:

- `calling-convention.md`
- `core-abi.md`
- `data-types.md`
- `exception-model.md`
- `memory-model.md`
- `register-model.md`
- `return-values.md`
- `stack-model.md`. 

So **do not rewrite those merely because scalability references them**.

---

# PART 3 — Required file/work order

Your requirement that each file be independently finishable is important.

I recommend this dependency order.

## Phase 0 — foundation documents

These should establish rules that later specifications can consume without modifying them.

| Order | File | What must be completed |
|---|---|---|
| 0.1 | `ULABI-DESIGN.md` | Master architecture |
| 0.2 | `ULABI-SPEC.md` | Normative Core |
| 0.3 | `ULABI-VERSIONING.md` | Version model |
| 0.4 | `ULABI-GOVERNANCE.md` | Governance |
| 0.5 | `docs/architecture/design-principles.md` | Design principles |
| 0.6 | `docs/architecture/overview.md` | Architecture overview |
| **0.7** | **`docs/architecture/scalability.md`** | **Scalability contract — the file above** |
| 0.8 | `docs/architecture/universal-model.md` | Universal computational model |

`scalability.md` can now be completed without waiting for the later implementation files.

---

# Phase 1 — Core ABI

These define the fundamental ABI contract.

1. `docs/abi/core-abi.md`
2. `docs/abi/data-types.md`
3. `docs/abi/calling-convention.md`
4. `docs/abi/return-values.md`
5. `docs/abi/exception-model.md`
6. `docs/abi/memory-model.md`
7. `docs/abi/stack-model.md`
8. `docs/abi/register-model.md`

Each should contain:

```text
Purpose
Scope
Terminology
Normative requirements
Interfaces
Data structures
Invariants
Security
Failure modes
Recovery
Compatibility
Scalability
Conformance tests
Reference implementation contract
Integration contract

That is important because otherwise later documents will force earlier files to be rewritten.


---

Phase 2 — Type system

Required files:

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/type-compatibility.md

They should consume data-types.md rather than redefine it.


---

Phase 3 — Interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

The important architecture is:

Language A
    |
 Adapter
    |
  ULABI
    |
 Adapter
    |
Language B

rather than language-to-language coupling.


---

Phase 4 — Runtime

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

These consume the scalability/resource contracts rather than inventing independent resource semantics.


---

Phase 5 — Memory

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

These should consume:

Core ABI

memory model

scalability

security


without redefining the universal resource-limit philosophy.


---

Phase 6 — Security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Critical integration rule:

> More resources never automatically means more authority.



That rule is now established in scalability.md.


---

Phase 7 — Reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md
docs/reliability/self-healing.md

self-healing.md should consume the scalability failure model instead of creating a separate definition of resource exhaustion.


---

Phase 8 — Distributed system

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

These consume:

scalability;

runtime;

errors;

compatibility;

security.


They should not redefine local ABI semantics.


---

Phase 9 — Compatibility

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

This is where the scalability profile negotiation model becomes operational.


---

Phase 10 — Platform/hardware profiles

docs/platforms/operating-systems.md
docs/platforms/architectures.md
docs/platforms/embedded.md
docs/platforms/mobile.md
docs/platforms/desktop.md
docs/platforms/server.md
docs/platforms/cloud.md
docs/platforms/webassembly.md
docs/platforms/accelerators.md

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md
docs/hardware/future-architectures.md

These should be profiles/adapters, not alternate ULABI definitions.


---

Phase 11 — Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md

The validator becomes particularly important because it can check:

ABI
Types
Memory
Security
Compatibility
Scalability
Conformance


---

Phase 12 — Observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md

These consume the scalability telemetry requirements rather than creating their own resource model.


---

Phase 13 — Standards

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

This is where the actual claim:

> "ULABI compatible"



becomes measurable.


---

PART 4 — Required schemas

The repository should eventually have machine-readable schemas rather than relying exclusively on Markdown.

Recommended:

schemas/
├── interface.schema.json
├── function.schema.json
├── type.schema.json
├── error.schema.json
├── profile.schema.json
├── capability.schema.json
├── version.schema.json
├── resource.schema.json
├── stream.schema.json
├── compatibility.schema.json
├── conformance.schema.json
├── diagnostic.schema.json
└── scalability.schema.json

scalability.schema.json should validate concepts such as:

resource limits
concurrency limits
payload limits
stream limits
timeouts
quotas
profiles
scaling capabilities


---

PART 5 — Required code architecture

One important correction to the current repository direction:

ULABI does not currently have a mature implementation tree.

The repository tree currently contains the specification/documentation foundation rather than a completed implementations/ source tree. The existing tree already contains the ABI and architecture documents, but the implementation layer should be added deliberately rather than pretending that a specific language is already ULABI's implementation language.

I recommend this implementation architecture:

implementations/
│
├── core/
│   ├── identity/
│   ├── types/
│   ├── interfaces/
│   ├── calls/
│   ├── errors/
│   ├── encoding/
│   ├── validation/
│   ├── compatibility/
│   └── capabilities/
│
├── memory/
│   ├── ownership/
│   ├── lifetime/
│   ├── allocation/
│   ├── buffers/
│   └── shared-memory/
│
├── runtime/
│   ├── process/
│   ├── threading/
│   ├── async/
│   ├── scheduling/
│   ├── resources/
│   └── cancellation/
│
├── transport/
│   ├── in-process/
│   ├── ipc/
│   ├── shared-memory/
│   ├── network/
│   └── wasm/
│
├── serialization/
│   ├── encoder/
│   ├── decoder/
│   ├── canonical/
│   ├── streaming/
│   └── validation/
│
├── security/
│   ├── identity/
│   ├── authentication/
│   ├── authorization/
│   ├── capabilities/
│   ├── sandbox/
│   └── policy/
│
├── distributed/
│   ├── discovery/
│   ├── routing/
│   ├── remote-calls/
│   ├── retry/
│   ├── deadlines/
│   └── load-balancing/
│
├── reliability/
│   ├── detection/
│   ├── isolation/
│   ├── recovery/
│   ├── rollback/
│   └── health/
│
├── scalability/
│   ├── quotas/
│   ├── rate-limits/
│   ├── concurrency/
│   ├── backpressure/
│   ├── batching/
│   ├── streaming/
│   ├── partitioning/
│   ├── sharding/
│   └── resource-accounting/
│
└── adapters/
    ├── c/
    ├── cpp/
    ├── rust/
    ├── go/
    ├── python/
    ├── java/
    ├── csharp/
    ├── swift/
    ├── kotlin/
    ├── fortran/
    ├── ada/
    ├── wasm/
    └── reference/

The important point is that the adapters depend on ULABI; ULABI does not depend on the adapters.


---

PART 6 — Exact required code modules

The initial reference implementation should eventually contain these conceptual modules.

Core

ulabi_identity
ulabi_types
ulabi_interface
ulabi_function
ulabi_call
ulabi_error
ulabi_result
ulabi_encoding
ulabi_validation
ulabi_version
ulabi_compatibility
ulabi_capability

Memory

ulabi_memory
ulabi_ownership
ulabi_lifetime
ulabi_allocator
ulabi_buffer
ulabi_shared_memory
ulabi_handle

Runtime

ulabi_runtime
ulabi_process
ulabi_thread
ulabi_scheduler
ulabi_async
ulabi_future
ulabi_cancellation
ulabi_resource

Serialization

ulabi_encoder
ulabi_decoder
ulabi_canonical
ulabi_stream_encoder
ulabi_stream_decoder
ulabi_schema_validator

Transport

ulabi_transport
ulabi_inprocess
ulabi_ipc
ulabi_shared_memory_transport
ulabi_network_transport
ulabi_wasm_transport

Security

ulabi_identity_security
ulabi_authentication
ulabi_authorization
ulabi_capability
ulabi_sandbox
ulabi_security_policy

Distributed

ulabi_discovery
ulabi_remote_call
ulabi_router
ulabi_retry
ulabi_deadline
ulabi_load_balancer
ulabi_partition

Scalability

ulabi_resource_quota
ulabi_resource_accounting
ulabi_rate_limit
ulabi_concurrency_limit
ulabi_backpressure
ulabi_batch
ulabi_stream
ulabi_sharding
ulabi_scaling_policy

Reliability

ulabi_fault_detector
ulabi_fault_isolation
ulabi_recovery
ulabi_rollback
ulabi_health

Tooling

ulabi_validator
ulabi_abi_inspector
ulabi_compatibility_checker
ulabi_schema_compiler
ulabi_interface_registry
ulabi_profile_registry


---

PART 7 — Tests that must eventually exist

The test tree should mirror the specification rather than becoming one giant test directory.

tests/
├── core/
│   ├── identity/
│   ├── types/
│   ├── calls/
│   ├── errors/
│   ├── encoding/
│   └── compatibility/
│
├── abi/
│   ├── calling/
│   ├── memory/
│   ├── stack/
│   ├── registers/
│   ├── exceptions/
│   └── returns/
│
├── interoperability/
├── runtime/
├── memory/
├── security/
├── reliability/
├── distributed/
├── scalability/
│   ├── resource_limits/
│   ├── concurrency/
│   ├── queues/
│   ├── streaming/
│   ├── backpressure/
│   ├── large_payloads/
│   ├── cancellation/
│   ├── retries/
│   ├── sharding/
│   └── long_running/
│
├── compatibility/
├── platforms/
└── conformance/


---

PART 8 — Conformance architecture

Eventually:

conformance/
├── core/
├── types/
├── abi/
├── memory/
├── ffi/
├── runtime/
├── async/
├── security/
├── distributed/
├── scalability/
├── reliability/
└── profiles/

The scalability suite should specifically test:

ULABI Core          ✓
ULABI Types         ✓
ULABI Memory        ✓
ULABI FFI           ✓
ULABI Async         ✓
ULABI Security      ✓
ULABI Distributed   ✓
ULABI Scalability   ✓
ULABI Reliability   ✓

rather than allowing an implementation to claim compatibility based merely on successful sample calls.


---

PART 9 — Critical integration rule for the entire repository

From this point onward, I recommend that every ULABI specification file contain an "Integration Contract" section.

It should say:

1. what this file defines;


2. what it consumes;


3. what it owns;


4. what it explicitly does not define;


5. which later files consume it;


6. which invariants must remain stable;


7. which schemas correspond to it;


8. which tests correspond to it;


9. which implementation modules implement it.



That directly satisfies your requirement:

> When one file is finished, later files should not force us to reopen it merely to add missing integration information.



The existing overview.md already follows much of this philosophy, explicitly separating semantic call contracts, calling convention, platform adapters, memory architecture, error architecture, compatibility, and profiles.


---

Recommended next implementation order

Now that scalability.md is defined, I would not immediately create more random documents.

The clean order is:

1. ULABI-DESIGN.md
2. ULABI-SPEC.md
3. ULABI-VERSIONING.md
4. architecture/design-principles.md
5. architecture/overview.md
6. architecture/scalability.md        ← THIS FILE
7. architecture/universal-model.md

8. abi/core-abi.md
9. abi/data-types.md
10. abi/calling-convention.md
11. abi/return-values.md
12. abi/exception-model.md
13. abi/memory-model.md
14. abi/stack-model.md
15. abi/register-model.md

16. type-system/*
17. interoperability/*
18. runtime/*
19. memory/*
20. security/*
21. reliability/*
22. distributed/*
23. compatibility/*
24. platforms/*
25. hardware/*
26. tooling/*
27. observability/*
28. standards/*

29. schemas/*
30. tests/*
31. conformance/*
32. reference/*
33. implementations/*
34. language adapters/*

Do not rewrite the already substantial ABI documents just to make scalability.md fit them. Instead, when we next review each ABI document, its own integration contract should be made complete. This avoids the exact repeated-edit problem you are trying to eliminate.

