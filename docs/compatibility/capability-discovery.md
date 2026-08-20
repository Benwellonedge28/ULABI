I read the repository before writing this. In particular, I checked the current ULABI-DESIGN.md, ULABI-SPEC.md, the existing compatibility documents, and the existing capability-discovery.md. The master design explicitly establishes language/runtime/platform neutrality, a minimal stable Core, profiles, explicit semantics, compatibility, capability discovery, and implementation independence.  The normative specification already makes capability discovery part of the Core Profile.

There is one important repository correction: the canonical path already present in the repository is:

docs/compatibility/capability-discovery.md

not docs/compatibility/compatibility-capability-discovery.md. The existing file is currently only a placeholder, so replace docs/compatibility/capability-discovery.md, rather than creating a duplicate document.

I also checked the related compatibility documents. backwards-compatibility.md, feature-negotiation.md, and forwards-compatibility.md are separate concerns, so the document below deliberately does not duplicate their responsibilities.

1. File to create/replace

Canonical file:

docs/compatibility/capability-discovery.md

Purpose: define how a ULABI implementation discovers what another implementation, interface, component, profile, endpoint, or execution environment is capable of supporting.

The specification below is deliberately self-contained. It establishes its integration points in advance so that later documents do not need to redefine its fundamental model.


---

# ULABI Capability Discovery

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/compatibility/capability-discovery.md`  
**Specification Family:** Compatibility / Capability Discovery  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**ULABI Core Specification:** 0.1.0  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI capability-discovery contract.

Capability discovery allows an implementation to determine, in a
machine-readable and security-aware manner, which ULABI capabilities are
available before attempting to use them.

Capability discovery exists to answer questions such as:

- Which ULABI profile does this endpoint support?
- Which interface versions are supported?
- Which functions are available?
- Which types are supported?
- Which execution modes are supported?
- Which transports are supported?
- Which optional extensions are available?
- Which security mechanisms are available?
- Which capabilities are required to invoke an operation?
- Which capabilities are granted to the current caller?
- Which limits apply?
- Which compatibility conditions exist?

Capability discovery MUST NOT silently grant permissions.

Discovery tells a component what is available.

Authorization determines what a component is permitted to use.

These are separate concepts.

---

# 2. Scope

This document defines:

1. capability terminology;
2. capability identity;
3. capability descriptions;
4. capability sets;
5. discovery requests;
6. discovery responses;
7. interface discovery;
8. profile discovery;
9. feature discovery;
10. version discovery;
11. execution-mode discovery;
12. transport discovery;
13. resource-limit discovery;
14. security capability discovery;
15. conditional capability discovery;
16. capability requirements;
17. capability availability;
18. capability authorization boundaries;
19. discovery freshness;
20. discovery integrity;
21. discovery failure behavior;
22. compatibility classification;
23. caching rules;
24. capability-change handling;
25. security requirements;
26. conformance requirements.

This document does NOT define:

- general authorization policy;
- complete authentication protocols;
- transport-specific protocols;
- complete versioning policy;
- backward compatibility rules;
- forward compatibility rules;
- feature negotiation procedures;
- graceful degradation procedures.

Those concerns belong to their respective specifications.

---

# 3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.

Capability discovery is a Core interoperability mechanism because an implementation
must be able to determine whether an optional or required contract is available
before depending on it.

The capability-discovery mechanism MUST remain:

- language-neutral;
- compiler-neutral;
- runtime-neutral;
- operating-system-neutral;
- processor-neutral;
- transport-neutral;
- vendor-neutral.

A capability MAY be implemented differently internally by different
implementations.

Only its ULABI-visible semantics are standardized.

---

# 4. Relationship to Other Specifications

Capability discovery participates in the following architecture:

```text
ULABI-DESIGN.md
        |
        +-- ULABI-SPEC.md
        |
        +-- Interface Identity
        |
        +-- Versioning
        |
        +-- Capability Discovery
        |       |
        |       +-- Feature Negotiation
        |       +-- Profile Selection
        |       +-- Compatibility
        |       +-- Authorization
        |
        +-- Runtime
        |
        +-- Security
        |
        +-- Distributed Execution
        |
        +-- Conformance

Capability discovery MUST provide information that can be consumed by:

docs/compatibility/feature-negotiation.md

docs/compatibility/backwards-compatibility.md

docs/compatibility/forwards-compatibility.md

docs/compatibility/graceful-degradation.md

docs/standards/conformance.md

docs/standards/test-suite.md

docs/security/security-model.md

docs/security/capability-security.md

docs/runtime/runtime-interface.md

docs/distributed/distributed-abi.md

docs/abi/core-abi.md


Those documents MUST consume this capability model rather than inventing independent capability semantics.


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

Capability discovery answers:

> What can this implementation expose or support?



It does NOT answer:

> What is this caller allowed to do?



Therefore:

Discovery
    |
    v
Available capability
    |
    v
Authorization
    |
    v
Usable capability

The following MUST NOT be assumed:

Discovered
   =
Authorized

and:

Supported
   =
Enabled

and:

Enabled
   =
Authorized

and:

Authorized
   =
Currently Available

A conforming implementation MUST preserve these distinctions.


---

7. Capability Definition

A ULABI capability is a formally identified ability, feature, property, resource, execution mode, or protocol behavior that an implementation may support, expose, grant, or require.

Examples include:

interface.ulabi.core
profile.ulabi.streaming
feature.ulabi.async
transport.ulabi.shared-memory
execution.ulabi.streaming
security.ulabi.capability-token
type.ulabi.tensor
resource.ulabi.gpu
operation.ulabi.cancel

Capability identifiers MUST be stable.

Capability identifiers MUST NOT depend solely on:

source-language names;

memory addresses;

compiler-generated symbols;

implementation-specific object names;

process-local numeric IDs.



---

8. Capability Identity

Every discoverable capability MUST have a unique identifier within its defined namespace.

A capability identity SHOULD contain:

namespace
name
major version

A conceptual representation is:

CapabilityIdentity {
    namespace
    name
    major_version
}

Minor revisions SHOULD NOT require a new identity when their semantics remain backward compatible.

A major semantic change MUST create a new capability identity or otherwise follow the applicable versioning rules.

Capability identifiers MUST NOT be silently reused for unrelated semantics.


---

9. Capability Categories

ULABI capabilities SHOULD be classified into explicit categories.

The initial categories are:

Interface
Profile
Feature
Type
Operation
Execution
Transport
Security
Resource
Platform
Hardware
Encoding
Observability
Reliability

Implementations MAY define additional categories through registered extensions.

Unknown categories MUST NOT cause unsafe behavior.


---

10. Capability States

A capability MAY exist in several states.

The canonical conceptual states are:

Unknown
Supported
Available
Enabled
Authorized
Unavailable
Revoked
Deprecated

These states have distinct meanings.

10.1 Unknown

The implementation has insufficient information to determine capability state.

10.2 Supported

The implementation can provide the capability in principle.

10.3 Available

The capability is currently available in the relevant execution context.

10.4 Enabled

The capability has been activated for the relevant interface or context.

10.5 Authorized

The current caller has permission to use the capability.

10.6 Unavailable

The capability is known but cannot currently be used.

10.7 Revoked

The capability was previously available or authorized but has been explicitly withdrawn.

10.8 Deprecated

The capability remains recognized but SHOULD NOT be used for new interactions.


---

11. Capability State Is Contextual

Capability state MAY depend on:

caller identity;

authentication state;

authorization;

profile;

interface version;

transport;

execution locality;

platform;

hardware;

resource availability;

security policy;

quota;

time;

feature configuration.


Therefore:

CapabilityState

MUST be interpreted within a declared context.

An implementation MUST NOT advertise a capability as universally available if it is only available under a specific condition.


---

12. Capability Descriptor

A capability descriptor SHOULD contain at least:

CapabilityDescriptor {
    identity
    category
    version
    state
    description
    dependencies
    conflicts
    requirements
    constraints
    security_requirements
    resource_requirements
    validity
}

Implementations MAY include additional metadata.

Unknown metadata MUST be safely ignorable.


---

13. Capability Dependencies

A capability MAY depend on other capabilities.

Example:

distributed.remote-call
    requires:
        serialization.canonical
        transport.quic
        security.authentication

Dependencies MUST be explicit.

A capability MUST NOT be advertised as unconditionally usable when a mandatory dependency is absent.

Dependency evaluation SHOULD produce a machine-readable result.


---

14. Capability Conflicts

A capability MAY conflict with another capability.

Example:

execution.deterministic
conflicts:
    execution.nondeterministic-randomness

Conflicts MUST be explicitly represented when they affect safe execution.

An implementation MUST NOT silently enable mutually incompatible capabilities.


---

15. Capability Requirements

An operation MAY declare required capabilities.

Example:

Operation:
    gpu.compute

Requires:
    resource.gpu
    execution.accelerated

A caller MUST establish that required capabilities are usable before invoking an operation that depends on them.

Failure to satisfy a mandatory capability requirement MUST result in a defined failure.


---

16. Optional Capabilities

An operation MAY declare optional capabilities.

Example:

Operation:
    process-data

Required:
    core.types

Optional:
    streaming
    zero-copy
    gpu

Optional capabilities MAY improve performance or functionality without being necessary for correctness.

An implementation MUST NOT treat an optional capability as mandatory unless the contract explicitly says so.


---

17. Capability Discovery Request

A conceptual discovery request is:

CapabilityDiscoveryRequest {
    protocol_version
    target_interface
    requested_categories
    requested_capabilities
    requested_profiles
    caller_context
}

The request MAY limit discovery to a subset of capabilities.

For example:

Requested:
    Profile
    Security
    Transport

An implementation MAY return only the requested categories.


---

18. Capability Discovery Response

A conceptual response is:

CapabilityDiscoveryResponse {
    protocol_version
    interface_identity
    interface_version
    capabilities
    profiles
    constraints
    validity
    status
}

The response MUST clearly identify:

1. the interface being described;


2. the capability-discovery protocol version;


3. the capability set;


4. applicable constraints;


5. the validity context.




---

19. Partial Discovery

An implementation MAY provide partial discovery results.

A partial response MUST explicitly indicate that it is partial.

For example:

status = Partial

A caller MUST NOT interpret a partial result as a complete statement of unsupported capabilities.

Missing information means:

Unknown

unless the protocol explicitly states otherwise.


---

20. Unknown Capabilities

Implementations MUST tolerate unknown capability identifiers.

Unknown capabilities MUST NOT automatically cause the entire interface to be rejected.

The caller MAY:

ignore the capability;

preserve the descriptor;

request additional information;

negotiate an alternative;

reject the interaction if the capability is mandatory.


This allows ULABI to evolve without requiring every implementation to understand every future extension.


---

