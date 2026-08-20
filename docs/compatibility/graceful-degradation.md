I read ULABI-DESIGN.md first, then checked the existing compatibility documents and the related architecture/ABI documents before defining this file. The repository currently establishes the Minimal Core + Standard Profiles + Extensible Ecosystem architecture, explicit semantics, versioning, capability discovery, negotiation, compatibility, language/runtime/platform neutrality, and implementation independence.

I also checked the compatibility family. backwards-compatibility.md and forwards-compatibility.md already contain substantial specifications, while graceful-degradation.md is still a placeholder.    The existing feature-negotiation design explicitly separates capability discovery, compatibility analysis, negotiation, and graceful degradation, so the document below deliberately does not redefine those mechanisms.

docs/compatibility/graceful-degradation.md

Copy the following entire document into that file:

# ULABI Graceful Degradation

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/compatibility/graceful-degradation.md`  
**Specification Family:** Compatibility / Reliability / Resilience  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`, `ULABI-VERSIONING.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI graceful-degradation contract.

Graceful degradation defines how a ULABI implementation behaves when the
preferred or full interoperability contract cannot be provided but a
reduced contract can still operate safely and correctly.

The fundamental principle is:

> When full functionality is unavailable, ULABI MAY operate with a
> deliberately reduced contract, but MUST NOT silently violate semantic,
> security, safety, integrity, ownership, or correctness requirements.

Graceful degradation is therefore not:

- silently ignoring failures;
- silently changing semantics;
- silently reducing security;
- silently dropping required data;
- silently changing locality;
- silently changing ownership;
- silently enabling unsafe fallbacks;
- pretending unsupported functionality exists.

Instead:

```text
Full Contract Available
        |
        v
    Use Full Mode

Full Contract Unavailable
        |
        v
Determine Safe Reduced Contract
        |
        +-------------------+
        |                   |
      Safe                Unsafe
        |                   |
        v                   v
  Degrade Safely          Reject


---

2. Scope

This specification governs graceful degradation for:

unavailable optional features;

unavailable profiles;

unsupported versions;

unavailable transports;

unavailable accelerators;

unavailable execution modes;

unavailable resource classes;

reduced hardware capabilities;

reduced memory availability;

reduced concurrency;

unavailable optimization features;

reduced streaming capabilities;

reduced distributed capabilities;

temporary subsystem failures;

partial service availability;

capability loss;

negotiated fallback;

compatibility fallback;

local fallback;

bounded performance degradation;

safe rejection when degradation is impossible.


This document does NOT redefine:

backward compatibility;

forward compatibility;

capability discovery;

feature negotiation;

security architecture;

authorization;

memory ownership;

calling conventions;

transport protocols;

self-healing;

distributed consensus;

certification.


Those specifications remain authoritative for their respective domains.


---

3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



Graceful degradation MUST operate inside that architecture.

The Core MUST NOT be weakened merely to enable degradation.

Optional functionality SHOULD be implemented as profiles or extensions so that an implementation can determine whether a reduced profile remains valid.

The compatibility architecture is:

ULABI-DESIGN.md
                           |
                    ULABI-SPEC.md
                           |
                  ULABI-VERSIONING.md
                           |
             +-------------+-------------+
             |             |             |
        Backward       Forward       Capability
      Compatibility  Compatibility    Discovery
             |             |             |
             +-------------+-------------+
                           |
                  Feature Negotiation
                           |
                  Compatibility Result
                           |
                 Graceful Degradation
                           |
             +-------------+-------------+
             |                           |
        Reduced Contract             Reject
             |
             v
         Safe Execution

Graceful degradation consumes compatibility and negotiation results.

It MUST NOT invent compatibility.


---

4. Normative Language

The following terms are normative:

MUST

MUST NOT

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY

OPTIONAL


A conforming implementation MUST satisfy every applicable MUST and MUST NOT requirement.


---

5. Fundamental Principle

ULABI distinguishes:

Available
Supported
Compatible
Authorized
Negotiated
Selected
Enabled
Required
Optional

These concepts MUST NOT be treated as equivalent.

In particular:

Unsupported
    !=
Degraded

Unavailable
    !=
Optional

Optional
    !=
Safe to Disable

Compatible
    !=
Authorized

Preferred
    !=
Required

A feature may be unavailable while a safe reduced contract remains possible.

A feature may also be unavailable while no safe reduced contract exists.


---

6. Degradation Outcomes

ULABI defines four primary degradation outcomes:

1. Full Operation


2. Reduced Operation


3. Safe Fallback


4. Rejection



6.1 Full Operation

The complete negotiated contract is available.

Requested Contract
        |
        v
All Requirements Satisfied
        |
        v
Full Operation

No degradation occurs.


---

6.2 Reduced Operation

One or more optional capabilities are removed while the remaining contract continues to satisfy all mandatory requirements.

Example:

Requested:

Core
Async
Streaming
GPU
Zero-Copy

Available:

Core
Async
Streaming

Result:

Core
Async
Streaming

GPU and zero-copy are omitted.

The resulting contract MUST remain valid.


---

6.3 Safe Fallback

The preferred mechanism is unavailable but another explicitly defined mechanism provides equivalent required semantics.

Example:

Preferred:
GPU acceleration

Fallback:
CPU execution

The fallback is valid only if the CPU execution preserves the required semantic and security contract.

Performance differences alone do not necessarily constitute incompatibility.

Semantic differences do.


---

6.4 Rejection

No safe reduced contract exists.

The implementation MUST reject the operation or contract.

Requested
   |
Unavailable
   |
No Safe Fallback
   |
   v
Reject

Rejection is preferable to unsafe degradation.


---

7. Degradation Safety Invariant

A conforming implementation MUST preserve all mandatory contract properties during degradation.

Mandatory properties include, where applicable:

semantic correctness;

type correctness;

ownership;

lifetime;

integrity;

confidentiality;

authorization;

authentication;

isolation;

capability restrictions;

safety requirements;

determinism requirements;

resource limits;

data validity;

error semantics.


A degraded implementation MUST NOT sacrifice these properties merely to continue execution.


---

8. Mandatory vs Optional Functionality

Every degradable feature MUST be classified as:

Mandatory;

Optional;

Conditional.


8.1 Mandatory

A mandatory feature cannot be removed through graceful degradation.

If it becomes unavailable:

Mandatory Feature Lost
        |
        v
Safe Alternative?
   +----+----+
   |         |
  YES       NO
   |         |
   v         v
Fallback   Reject


---

8.2 Optional

An optional feature MAY be removed if its absence does not violate the remaining contract.


---

