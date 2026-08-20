I read the repository first rather than treating the directory tree you supplied as authoritative.

The current repository confirms that ULABI-DESIGN.md is the architectural authority, currently version 0.2.0-draft, and that the project already establishes Minimal Core + Standard Profiles + Extensible Ecosystem, explicit semantics, capability discovery, ABI negotiation, compatibility, implementation independence, and language/runtime/platform neutrality.

I also checked the existing compatibility documents. capability-discovery.md is already a substantial specification and explicitly says that capability discovery supplies evidence while feature negotiation determines what both parties will actually use.  backwards-compatibility.md separately owns preservation of older contracts.  The current feature-negotiation.md, however, is still only a placeholder.

So this document should not duplicate capability discovery, backward compatibility, forward compatibility, or graceful degradation. It should define the actual negotiation protocol and state machine that consumes those contracts.

Also, the repository already has substantial ABI architecture documents, including calling convention, core ABI, data types, memory, registers, stack, return values, and exception model, so the negotiation document should integrate with them rather than redefine them.

docs/compatibility/feature-negotiation.md

Copy everything below into:

docs/compatibility/feature-negotiation.md

# ULABI Feature Negotiation

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/compatibility/feature-negotiation.md`  
**Specification Family:** Compatibility / Feature Negotiation  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**ULABI Core Specification:** 0.1.0  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI feature-negotiation contract.

Feature negotiation determines which compatible ULABI capabilities, profiles,
features, interface versions, execution modes, encodings, transports, and
optional behaviors will actually be used when two implementations interact.

Feature negotiation occurs after sufficient capability information has been
obtained and before negotiated functionality is used.

The fundamental model is:

```text
Capability Discovery
        |
        v
Capabilities Known
        |
        v
Compatibility Analysis
        |
        v
Feature Negotiation
        |
        v
Selected Contract
        |
        v
Execution

Capability discovery answers:

> What is available?



Compatibility analysis answers:

> What can safely interoperate?



Feature negotiation answers:

> Which compatible options will both parties use?



Therefore:

Discovered
    !=
Negotiated

Supported
    !=
Selected

Available
    !=
Authorized

Compatible
    !=
Automatically Enabled

A ULABI implementation MUST preserve these distinctions.


---

2. Scope

This specification defines:

1. negotiation terminology;


2. negotiation participants;


3. negotiation scope;


4. negotiation identifiers;


5. negotiation capabilities;


6. negotiation requirements;


7. mandatory and optional features;


8. feature preferences;


9. feature dependencies;


10. feature conflicts;


11. compatibility evaluation;


12. negotiation requests;


13. negotiation responses;


14. proposal and selection semantics;


15. deterministic selection;


16. version negotiation;


17. profile negotiation;


18. encoding negotiation;


19. execution-mode negotiation;


20. transport negotiation;


21. security-feature negotiation;


22. resource-aware negotiation;


23. locality negotiation;


24. asynchronous negotiation;


25. streaming negotiation;


26. extension negotiation;


27. renegotiation;


28. negotiation failure;


29. downgrade prevention;


30. security requirements;


31. caching;


32. state transitions;


33. conformance requirements.



This document does NOT define:

the complete capability-discovery model;

general authorization policy;

authentication protocols;

the complete versioning system;

backward compatibility rules;

forward compatibility rules;

graceful degradation behavior;

transport-specific protocols;

the internal implementation of any feature.


Those concerns belong to their respective specifications.


---

3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



Feature negotiation is a mechanism for selecting among already-defined contracts.

It MUST NOT become a mechanism for inventing new ABI semantics dynamically.

Negotiation MAY select:

a version;

a profile;

an optional feature;

a compatible encoding;

an execution mode;

a transport;

a supported optimization;

a resource configuration;

a security mechanism.


Negotiation MUST NOT silently redefine:

type semantics;

ownership semantics;

error semantics;

security guarantees;

interface identity;

function identity;

mandatory Core behavior.



---

4. Relationship to Other Specifications

Feature negotiation integrates with:

ULABI-DESIGN.md
       |
       +-- ULABI-SPEC.md
       |
       +-- ULABI-VERSIONING.md
       |
       +-- Capability Discovery
       |       |
       |       v
       |   Feature Negotiation
       |
       +-- Backward Compatibility
       |
       +-- Forward Compatibility
       |
       +-- Graceful Degradation
       |
       +-- Core ABI
       |
       +-- Calling Convention
       |
       +-- Data Types
       |
       +-- Memory Model
       |
       +-- Runtime
       |
       +-- Security
       |
       +-- Distributed ABI
       |
       +-- Conformance

The following specifications MUST remain authoritative for their respective domains:

docs/compatibility/capability-discovery.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/graceful-degradation.md

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

docs/security/security-model.md
docs/security/capability-security.md

docs/runtime/runtime-interface.md

docs/standards/conformance.md
docs/standards/test-suite.md

This document consumes those contracts.

It MUST NOT redefine them.


---

5. Normative Language

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


A conforming implementation MUST satisfy every applicable MUST and MUST NOT requirement.


---

6. Fundamental Principle

Negotiation MUST be explicit.

An implementation MUST NOT assume that an optional feature is usable merely because it exists.

The canonical model is:

Advertised
    |
    v
Compatible
    |
    v
Eligible
    |
    v
Negotiable
    |
    v
Selected
    |
    v
Enabled
    |
    v
Used

A feature MUST NOT be used before it reaches the appropriate negotiated state.


---

7. Negotiation Participants

A negotiation consists of at least two logical participants.

They are identified as:

Initiator
Responder

The Initiator starts the negotiation.

The Responder evaluates the proposal and returns a negotiation response.

For symmetric protocols, either participant MAY initiate a negotiation.

The roles MUST be explicit within each negotiation transaction.


---

8. Negotiation Identity

Every negotiation transaction MUST have a unique negotiation identifier.

Conceptually:

NegotiationID {
    namespace
    identifier
}