21. Capability Discovery and Compatibility

Capability discovery MUST NOT itself claim semantic compatibility.

Instead:

Discovery
    |
    v
Capabilities known
    |
    v
Compatibility analysis
    |
    v
Negotiation
    |
    v
Usable contract

Capability discovery provides evidence.

Compatibility analysis determines whether the evidence satisfies the required contract.

Therefore:

Capability Present
    !=
Compatible

and:

Capability Absent
    =>
Required feature cannot be used

when that capability is mandatory.


---

22. Capability Discovery and Backward Compatibility

Backward compatibility rules are defined by:

docs/compatibility/backwards-compatibility.md

Capability discovery MUST provide sufficient information for a newer implementation to determine whether an older required capability remains available.

A newer implementation MUST NOT claim unconditional backward compatibility when a mandatory capability from the older contract has disappeared.


---

23. Capability Discovery and Forward Compatibility

Forward compatibility is defined separately.

Capability discovery SHOULD expose enough information for an older implementation to recognize:

known capabilities;

unknown capabilities;

mandatory requirements;

optional extensions;

compatibility constraints.


Unknown optional capabilities SHOULD be safely ignorable.

Unknown mandatory requirements MUST NOT be ignored.


---

24. Capability Discovery and Feature Negotiation

Capability discovery answers:

What exists?

Feature negotiation answers:

What will both parties use?

Therefore:

Discovery
    |
    v
Intersection
    |
    v
Negotiation
    |
    v
Selected Contract

Capability discovery MUST NOT silently perform feature negotiation.


---

25. Capability Intersection

When two implementations interact, the usable capability set is conceptually:

UsableCapabilities =
    Supported(A)
    ∩
    Supported(B)
    ∩
    Authorized(A,B)
    ∩
    PolicyAllowed
    ∩
    ResourceAvailable

The exact implementation MAY differ, but the observable result MUST preserve these semantics.


---

26. Profile Discovery

Implementations MUST be able to identify the ULABI profiles they support.

A profile descriptor SHOULD contain:

ProfileDescriptor {
    profile_id
    version
    required_capabilities
    optional_capabilities
    dependencies
    constraints
}

Examples:

ulabi.core
ulabi.security
ulabi.streaming
ulabi.async
ulabi.distributed
ulabi.embedded
ulabi.realtime
ulabi.hardware

A Core implementation MUST NOT claim support for an extension profile merely because it supports the Core.


---

27. Interface Discovery

Capability discovery SHOULD identify:

interface identity;

interface version;

supported operations;

supported types;

supported profiles;

required capabilities;

optional capabilities;

execution semantics;

locality;

transport constraints.


An implementation MUST NOT expose an operation as usable when its mandatory dependencies cannot be satisfied.


---

28. Operation Discovery

Operations MAY expose capability metadata.

Conceptually:

OperationDescriptor {
    operation_id
    version
    parameters
    returns
    required_capabilities
    optional_capabilities
    effects
    execution_mode
    locality
    cancellation
}

This metadata allows callers and validators to inspect requirements before invocation.


---

29. Resource Capability Discovery

Resources MAY be represented as capabilities.

Examples:

resource.cpu
resource.gpu
resource.npu
resource.fpga
resource.memory
resource.shared-memory
resource.storage
resource.network

Resource discovery SHOULD include applicable limits.

For example:

GPU:
    available = true
    memory = 16 GiB
    queues = 8

Exact resource representations belong to the relevant resource specifications.


---

30. Limits

Capability discovery SHOULD expose relevant operational limits.

Examples:

maximum_message_size
maximum_arguments
maximum_nesting_depth
maximum_stream_size
maximum_concurrent_calls
maximum_memory
maximum_execution_time
maximum_payload_size

Limits MUST NOT be represented as unlimited unless the implementation can actually satisfy the corresponding contract.

Unknown limits MUST be represented as unknown rather than fabricated.


---

31. Conditional Capabilities

A capability MAY be available only under conditions.

Example:

Capability:
    gpu.compute

Condition:
    authenticated caller
    |
    sufficient quota
    |
    compatible device

Conditional capabilities MUST identify their conditions.

The implementation MUST NOT advertise them as unconditionally available.


---

32. Capability Constraints

A capability MAY have constraints.

Conceptually:

CapabilityConstraint {
    capability
    condition
    minimum
    maximum
    units
}

Examples:

maximum_payload = 16 MiB
maximum_streams = 32
minimum_alignment = 16 bytes
maximum_execution_time = 100 ms

Constraints MUST be interpreted according to the applicable ULABI type and resource semantics.


---

33. Security Capabilities

Security-related capabilities MAY include:

authentication
authorization
capability-tokens
integrity-verification
confidentiality
sandboxing
secure-loading
auditability
cryptographic-agility
post-quantum-cryptography

Security capability discovery MUST NOT expose secrets.

Discovery metadata MUST NOT reveal:

private keys;

credentials;

bearer tokens;

confidential policy data;

sensitive implementation internals.



---

34. Discovery Authorization

Capability discovery itself MAY require authorization.

An implementation MAY expose different discovery results to different callers.

For example:

Anonymous caller
    ->
public capabilities

Authenticated caller
    ->
additional capabilities

Privileged caller
    ->
administrative capabilities

The implementation MUST NOT reveal protected capability information to an unauthorized caller.


---

35. Capability Discovery Is Not Permission Granting

The following is prohibited:

Discover capability
        |
        v
Automatically grant capability