8.3 Conditional

A conditional feature may be used only when explicit conditions are satisfied.

For example:

GPU Profile
requires:
    GPU capability
    accelerator authorization
    compatible tensor profile

If those conditions are unavailable, the implementation MAY select a defined fallback.


---

9. Degradation Decision Model

The conceptual algorithm is:

Requested Contract
        |
        v
Validate Contract
        |
        v
Discover Capabilities
        |
        v
Evaluate Compatibility
        |
        v
Negotiate
        |
        v
All Required Features Available?
       / \
     YES  NO
      |    |
      v    v
    Full  Find Safe Degradation
             |
             v
       Mandatory Feature Missing?
            / \
          YES  NO
           |    |
           v    v
       Find Fallback
              |
              v
        Safe Fallback Exists?
            / \
          YES  NO
           |    |
           v    v
       Reduced       Reject
       Contract

The exact discovery and negotiation mechanisms are defined by their respective specifications.


---

10. Degradation Must Be Explicit

A ULABI implementation MUST NOT silently degrade when the degradation changes observable semantics.

Examples of observable changes include:

local execution becoming remote execution;

synchronous execution becoming asynchronous;

authenticated execution becoming unauthenticated;

encrypted transport becoming plaintext;

deterministic execution becoming nondeterministic;

immutable data becoming mutable;

borrowed data becoming owned;

bounded memory becoming unbounded;

guaranteed ordering becoming unordered;

transactional behavior becoming best-effort.


Such changes require explicit contract selection, negotiation, or rejection.


---

11. Performance Degradation

Performance degradation MAY occur when semantics remain unchanged.

For example:

Preferred:
GPU

Fallback:
CPU

is potentially valid when both preserve the same required result semantics.

Likewise:

Preferred:
Zero-copy

Fallback:
Copy-based transfer

MAY be valid if:

ownership remains correct;

lifetime remains correct;

data remains valid;

security properties remain intact;

resource limits remain enforceable.


Performance MUST NOT be treated as semantic equivalence automatically.


---

12. Semantic Preservation

Graceful degradation MUST preserve semantic meaning.

For example:

Preferred:
Vectorized addition

Fallback:
Scalar addition

may be valid.

But:

Preferred:
Strong transaction

Fallback:
Best-effort write

is not automatically valid.

A fallback MUST be rejected when it changes a mandatory semantic guarantee.


---

13. Security Preservation

Security properties MUST NOT be silently degraded.

The following MUST NOT be silently downgraded:

authentication;

authorization;

confidentiality;

integrity;

capability restrictions;

sandboxing;

isolation;

cryptographic verification;

secure loading;

trust requirements.


For example:

Preferred:
Authenticated encrypted channel

Fallback:
Unauthenticated plaintext channel

MUST NOT be treated as a graceful fallback unless the applicable contract explicitly permits it and the security policy authorizes it.

Otherwise:

Security Requirement Lost
          |
          v
        Reject


---

14. Capability Loss

Capabilities may disappear during execution.

Examples:

GPU removed;

network unavailable;

filesystem unavailable;

device unavailable;

remote service unavailable;

memory pressure;

process quota reached.


Capability loss MUST be observable by the relevant runtime or interface layer.

A capability MUST NOT be assumed available indefinitely.


---

15. Capability Loss State Machine

A conceptual capability-loss model is:

Capability Active
       |
       v
Capability Failure
       |
       v
Evidence / Diagnosis
       |
       v
Alternative Available?
      / \
    YES  NO
     |    |
     v    v
Validate Reject / Escalate
Fallback
     |
     v
Commit Reduced Contract
     |
     v
Continue

The implementation MUST NOT continue using a capability after it has become invalid unless the capability specification explicitly permits that behavior.


---

16. Transport Degradation

ULABI is transport-independent.

A system MAY select another transport when the preferred transport becomes unavailable.

Example:

Preferred:
Shared Memory

Fallback:
Local IPC

Fallback:
Socket

Fallback:
Remote Transport

However, transport changes MUST preserve applicable:

security;

locality;

ordering;

reliability;

authentication;

authorization;

serialization;

resource;

latency;

failure semantics.


A transport change that alters meaningful semantics MUST be explicit.


---

17. Locality Preservation

ULABI distinguishes:

LocalOnly;

ProcessLocal;

HostLocal;

NetworkCapable;

RemoteCapable.


A fallback MUST NOT silently cross a locality boundary.

For example:

LocalOnly

MUST NOT silently become:

RemoteCall

because the local implementation is unavailable.

Remote execution introduces additional:

latency;

failure modes;

authentication;

authorization;

data exposure;

resource usage;

consistency considerations.


Such a transition requires an explicit contract.


---

18. Encoding Degradation

When multiple compatible encodings exist, a system MAY use an alternative encoding.

Example:

Preferred:
ULABI Binary

Fallback:
Canonical CBOR

provided the alternative is part of the negotiated and supported contract.

An implementation MUST NOT silently reinterpret bytes using a different encoding.


---

19. Streaming Degradation

A streaming feature MAY degrade to bounded chunked or buffered operation only when the resulting behavior remains within the contract.

Example:

Streaming
    |
    v
Chunked Transfer
    |
    v
Bounded Buffering

The implementation MUST enforce resource limits.

It MUST NOT convert an arbitrarily large stream into unbounded memory buffering.

If no bounded alternative exists:

Reject


---

20. Async Degradation

Asynchronous execution MAY degrade to synchronous execution only when:

the contract permits it;

blocking behavior is acceptable;

resource constraints permit it;

cancellation semantics are preserved or explicitly changed;

the caller has selected or accepted the reduced execution mode.


A synchronous operation MUST NOT be silently introduced where blocking is forbidden.


---

21. Concurrency Degradation

A system MAY reduce concurrency when semantics permit.

Example:

Requested:
64 workers

Available:
8 workers

Reduced:
8 workers

This is valid only if the interface does not require a minimum concurrency level for correctness.

The implementation MUST NOT use reduced concurrency to violate:

ordering requirements;

progress guarantees;

real-time deadlines;

transactional semantics;

synchronization requirements.



---

22. Memory Degradation

Memory-intensive optimizations MAY be disabled under memory pressure.

Examples:

Zero-copy
    ->
Copy-based transfer

or:

Large cache
    ->
Bounded cache

Memory degradation MUST preserve:

ownership;

lifetime;

mutability;

data integrity;

bounds;

release semantics.


An implementation MUST NOT solve memory pressure by violating ownership or lifetime rules.


---

23. Accelerator Degradation

ULABI may support:

CPU;

GPU;

NPU;

FPGA;