The identifier MUST be unique within the relevant protocol scope.

It MUST NOT depend solely on:

memory addresses;

process-local pointers;

source-language names;

compiler-generated identifiers.


The negotiation identifier MAY be cryptographically random, monotonically generated, or derived from another collision-resistant mechanism.


---

9. Negotiation Context

A negotiation MUST occur within an explicit context.

Conceptually:

NegotiationContext {
    interface_identity
    interface_version
    protocol_version
    participant_identity
    locality
    transport
    security_context
    profile
}

The exact fields MAY vary according to the profile.

A negotiation result MUST NOT be interpreted outside its declared context without revalidation.


---

10. Feature Identity

Every negotiable feature MUST have a stable identifier.

Conceptually:

FeatureIdentity {
    namespace
    name
    major_version
}

Examples:

ulabi.async
ulabi.streaming
ulabi.zero-copy
ulabi.shared-memory
ulabi.canonical-json
ulabi.canonical-binary
ulabi.security.capability
ulabi.transport.quic

Feature identifiers MUST NOT be silently reused for unrelated semantics.


---

11. Feature Descriptor

A negotiable feature SHOULD expose a descriptor containing:

FeatureDescriptor {
    identity
    version
    category
    state
    mandatory
    dependencies
    conflicts
    constraints
    security_requirements
    resource_requirements
    compatibility_requirements
}

Unknown metadata MUST be safely ignorable unless explicitly marked mandatory.


---

12. Mandatory and Optional Features

Features SHALL be classified as either:

Mandatory
Optional

A mandatory feature is required for the selected contract.

An optional feature MAY be used if both parties agree.

The distinction is critical.

For example:

Core Types        Mandatory
Canonical Encoding Mandatory
GPU Acceleration  Optional
Zero Copy         Optional

A negotiation MUST fail if a mandatory feature cannot be satisfied.

A negotiation MAY continue without an optional feature.


---

13. Feature Requirements

A feature MAY declare required dependencies.

Example:

ulabi.zero-copy
requires:
    ulabi.shared-memory
    ulabi.memory-safety

A feature MUST NOT be selected unless all mandatory dependencies are satisfied.

Dependency resolution MUST occur before final selection.


---

14. Feature Conflicts

Features MAY declare conflicts.

Example:

ulabi.deterministic-execution
conflicts:
    ulabi.uncontrolled-randomness

Conflicting features MUST NOT both be selected unless the relevant specification explicitly defines a compatible combined mode.


---

15. Feature Preferences

A participant MAY express preferences.

Examples:

preferred:
    canonical-binary

acceptable:
    canonical-binary
    canonical-json
    canonical-cbor

Preferences MUST NOT override mandatory requirements.

A participant MUST distinguish:

Required
Preferred
Acceptable
Unsupported


---

16. Preference Ordering

A participant MAY provide an ordered preference list.

Example:

1. binary-v2
2. binary-v1
3. canonical-cbor
4. canonical-json

If multiple options satisfy all mandatory requirements, the negotiation SHOULD select the highest mutually acceptable preference according to the applicable selection rules.


---

17. Negotiation Proposal

A conceptual negotiation proposal is:

NegotiationProposal {
    negotiation_id
    protocol_version
    interface_identity
    requested_version
    required_features
    optional_features
    preferred_features
    acceptable_features
    required_profiles
    optional_profiles
    constraints
    security_requirements
    resource_requirements
    locality_requirements
}

The proposal MUST clearly distinguish mandatory requirements from preferences.


---

18. Negotiation Response

A conceptual response is:

NegotiationResponse {
    negotiation_id
    protocol_version
    result
    selected_version
    selected_features
    selected_profiles
    selected_encoding
    selected_transport
    selected_execution_mode
    constraints
    rejection_reason
}

The response MUST identify whether negotiation:

Succeeded
PartiallySucceeded
Rejected
Failed


---

19. Successful Negotiation

A negotiation succeeds only when:

1. all mandatory requirements are satisfied;


2. selected features are mutually supported;


3. selected features are compatible;


4. dependencies are satisfied;


5. conflicts are resolved;


6. security requirements are satisfied;


7. authorization requirements are satisfied;


8. resource constraints are satisfied;


9. the selected contract is valid for the context.



Conceptually:

Success =
Requirements
AND Compatibility
AND Dependencies
AND Security
AND Authorization
AND Resources


---

20. Partial Negotiation

Partial negotiation MAY occur when optional functionality cannot be selected.

Example:

Requested:
    Core
    Async
    Streaming
    GPU

Available:
    Core
    Async
    Streaming

Selected:
    Core
    Async
    Streaming

Not Selected:
    GPU

Partial success MUST NOT be reported as complete feature satisfaction.

The response MUST identify which optional features were not selected.


---

21. Negotiation Failure

Negotiation MUST fail when a mandatory requirement cannot be satisfied.

Examples:

Required profile unavailable
Required feature unavailable
Incompatible version
Security requirement unsatisfied
Mandatory dependency unavailable
Mandatory capability unauthorized
Resource requirement impossible
Conflicting mandatory features

Failure MUST produce a machine-readable reason.

Human-readable text MAY additionally be supplied.


---

22. Safe Failure

A failed negotiation MUST NOT result in partial activation of mandatory features.

The implementation MUST leave the negotiated contract in a safe state.

The default failure state is:

No Negotiated Contract

The caller MAY then:

terminate;

retry;

select an alternative;

invoke graceful degradation;

use a different interface.



---

23. Negotiation State Machine

A conforming implementation SHOULD model negotiation using explicit states.

Idle
  |
  v
Discovery
  |
  v
Proposal
  |
  v
Evaluation
  |
  +-------> Rejected
  |
  v
Selection
  |
  v
Validation
  |
  +-------> Failed
  |
  v
Committed
  |
  v
Active

A negotiated feature MUST NOT become active before the Committed state.


---

24. Atomic Commitment