Instead:

Discover
   |
   v
Evaluate
   |
   v
Authorize
   |
   v
Grant / Deny

This distinction is mandatory.


---

36. Capability Revocation

Capabilities MAY be revoked.

Revocation MAY occur because of:

security policy;

resource exhaustion;

session termination;

quota exhaustion;

hardware failure;

administrative action;

expiration;

isolation;

safety policy.


A revoked capability MUST NOT continue to be treated as authorized.


---

37. Capability Expiration

A capability MAY have a validity interval.

Conceptually:

valid_from
valid_until

Expiration MUST be evaluated before using a capability whose validity is time-dependent.

Time-sensitive capabilities SHOULD use a trusted clock or explicitly defined time source.


---

38. Discovery Freshness

Capability information MAY become stale.

A discovery response SHOULD include freshness information where capability state can change.

Possible metadata:

issued_at
expires_at
revision
generation
etag

A cached capability result MUST NOT be treated as permanently authoritative when the capability is mutable.


---

39. Capability Generation

An implementation SHOULD maintain a monotonically changing capability generation identifier when capability state changes.

Example:

Generation 41
    |
    v
GPU available

Generation 42
    |
    v
GPU unavailable

A generation change indicates that previously discovered capability information MAY need to be refreshed.


---

40. Capability Caching

Capability discovery results MAY be cached.

A cache MUST respect:

validity;

expiration;

generation;

authorization context;

interface identity;

profile;

transport context;

security context.


A cached capability result MUST NOT be reused across incompatible security or authorization contexts.


---

41. Capability Integrity

Capability discovery data SHOULD be integrity protected when discovery crosses a trust boundary.

Integrity protection MAY be provided through:

authenticated channels;

signed metadata;

cryptographic attestations;

trusted local mechanisms.


The mechanism is transport- and implementation-independent at the ULABI semantic layer.


---

42. Capability Authenticity

When authenticity matters, an implementation SHOULD be able to establish the origin of capability information.

A caller MUST NOT blindly trust capability claims received from an untrusted source when those claims influence:

security decisions;

resource allocation;

code loading;

authorization;

privileged execution;

safety-critical behavior.



---

43. Capability Attestation

Future ULABI security profiles MAY define cryptographic attestation of capability sets.

Attestation MAY establish:

Implementation identity
+
Interface identity
+
Capability set
+
Profile
+
Version
+
Security state

Attestation is an extension mechanism and MUST NOT be required by Core implementations unless the applicable security profile requires it.


---

44. Capability Discovery Across Locality Boundaries

Capability discovery MUST preserve locality semantics.

The discovery of a remote capability MUST NOT imply that execution is local.

For example:

Remote GPU

MUST NOT be represented simply as:

Local GPU

if the execution semantics differ.

Relevant locality information SHOULD include:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

This follows the locality model established by ULABI's architecture.


---

45. Capability Discovery Across Transports

Capability discovery MUST be transport-independent at the semantic level.

The same capability model MAY be transported through:

direct calls;

shared memory;

IPC;

pipes;

sockets;

message queues;

network protocols;

WebAssembly host calls;

device interfaces.


Transport-specific discovery mechanisms MUST map to the common ULABI capability semantics.


---

46. Discovery Failure

If capability discovery fails, the result MUST NOT be interpreted as:

No capabilities exist

unless the protocol explicitly establishes that fact.

Possible discovery results include:

Success
Partial
Unauthorized
Unavailable
Unsupported
Timeout
InvalidRequest
IntegrityFailure
Unknown

The exact error representation MUST use the ULABI error model.


---

47. Fail-Closed Security Rule

When capability information is required for a security-sensitive operation and the required capability cannot be established, the implementation MUST fail closed.

For example:

Required capability
        |
        v
Discovery failed
        |
        v
Do not execute privileged operation

An implementation MUST NOT assume that an undiscovered security capability exists.


---

48. Discovery of Unknown Future Features

ULABI is designed for long-term extensibility.

Therefore an implementation MUST be able to encounter capabilities it does not understand.

Unknown capabilities SHOULD be preserved when forwarding metadata where doing so is safe.

Unknown capabilities MUST NOT automatically be executed or enabled.


---

49. Capability Namespaces

Capability identifiers SHOULD use controlled namespaces.

Examples:

ulabi.core.*
ulabi.profile.*
ulabi.feature.*
ulabi.security.*
ulabi.transport.*
ulabi.hardware.*

Private implementations MAY use private namespaces.

Private capabilities MUST NOT be represented as standardized ULABI capabilities unless they are registered according to the governance process.


---

50. Private Capabilities

Private capability extensions MAY be used for implementation-specific functionality.

A private capability MUST:

have an unambiguous namespace;

define its semantics;

avoid collisions with standardized identifiers;

declare whether it is required or optional;

define failure behavior.


Private capabilities MUST NOT silently redefine standardized capabilities.


---

51. Capability Set

A capability set is the collection of capability descriptors applicable to a specific context.

Conceptually:

CapabilitySet {
    interface
    profile
    version
    generation
    capabilities[]
}

Capability sets MUST be evaluated within their declared context.


---

52. Capability Fingerprint

Implementations MAY expose a deterministic fingerprint of a capability set.

A capability fingerprint MAY be used to:

detect capability changes;

validate cached discovery data;

compare environments;

support diagnostics;

assist negotiation.


The fingerprint MUST be calculated from canonicalized capability metadata.