other accelerators.


When an accelerator is unavailable, an implementation MAY select another execution backend when the semantic contract remains valid.

Example:

NPU
 |
 +-- unavailable
       |
       v
     GPU
       |
       +-- unavailable
               |
               v
              CPU

The fallback order MUST be determined by policy, capability, compatibility, and resource constraints.


---

24. Hardware Feature Degradation

Hardware-specific optimizations MUST remain optional unless explicitly required by a profile.

An implementation MUST NOT claim hardware-profile conformance merely because it can emulate a feature.

Where emulation is permitted, the implementation MUST identify the mode correctly.

For example:

Hardware acceleration

MUST NOT be reported when execution is actually software-emulated unless the profile explicitly defines that as conforming behavior.


---

25. Resource Degradation

ULABI implementations MAY reduce optional resource consumption.

Examples include:

fewer worker threads;

smaller caches;

smaller batches;

reduced prefetching;

reduced parallelism;

disabled optional telemetry;

disabled optional optimization.


Resource degradation MUST respect mandatory resource guarantees.


---

26. Determinism

If a contract requires deterministic behavior, degradation MUST preserve determinism.

For example:

Deterministic GPU execution

MUST NOT degrade into:

Nondeterministic CPU execution

without an explicit contract change.

A performance fallback is not valid when it violates deterministic semantics.


---

27. Real-Time Constraints

Real-time profiles require special handling.

A degraded implementation MUST NOT continue operating under a real-time contract if it can no longer satisfy mandatory deadlines.

Possible outcomes include:

Real-Time Capability Lost
          |
          +-- Safe alternate implementation
          |
          +-- Controlled failure
          |
          +-- Escalation
          |
          +-- Reject

An implementation MUST NOT silently claim real-time conformance when deadlines cannot be guaranteed.


---

28. Safety-Critical Degradation

Safety-critical profiles MUST define degradation policies explicitly.

A safety-critical implementation MUST identify:

allowable degraded states;

forbidden degraded states;

safe-state behavior;

transition rules;

verification requirements;

recovery requirements;

escalation behavior.


When no approved degraded state exists:

Unsafe Degradation
       |
       v
Safe State / Reject


---

29. Data Integrity

Graceful degradation MUST NOT silently discard mandatory data.

Data loss MAY occur only when the contract explicitly defines data as:

optional;

lossy;

cacheable;

reconstructible;

non-authoritative.


Mandatory data MUST either:

be preserved;

be safely transferred through another mechanism;

or cause controlled failure.



---

30. Lossy Degradation

Some profiles MAY explicitly allow lossy behavior.

Examples:

image quality reduction;

compression-level reduction;

optional metadata removal;

precision reduction.


Lossy behavior MUST be declared.

A system MUST NOT silently convert a lossless contract into a lossy contract.


---

31. Numeric Precision Degradation

Precision reduction is semantic unless explicitly permitted.

For example:

Float64

MUST NOT silently become:

Float32

if the contract requires Float64 semantics.

Precision reduction MAY be permitted when:

explicitly negotiable;

bounded;

declared;

acceptable to the caller;

covered by the applicable profile.



---

32. Optional Metadata

Optional metadata MAY be removed during degradation when the metadata is explicitly classified as non-semantic.

Examples may include:

performance hints;

optional diagnostics;

non-authoritative cache information.


Security, authorization, ownership, integrity, and execution metadata MUST NOT be treated as optional unless the governing specification explicitly says so.


---

33. Degradation and Feature Negotiation

Feature negotiation determines the selected feature set.

Graceful degradation determines what safe reduced operation is possible when the preferred set cannot be fully satisfied.

The relationship is:

Capability Discovery
        |
        v
Compatibility Analysis
        |
        v
Feature Negotiation
        |
        +------ Full Contract ------> Execute
        |
        +------ Partial Contract ---> Degrade
        |
        +------ No Safe Contract ---> Reject

This document MUST NOT replace the feature-negotiation state machine.


---

34. Degradation and Capability Discovery

Capability discovery answers:

> What is available?



Graceful degradation asks:

> What safe operation remains possible?



An implementation MUST NOT infer capabilities that were not discovered or otherwise established.

Capability discovery remains authoritative for capability identity and availability.


---

35. Degradation and Backward Compatibility

Backward compatibility determines whether a newer implementation can continue supporting an older contract.

Graceful degradation may be used when:

a compatible optional feature is unavailable;

a profile is absent;

an optimization cannot be provided.


It MUST NOT be used to hide an actual backward-compatibility break.


---

36. Degradation and Forward Compatibility

Forward compatibility determines how older implementations handle newer information.

Graceful degradation may provide a reduced behavior when future functionality is unavailable, but it MUST follow the forward-compatibility rules for unknown information.

Unknown security requirements MUST fail closed.

Unknown semantic requirements MUST NOT be guessed.


---

37. Degradation and Security

Security policy has authority over degradation.

The following precedence applies:

Security Requirement
        >
Safety Requirement
        >
Correctness Requirement
        >
Contract Requirement
        >
Optional Feature
        >
Optimization
        >
Performance Preference

An optimization MUST never override a security requirement.


---

38. Degradation and Authorization

Authorization MUST be evaluated for the degraded contract.

A fallback that requires a capability unavailable to the caller MUST NOT be enabled merely because it technically exists.

Example:

Preferred:
Local GPU

Fallback:
Remote GPU

The remote GPU fallback requires separate authorization.

It MUST NOT be automatically activated.


---

39. Degradation and Resource Limits

Degradation MUST remain bounded.

A fallback MUST NOT create unbounded:

memory use;

CPU use;

thread creation;

network traffic;

retries;

recursion;

queue growth;

handle creation;

storage;

logging.


A fallback that cannot satisfy resource limits MUST be rejected or escalated.


---

40. Retry Limits

Retries are not themselves graceful degradation.

Retries MAY be part of a fallback strategy, but MUST be:

bounded;

policy-controlled;

observable;

cancellable where applicable.


An implementation MUST NOT retry indefinitely.


---

41. Cascading Failure Prevention

A degraded mode MUST NOT create uncontrolled cascading failure.

For example:

GPU unavailable
    |
    v
CPU fallback
    |
    v
CPU overloaded
    |
    v
Unlimited worker creation

is forbidden.

Instead:

GPU unavailable
    |
    v
CPU fallback
    |
    v
Resource Check
    |
    +-- capacity available --> Continue
    |
    +-- capacity unavailable -> Controlled Failure


---

42. Degradation Budget

Profiles MAY define a degradation budget.

A degradation budget identifies how far an implementation may reduce:

performance;

concurrency;

precision;

availability;

optional functionality;

resource consumption.


The budget MUST be explicit.

Example:

Maximum tolerated latency:
100 ms

Minimum concurrency:
4 workers

Precision:
Float32 acceptable

Remote execution:
Forbidden

A fallback outside the budget MUST NOT be selected.


---

43. Degradation Policies

Degradation policy SHOULD be represented explicitly.

Conceptually:

DegradationPolicy {
    policy_id
    allowed_fallbacks
    forbidden_fallbacks
    mandatory_features
    optional_features
    resource_limits
    security_requirements
    locality_constraints
    safety_constraints
    performance_constraints
    escalation_policy
}

The actual serialized representation belongs to the applicable schema.


---

44. Fallback Ordering

Where multiple fallbacks exist, the implementation SHOULD use a deterministic ordering.

Example:

1. Native GPU
2. Alternate GPU backend
3. NPU
4. CPU SIMD
5. CPU scalar
6. Reject

Fallback ordering MUST consider:

compatibility;

security;

authorization;

resources;

locality;

policy;

correctness.


Performance preference alone MUST NOT determine fallback selection.


---

45. Fallback Equivalence

A fallback SHOULD declare what form of equivalence it provides.

Possible classes:

Semantic Equivalent
Security Equivalent
Performance Equivalent
Interface Equivalent
Transport Equivalent
Best Effort

Only the equivalence properties explicitly guaranteed by the fallback may be relied upon.

For example:

CPU fallback
Semantic Equivalent: YES
Performance Equivalent: NO

The implementation MUST NOT claim performance equivalence.


---

46. Contract Transformation

Graceful degradation can be modeled as a controlled transformation:

Requested Contract
        |
        v
Degradation Policy
        |
        v
Reduced Contract

The reduced contract MUST be independently validated.

Conceptually:

Validate(Requested Contract)
            |
            v
Apply Approved Degradation
            |
            v
Validate(Reduced Contract)
            |
            +-- Valid --> Commit
            |
            +-- Invalid -> Reject

An implementation MUST NOT activate an unvalidated degraded contract.


---

47. Atomic Degradation

Degradation SHOULD be committed atomically.

The implementation MUST NOT expose a partially degraded state that violates the interface contract.

Example:

Current Contract
      |
      v
Prepare Fallback
      |
      v
Validate
      |
      v
Atomic Commit
      |
      v
Reduced Contract

If commitment fails, the implementation MUST retain the last valid contract or enter a defined safe state.


---

48. In-Flight Operations

An implementation MUST define how degradation affects operations already in progress.

Possible policies include:

complete using the old contract;

migrate safely;

cancel;

fail deterministically;

drain and then switch.


The policy MUST be explicit.

An implementation MUST NOT change the semantics of an in-flight operation mid-operation without a contract permitting that behavior.


---

49. State Preservation

When switching to a fallback, state required by the operation MUST be preserved or safely reconstructed.

The implementation MUST NOT lose:

ownership information;

transaction state;

authentication context;

authorization context;

cancellation state;

resource accounting;

integrity state.



---

50. Graceful Degradation and Handles

Handles remain bound to their declared resource semantics.

A fallback MUST NOT silently reinterpret a handle.

For example:

Handle<GPUBuffer>

MUST NOT automatically become:

Handle<File>

If migration to another resource type is supported, it MUST be explicit and validated.


---

51. Graceful Degradation and Memory Ownership

Fallback implementations MUST preserve ownership semantics.

Examples:

Borrowed

MUST remain borrowed unless the contract explicitly permits transfer.

Owned

MUST remain owned.

A copy-based fallback MUST NOT accidentally create double ownership.


---

52. Graceful Degradation and Errors

Degraded operation MUST expose errors using the ULABI error model.

The implementation SHOULD identify:

original failure;

degradation decision;

selected fallback;

fallback result;

whether semantics were reduced.


Conceptually:

DegradationResult {
    status
    original_failure
    selected_mode
    reduced_features
    fallback_identity
}

The actual error representation remains governed by the error model.


---

53. Observability

A degradation event SHOULD be observable through the ULABI observability mechanisms.

At minimum, where supported, the event should identify:

interface;

operation;

original mode;

failure;

degraded mode;

reason;

policy;

timestamp or sequence;

resulting state.


Observability MUST NOT leak protected data.


---

54. Deterministic Degradation

Given the same:

contract;

capabilities;

policy;

resource state;

security context;


a degradation decision SHOULD be deterministic unless the profile explicitly allows nondeterministic selection.

Deterministic degradation simplifies:

testing;

debugging;

certification;

reproducibility;

formal verification.



---

55. Failure Containment

A failed fallback MUST remain contained.

Example:

Primary Failure
      |
      v
Fallback
      |
      v
Fallback Failure
      |
      v
No More Fallbacks
      |
      v
Controlled Failure

The implementation MUST NOT recursively generate increasingly unsafe fallback modes.


---

56. No Automatic Semantic Downgrade

ULABI MUST NOT automatically perform transformations such as:

secure -> insecure
authenticated -> unauthenticated
local -> remote
lossless -> lossy
deterministic -> nondeterministic
transactional -> best-effort
immutable -> mutable
owned -> borrowed
bounded -> unbounded

unless the applicable contract explicitly permits and authorizes the transformation.


---

57. Degradation and Self-Healing

Graceful degradation and self-healing are related but distinct.

Graceful degradation answers:

> What reduced contract can safely continue operating?



Self-healing answers:

> How can a failed component be diagnosed, recovered, verified, and restored?



The relationship is:

Failure
   |
   +---- Safe Reduced Contract Available
   |             |
   |             v
   |        Graceful Degradation
   |
   +---- Recovery Possible
                 |
                 v
             Self-Healing

Graceful degradation MUST NOT become uncontrolled self-modification.

Self-healing remains governed by:

docs/reliability/self-healing.md


---

58. Degradation and Rollback

When a degraded transition fails verification, the implementation SHOULD:

1. stop the failed transition;


2. restore the previous valid state when safe;


3. otherwise enter the defined safe state;


4. report the failure;


5. escalate when required.



Conceptually:

Prepare Fallback
      |
      v
Activate
      |
      v
Verify
   /     \
Healthy  Failed
  |        |
  v        v
Continue  Rollback
             |
             v
          Verify
             |
        +----+----+
        |         |
      Success    Failure
        |         |
        v         v
      Resume    Escalate


---

59. Escalation

Escalation MUST be policy-controlled.

Possible escalation targets include:

caller;

supervisor;

runtime;

orchestrator;

health monitor;