Negotiated state SHOULD be committed atomically.

The implementation MUST NOT expose a partially committed negotiation to normal application execution.

Conceptually:

Proposal
   |
Evaluation
   |
Selection
   |
Validation
   |
Atomic Commit
   |
Active Contract

If commitment fails, the implementation MUST revert to the previous valid state.


---

25. Version Negotiation

Version negotiation MUST distinguish:

Major
Minor
Patch

The exact compatibility rules are defined by:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

Feature negotiation MUST consume those rules.

It MUST NOT invent an alternative versioning model.


---

26. Major Version Selection

A major version MAY be incompatible with another major version.

A negotiation MUST NOT assume compatibility between different major versions.

Example:

Initiator:
    2.x

Responder:
    1.x

The negotiation MAY:

select another supported interface;

select a compatibility adapter;

use a defined translation layer;

fail safely.


It MUST NOT silently pretend that 2.x and 1.x are equivalent.


---

27. Minor Version Selection

When multiple compatible minor versions exist, the participants SHOULD select the highest mutually compatible version satisfying all requirements.

Example:

Initiator:
    1.4
    1.3
    1.2

Responder:
    1.3
    1.2

Selected:
    1.3

The selection algorithm MUST be deterministic.


---

28. Profile Negotiation

Profiles MUST be negotiated independently from Core support.

Example:

Core
Security
Streaming
Async
Distributed

An implementation supporting Core MUST NOT automatically claim support for all profiles.

Profile selection MUST consider:

required profiles;

optional profiles;

profile dependencies;

profile conflicts;

profile versions;

security requirements.



---

29. Profile Dependencies

A profile MAY require another profile.

Example:

Distributed
    requires:
        Serialization
        Security

The dependent profile MUST NOT be selected unless its mandatory dependencies are selected.


---

30. Encoding Negotiation

Where multiple encodings are supported, encoding selection MUST be explicit.

Example:

Preferred:
    ulabi.binary

Acceptable:
    ulabi.binary
    ulabi.canonical-cbor

The selected encoding MUST be recorded as part of the negotiated contract.

The implementation MUST NOT silently change encoding during an active contract unless the applicable protocol explicitly supports renegotiation.


---

31. Canonical Encoding

Where the ULABI contract requires deterministic encoding, the selected encoding MUST provide the canonical properties required by that contract.

Encoding negotiation MUST NOT weaken:

determinism;

integrity;

validation;

type safety;

compatibility.



---

32. Transport Negotiation

Transport MAY be negotiated where the applicable profile supports multiple transports.

Examples:

Direct
SharedMemory
IPC
UnixSocket
TCP
QUIC
WebAssemblyHost
DeviceBus

Transport selection MUST NOT silently alter the semantic contract.

A transport change MUST NOT silently convert a local operation into an uncontrolled remote operation.


---

33. Locality Negotiation

ULABI locality values include:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A negotiation MAY select a locality supported by both parties.

A locality change MUST preserve all required security and execution semantics.

Latency, availability, failure, authentication, authorization, and consistency differences MUST remain explicit.


---

34. Execution-Mode Negotiation

Execution modes MAY include:

Synchronous
Asynchronous
Blocking
NonBlocking
Streaming
OneShot
LongRunning
Cancellable
Idempotent

An execution mode MUST NOT be selected if it violates mandatory interface semantics.

For example:

Required:
    Synchronous

Available:
    Asynchronous

is a negotiation failure unless the interface specification defines a valid adaptation.


---

35. Streaming Negotiation

Streaming features MAY negotiate:

element type;

direction;

framing;

backpressure;

buffering;

cancellation;

maximum message size;

completion semantics.


The selected streaming contract MUST be explicit before streaming begins.


---

36. Async Negotiation

Asynchronous operation support MAY be negotiated.

The negotiation MUST establish:

completion mechanism;

failure delivery;

cancellation;

lifetime;

ownership;

resource limits.


Async support MUST NOT be inferred solely from the existence of a callback mechanism.


---

37. Memory and Zero-Copy Negotiation

Zero-copy MAY be negotiated as an optimization.

It MUST NOT be selected unless all applicable memory requirements are satisfied.

These include:

ownership;

lifetime;

alignment;

mutability;

synchronization;

isolation;

access permissions.


If zero-copy cannot be safely established, the implementation MAY fall back to copying when the interface permits it.


---

38. Shared-Memory Negotiation

Shared memory MUST be explicitly negotiated.

The negotiation MUST establish:

Region Identity
Size
Permissions
Ownership
Lifetime
Synchronization
Security Context

A component MUST NOT access another component's memory merely because a shared memory mechanism exists.


---

39. Security Feature Negotiation

Security features MAY include:

Authentication
Authorization
Capability Tokens
Integrity Verification
Encryption
Sandboxing
Attestation
Post-Quantum Cryptography

Security negotiation MUST NOT downgrade mandatory security requirements.

For example:

Required:
    authenticated

MUST NOT negotiate:

Unauthenticated

merely because it is available.


---

40. Downgrade Prevention

A negotiation MUST prevent unauthorized security downgrades.

If a participant requires a minimum security level:

MinimumSecurityLevel = N

the selected contract MUST satisfy:

SelectedSecurityLevel >= N

An implementation SHOULD reject suspicious downgrade attempts.

Security policy MUST be evaluated before commitment.


---

41. Capability Requirements

Feature negotiation MUST consume the capability model defined by:

docs/compatibility/capability-discovery.md

Discovery establishes what exists.

Authorization establishes what the caller may use.

Negotiation selects what will actually be used.

Therefore:

Discovered
    |
    v
Supported
    |
    v
Authorized
    |
    v
Eligible
    |
    v
Negotiated

A discovered capability MUST NOT be treated as automatically authorized.


---

42. Resource-Aware Negotiation

A feature MAY require resources.

Examples:

GPU
Memory
CPU
Storage
Network
Threads
Shared Memory
Device