The fingerprint MUST NOT replace semantic compatibility analysis.


---

53. Canonical Ordering

Where capability sets are serialized, their canonical representation MUST use deterministic ordering.

Equivalent capability sets MUST produce equivalent canonical representations.

Canonicalization rules SHALL be defined by the applicable ULABI encoding specification.


---

54. Capability Comparison

Capability comparison SHOULD produce explicit results.

Example:

Capability A:
    supported = true

Capability B:
    supported = true

Comparison:
    common = true

A more complete comparison SHOULD distinguish:

Common
A-only
B-only
Conflicting
Conditionally compatible
Unknown

This information may be consumed by feature negotiation and compatibility validation.


---

55. Capability Selection

Capability selection is separate from capability discovery.

Discovery:

What is available?

Selection:

Which available capability should be used?

Negotiation:

Which capability will both parties use?

These stages MUST remain semantically distinct.


---

56. Capability Discovery and Graceful Degradation

Capability discovery MAY be used to determine whether an implementation can degrade safely.

Example:

Streaming available?
    |
    +-- YES -> use streaming
    |
    +-- NO -> use bounded batch mode

However, capability discovery MUST NOT define the fallback behavior.

Fallback behavior belongs to:

docs/compatibility/graceful-degradation.md


---

57. Capability Discovery and Self-Healing

Capability discovery MAY expose whether recovery-related capabilities exist.

Examples:

recovery.restart
recovery.rollback
recovery.checkpoint
recovery.health-check

Discovery MUST NOT authorize autonomous recovery.

Self-healing behavior is governed by:

docs/reliability/self-healing.md

and related reliability specifications.


---

58. Capability Discovery and Hardware

Hardware capabilities MAY include:

cpu
gpu
npu
fpga
quantum
accelerator
tensor
vector

Hardware capability descriptors SHOULD distinguish:

supported;

available;

allocated;

authorized;

remotely accessible;

locally accessible.


Hardware availability MUST NOT imply ownership or authorization.


---

59. Capability Discovery and Real-Time Systems

Real-time profiles MAY expose:

maximum_latency
deadline_support
priority_support
deterministic_execution
jitter_bound

Real-time capability claims MUST be backed by measurable and testable properties.

An implementation MUST NOT claim deterministic real-time guarantees solely because it exposes a capability identifier.


---

60. Capability Discovery and Embedded Systems

Embedded implementations MAY provide reduced discovery mechanisms.

A reduced mechanism MAY use:

static capability tables;

compile-time descriptors;

fixed-size representations;

read-only capability metadata.


Resource-constrained implementations MUST still preserve the semantic distinction between supported, available, and authorized capabilities where the applicable profile requires it.


---

61. Capability Discovery and Distributed Systems

Distributed implementations SHOULD distinguish:

Local capability
Remote capability
Delegated capability
Forwarded capability
Temporary capability

A remote capability MUST NOT automatically become a local capability.

Delegation MUST be explicit.


---

62. Capability Delegation

A capability MAY be delegated to another component.

Delegation MUST define:

capability identity;

scope;

authority;

expiration;

restrictions;

issuer;

recipient;

revocation behavior.


Capability delegation belongs to the security model and MUST NOT be inferred solely from discovery.


---

63. Capability Forwarding

A component MAY forward capability metadata to another component.

Forwarding MUST NOT imply authorization.

For example:

A discovers capability X on B
A forwards metadata about X to C

does not mean:

C is authorized to use X


---

64. Capability Discovery Security Invariants

A conforming implementation MUST preserve the following invariants:

1. Discovery MUST NOT grant authorization.


2. Unknown MUST NOT be treated as supported.


3. Supported MUST NOT be treated as authorized.


4. Available MUST NOT be treated as unrestricted.


5. Revoked capabilities MUST NOT remain authorized.


6. Expired capabilities MUST NOT remain valid.


7. Security-sensitive decisions MUST NOT depend on untrusted capability claims.


8. Capability identifiers MUST NOT be silently reused.


9. Mandatory dependencies MUST NOT be omitted.


10. Capability changes MUST invalidate stale state where required.


11. Discovery MUST NOT expose secrets.


12. Capability metadata MUST NOT silently change semantic meaning.




---

65. Failure Modes

Capability discovery implementations MUST account for:

malformed requests;

malformed responses;

unknown capability identifiers;

unsupported discovery protocol versions;

unavailable endpoints;

authorization failures;

stale capability data;

capability revocation;

capability expiration;

resource exhaustion;

integrity failures;

authentication failures;

inconsistent capability sets;

conflicting capability declarations;

missing dependencies;

invalid constraints.


Each failure MUST map to a defined ULABI error or status.


---

66. Recovery Behavior

Capability discovery failures SHOULD be recoverable where safe.

Possible behavior:

Discovery request
      |
      v
Failure
      |
      +-- Retry permitted?
      |       |
      |      YES
      |       |
      |      Retry
      |
      +-- Cached result valid?
      |       |
      |      YES
      |       |
      |      Use cache
      |
      +-- Otherwise
              |
              v
           Unknown

A stale or unverified capability result MUST NOT be used for security-sensitive operations merely because it is convenient.


---

67. No Unsafe Fallback

An implementation MUST NOT silently substitute an unsafe capability when a required capability is unavailable.

For example:

secure transport unavailable
        |
        X
fall back to insecure transport

MUST NOT occur unless the contract explicitly permits that downgrade and the caller has explicitly accepted it.


---

68. Capability Discovery API Independence