security subsystem;

safety controller.


An implementation MUST NOT escalate by granting itself additional capabilities.


---

60. Distributed Degradation

Distributed degradation requires special care because a degraded participant may affect other participants.

A participant MUST NOT unilaterally change a shared distributed contract when the change affects another participant's assumptions.

For distributed operation:

Participant A
      |
      v
Failure
      |
      v
Local Degradation Proposal
      |
      v
Validate / Negotiate
      |
      v
Shared Reduced Contract

The exact distributed protocol is governed by:

docs/distributed/distributed-abi.md

and related specifications.


---

61. Network Partition

A network partition MUST NOT automatically be interpreted as permission to switch to an unsafe local mode.

The correct response depends on the consistency contract.

Possible outcomes:

continue using cached data;

enter read-only mode;

pause;

fail;

switch to an explicitly defined offline profile.



---

62. Consistency Preservation

A degraded distributed system MUST preserve its declared consistency model.

For example:

Strong Consistency

MUST NOT silently become:

Eventual Consistency

unless the contract explicitly permits negotiated degradation.


---

63. Offline Degradation

An interface MAY define an offline mode.

Offline mode MUST declare:

operations permitted;

operations forbidden;

stale-data rules;

synchronization rules;

conflict rules;

security requirements;

expiration;

recovery behavior.


An implementation MUST NOT silently treat stale data as authoritative when the contract requires current data.


---

64. Graceful Degradation and Caching

Caching MAY be used as a fallback when the contract permits stale or cached data.

The cache policy MUST define:

freshness;

expiration;

invalidation;

authority;

integrity;

security context.


Expired or untrusted data MUST NOT be used merely because the primary service is unavailable.


---

65. Graceful Degradation and Internationalization

If an optional localization resource is unavailable, a system MAY fall back to another explicitly supported locale.

Example:

Requested:
Language A

Fallback:
Language B

The fallback MUST NOT silently alter security-sensitive identifiers, protocol fields, or machine-readable values.


---

66. Graceful Degradation and Diagnostics

If optional diagnostics are unavailable, execution MAY continue when the diagnostics are not required for correctness or safety.

However, safety-critical profiles MAY require diagnostics.

In such profiles:

Required Diagnostics Lost
          |
          v
Controlled Failure


---

67. Graceful Degradation and Telemetry

Optional telemetry MAY be disabled when:

it is not mandatory;

disabling it does not violate safety requirements;

disabling it does not hide mandatory security events.


Security and audit events MUST remain subject to their governing security policy.


---

68. Graceful Degradation and Debugging

A debugging feature MAY be unavailable without making the underlying interface incompatible.

However, debugging MUST NOT be required as a hidden dependency for normal operation unless the relevant profile explicitly requires it.


---

69. Degradation Policy Precedence

When policies conflict, the following precedence SHOULD apply:

1. Safety
2. Security
3. Correctness
4. Data Integrity
5. Ownership / Lifetime
6. Contract Requirements
7. Resource Constraints
8. Availability
9. Performance
10. Optimization

Lower-priority preferences MUST NOT override higher-priority requirements.


---

70. Degradation Invariants

A conforming implementation MUST maintain the following invariants:

Invariant 1 — No Silent Semantic Violation

Degradation MUST NOT silently change mandatory semantics.

Invariant 2 — No Security Downgrade

Degradation MUST NOT silently weaken mandatory security.

Invariant 3 — No Ownership Violation

Degradation MUST NOT alter ownership without authorization.

Invariant 4 — No Unbounded Fallback

Fallback behavior MUST remain resource-bounded.

Invariant 5 — Explicit Contract

The resulting degraded contract MUST be identifiable.

Invariant 6 — Validated Transition

A degraded contract MUST be validated before activation.

Invariant 7 — Safe Failure

If no valid degraded state exists, the implementation MUST fail safely.

Invariant 8 — No Hidden Remote Execution

A local contract MUST NOT silently become remote.

Invariant 9 — No Infinite Recovery

Fallback and retry behavior MUST be bounded.

Invariant 10 — Policy Authority

Degradation MUST remain subject to explicit policy.


---

71. Security Requirements

A graceful-degradation implementation MUST:

1. fail closed for unsupported security requirements;


2. preserve authorization requirements;


3. preserve capability restrictions;


4. preserve authentication where mandatory;


5. preserve integrity requirements;


6. preserve isolation requirements;


7. enforce resource limits;


8. validate fallback contracts;


9. prevent downgrade attacks;


10. prevent unauthorized fallback activation.




---

72. Downgrade Attack Prevention

An attacker MUST NOT be able to intentionally force a system into a weaker security mode merely by making a preferred capability appear unavailable.

Implementations SHOULD distinguish:

Naturally Unavailable

from:

Maliciously Suppressed

where the surrounding security architecture supports such detection.

Security-critical degradation decisions SHOULD be authenticated and policy-validated.


---

73. Failure Modes

Potential degradation failures include:

no compatible fallback;

fallback unavailable;

fallback unauthorized;

fallback incompatible;

resource exhaustion;

security requirement violation;

ownership violation;

timeout;

partial transition;

rollback failure;

state migration failure;

inconsistent distributed state;

invalid degraded contract;

cascading fallback;

repeated failure.


Each MUST result in a defined behavior.


---

74. Recovery From Degradation

A system MAY later restore the preferred contract.

The restoration sequence SHOULD be:

Reduced Mode
     |
     v
Capability Recovered
     |
     v
Validate Preferred Contract
     |
     v
Negotiate if Required
     |
     v
Prepare Restoration
     |
     v
Verify
     |
     v
Atomic Commit
     |
     v
Preferred Mode

Restoration MUST NOT interrupt active operations unsafely.


---

75. Hysteresis

Implementations SHOULD avoid rapidly switching between modes when a capability repeatedly appears and disappears.

For example:

GPU available
GPU unavailable
GPU available
GPU unavailable
...

could cause instability.

A policy MAY define:

minimum stable duration;

retry delay;

recovery threshold;

failure threshold;

cooldown period.



---

76. Degradation State Identity

A degraded contract SHOULD have an explicit identity.

Conceptually:

DegradedContract {
    base_contract
    degradation_policy
    removed_features
    selected_fallbacks
    constraints
    security_context
    version
}

This allows:

observability;

debugging;

testing;

certification;

reproducibility.



---

77. Conformance Requirements

An implementation claiming graceful-degradation conformance MUST demonstrate:

1. identification of mandatory features;


2. identification of optional features;


3. explicit degradation policy;


4. safe fallback selection;


5. validation before activation;


6. security-preserving degradation;