A feature MUST NOT be selected when its mandatory resource requirements cannot be satisfied.

Resource availability MAY change after negotiation.

Such changes MUST be handled through the applicable runtime and reliability specifications.


---

43. Constraint Negotiation

A participant MAY declare constraints.

Examples:

maximum_message_size = 16 MiB
maximum_memory = 1 GiB
maximum_latency = 10 ms
maximum_concurrent_calls = 32

The selected contract MUST satisfy all mandatory constraints.

If constraints conflict, negotiation MUST fail or select a valid alternative.


---

44. Deterministic Selection

When more than one valid negotiated result exists, selection SHOULD be deterministic.

A conceptual ordering is:

1. Mandatory requirements
2. Security requirements
3. Compatibility
4. Dependency satisfaction
5. Explicit preferences
6. Profile preferences
7. Performance preferences
8. Optional optimizations

Implementations MUST NOT select a less secure option merely because it is faster when doing so violates policy.


---

45. Negotiation Intersection

The usable feature set is conceptually:

Usable =
    Supported(A)
    ∩
    Supported(B)
    ∩
    Compatible
    ∩
    Authorized
    ∩
    PolicyAllowed
    ∩
    ResourceAvailable

The exact implementation MAY differ.

The observable result MUST preserve these semantics.


---

46. Negotiation Algorithm

A conforming implementation SHOULD follow this conceptual algorithm:

1. Identify participants.
2. Establish negotiation context.
3. Obtain capability information.
4. Validate identities and versions.
5. Determine mandatory requirements.
6. Determine optional requirements.
7. Remove unsupported features.
8. Remove incompatible features.
9. Resolve dependencies.
10. Resolve conflicts.
11. Evaluate security policy.
12. Evaluate authorization.
13. Evaluate resource constraints.
14. Apply participant preferences.
15. Select the highest-priority valid configuration.
16. Validate the resulting contract.
17. Atomically commit the negotiated state.
18. Activate the selected contract.

Failure at any mandatory stage MUST prevent activation.


---

47. Negotiation Commit

The selected configuration MUST be represented as a negotiated contract.

Conceptually:

NegotiatedContract {
    negotiation_id
    interface_identity
    interface_version
    selected_profiles
    selected_features
    selected_encoding
    selected_transport
    selected_execution_mode
    selected_locality
    security_level
    constraints
    validity
}

The negotiated contract MUST be immutable for the duration of the negotiated session unless renegotiation is explicitly supported.


---

48. Contract Immutability

Once committed, a negotiated contract MUST NOT be silently changed.

Changes MUST occur through:

explicit renegotiation;

session restart;

interface replacement;

failure recovery;

another mechanism defined by the applicable profile.


An implementation MUST NOT silently modify negotiated semantics.


---

49. Renegotiation

Renegotiation MAY be supported.

It MUST use an explicit state transition:

Active
   |
RenegotiationRequested
   |
Evaluating
   |
NewContract
   |
Committed
   |
Active

The previous contract MUST remain valid until the new contract is committed, unless the applicable protocol explicitly permits temporary suspension.


---

50. Renegotiation Failure

If renegotiation fails, the implementation SHOULD preserve the previous valid contract where safe.

Conceptually:

Old Contract
     |
Renegotiate
     |
   Failed
     |
Old Contract remains active

unless security or correctness requires termination.


---

51. Feature Revocation

A previously negotiated feature MAY become unavailable or revoked.

Revocation MUST be distinguishable from ordinary negotiation failure.

Examples:

Capability revoked
Resource removed
Security policy changed
Credential expired
Hardware unavailable

The runtime MUST follow the applicable resource, security, and reliability specification for the resulting state.


---

52. Negotiation Expiration

Negotiated state MAY have a validity period.

Conceptually:

valid_from
valid_until

An expired negotiation MUST NOT be treated as active indefinitely.

The implementation MAY:

renegotiate;

renew;

terminate;

fall back according to the applicable graceful-degradation policy.



---

53. Caching

Negotiation results MAY be cached.

A cache entry MUST be associated with sufficient context to ensure that it is safe to reuse.

At minimum, cached negotiation state SHOULD consider:

interface identity;

interface version;

participant identity;

capability state;

security context;

profile;

transport;

validity period.


A stale or invalid cache entry MUST NOT be treated as authoritative.


---

54. Security of Negotiation Metadata

Negotiation metadata MAY influence:

code paths;

memory access;

resource allocation;

security policy;

transport selection;

cryptographic selection.


Therefore negotiation metadata MUST be treated as untrusted input until validated.

Implementations MUST validate:

lengths;

identifiers;

versions;

feature dependencies;

constraints;

cryptographic parameters;

resource claims.



---

55. Negotiation Authentication

Where negotiation affects protected resources or security-sensitive operations, the negotiation context SHOULD be authenticated according to the applicable security profile.

Authentication MUST NOT be inferred merely from successful capability discovery.


---

56. Negotiation Integrity

Where negotiation crosses an untrusted boundary, negotiation messages SHOULD have integrity protection.

A tampered negotiation MUST NOT result in activation of an unauthorized or weaker contract.


---

57. Unknown Features

Unknown optional features SHOULD be ignored safely.

Unknown mandatory features MUST NOT be ignored.

For example:

Feature:
    optional-new-compression

may be ignored by an older implementation.

But:

Feature:
    mandatory-new-type-semantics

MUST result in safe incompatibility handling when the implementation cannot understand it.


---

58. Unknown Metadata

Unknown optional metadata SHOULD be preserved or ignored safely.

Unknown metadata MUST NOT alter established semantics unless the applicable specification explicitly defines it as mandatory.


---

59. Forward Compatibility

Feature negotiation MUST integrate with:

docs/compatibility/forwards-compatibility.md

It MUST NOT redefine forward-compatibility rules.

Negotiation SHOULD permit an older implementation to safely reject unsupported optional functionality without corrupting the established contract.


---