ULABI defines semantics, not one mandatory source-language API.

Implementations MAY expose discovery through:

functions;

objects;

system calls;

metadata;

descriptors;

messages;

static tables;

compiler interfaces;

runtime interfaces.


All implementations MUST map their mechanism to the same ULABI semantic contract.


---

69. Reference Data Model

A conceptual language-neutral model is:

CapabilityIdentity
    namespace
    name
    major_version

CapabilityDescriptor
    identity
    category
    state
    description
    dependencies[]
    conflicts[]
    requirements[]
    constraints[]
    security_requirements[]
    validity

CapabilitySet
    interface_identity
    interface_version
    profile[]
    generation
    capabilities[]

DiscoveryRequest
    protocol_version
    target
    requested_categories[]
    requested_capabilities[]

DiscoveryResponse
    status
    capability_set
    validity
    integrity_metadata

This model is semantic.

It does not prescribe the internal representation of any implementation.


---

70. Reference Discovery Algorithm

A conforming implementation MAY use the following conceptual algorithm:

1. Identify the target interface.
2. Authenticate when required.
3. Authorize discovery when required.
4. Determine the applicable context.
5. Collect supported capabilities.
6. Determine currently available capabilities.
7. Apply security policy.
8. Apply resource constraints.
9. Apply profile constraints.
10. Construct the capability set.
11. Canonicalize the representation.
12. Attach validity metadata.
13. Protect integrity when required.
14. Return the discovery result.

The algorithm is descriptive of semantics, not a mandatory internal implementation.


---

71. Discovery Validation

A validator SHOULD verify:

capability identity;

version;

category;

state;

dependencies;

conflicts;

constraints;

profile requirements;

interface identity;

validity;

security metadata;

canonical representation.


Invalid capability declarations MUST NOT be treated as trusted evidence.


---

72. Conformance Requirements

A Core-conformant ULABI implementation:

MUST:

1. expose its applicable Core identity;


2. identify its supported ULABI version;


3. expose required Core capabilities;


4. distinguish unknown from unsupported;


5. distinguish discovery from authorization;


6. support capability identity;


7. preserve stable capability identifiers;


8. represent mandatory dependencies;


9. report discovery failures;


10. prevent unauthorized capability disclosure where required by the security profile.



SHOULD:

1. expose profile information;


2. expose capability generation;


3. expose capability constraints;


4. expose validity information;


5. support capability comparison;


6. support machine-readable discovery;


7. provide deterministic capability representations.



MAY:

1. provide capability fingerprints;


2. provide signed capability sets;


3. provide attestation;


4. expose private capability extensions.




---

73. Conformance Test Categories

The ULABI conformance suite MUST eventually test at least:

Capability Identity
Capability Categories
Capability States
Unknown Capability Handling
Capability Dependencies
Capability Conflicts
Required Capabilities
Optional Capabilities
Profile Discovery
Interface Discovery
Operation Discovery
Resource Discovery
Security Discovery
Conditional Capabilities
Capability Revocation
Capability Expiration
Discovery Freshness
Capability Caching
Capability Integrity
Authorization Separation
Unknown Future Capabilities
Capability Comparison
Capability Fingerprinting
Locality
Distributed Discovery
Failure Handling
Fail-Closed Security

Tests MUST include positive and negative cases.


---

74. Required Negative Tests

The conformance suite MUST verify that an implementation rejects or safely handles:

unknown mandatory capability
missing dependency
conflicting capability
expired capability
revoked capability
unauthorized capability
tampered capability set
invalid capability identifier
invalid version
invalid constraint
stale security-sensitive discovery


---

75. Reference Implementation Requirements

A future ULABI reference implementation SHOULD provide:

CapabilityRegistry
CapabilityDescriptor
CapabilitySet
CapabilityResolver
CapabilityValidator
CapabilityCache
CapabilityComparator
DiscoveryRequest
DiscoveryResponse
DiscoveryService
CapabilityPolicy
CapabilityAuthorizationAdapter
CapabilityFingerprint

These are implementation modules, not language-specific requirements.


---

76. Required Integration Points

This specification establishes the following integration contracts.

76.1 Core ABI

docs/abi/core-abi.md

MUST consume:

capability identity;

capability requirements;

operation capability metadata;

discovery status.


It MUST NOT redefine capability semantics.

76.2 Security

docs/security/security-model.md

MUST consume:

authorization distinction;

capability state;

revocation;

delegation;

integrity;

authenticity.


It defines security mechanisms rather than redefining discovery.

76.3 Capability Security

docs/security/capability-security.md

MUST consume this document's capability identity and state model.

It defines how capabilities are granted, restricted, delegated, and revoked.

76.4 Feature Negotiation

docs/compatibility/feature-negotiation.md

MUST consume discovered capability sets as negotiation input.

It defines selection rather than discovery.

76.5 Backward Compatibility

docs/compatibility/backwards-compatibility.md

MUST use capability discovery when evaluating whether required capabilities remain available.

76.6 Forward Compatibility

docs/compatibility/forwards-compatibility.md

MUST use the unknown-capability rules defined here.

76.7 Graceful Degradation

docs/compatibility/graceful-degradation.md

MUST consume capability availability to determine whether an explicitly defined fallback can be selected.

76.8 Runtime

docs/runtime/runtime-interface.md

MUST expose runtime capabilities according to this semantic model.

76.9 Distributed ABI

docs/distributed/distributed-abi.md