7. bounded resource usage;


8. deterministic failure where required;


9. safe rejection;


10. rollback or safe-state behavior;


11. observable degradation state;


12. prevention of unauthorized downgrade.




---

78. Conformance Test Categories

The conformance suite SHOULD test at least:

GD-001 Full Operation
GD-002 Optional Feature Removal
GD-003 Mandatory Feature Failure
GD-004 Safe CPU Fallback
GD-005 Unsafe Fallback Rejection
GD-006 Security Downgrade Rejection
GD-007 Local-to-Remote Rejection
GD-008 Ownership Preservation
GD-009 Lifetime Preservation
GD-010 Resource-Bounded Fallback
GD-011 Encoding Fallback
GD-012 Streaming Fallback
GD-013 Async Fallback
GD-014 Concurrency Reduction
GD-015 Accelerator Fallback
GD-016 Determinism Preservation
GD-017 Real-Time Constraint Enforcement
GD-018 Distributed Degradation
GD-019 Network Partition Handling
GD-020 Cache Expiration
GD-021 Atomic Degradation
GD-022 Rollback
GD-023 Repeated Failure / Hysteresis
GD-024 Downgrade Attack Prevention
GD-025 Degraded Contract Identity
GD-026 Observability
GD-027 Recovery to Preferred Mode


---

79. Reference Decision Algorithm

A reference conceptual algorithm is:

function select_mode(request, capabilities, policy):

    validate(request)

    available = discover(capabilities)

    compatible = evaluate_compatibility(
        request,
        available
    )

    if compatible.full_contract_available:
        return FULL_CONTRACT

    candidates = generate_policy_allowed_fallbacks(
        request,
        available,
        policy
    )

    for candidate in candidates:

        if violates_security(candidate, policy):
            continue

        if violates_safety(candidate, policy):
            continue

        if violates_ownership(candidate):
            continue

        if violates_lifetime(candidate):
            continue

        if violates_locality(candidate):
            continue

        if violates_resource_limits(candidate, policy):
            continue

        if violates_semantics(candidate):
            continue

        if validate(candidate):
            return candidate

    return REJECT

The implementation MAY use another algorithm.

The observable behavior MUST satisfy this specification.


---

80. Reference State Machine

+----------------+
                 | Full Contract  |
                 +-------+--------+
                         |
                    failure
                         v
                 +----------------+
                 | Detect Failure |
                 +-------+--------+
                         |
                         v
                 +----------------+
                 | Collect State  |
                 +-------+--------+
                         |
                         v
                 +----------------+
                 | Check Policy   |
                 +-------+--------+
                         |
                 +-------+-------+
                 |               |
             Allowed          Forbidden
                 |               |
                 v               v
         +---------------+     Reject
         | Find Fallback |
         +-------+-------+
                 |
          +------+------+
          |             |
        Found         None
          |             |
          v             v
      Validate        Reject
          |
     +----+----+
     |         |
   Valid     Invalid
     |         |
     v         v
   Commit    Next Candidate
     |
     v
Reduced Contract
     |
     v
Verify
     |
 +---+---+
 |       |
Healthy Failed
 |       |
 v       v
Done   Rollback
           |
      +----+----+
      |         |
    Success   Failure
      |         |
      v         v
    Continue Escalate


---

81. Reference Implementation Boundary

The reference implementation SHOULD expose a policy-independent core decision interface while keeping policy configuration external.

Conceptually:

DegradationEngine
    |
    +-- ContractValidator
    |
    +-- CapabilityProvider
    |
    +-- CompatibilityEvaluator
    |
    +-- FallbackRegistry
    |
    +-- PolicyEvaluator
    |
    +-- SecurityEvaluator
    |
    +-- ResourceEvaluator
    |
    +-- StateManager
    |
    +-- TransitionManager
    |
    +-- RollbackManager
    |
    +-- HealthVerifier
    |
    +-- ObservabilitySink

The reference implementation MUST NOT become part of the normative ULABI standard itself.

Independent implementations MAY use completely different internal architectures.


---

82. Required Interfaces

A conforming implementation SHOULD expose equivalent logical interfaces for:

discover_capabilities()
evaluate_compatibility()
evaluate_policy()
find_fallbacks()
validate_degraded_contract()
prepare_transition()
commit_transition()
verify_transition()
rollback_transition()
report_degradation()
restore_preferred_contract()

These are conceptual interfaces.

They do not prescribe a programming language or source-level API.


---

83. Required Data Structures

The implementation SHOULD represent the following concepts:

DegradationPolicy
DegradationCandidate
DegradedContract
FallbackDescriptor
DegradationDecision
DegradationTransition
DegradationFailure
DegradationState
RecoveryAttempt

Their concrete representations MAY differ between implementations.


---

84. Integration With ULABI Core

This specification depends on the Core for:

interface identity;

type identity;

function identity;

errors;

versioning;

contract semantics.


It MUST NOT redefine those concepts.


---

85. Integration With ABI Specifications

This specification consumes:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

A fallback MUST remain conformant to the applicable ABI rules.


---

86. Integration With Type System

Fallbacks MUST preserve ULABI type semantics.

They MUST NOT silently alter:

type identity;

signedness;

range;

ownership;

lifetime;

encoding;

semantic meaning.


Type-specific compatibility remains defined by the type-system specifications.


---

87. Integration With Memory

Fallbacks MUST obey:

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

Memory pressure MUST NOT justify violating those contracts.


---

88. Integration With Runtime

Runtime degradation MUST integrate with:

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

The runtime MUST remain aware of the selected degraded execution mode.


---

89. Integration With Security

Security remains authoritative through:

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Graceful degradation MUST NOT override security policy.


---

90. Integration With Distributed Systems

Distributed degradation integrates with:

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Distributed fallback MUST respect shared contract semantics.


---

91. Integration With Reliability

Graceful degradation integrates with:

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md
docs/reliability/self-healing.md

Degradation provides a controlled reduced operating mode.

It does not replace recovery or self-healing.


---

92. Integration With Compatibility

The compatibility family is:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Responsibility boundaries:

Backward Compatibility
    |
    +-- Can newer implementation serve older contract?

Forward Compatibility
    |
    +-- Can older implementation safely encounter newer contract?

Capability Discovery
    |
    +-- What exists?

Feature Negotiation
    |
    +-- What will actually be selected?

Graceful Degradation
    |
    +-- What safe reduced contract can operate?

No document should duplicate another's primary responsibility.


---

93. Integration With Observability

Degradation events SHOULD integrate with:

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md

The observability system MUST distinguish:

Normal
Degraded
Recovering
Failed
Rejected