60. Backward Compatibility

Feature negotiation MUST integrate with:

docs/compatibility/backwards-compatibility.md

A newer implementation MUST NOT use negotiation as an excuse to silently break an older mandatory contract.

If a previously mandatory feature has been removed, the implementation MUST report incompatibility or use an explicitly defined compatibility mechanism.


---

61. Graceful Degradation

Feature negotiation MUST integrate with:

docs/compatibility/graceful-degradation.md

Negotiation MAY identify a reduced valid feature set.

Example:

Requested:
    Core
    Streaming
    ZeroCopy
    GPU

Available:
    Core
    Streaming

Selected:
    Core
    Streaming

The reduced configuration is valid only if the remaining features preserve correctness and the interface contract allows the omitted functionality.


---

62. No Silent Semantic Substitution

Negotiation MUST NOT silently substitute a feature with another feature that has different semantics.

For example:

Requested:
    exactly-once

must not silently become:

at-least-once

unless the contract explicitly permits that substitution.

Performance equivalence does not imply semantic equivalence.


---

63. Optimization Negotiation

Optimizations MAY be negotiated separately from semantic requirements.

Examples:

ZeroCopy
Compression
Vectorization
GPU
Caching
Batching

An optimization MUST NOT alter the externally observable semantics required by the interface.


---

64. Semantic Equivalence

Two feature implementations MAY be considered equivalent when their observable ULABI contract is equivalent.

Internal implementation differences MUST NOT affect the negotiated identity when they do not affect the observable contract.


---

65. Adapter-Based Negotiation

An implementation MAY use an adapter to satisfy a negotiation.

Example:

Implementation A
      |
      v
ULABI Adapter
      |
      v
Negotiated Contract
      |
      v
Implementation B

The adapter MUST preserve the negotiated semantics.

An adapter MUST NOT silently weaken mandatory security, ownership, lifetime, or error guarantees.


---

66. Translation Layers

A translation layer MAY convert between compatible representations.

Examples:

ULABI Binary v1
       |
   Translator
       |
ULABI Binary v2

Translation MUST preserve semantic meaning.

Lossy translation MUST NOT be presented as lossless compatibility.


---

67. Feature Negotiation and Memory Safety

Negotiation MUST NOT permit a feature to bypass the applicable memory model.

For example, selecting:

zero-copy

MUST NOT implicitly grant:

arbitrary-memory-access

Memory permissions remain governed by:

docs/abi/memory-model.md

and the applicable security specifications.


---

68. Feature Negotiation and Capability Security

Feature negotiation MUST integrate with:

docs/security/capability-security.md

Negotiating a feature does not grant authority.

The correct model is:

Feature Supported
        |
        v
Feature Compatible
        |
        v
Capability Available
        |
        v
Authorization
        |
        v
Feature Negotiated


---

69. Feature Negotiation and Runtime

Runtime implementations MUST expose sufficient information for negotiated execution modes to be honored.

The runtime MUST NOT claim support for:

Async
Streaming
Cancellation
Concurrency

unless the selected runtime contract can actually provide their required semantics.


---

70. Feature Negotiation and Distributed Execution

Distributed negotiation MUST account for the fact that remote execution is not semantically identical to local execution.

Distributed negotiation MUST consider:

transport;

serialization;

latency;

availability;

authentication;

authorization;

timeout;

retry;

cancellation;

failure semantics.


The distributed specifications remain authoritative for these details.


---

71. Negotiation and Hardware

Hardware capabilities MAY be negotiated.

Examples:

GPU
NPU
FPGA
Quantum
Vector
Tensor

Hardware capability selection MUST NOT make the interface dependent on one hardware architecture.

Hardware-specific behavior belongs to the applicable hardware or accelerator profile.


---

72. Negotiation and Embedded Systems

Embedded implementations MAY expose a restricted capability set.

A valid negotiation MAY therefore select:

Core
Memory
RealTime

without supporting:

Distributed
GPU
Cloud

A conforming implementation MUST NOT be required to implement capabilities that are outside its claimed profile.


---

73. Negotiation and Real-Time Systems

For real-time profiles, negotiation MUST preserve declared timing constraints.

A feature MUST NOT be selected if its known behavior violates a mandatory real-time constraint.


---

74. Negotiation and Determinism

If deterministic execution is mandatory, negotiation MUST reject features that violate deterministic requirements.

For example:

Required:
    deterministic

Conflicting:
    uncontrolled-randomness

The implementation MUST NOT silently select the conflicting feature.


---

75. Negotiation Auditability

Security-sensitive negotiations SHOULD produce auditable records containing:

negotiation identity;

participants;

selected contract;

rejected mandatory features;

selected security properties;

applicable policy;

timestamp or validity period;

reason for rejection where applicable.


Audit records MUST NOT expose protected information unnecessarily.


---

76. Negotiation Logging

Diagnostic logging MAY record negotiation decisions.

Logs SHOULD distinguish:

Discovered
Supported
Eligible
Selected
Rejected
Unavailable
Revoked

Human-readable logs MUST NOT be the only source of machine-verifiable negotiation state.


---

77. Negotiation Errors

Negotiation errors SHOULD use structured ULABI error semantics.

Possible categories include:

Unsupported
Incompatible
InvalidArgument
PermissionDenied
SecurityViolation
ResourceExhausted
Unavailable
ProtocolError
Timeout
Cancelled
IntegrityFailure

Error identifiers MUST follow the ULABI error model.


---

78. Retry Semantics

A failed negotiation MAY be retried.

Retries MUST NOT blindly repeat security-sensitive negotiation indefinitely.

The implementation SHOULD distinguish:

Permanent Failure
Temporary Failure
Policy Failure
Resource Failure
Protocol Failure

Retry behavior SHOULD follow the applicable runtime and distributed profiles.


---

79. Replay Protection

Security-sensitive negotiation SHOULD protect against replay.

A replayed negotiation MUST NOT cause:

unauthorized feature activation;

security downgrade;

stale capability use;

stale credential use;

unintended resource allocation.



---

80. Resource Exhaustion Protection

Negotiation itself MUST be resource-bounded.

Implementations SHOULD enforce limits on:

proposal size;

number of features;

number of dependencies;

nesting depth;

negotiation duration;

number of alternatives;

number of participants.


A malicious peer MUST NOT be able to cause unbounded resource consumption merely by submitting negotiation metadata.


---

81. Negotiation Determinism

Given identical:

inputs
capabilities
requirements
policies
preferences
constraints

a deterministic implementation SHOULD produce the same negotiated result.

If implementation-specific behavior affects selection, that behavior MUST be documented.


---

82. Compatibility Result

Negotiation SHOULD produce one of:

FullyCompatible
ConditionallyCompatible
PartiallyCompatible
Incompatible
Unknown

These results MUST remain distinct from the final execution state.


---

83. Machine-Readable Result

The negotiation result MUST be machine-readable.

A conceptual result is:

NegotiationResult {
    status
    compatibility
    selected_contract
    unmet_requirements
    rejected_features
    warnings
}

Human-readable explanations MAY accompany it.


---

84. Warnings

Warnings MAY identify:

deprecated features;

optional features not selected;

performance fallbacks;

compatibility adapters;

resource constraints;

non-preferred selections.


Warnings MUST NOT override mandatory failures.


---

85. Negotiation Invariants

A conforming implementation MUST preserve these invariants:

1. Unsupported mandatory features are never selected.


2. Unauthorized features are never activated.


3. Conflicting mandatory features cannot both be selected.


4. Missing dependencies prevent dependent feature selection.


5. Negotiated contracts are explicit.


6. Negotiated semantics are not silently changed.


7. Security requirements cannot be silently weakened.


8. Unknown optional features do not cause unsafe behavior.


9. Unknown mandatory features cannot be silently ignored.


10. Failed negotiation cannot produce an active invalid contract.


11. Negotiation state is context-bound.


12. Stale negotiation state cannot be treated as current.


13. Feature identity is stable.


14. Interface identity is stable.


15. Internal implementation details do not define feature semantics.


16. Negotiation does not grant authorization.


17. Capability discovery does not equal feature selection.


18. Graceful degradation cannot violate mandatory correctness guarantees.




---

86. Security Requirements

A conforming implementation MUST:

validate negotiation input;

enforce mandatory security requirements;

prevent unauthorized downgrade;

distinguish capability discovery from authorization;

protect sensitive negotiation metadata;

prevent malformed negotiation from causing unsafe execution;

enforce resource limits;

reject invalid dependency graphs;

reject conflicting mandatory features;

prevent stale negotiated contracts from being reused incorrectly.


Security-sensitive deployments SHOULD additionally provide:

authenticated negotiation;

integrity protection;

replay protection;

audit logging;

cryptographic agility;

policy-controlled minimum security levels.



---

87. Failure Modes

Possible negotiation failures include:

InvalidProposal
UnknownInterface
UnsupportedVersion
UnsupportedFeature
MissingDependency
FeatureConflict
SecurityFailure
AuthorizationFailure
ResourceUnavailable
ConstraintConflict
PolicyDenied
Timeout
Cancelled
IntegrityFailure
ProtocolError

Failures MUST be distinguishable where programmatic recovery depends on the distinction.


---

88. Recovery Behavior

When negotiation fails, the implementation MAY:

Retry
SelectAlternative
InvokeAdapter
UseGracefulDegradation
Terminate

Recovery MUST NOT silently violate mandatory contract requirements.

Security failures SHOULD normally result in rejection rather than automatic downgrade.


---

89. Conformance Requirements

A ULABI implementation claiming feature-negotiation conformance MUST demonstrate:

Identity

stable negotiation identifiers;

stable feature identifiers;

interface identification.


Discovery Integration

consumption of capability-discovery results;

distinction between supported and authorized capabilities.


Requirements

mandatory feature enforcement;

optional feature handling;

dependency resolution;

conflict detection.


Selection

deterministic selection;

preference handling;

profile negotiation;

version negotiation.


Security

downgrade prevention;

authorization enforcement;

malformed-input handling.


State

explicit negotiation states;

atomic commitment;

safe failure;

renegotiation handling where supported.


Compatibility

backward compatibility integration;

forward compatibility integration;

graceful degradation integration.


Resource Safety

bounded negotiation;

resource-limit enforcement.



---

90. Conformance Test Categories

The conformance suite SHOULD contain at least:

feature_negotiation/
    identity/
    versions/
    mandatory/
    optional/
    preferences/
    dependencies/
    conflicts/
    profiles/
    encodings/
    transports/
    execution_modes/
    security/
    authorization/
    resources/
    downgrade/
    unknown_features/
    malformed_input/
    deterministic_selection/
    commitment/
    renegotiation/
    expiration/
    caching/
    graceful_degradation/


---

91. Required Test Cases

At minimum, the test suite SHOULD verify:

1. Two identical implementations negotiate successfully.


2. Compatible versions select the correct version.


3. Incompatible major versions fail safely.


4. Optional unsupported features are omitted.


5. Unsupported mandatory features fail negotiation.


6. Missing dependencies prevent selection.


7. Conflicting features are rejected.


8. Preferences produce deterministic selection.


9. Unauthorized features are rejected.


10. Security downgrade is rejected.


11. Unknown optional features are ignored safely.


12. Unknown mandatory features cause incompatibility.


13. Resource constraints are respected.


14. Invalid proposals are rejected.


15. Negotiation commitment is atomic.


16. Failed commitment leaves no invalid active contract.


17. Expired negotiation state is rejected.


18. Renegotiation preserves the previous valid contract on safe failure.


19. Graceful degradation never removes mandatory semantics.


20. Negotiation metadata cannot bypass memory or security boundaries.




---

92. Reference Data Structures