MUST preserve locality and delegation semantics when capability discovery crosses process or machine boundaries.

76.10 Conformance

docs/standards/conformance.md

MUST use the conformance requirements defined here.

76.11 Test Suite

docs/standards/test-suite.md

MUST implement the capability-discovery test categories defined here.


---

77. Integration Stability Rule

Once this specification is accepted into a ULABI specification version:

later documents MUST reference these capability semantics;

later documents MUST NOT silently redefine capability states;

later documents MUST NOT redefine capability identity;

later documents MUST NOT redefine discovery-versus-authorization semantics;

later documents MUST NOT introduce incompatible meanings for standardized capability identifiers.


Any semantic change MUST occur through the ULABI versioning and governance process.

This allows this document to be completed independently without requiring later documents to rewrite its fundamental contract.


---

78. Separation of Concerns

The following boundaries are normative:

Capability Discovery
        |
        | What exists?
        v
Feature Negotiation
        |
        | What will be used?
        v
Authorization
        |
        | What may this caller use?
        v
Execution
        |
        | What actually happened?
        v
Observability / Diagnostics

No component SHOULD collapse these layers into one opaque mechanism.


---

79. Security Principle

Capability discovery is information.

Authorization is authority.

Execution is action.

ULABI MUST preserve the distinction:

Information
    !=
Authority
    !=
Execution

This is a foundational ULABI security invariant.


---

80. Final Normative Principle

The fundamental rule of ULABI capability discovery is:

> An implementation MUST be able to determine what a ULABI interface supports without that discovery operation itself granting authority to use the discovered capabilities.



Capability discovery therefore enables interoperability while preserving:

security;

compatibility;

implementation independence;

profile independence;

transport independence;

language independence;

long-term extensibility.


End of specification.

## 2. What should be done in each file

The important thing is **not** to start modifying all dependent files immediately. The capability-discovery specification above is sufficiently self-contained that it can be completed first.

I recommend this implementation/documentation order.

### Phase 1 — independent semantic foundations

These should establish contracts without depending on implementation modules.

| Order | File | What it owns |
|---|---|---|
| 1 | `docs/architecture/design-principles.md` | Architectural principles |
| 2 | `docs/architecture/overview.md` | Overall architecture |
| 3 | `docs/architecture/universal-model.md` | Universal semantic model |
| 4 | `docs/architecture/scalability.md` | Scaling model |
| 5 | `docs/compatibility/capability-discovery.md` | **Capability discovery — this file** |
| 6 | `docs/compatibility/backwards-compatibility.md` | Backward compatibility |
| 7 | `docs/compatibility/forwards-compatibility.md` | Forward compatibility |
| 8 | `docs/compatibility/feature-negotiation.md` | Negotiation |
| 9 | `docs/compatibility/graceful-degradation.md` | Safe fallback |
| 10 | `ULABI-VERSIONING.md` | Version lifecycle |

This separation is important because the existing repository already treats these as distinct documents rather than one large compatibility specification. 

### Phase 2 — Core ABI

| File | Required responsibility |
|---|---|
| `docs/abi/core-abi.md` | Core ABI contract and how capabilities attach to interfaces |
| `docs/abi/calling-convention.md` | Calling/argument/return ABI |
| `docs/abi/data-types.md` | Boundary representations |
| `docs/abi/memory-model.md` | Ownership/lifetime/memory boundaries |
| `docs/abi/stack-model.md` | Stack semantics |
| `docs/abi/register-model.md` | Register abstraction |
| `docs/abi/exception-model.md` | Exception/error boundary |
| `docs/abi/return-values.md` | Return semantics |

The repository already contains these ABI documents, including `core-abi.md`, `calling-convention.md`, `memory-model.md`, `stack-model.md`, and `register-model.md`.   

### Phase 3 — Type system

| File | Responsibility |
|---|---|
| `docs/type-system/universal-types.md` | Primitive semantic types |
| `docs/type-system/type-descriptors.md` | Runtime/boundary type metadata |
| `docs/type-system/generics.md` | Generic contracts |
| `docs/type-system/enums.md` | Enum identity/evolution |
| `docs/type-system/structs.md` | Record/struct layout |
| `docs/type-system/unions.md` | Union semantics |
| `docs/type-system/type-compatibility.md` | Type compatibility |

### Phase 4 — Interoperability

| File | Responsibility |
|---|---|
| `docs/interoperability/language-interoperability.md` | Cross-language model |
| `docs/interoperability/foreign-function-interface.md` | FFI |
| `docs/interoperability/object-model.md` | Object boundary |
| `docs/interoperability/name-mangling.md` | Symbol naming |
| `docs/interoperability/symbol-resolution.md` | Symbol resolution |
| `docs/interoperability/cross-language-data.md` | Cross-language data |

### Phase 5 — Runtime

| File | Responsibility |
|---|---|
| `docs/runtime/runtime-interface.md` | Runtime contract |
| `docs/runtime/process-model.md` | Process abstraction |
| `docs/runtime/threading.md` | Threads |
| `docs/runtime/async-model.md` | Async |
| `docs/runtime/concurrency.md` | Concurrency |
| `docs/runtime/resource-management.md` | Resources |

### Phase 6 — Security