---

94. Integration With Conformance

Conformance is defined through:

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md

Graceful-degradation support SHOULD be independently testable.

An implementation MUST NOT claim complete ULABI conformance merely because it implements graceful degradation.


---

95. Compatibility Requirements

A degraded contract MUST have:

an identifiable contract version;

identifiable selected features;

identifiable removed features;

identifiable fallback mechanisms;

explicit constraints;

applicable security requirements;

applicable resource requirements.


The resulting contract MUST be machine-verifiable where practical.


---

96. Implementation Independence

Nothing in this specification requires:

C;

C++;

Rust;

Go;

Java;

Python;

Swift;

Kotlin;

Zamani;

Sankofa;

any particular runtime;

any particular operating system;

any particular processor;

any particular vendor.


ULABI defines the contract.

Implementations define their internal mechanisms.


---

97. Security Boundary

Graceful degradation MUST remain below the authority of the security policy.

Conceptually:

Security Policy
      |
      v
Degradation Policy
      |
      v
Fallback Selection
      |
      v
Reduced Contract

Not:

Fallback Selection
      |
      v
Override Security Policy

The latter is forbidden.


---

98. No Autonomous Contract Expansion

Graceful degradation MUST NOT grant an implementation new capabilities.

A degraded implementation MAY remove functionality.

It MUST NOT obtain additional authority merely because its preferred mode failed.

For example:

GPU unavailable
    |
    X
Request unrestricted network access

is prohibited.


---

99. Formal Invariants

For any degraded contract D derived from requested contract R:

D MUST satisfy:

Security(D) >= RequiredSecurity(R)

Safety(D) >= RequiredSafety(R)

Correctness(D) >= RequiredCorrectness(R)

Ownership(D) = RequiredOwnership(R)

Lifetime(D) = RequiredLifetime(R)

Authorization(D) <= AuthorizedCapabilities

Resources(D) <= ResourcePolicy

Locality(D) <= AllowedLocality

Semantics(D) = RequiredSemantics

Performance MAY decrease where explicitly permitted.

Optional functionality MAY decrease where explicitly permitted.

Mandatory semantics MUST NOT decrease.


---

100. Formal Acceptance Condition

A degraded contract is valid only if:

ValidDegradation =
    ContractValid
    AND
    SecurityValid
    AND
    SafetyValid
    AND
    AuthorizationValid
    AND
    OwnershipValid
    AND
    LifetimeValid
    AND
    ResourceValid
    AND
    LocalityValid
    AND
    SemanticValid
    AND
    PolicyAllowed

If any mandatory condition is false:

ValidDegradation = false

and the implementation MUST NOT activate the degraded contract.


---

101. Reference Examples

Example A — GPU Failure

Requested:
GPU + CPU fallback

GPU unavailable

CPU available

Result:
CPU fallback

Preserved:
Semantics
Security
Ownership

Changed:
Performance

Valid if the contract permits CPU fallback.


---

Example B — Security Failure

Requested:
Authenticated encrypted transport

Encryption unavailable

Fallback:
Plaintext transport

Result:

REJECT

unless an explicitly authorized weaker profile exists.


---

Example C — Zero-Copy Failure

Requested:
Zero-copy buffer

Shared memory unavailable

Fallback:
Copy buffer

Potentially valid if:

ownership remains correct;

lifetime remains correct;

data integrity remains intact;

resource limits remain satisfied.



---

Example D — Remote Fallback

Local service unavailable

Fallback:
Remote service

Not automatically valid.

The implementation MUST verify:

remote execution is allowed;

authentication is available;

authorization is available;

locality permits remote execution;

data-transfer policy permits it;

failure semantics are acceptable.



---

Example E — Optional Streaming

Requested:
Streaming

Streaming unavailable

Fallback:
Bounded chunking

Potentially valid if the profile permits bounded chunking.

Unbounded buffering is forbidden.


---

102. Reference Conformance Checklist

A conforming implementation should be able to answer:

[ ] Can mandatory and optional features be distinguished?
[ ] Can unavailable features be detected?
[ ] Can safe fallbacks be identified?
[ ] Can unsafe fallbacks be rejected?
[ ] Are security properties preserved?
[ ] Are ownership rules preserved?
[ ] Are lifetime rules preserved?
[ ] Are locality constraints preserved?
[ ] Are resource limits enforced?
[ ] Are degraded contracts validated?
[ ] Is activation atomic?
[ ] Is rollback supported where required?
[ ] Are degradation events observable?
[ ] Are fallback retries bounded?
[ ] Are downgrade attacks prevented?
[ ] Can the preferred mode be restored safely?
[ ] Are distributed changes coordinated?
[ ] Are safety-critical degraded states explicit?
[ ] Are conformance tests available?


---

103. Final Principle

ULABI graceful degradation follows one central rule:

> Reduce optional capability before reducing mandatory guarantees.



Therefore:

Optional Feature
      |
      v
Can Remove Safely?
      |
     YES
      |
      v
Remove Feature
      |
      v
Validate Contract
      |
      v
Continue

But:

Mandatory Guarantee
      |
      v
Can Preserve?
   /       \
 YES       NO
  |         |
  v         v
Continue   Reject / Safe State

ULABI MUST prefer controlled failure over unsafe interoperability.

Graceful degradation exists to preserve useful operation without weakening the universal contract that makes ULABI trustworthy.

## What should be done under which files

The important correction from the repository review is that **we should not implement graceful degradation by scattering its logic across every compatibility document**. The documents should establish contracts first; code should then implement those contracts.

### Phase 1 — independent normative contracts

These should be completed first because they establish concepts other files consume.

| Order | File | Responsibility | Must integrate with |
|---|---|---|---|
| 1 | `ULABI-DESIGN.md` | Master architecture | Everything |
| 2 | `ULABI-SPEC.md` | Normative Core rules | Everything |
| 3 | `ULABI-VERSIONING.md` | Version identity and evolution | All compatibility |
| 4 | `docs/architecture/design-principles.md` | Architectural principles | Architecture |
| 5 | `docs/architecture/universal-model.md` | Universal semantic model | ABI/type/interoperability |
| 6 | `docs/architecture/scalability.md` | Scaling model | Runtime/distributed |
| 7 | `docs/architecture/overview.md` | Architecture navigation | All documentation |

### Phase 2 — foundational ABI contracts