The normative conceptual structures are:

NegotiationID

NegotiationContext

FeatureIdentity

FeatureDescriptor

NegotiationProposal

NegotiationResponse

NegotiatedContract

NegotiationResult

These structures describe semantics.

They do not mandate a programming language, memory layout, serialization format, or implementation language.

The canonical wire representation belongs to the applicable ULABI encoding specification.


---

93. Reference Negotiation Flow

A complete negotiation may therefore be represented as:

Participant A
     |
     | Capability Discovery
     v
Participant B
     |
     | Capability Response
     v
Participant A
     |
     | Compatibility Analysis
     v
Feature Proposal
     |
     v
Participant B
     |
     | Evaluate Requirements
     |
     +--> Dependencies
     |
     +--> Conflicts
     |
     +--> Security
     |
     +--> Authorization
     |
     +--> Resources
     |
     +--> Preferences
     |
     v
Feature Selection
     |
     v
Validation
     |
     v
Atomic Commit
     |
     v
Negotiated Contract
     |
     v
Execution


---

94. Example

Suppose:

Initiator supports:

Core 1.0
Async
Streaming
ZeroCopy
GPU

Responder supports:

Core 1.0
Async
Streaming

The Initiator requests:

Required:
    Core 1.0

Preferred:
    GPU
    ZeroCopy
    Streaming
    Async

The negotiation may produce:

Selected:

Core 1.0
Async
Streaming

Not selected:

GPU
ZeroCopy

This is successful because GPU and ZeroCopy were optional.


---

95. Example of Mandatory Failure

Initiator:

Required:
    Streaming
    ZeroCopy

Responder:

Streaming

If ZeroCopy is mandatory, negotiation MUST fail.

It MUST NOT silently convert:

ZeroCopy

into:

Copy

unless the interface explicitly defines copying as an acceptable negotiated alternative.


---

96. Example of Security Downgrade Prevention

Initiator:

Required:
    authenticated
    integrity-protected

Responder:

authenticated
    available

integrity-protection
    unavailable

The negotiation MUST fail.

It MUST NOT select:

Unauthenticated

or:

Authenticated without integrity

unless the initiating security policy explicitly permits that configuration.


---

97. Example of Dependency Resolution

Requested:

ZeroCopy

Dependencies:

ZeroCopy
    requires:
        SharedMemory
        MemoryOwnership

If:

SharedMemory = supported
MemoryOwnership = supported

then ZeroCopy is eligible.

If:

SharedMemory = unavailable

then ZeroCopy MUST NOT be selected.


---

98. Example of Preference Selection

Initiator:

Preferred:
    binary-v2
    binary-v1
    canonical-cbor

Responder:

Supported:
    binary-v1
    canonical-cbor

Selected:

binary-v1

The selection is deterministic because binary-v1 is the highest-ranked mutually acceptable option.


---

99. Example of Graceful Degradation

Requested:

Core
Async
Streaming
GPU
ZeroCopy

Available:

Core
Async
Streaming

If GPU and ZeroCopy are optional:

Selected:
    Core
    Async
    Streaming

The implementation MAY continue using the reduced configuration.

The reduced configuration MUST remain semantically valid.


---

100. Implementation Independence

This specification does not require a particular implementation language.

Possible implementations include:

C
C++
Rust
Go
Java
Kotlin
Swift
Python
Fortran
Ada
Zig
WebAssembly
Sankofa
Zamani

The implementation language MUST NOT change the negotiated ULABI semantics.

Zamani and Sankofa are consumers or implementers of ULABI.

They are not normative authorities over ULABI.


---

101. Reference Implementation Boundary

A reference implementation SHOULD separate:

Discovery
    |
Compatibility
    |
Negotiation
    |
Selection
    |
Validation
    |
Commit

Each stage SHOULD be independently testable.

No stage should silently perform the responsibilities of another stage.


---

102. Integration Contract

This document establishes the following integration contract for future implementations.

Capability Discovery

Consumes:

CapabilityDescriptor
CapabilityState
ProfileDescriptor
OperationDescriptor

from:

docs/compatibility/capability-discovery.md

Compatibility

Consumes compatibility results from:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

Graceful Degradation

Delegates reduced-feature behavior to:

docs/compatibility/graceful-degradation.md

ABI

Uses contracts from:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

Security

Uses:

docs/security/security-model.md
docs/security/capability-security.md
docs/security/authentication.md
docs/security/authorization.md

Runtime

Uses:

docs/runtime/runtime-interface.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Distributed

Uses:

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md

Conformance

Consumes:

docs/standards/conformance.md
docs/standards/test-suite.md

This document is complete without requiring those documents to redefine feature negotiation semantics.


---

103. Required Implementation Modules

A reference implementation SHOULD provide independent modules for:

negotiation/
├── identity
├── context
├── descriptor
├── proposal
├── response
├── requirements
├── preferences
├── dependency
├── conflict
├── compatibility
├── version
├── profile
├── encoding
├── transport
├── execution
├── locality
├── security
├── authorization
├── resources
├── selection
├── validation
├── commitment
├── renegotiation
├── expiration
├── caching
├── downgrade_protection
├── errors
└── audit

These modules are implementation boundaries, not requirements for a particular programming language or repository layout.


---

104. Final Invariants

The following rules are fundamental:

> Discovery determines what exists.



> Compatibility determines what can interoperate.



> Negotiation determines what will be used.



> Authorization determines what may be used.



> Policy determines what is permitted.



> Runtime determines how it executes.



> The negotiated contract determines the active interoperability semantics.



Therefore:

Discovery
    !=
Negotiation
    !=
Authorization
    !=
Execution

ULABI feature negotiation MUST preserve these boundaries.


---

105. Final Architectural Rule

ULABI feature negotiation exists to select among standardized, compatible contracts.

It MUST NOT become a mechanism for silently changing the meaning of ULABI.

The governing principle is:

> Negotiate capabilities and options; never negotiate away the fundamental semantics required for a valid ULABI contract.



## What should be done under which files

I would **not** now start editing every document to accommodate this one. The repository architecture should be completed in dependency order, with each document defining its integration contract in advance.

The order I recommend is:

### Phase 1 — independent architectural contracts

These should be completed first because other documents consume their definitions.

| File | What it owns |
|---|---|
| `ULABI-DESIGN.md` | Master architecture and boundaries |
| `docs/architecture/design-principles.md` | Fundamental principles; no component-specific duplication |
| `docs/architecture/overview.md` | Complete architecture and layer relationships |
| `docs/architecture/universal-model.md` | Universal abstraction model |
| `docs/architecture/scalability.md` | Scale dimensions, limits and scaling invariants |
| `ULABI-VERSIONING.md` | Version identity and version evolution |
| `ULABI-GOVERNANCE.md` | Ownership, standards process and change authority |
| `ULABI-SPEC.md` | Normative Core requirements |

### Phase 2 — Core ABI

| File | Owns |
|---|---|
| `docs/abi/core-abi.md` | Core ABI contract |
| `docs/abi/data-types.md` | ABI data representations |
| `docs/abi/calling-convention.md` | Calls and argument ABI |
| `docs/abi/memory-model.md` | Boundary memory semantics |
| `docs/abi/stack-model.md` | Stack semantics |
| `docs/abi/register-model.md` | Register abstraction |
| `docs/abi/return-values.md` | Return-value ABI |
| `docs/abi/exception-model.md` | Error/exception boundary |

### Phase 3 — Type system

```text
docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/generics.md
docs/type-system/enums.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/type-compatibility.md

Phase 4 — Interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Phase 5 — Runtime

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Phase 6 — Memory

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

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

Phase 8 — Compatibility

The correct dependency order is:

docs/compatibility/backwards-compatibility.md
                    |
                    v
docs/compatibility/forwards-compatibility.md
                    |
                    v
docs/compatibility/capability-discovery.md
                    |
                    v
docs/compatibility/feature-negotiation.md
                    |
                    v
docs/compatibility/graceful-degradation.md

With:

docs/compatibility/capability-discovery.md
        |
        +----> feature-negotiation.md
        |
        +----> conformance.md
        |
        +----> security
        |
        +----> runtime

The current repository already has the first and third substantial documents, while feature-negotiation.md is currently just a placeholder.

Phase 9 — Reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md
docs/reliability/self-healing.md

Self-healing should consume the already-defined recovery, rollback, security and runtime contracts rather than redefine them.

Phase 10 — Distributed

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Phase 11 — Platform and hardware profiles

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

Phase 12 — Tooling

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md
docs/tooling/debugger-interface.md
docs/tooling/profiler-interface.md
docs/tooling/validator.md

Phase 13 — Observability

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md
docs/observability/deterministic-debugging.md

Phase 14 — Standards

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

These should be written after the contracts they test have been defined, but the contracts themselves should contain their conformance hooks in advance—as the document above does.


---

Required non-document implementation structure

The important point is that the repository currently has extensive specification material but the implementation side needs to be treated as a separate layer. The Git tree shows the project currently contains the specification hierarchy but not a mature implementation tree.

I recommend the eventual implementation structure:

schemas/
├── abi/
├── types/
├── interfaces/
├── capabilities/
├── negotiation/
├── errors/
├── security/
├── profiles/
└── versioning/

reference/
├── abi/
├── types/
├── memory/
├── compatibility/
├── negotiation/
├── security/
├── runtime/
└── validation/

implementations/
├── common/
├── c/
├── cpp/
├── rust/
├── go/
├── python/
├── java/
├── kotlin/
├── swift/
├── ada/
├── fortran/
├── wasm/
└── adapters/

tests/
├── abi/
├── types/
├── memory/
├── interoperability/
├── runtime/
├── security/
├── compatibility/
├── negotiation/
├── distributed/
├── reliability/
└── property/

conformance/
├── core/
├── types/
├── memory/
├── ffi/
├── security/
├── async/
├── streaming/
├── distributed/
├── self-healing/
└── profiles/

examples/
├── c-rust/
├── rust-go/
├── python-rust/
├── java-c/
├── wasm-host/
├── local/
├── process/
└── distributed/

Required feature-negotiation implementation modules

The code-level implementation should eventually map directly to the contract we just defined:

negotiation/
├── mod
├── identity
├── context
├── feature
├── descriptor
├── capability_adapter
├── proposal
├── response
├── requirement
├── preference
├── dependency
├── conflict
├── compatibility
├── version
├── profile
├── encoding
├── transport
├── execution_mode
├── locality
├── security
├── authorization
├── resource_constraints
├── selector
├── validator
├── commit
├── renegotiation
├── expiration
├── cache
├── downgrade_protection
├── errors
└── audit

The critical architectural boundary is:

capability_discovery
        ↓
compatibility
        ↓
negotiation
        ↓
validation
        ↓
commit
        ↓
runtime

Do not merge those into one giant compatibility.rs, negotiator.rs, or equivalent module. That would recreate exactly the architectural coupling we're trying to avoid.

Integration rule for every future code module

Every implementation module should document:

Purpose
Public API
Inputs
Outputs
Owned invariants
Dependencies
Security boundary
Failure modes
Recovery behavior
Thread/concurrency requirements
Resource limits
Serialization boundary
Compatibility behavior
Conformance tests
Consumers

That means when you eventually implement, for example, negotiation/selector, you can finish that module against a stable contract without having to redesign it because a later document suddenly introduced an undisclosed requirement.

One important correction from your requested filename: keep the canonical repository name as docs/compatibility/feature-negotiation.md. The repository already has that file, and capability-discovery.md similarly establishes the canonical naming. Do not create a duplicate compatibility-feature-negotiation.md or another similarly named file.

This gives ULABI a clean dependency graph instead of a collection of documents that repeatedly redefine one another.