| File | Responsibility |
|---|---|
| `docs/security/security-model.md` | Overall security model |
| `docs/security/capability-security.md` | Capability authority/delegation |
| `docs/security/sandboxing.md` | Isolation |
| `docs/security/cryptography.md` | Cryptographic contracts |
| `docs/security/authentication.md` | Identity authentication |
| `docs/security/authorization.md` | Permission model |
| `docs/security/zero-trust.md` | Trust boundaries |
| `docs/security/secure-loading.md` | Secure loading |
| `docs/security/supply-chain-security.md` | Supply-chain integrity |

**Important:** `capability-discovery.md` says **what capabilities exist**. `capability-security.md` must say **how capabilities become authority**. That prevents the two specifications from becoming duplicates.

### Phase 7 — Reliability

| File | Responsibility |
|---|---|
| `docs/reliability/fault-detection.md` | Detection |
| `docs/reliability/fault-isolation.md` | Isolation |
| `docs/reliability/recovery.md` | Recovery |
| `docs/reliability/rollback.md` | Rollback |
| `docs/reliability/health-monitoring.md` | Health |
| `docs/reliability/self-healing.md` | Controlled recovery orchestration |

### Phase 8 — Distributed

| File | Responsibility |
|---|---|
| `docs/distributed/distributed-abi.md` | Distributed ABI |
| `docs/distributed/remote-calls.md` | Remote invocation |
| `docs/distributed/serialization.md` | Serialization |
| `docs/distributed/service-discovery.md` | Service discovery |
| `docs/distributed/distributed-errors.md` | Distributed failures |
| `docs/distributed/consensus-boundaries.md` | Consensus boundaries |

Capability discovery becomes especially important here because a remote capability must never silently become a local capability.

---

# 3. Required implementation code/modules

The documentation should remain language-neutral, but ULABI eventually needs executable infrastructure.

I would define the following **logical modules**, without forcing a particular programming language:

```text
implementations/
│
├── core/
│   ├── interface_identity
│   ├── version
│   ├── capability
│   ├── capability_descriptor
│   ├── capability_set
│   ├── capability_state
│   ├── capability_dependency
│   ├── capability_constraint
│   └── capability_generation
│
├── discovery/
│   ├── discovery_request
│   ├── discovery_response
│   ├── discovery_service
│   ├── discovery_client
│   ├── discovery_registry
│   ├── capability_resolver
│   ├── capability_validator
│   ├── capability_comparator
│   └── capability_fingerprint
│
├── compatibility/
│   ├── compatibility_engine
│   ├── version_resolver
│   ├── profile_resolver
│   ├── feature_negotiator
│   └── degradation_selector
│
├── security/
│   ├── capability_authorizer
│   ├── capability_policy
│   ├── capability_delegation
│   ├── capability_revocation
│   ├── capability_attestation
│   └── discovery_integrity
│
├── runtime/
│   ├── runtime_capabilities
│   ├── resource_capabilities
│   ├── execution_capabilities
│   └── transport_capabilities
│
├── encoding/
│   ├── canonical_encoding
│   ├── capability_codec
│   └── discovery_codec
│
├── cache/
│   ├── capability_cache
│   ├── freshness
│   └── invalidation
│
└── validation/
    ├── schema_validator
    ├── contract_validator
    └── conformance_validator

These are logical modules, not yet filenames in a particular implementation language. That is intentional: ULABI's architecture says the implementation must remain independent of the language used to implement it.


---

4. Required schemas

The capability system will eventually need machine-readable schemas under:

schemas/
├── capability-identity.schema.*
├── capability-descriptor.schema.*
├── capability-set.schema.*
├── discovery-request.schema.*
├── discovery-response.schema.*
├── capability-constraint.schema.*
├── capability-dependency.schema.*
├── profile-descriptor.schema.*
└── operation-capability.schema.*

The actual schema format should be selected once ULABI's canonical serialization specification is finalized. Do not prematurely duplicate the semantic model into several incompatible schema formats.


---

5. Required executable tests

Eventually:

tests/
├── capability_identity/
├── capability_states/
├── capability_dependencies/
├── capability_conflicts/
├── capability_discovery/
├── profile_discovery/
├── operation_discovery/
├── resource_discovery/
├── security_discovery/
├── authorization_separation/
├── revocation/
├── expiration/
├── freshness/
├── caching/
├── integrity/
├── unknown_capabilities/
├── compatibility/
├── locality/
├── distributed/
└── failure_handling/

And:

conformance/
├── core/
├── capability-discovery/
├── compatibility/
├── security/
├── runtime/
└── distributed/


---

6. Integration rule that prevents future re-editing

Your requirement that "when we finish one file, we should not need to re-edit it because another file was later written" is exactly the right approach.

For ULABI I recommend this dependency direction:

ULABI-DESIGN.md
                           |
                           v
                     ULABI-SPEC.md
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          Identity      Types       Capability
             |             |             |
             +-------------+-------------+
                           |
                           v
                     Core ABI
                           |
            +--------------+--------------+
            |              |              |
            v              v              v
          Memory         Calls         Errors
            |              |              |
            +--------------+--------------+
                           |
                           v
                    Compatibility
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        Negotiation   Backward       Forward
             |
             v
          Runtime
             |
             v
          Security
             |
             v
       Distributed
             |
             v
        Conformance

A later document references an earlier contract.

It does not redefine it.

That means once capability-discovery.md is finalized, the later feature-negotiation.md should say, in effect:

> "Use the capability model defined by capability-discovery.md."



It should not create a second definition of Supported, Available, Authorized, Revoked, capability identity, etc.

That is the key architectural discipline that will keep ULABI from developing the repetition problem you are specifically trying to avoid.