```text
docs/abi/core-abi.md
docs/abi/data-types.md
docs/abi/calling-convention.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

These establish the actual binary boundary.

Phase 3 — type system

docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/type-compatibility.md

These should define the semantic type contracts that compatibility and degradation consume.

Phase 4 — interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Phase 5 — memory/runtime

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Phase 6 — security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/cryptography.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

Security must be completed before implementing automatic fallback selection because degradation cannot be allowed to weaken the security boundary.

Phase 7 — compatibility

The responsibility split should remain:

backwards-compatibility.md
        ↓
preserve old contracts

forwards-compatibility.md
        ↓
safely encounter future contracts

capability-discovery.md
        ↓
discover available capabilities

feature-negotiation.md
        ↓
select a mutually supported contract

graceful-degradation.md
        ↓
operate safely with a reduced contract

Then:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Finally:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

should all be cross-checked against ULABI-VERSIONING.md rather than repeatedly editing one another.


---

Required implementation/code modules

The repository currently has the documentation foundation, but the architecture should eventually produce implementation modules along these boundaries.

I would not put all of this into one compatibility.rs or equivalent file.

Core

core/
├── interface_identity
├── function_identity
├── type_identity
├── contract
├── version
├── encoding
├── errors
├── validation
└── effects

Compatibility

compatibility/
├── compatibility_engine
├── backward_checker
├── forward_checker
├── version_resolver
├── compatibility_result
└── compatibility_report

Capability system

capabilities/
├── capability_id
├── capability_descriptor
├── capability_registry
├── capability_provider
├── capability_snapshot
├── capability_validator
└── capability_policy

Feature negotiation

negotiation/
├── negotiation_id
├── negotiation_context
├── feature_descriptor
├── feature_registry
├── dependency_resolver
├── conflict_resolver
├── proposal
├── response
├── selection
├── state_machine
├── commitment
└── negotiation_validator

Graceful degradation

This is the code family directly corresponding to the document I just wrote:

degradation/
├── degradation_engine
├── degradation_policy
├── degradation_candidate
├── degradation_decision
├── degraded_contract
├── fallback_descriptor
├── fallback_registry
├── fallback_selector
├── contract_transformer
├── transition_manager
├── transition_validator
├── state_manager
├── rollback_manager
├── degradation_verifier
├── degradation_report
├── degradation_error
├── degradation_state
└── degradation_observer

Important separation

Do not make:

degradation_engine

responsible for discovering capabilities.

Instead:

CapabilityProvider
       ↓
CompatibilityEngine
       ↓
NegotiationEngine
       ↓
DegradationEngine
       ↓
ContractValidator
       ↓
TransitionManager

That separation is important for independent testing and avoids repeating logic.


---

Runtime/resource integration modules

runtime/
├── execution_context
├── resource_manager
├── resource_limits
├── execution_mode
├── locality
├── async_context
├── concurrency_manager
└── health_state

Security integration modules

security/
├── security_context
├── authorization_context
├── capability_guard
├── security_policy
├── downgrade_guard
├── trust_context
└── integrity_validator

Reliability integration modules

reliability/
├── fault_detector
├── fault_classifier
├── health_monitor
├── recovery_manager
├── rollback_manager
├── safe_state
└── escalation_manager

The reliability modules should consume degradation rather than duplicate its policy engine.


---

Distributed integration modules

distributed/
├── remote_contract
├── distributed_negotiation
├── distributed_degradation
├── consistency_policy
├── partition_handler
├── service_state
└── distributed_failure

These should build on the generic degradation engine rather than creating a second independent degradation framework.


---

Schemas

The implementation will eventually need machine-readable schemas for the contracts:

schemas/
├── interface.schema
├── type.schema
├── function.schema
├── version.schema
├── capability.schema
├── feature.schema
├── negotiation.schema
├── degradation-policy.schema
├── degraded-contract.schema
├── fallback.schema
├── error.schema
└── conformance.schema

The exact schema format should be selected later according to the repository's normative serialization decision. The documentation should not prematurely couple ULABI to JSON, YAML, CBOR, Protobuf, or another representation.


---

Tests

The test tree should mirror the specification boundaries:

tests/
├── core/
├── types/
├── abi/
├── memory/
├── runtime/
├── security/
├── interoperability/
├── compatibility/
│   ├── backward/
│   ├── forward/
│   ├── discovery/
│   ├── negotiation/
│   └── degradation/
├── distributed/
├── reliability/
└── conformance/

For graceful degradation specifically:

tests/compatibility/degradation/
├── full_operation
├── optional_feature_removal
├── mandatory_feature_failure
├── safe_fallback
├── unsafe_fallback
├── security_downgrade
├── locality_downgrade
├── ownership_preservation
├── lifetime_preservation
├── resource_bounds
├── encoding_fallback
├── streaming_fallback
├── async_fallback
├── accelerator_fallback
├── deterministic_fallback
├── atomic_transition
├── rollback
├── distributed_degradation
├── downgrade_attack
└── restoration


---

Reference implementation

The reference/ directory should eventually contain a deliberately small implementation of the normative contracts:

reference/
├── core/
├── types/
├── compatibility/
├── capabilities/
├── negotiation/
├── degradation/
├── security/
├── runtime/
└── validation/

It should not become the definition of ULABI. The specification remains the authority.


---

Conformance implementation

Eventually:

conformance/
├── core/
├── types/
├── abi/
├── memory/
├── security/
├── async/
├── distributed/
├── compatibility/
├── degradation/
├── reliability/
└── certification/

This allows an implementation to report something meaningful such as:

ULABI Core                 PASS
ULABI Types                PASS
ULABI ABI                  PASS
ULABI Memory               PASS
ULABI Security             PASS
ULABI Capability Discovery PASS
ULABI Negotiation           PASS
ULABI Graceful Degradation  PASS
ULABI Distributed           PARTIAL
ULABI Self-Healing          NOT IMPLEMENTED

rather than the vague claim "ULABI compatible."

Most important architectural result

The repository is now naturally converging on this pipeline:

ULABI CONTRACT
                       |
                       v
              Capability Discovery
                       |
                       v
             Compatibility Analysis
                       |
                       v
              Feature Negotiation
                       |
              +--------+--------+
              |                 |
          Full Contract     Partial Contract
              |                 |
              v                 v
          Execution      Graceful Degradation
                                |
                         +------+------+
                         |             |
                    Safe Fallback    Reject
                         |
                         v
                    Validation
                         |
                         v
                    Atomic Commit
                         |
                         v
                     Execution
                         |
                         v
                     Recovery
                         |
                         v
                 Preferred Contract

That is the boundary I recommend locking in. Graceful degradation should be a policy-controlled contract transformation layer, not a generic "try something else" mechanism. It should never be allowed to silently cross security, ownership, locality, safety, or semantic boundaries.

