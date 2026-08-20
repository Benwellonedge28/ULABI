I read the repository first, including the current ULABI-DESIGN.md, the distributed-document directory, and the existing distributed ABI material. The repository confirms that docs/distributed/service-discovery.md is currently only a placeholder, while distributed-abi.md, remote-calls.md, serialization.md, distributed-errors.md, and consensus-boundaries.md already contain substantial specifications.

That means service discovery should be the authoritative specification for discovering and selecting endpoints, rather than duplicating remote-call, serialization, distributed-error, or consensus semantics.

The repository's architecture already establishes Minimal Core + Standard Profiles + Extensible Ecosystem, transport independence, explicit locality, interface identity, negotiation/discovery, and implementation independence.

Below is the complete file to copy and paste.

docs/distributed/service-discovery.md

# ULABI Service Discovery Specification

Project: ULABI  
Full Name: Universal Language Application Binary Interface  
Document: `docs/distributed/service-discovery.md`  
Specification Family: Distributed Interoperability  
Specification Area: Service Discovery  
Status: Normative Specification  
Specification Version: 0.1.0-draft  
ULABI Architecture Version: 0.2.0-draft  
Normative Authority:
- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

---

# 1. Purpose

This document defines the ULABI Service Discovery contract.

Service discovery is the mechanism by which a ULABI implementation determines:

- which interfaces are available;
- which versions are supported;
- which endpoints currently implement those interfaces;
- which transports are available;
- which profiles are supported;
- which capabilities are exposed;
- which security requirements apply;
- which locality constraints apply;
- which resource limits apply;
- whether an endpoint is currently eligible for use.

Service discovery exists above transport selection and below application-level service selection.

The fundamental model is:

```text
Interface Identity
       |
       v
Service Discovery
       |
       +---- Endpoint Discovery
       |
       +---- Version Discovery
       |
       +---- Profile Discovery
       |
       +---- Capability Discovery
       |
       +---- Security Requirement Discovery
       |
       +---- Health / Availability
       |
       +---- Policy Filtering
       |
       v
Eligible Endpoint Set
       |
       v
Endpoint Selection
       |
       v
Remote Invocation

Service discovery MUST identify what is available.

It MUST NOT silently determine what an application is authorized to do.

Discovery and authorization are separate concerns.


---

2. Scope

This specification defines:

1. service identity;


2. interface identity;


3. endpoint identity;


4. service instances;


5. discovery mechanisms;


6. registry-based discovery;


7. direct discovery;


8. static discovery;


9. dynamic discovery;


10. endpoint advertisements;


11. endpoint metadata;


12. version discovery;


13. profile discovery;


14. capability discovery;


15. transport discovery;


16. locality discovery;


17. security metadata;


18. health metadata;


19. resource metadata;


20. endpoint eligibility;


21. endpoint selection;


22. discovery freshness;


23. registration;


24. deregistration;


25. expiration;


26. leases;


27. liveness;


28. stale endpoints;


29. duplicate endpoints;


30. endpoint identity;


31. service identity;


32. discovery security;


33. discovery authorization;


34. discovery confidentiality;


35. discovery integrity;


36. discovery failure;


37. discovery consistency;


38. discovery caching;


39. discovery invalidation;


40. discovery observability;


41. compatibility;


42. graceful degradation;


43. conformance requirements.



This document does NOT define:

a specific network protocol;

DNS;

HTTP;

TCP;

QUIC;

mDNS;

a specific registry;

a specific database;

a specific consensus algorithm;

a specific authentication algorithm;

a specific authorization policy;

application business logic;

remote invocation semantics;

serialization encoding.


Those concerns belong to their respective ULABI specifications or implementation profiles.


---

3. Architectural Position

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



Service discovery is primarily part of the Distributed Interoperability Profile.

ULABI Core
    |
    +-- Interface Identity
    |
    +-- Versioning
    |
    +-- Capability Model
    |
    +-- Contract Validation
    |
    v
Distributed Profile
    |
    +-- Service Discovery
    |
    +-- Remote Calls
    |
    +-- Serialization
    |
    +-- Distributed Errors
    |
    +-- Consensus Boundaries

Service discovery MUST consume existing ULABI contracts.

It MUST NOT redefine:

the core ABI;

type semantics;

canonical serialization;

remote-call semantics;

distributed error semantics;

consensus semantics;

authorization semantics.



---

4. Relationship With Other Specifications

This document owns:

> discovery and selection of potentially usable service endpoints.



The following documents retain their own authority:

docs/distributed/distributed-abi.md

Defines the overall distributed ABI boundary.

docs/distributed/remote-calls.md

Defines invocation semantics.

docs/distributed/serialization.md

Defines representation and serialization of discovery records and distributed values.

docs/distributed/distributed-errors.md

Defines distributed failure and error semantics.

docs/distributed/consensus-boundaries.md

Defines consensus-related boundaries and semantics.

docs/compatibility/capability-discovery.md

Defines the general capability-discovery model.

docs/compatibility/feature-negotiation.md

Defines mutual feature negotiation.

docs/compatibility/backwards-compatibility.md

Defines compatibility with previous contracts.

docs/compatibility/forwards-compatibility.md

Defines handling of contracts containing newer features.

docs/compatibility/graceful-degradation.md

Defines safe reduction of functionality when complete compatibility is unavailable.

docs/security/authentication.md

Defines authentication.

docs/security/authorization.md

Defines authorization.

docs/security/capability-security.md

Defines capability-based security.

docs/security/zero-trust.md

Defines zero-trust principles.

Service discovery MUST integrate with these specifications rather than duplicate them.


---

5. Normative Language

The terms:

MUST

MUST NOT

REQUIRED

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY

OPTIONAL


are normative.

A conforming implementation MUST satisfy all applicable MUST and MUST NOT requirements.


---

6. Fundamental Principle

Service discovery answers:

> "What potentially usable implementations of this contract are currently discoverable?"



It does not answer:

> "What is this caller allowed to do?"



Therefore:

Discovery
    !=
Authorization

An endpoint being discoverable MUST NOT imply that:

it is trusted;

it is authenticated;

it is authorized;

it is safe;

it is available;

it is compatible;

it may be invoked.


Discovery produces candidates.

Policy produces eligibility.

Authorization produces permission.


---

7. Service Identity

Every discoverable service MUST have a stable service identity.

Conceptually:

ServiceIdentity {
    namespace
    service_id
}

The service identity MUST NOT depend solely on:

IP address;

hostname;

port;

process ID;

memory address;

container ID;

temporary endpoint;

source-language name;

compiler-generated symbol.


A service MAY expose multiple interfaces.


---

8. Interface Identity

A service interface MUST have a stable interface identity.

Conceptually:

InterfaceIdentity {
    namespace
    interface_id
    major_version
}

Service discovery MUST preserve the distinction between:

Service
Interface
Endpoint
Instance

These are different identities.


---

9. Service, Interface, Instance, Endpoint

ULABI defines the following conceptual model:

Service
   |
   +-- Interface A
   |      |
   |      +-- Instance 1
   |      |      |
   |      |      +-- Endpoint 1
   |      |
   |      +-- Instance 2
   |             |
   |             +-- Endpoint 2
   |
   +-- Interface B
          |
          +-- Instance 3
                 |
                 +-- Endpoint 3

Service

Logical provider identity.

Interface

A versioned interoperability contract.

Instance

A running or provisioned implementation of an interface.

Endpoint

A concrete reachable access point.


---

10. Endpoint Identity

Every discoverable endpoint SHOULD have a stable endpoint identity.

Conceptually:

EndpointIdentity {
    namespace
    endpoint_id
}

The endpoint identity SHOULD remain stable for the lifetime of the endpoint instance.

If an endpoint's meaningful identity changes, the implementation SHOULD advertise a new endpoint identity.


---

11. Endpoint Location

An endpoint MAY contain location information.

Conceptually:

EndpointLocation {
    host
    port
    transport
    path
    region
    zone
    device
}

The exact fields depend on the transport.

ULABI MUST NOT require one universal addressing scheme.


---

12. Transport Independence

Service discovery MUST remain independent from transport.

Possible transports include:

direct invocation;

shared memory;

operating-system IPC;

pipes;

Unix sockets;

TCP;

QUIC;

WebAssembly host calls;

device buses;

accelerator interfaces;

future transports.


A service MAY advertise multiple transports.

Example:

Endpoint {
    transports:
        - ULABI-IPC
        - ULABI-QUIC
        - ULABI-SHM
}

Discovery MUST NOT assume that one transport is universally preferred.


---

13. Locality

Service discovery MUST expose sufficient locality information to prevent an implementation from accidentally treating a remote endpoint as local.

Conceptual locality classes:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

An endpoint MAY additionally advertise:

SameProcess
SameHost
SameZone
SameRegion
CrossRegion
Remote

Locality information MUST NOT be treated as an authorization decision.


---

14. Discovery Modes

ULABI supports four conceptual discovery modes.

14.1 Static Discovery

The endpoint is configured before execution.

Example:

configuration
    |
    v
Interface
    |
    v
Endpoint

Static discovery SHOULD be used where dynamic discovery is unnecessary.


---

14.2 Direct Discovery

A caller queries a known discovery endpoint or authority.

Caller
   |
   v
Known Discovery Authority
   |
   v
Endpoint Set


---

14.3 Registry Discovery

Endpoints register themselves with a registry.

Service Instance
       |
       v
Registry
       |
       v
Client Query


---

14.4 Dynamic Discovery

The implementation discovers services dynamically during execution.

Dynamic discovery MAY use:

local registries;

network registries;

multicast mechanisms;

DNS-based systems;

service meshes;

distributed registries;

platform-native discovery;

future discovery mechanisms.


ULABI standardizes the semantic contract rather than requiring one mechanism.


---

15. Discovery Authority

A discovery authority is an entity capable of providing endpoint information.

Conceptually:

DiscoveryAuthority {
    authority_id
    supported_interfaces
    supported_versions
    supported_profiles
    security_requirements
}

A discovery authority MUST NOT automatically gain authorization over the discovered service.

Discovery authority and service authority are distinct concepts.


---

16. Service Registration

A service instance MAY register itself with a discovery authority.

Conceptually:

Registration {
    service_identity
    interface_identity
    instance_identity
    endpoint_identity
    endpoint_metadata
    profiles
    capabilities
    security_requirements
    health_state
    resource_limits
    lease
}

Registration MUST contain sufficient information for clients to determine whether the endpoint is a potential candidate.


---

17. Registration Validation

A discovery authority MUST validate registration data according to its applicable policy.

Validation SHOULD include:

identity syntax;

interface identity;

supported versions;

supported profiles;

endpoint format;

capability metadata;

security metadata;

resource metadata;

lease validity.


Malformed registration MUST NOT be published as a valid endpoint.


---

18. Registration Authentication

Where registration security is required, the registering service MUST authenticate according to the applicable security profile.

Authentication MUST NOT be inferred merely from:

network location;

hostname;

endpoint reachability;

possession of a registration address.



---

19. Registration Authorization

A discovery authority MAY require authorization before accepting registration.

Authorization MUST be separate from authentication.

Conceptually:

Registration
     |
     v
Authenticate
     |
     v
Authorize
     |
     v
Validate
     |
     v
Publish


---

20. Endpoint Advertisement

A published endpoint SHOULD expose a canonical advertisement.

Conceptually:

ServiceAdvertisement {
    service_identity
    interface_identity
    interface_versions
    instance_identity
    endpoint_identity
    transports
    locality
    profiles
    capabilities
    security_requirements
    resource_limits
    health
    lease
    metadata
}

Unknown metadata MUST NOT cause a conforming implementation to misinterpret known fields.


---

21. Interface Versions

An endpoint MAY advertise multiple interface versions.

Example:

interface_versions:
    - 1.x
    - 2.x
    - 3.x

The advertised versions MUST correspond to actually supported contracts.

An implementation MUST NOT advertise compatibility it cannot provide.


---

22. Profile Advertisement

An endpoint MAY advertise supported ULABI profiles.

Example:

profiles:
    - Core
    - Distributed
    - Streaming
    - Security
    - Async

Profile identifiers MUST be stable.

Profile support MUST NOT imply support for every feature within another profile.


---

23. Capability Advertisement

Endpoints MAY advertise capabilities.

Example:

capabilities:
    - streaming
    - cancellation
    - compression
    - encryption
    - zero_copy

Capability advertisement is descriptive.

It is not authorization.

A capability advertised by an endpoint MUST still be subject to the caller's authorization.


---

24. Security Metadata

An endpoint MAY advertise security requirements.

Example:

security_requirements {
    authentication_required
    authorization_required
    encryption_required
    integrity_required
    trusted_execution_required
}

Security requirements MUST be machine-readable where interoperability depends on them.

A client MUST NOT invoke an endpoint when required security conditions cannot be satisfied.


---

25. Health Metadata

An endpoint MAY advertise health information.

Conceptually:

HealthState {
    Healthy
    Degraded
    Unhealthy
    Unknown
}

Health information is advisory unless a policy explicitly makes it a selection requirement.

A stale health record MUST NOT be treated as current indefinitely.


---

26. Resource Metadata

An endpoint MAY advertise resource limits.

Examples:

maximum_message_size
maximum_concurrent_calls
maximum_memory
maximum_execution_time
maximum_streams
maximum_bandwidth

Clients SHOULD use these limits before attempting operations that would exceed them.

The server MUST enforce its authoritative limits independently.


---

27. Endpoint Eligibility

Discovery produces candidate endpoints.

Eligibility is determined by applying:

1. interface compatibility;


2. version compatibility;


3. profile compatibility;


4. capability requirements;


5. locality requirements;


6. transport requirements;


7. security requirements;


8. resource requirements;


9. policy constraints;


10. health constraints.



Conceptually:

Discovered Endpoints
        |
        v
Compatibility Filter
        |
        v
Capability Filter
        |
        v
Security Filter
        |
        v
Policy Filter
        |
        v
Health / Resource Filter
        |
        v
Eligible Endpoints


---

28. Endpoint Selection

ULABI does not mandate one endpoint-selection algorithm.

Implementations MAY select according to:

locality;

latency;

load;

health;

cost;

availability;

capacity;

security level;

policy;

deterministic ordering;

weighted selection;

geographic proximity.


Selection MUST respect the applicable contract.

An implementation MUST NOT select an endpoint that violates a REQUIRED constraint merely because it appears faster or cheaper.


---

29. Deterministic Selection

Some applications require deterministic endpoint selection.

ULABI MAY support a deterministic selection policy.

For deterministic selection:

same discovery set
+
same policy
+
same selection algorithm
=
same selected endpoint

unless dynamic state explicitly participates in selection.


---

30. Load and Capacity

An endpoint MAY advertise:

load
capacity
available_capacity
concurrency
queue_depth

These values MAY become stale.

Clients MUST treat dynamic load information as time-sensitive.

A discovery authority MUST NOT guarantee that an advertised capacity remains available after advertisement unless it explicitly provides such a reservation guarantee.


---

31. Discovery Freshness

Every dynamic discovery record SHOULD have freshness information.

Conceptually:

Freshness {
    observed_at
    expires_at
    ttl
}

A client MUST NOT assume that an expired record remains valid.

An implementation MAY use an expired record only under an explicitly defined degraded-mode policy.


---

32. Time-To-Live

A discovery record MAY have a TTL.

Example:

ttl = 30 seconds

The TTL specifies the maximum recommended validity interval of the discovery information.

TTL does not guarantee endpoint availability.


---

33. Leases

Dynamic registration SHOULD support leases where endpoint liveness matters.

Conceptually:

Lease {
    lease_id
    issued_at
    expires_at
    renewal_policy
}

A service instance MUST renew its lease before expiration when continued advertisement is required.

An expired lease SHOULD cause the endpoint to become undiscoverable.


---

34. Lease Failure

If lease renewal fails, the discovery authority MUST follow its configured policy.

Possible behavior:

Lease renewal failed
        |
        +-- grace period
        |
        +-- temporary stale state
        |
        +-- deregistration

The policy MUST be explicit.

An implementation MUST NOT retain a dead endpoint indefinitely.


---

35. Deregistration

A service instance SHOULD explicitly deregister when it stops serving an interface.

Conceptually:

Running
   |
   v
Deregister
   |
   v
Unavailable

Failure to deregister MUST be recoverable through lease expiration or equivalent liveness detection.


---

36. Duplicate Registration

A discovery authority MAY receive duplicate registrations.

Duplicates MUST NOT produce ambiguous service identity.

The authority SHOULD identify duplicates using:

service identity;

interface identity;

instance identity;

endpoint identity;

registration authority.


Multiple endpoints MAY legitimately implement the same service.


---

37. Endpoint Replacement

A service instance MAY replace an endpoint.

The discovery system SHOULD support:

Endpoint A
    |
    v
Endpoint B

without changing the interface identity.

If endpoint replacement changes contract semantics, a new interface version MUST be advertised.


---

38. Service Migration

Services MAY migrate between:

hosts;

processes;

containers;

virtual machines;

regions;

devices;

accelerators.


Migration MUST NOT require clients to change the interface identity.

The endpoint information MAY change.


---

39. Discovery Caching

Clients MAY cache discovery results.

Cached records MUST retain freshness metadata.

A cache MUST NOT silently convert dynamic information into permanent configuration.

Caching behavior SHOULD define:

TTL;

maximum cache age;

invalidation;

refresh;

stale-use policy.



---

40. Cache Invalidation

A discovery system MAY support explicit invalidation.

Examples:

EndpointRemoved
EndpointChanged
InterfaceChanged
SecurityChanged
CapabilityChanged
VersionChanged

Invalidation messages SHOULD be authenticated where discovery security requires it.


---

41. Stale Discovery

A stale endpoint MAY still be reachable.

However:

Reachable
    !=
Current

A client MUST NOT assume that successful network reachability proves that cached contract metadata remains valid.


---

42. Discovery Failure

Discovery itself may fail.

Possible states include:

DiscoverySuccess
DiscoveryUnavailable
DiscoveryTimeout
DiscoveryRejected
DiscoveryUnauthorized
DiscoveryIncompatible
DiscoveryUnknown

Discovery failure MUST NOT be confused with service failure.


---

43. Discovery and Service Failure

The following states are distinct:

Cannot discover service
        !=
Discovered service but cannot connect
        !=
Connected but invocation failed
        !=
Invocation outcome unknown

Implementations SHOULD preserve these distinctions in diagnostics.


---

44. Discovery Consistency

Different clients MAY temporarily observe different endpoint sets.

ULABI does not require globally consistent discovery unless a specific profile requires it.

The discovery authority MUST state its consistency model where it materially affects correctness.

Possible models include:

EventuallyConsistent
ReadYourWrites
Monotonic
StronglyConsistent
Unknown

The chosen model MUST NOT be inferred.


---

45. Discovery and Consensus

Service discovery MUST NOT silently become a consensus mechanism.

Discovery may tell clients:

These endpoints are currently advertised.

Consensus determines:

Which state or decision is authoritative.

Those are different functions.

Consensus requirements belong to:

docs/distributed/consensus-boundaries.md


---

46. Discovery Security

Discovery is part of the distributed security surface.

A secure discovery system SHOULD provide:

authenticated registration;

authenticated queries where required;

integrity protection;

authorization;

replay protection;

endpoint authenticity;

metadata integrity;

confidentiality where discovery data is sensitive.



---

47. Discovery Does Not Grant Trust

A discovered endpoint MUST NOT automatically become trusted.

Conceptually:

Discover
   |
   v
Authenticate
   |
   v
Authorize
   |
   v
Invoke

The exact security flow belongs to the applicable security profile.


---

48. Discovery Does Not Grant Capabilities

A discovery record MUST NOT itself grant:

filesystem access;

network access;

device access;

cryptographic authority;

process control;

memory access;

administrative privileges.


Discovery metadata is not a capability token unless a separate security specification explicitly defines such semantics.


---

49. Confidential Discovery

Some services may reveal sensitive information through discovery.

Examples:

internal topology;

private addresses;

hardware capabilities;

tenant identities;

security configuration;

service names;

geographic locations.


Discovery systems MAY restrict metadata visibility according to authorization policy.

The implementation SHOULD disclose only metadata required for the requesting party's discovery task.


---

50. Discovery Poisoning

Implementations MUST consider malicious or incorrect endpoint advertisements.

A client SHOULD validate:

identity;

interface;

version;

endpoint;

security metadata;

authority;

signature;

freshness.


Untrusted discovery information MUST NOT bypass normal security validation.


---

51. Replay Protection

Discovery advertisements with security significance SHOULD include freshness or replay protection.

Possible mechanisms include:

timestamps;

sequence numbers;

nonces;

epochs;

leases;

signed freshness metadata.


The specific cryptographic mechanism belongs to the applicable security profile.


---

52. Endpoint Authenticity

An endpoint advertisement SHOULD be bound to the identity of the entity authorized to provide that endpoint.

The client MUST be able to distinguish:

Endpoint claims to implement Interface X

from:

Authorized entity actually implements Interface X

when authentication is required.


---

53. Interface Verification

After discovery, a client SHOULD verify that the endpoint actually implements the advertised interface.

Verification MAY include:

authenticated handshake;

interface identity exchange;

version negotiation;

profile negotiation;

cryptographic attestation;

contract validation.


Discovery metadata alone MUST NOT be considered definitive proof of implementation correctness.


---

54. Version Compatibility

Discovery SHOULD expose sufficient version information for the client to determine compatibility.

The client MUST apply the rules defined by:

docs/compatibility/backwards-compatibility.md

and:

docs/compatibility/forwards-compatibility.md

Discovery MUST NOT invent compatibility rules.


---

55. Feature Negotiation

Discovery and feature negotiation are separate.

Discovery answers:

What does the endpoint claim to support?

Negotiation answers:

What mutually supported configuration will actually be used?

The detailed negotiation mechanism belongs to:

docs/compatibility/feature-negotiation.md


---

56. Capability Discovery

Capability discovery identifies available capabilities.

Service discovery MAY include capability metadata.

However:

Capability discovered
    !=
Capability authorized

The capability-discovery specification remains authoritative for capability semantics.


---

57. Graceful Degradation

If a discovered endpoint lacks an optional feature, the client MAY select a reduced configuration when permitted.

Example:

Requested:
Streaming + Compression + Encryption

Endpoint:
Streaming + Encryption

Result:
Streaming + Encryption

The implementation MUST NOT silently remove a REQUIRED feature.

Graceful degradation follows:

docs/compatibility/graceful-degradation.md


---

58. Required vs Optional Metadata

Discovery metadata SHOULD distinguish:

Required
Recommended
Optional
Informational

A missing OPTIONAL field MUST NOT invalidate the endpoint.

A missing REQUIRED field MUST cause the endpoint to be rejected for operations requiring that field.


---

59. Unknown Metadata

ULABI discovery records MUST be extensible.

A conforming implementation MUST ignore unknown optional metadata when it can safely do so.

Unknown metadata MUST NOT override known normative fields.

Unknown REQUIRED semantics MUST cause the endpoint to be rejected when the implementation cannot safely interpret them.


---

60. Canonical Discovery Record

A conceptual canonical discovery record is:

ServiceAdvertisement {
    service_identity
    interface_identity
    interface_versions
    instance_identity
    endpoint_identity

    transports
    locality

    profiles
    capabilities

    security_requirements

    resource_limits

    health

    consistency
    freshness
    lease

    metadata
}

The wire representation is defined through:

docs/distributed/serialization.md

and MUST NOT be redefined here.


---

61. Discovery Query

A discovery query SHOULD be expressible conceptually as:

DiscoveryQuery {
    service_identity?
    interface_identity?
    version_constraints?
    profile_requirements?
    capability_requirements?
    locality_requirements?
    transport_requirements?
    security_requirements?
    resource_requirements?
    health_requirements?
    policy_constraints?
}

The implementation MAY expose additional query fields.


---

62. Discovery Result

A discovery result SHOULD conceptually contain:

DiscoveryResult {
    query
    endpoints
    authority
    observed_at
    expires_at
    consistency
}

The result MUST make it possible to determine whether the endpoint information is current enough for the intended operation.


---

63. Empty Results

A valid discovery operation MAY return zero endpoints.

This MUST NOT automatically be interpreted as:

service does not exist;

service is permanently unavailable;

service is unauthorized;

service has failed.


The reason MAY be represented separately where the discovery authority is permitted to disclose it.


---

64. Discovery Ordering

A discovery authority MAY return endpoints in a defined order.

If ordering has semantic meaning, it MUST be explicitly documented.

Otherwise clients MUST NOT interpret list order as:

priority;

trust;

health;

preference.



---

65. Endpoint Priority

An endpoint MAY advertise a priority.

Priority semantics MUST be explicitly defined.

If multiple endpoints have equal priority, a client MAY use another defined selection policy.


---

66. Endpoint Weight

Endpoints MAY advertise weights.

Weights MAY be used for load distribution.

Weights MUST NOT override:

security requirements;

compatibility requirements;

locality restrictions;

authorization;

mandatory resource constraints.



---

67. Health Checking

A discovery implementation MAY actively check endpoint health.

Health checking MUST be distinguishable from actual service invocation.

A health check MUST NOT itself be treated as proof that all service methods are functioning.


---

68. Health Check Isolation

Health checks SHOULD be bounded.

A discovery system MUST NOT create an unbounded number of health checks.

Health checking SHOULD respect:

rate limits;

resource budgets;

security policy;

endpoint capacity.



---

69. Circuit Breaking

A discovery client MAY temporarily exclude an endpoint after repeated failures.

This is an implementation policy.

The endpoint MUST NOT be permanently removed solely because one client experienced failures unless authoritative discovery state confirms removal.


---

70. Failure Domains

Discovery metadata MAY identify failure domains.

Examples:

process
host
rack
zone
region
device
provider

Clients MAY use failure-domain information to avoid correlated failures.

Failure-domain semantics MUST NOT be confused with consensus semantics.


---

71. Locality-Aware Selection

When locality is important, clients SHOULD prefer an endpoint satisfying the required locality.

Example:

SameProcess
    >
SameHost
    >
SameZone
    >
SameRegion
    >
Remote

This ordering is only an example.

The actual policy MUST be explicit.


---

72. Security-Aware Selection

Endpoint selection MAY consider security properties.

Example:

Authenticated
Encrypted
Attested
TrustedExecution

Security requirements marked REQUIRED MUST be satisfied before endpoint selection.


---

73. Resource-Aware Selection

Endpoint selection MAY consider:

available memory;

CPU capacity;

GPU capacity;

bandwidth;

concurrent-call capacity;

execution limits.


Resource metadata is advisory unless a reservation mechanism explicitly guarantees the resource.


---

74. Discovery Across Administrative Domains

ULABI MAY support discovery across independent administrative domains.

A client MUST NOT assume that one administrative domain trusts another.

Cross-domain discovery SHOULD explicitly establish:

authority;

trust relationship;

identity;

authorization;

security requirements.



---

75. Multi-Registry Discovery

A client MAY query multiple discovery authorities.

Example:

Registry A
Registry B
Registry C
     |
     v
Merged Candidate Set

The client MUST prevent conflicting records from being treated as automatically equivalent.

Conflicts SHOULD be resolved according to explicit policy.


---

76. Discovery Authority Conflicts

If two authorities advertise conflicting information, the client MUST NOT silently choose one unless policy defines the precedence.

Possible policies:

PrimaryAuthority
MostRecent
HighestTrust
LocalAuthority
ExplicitPriority
ConsensusDerived

The policy MUST be explicit.


---

77. Discovery Trust Domains

Discovery information MAY carry a trust-domain identifier.

Conceptually:

TrustDomain {
    domain_id
    authority
}

A client MAY restrict discovery to trusted domains.


---

78. Multi-Tenant Discovery

A discovery system MAY isolate tenants.

Tenant-specific discovery MUST prevent unauthorized disclosure of:

private services;

private endpoints;

private metadata;

private capabilities;

private topology.



---

79. Namespace Isolation

Service identities SHOULD use namespaces.

Example:

namespace = example.organization
service_id = storage

Namespaces MUST NOT be interpreted as proof of ownership without authentication or trust policy.


---

80. Service Aliases

A discovery system MAY support aliases.

Aliases MUST resolve to stable service identities.

An alias MUST NOT silently change the semantic contract of the referenced service.


---

81. Discovery Events

A dynamic discovery system MAY expose events:

ServiceAdded
ServiceRemoved
EndpointAdded
EndpointRemoved
EndpointChanged
VersionChanged
CapabilityChanged
HealthChanged
SecurityChanged

Event semantics MUST remain distinguishable from invocation events.


---

82. Event Ordering

Discovery events MAY arrive out of order.

If event ordering matters, the discovery system SHOULD provide:

sequence numbers;

epochs;

versions;

timestamps;

monotonic revision identifiers.


Clients MUST NOT infer ordering solely from network arrival.


---

83. Discovery Reconciliation

Clients SHOULD periodically reconcile cached state against an authoritative discovery source when correctness requires current information.

Conceptually:

Cached State
     |
     v
Refresh
     |
     v
Authoritative State
     |
     v
Reconcile


---

84. Discovery Recovery

If a discovery authority becomes unavailable, an implementation MAY use cached discovery data under a defined degraded-mode policy.

The policy MUST define:

maximum stale age;

allowed operations;

security restrictions;

retry behavior;

failure behavior.


Stale discovery MUST NOT silently restore capabilities that have been revoked.


---

85. Security Revocation

Security-sensitive discovery information MUST support revocation or invalidation where required.

Examples:

revoked endpoint;

revoked credential;

revoked capability;

compromised instance;

invalidated trust domain.


A cached record MUST NOT override authoritative revocation.


---

86. Discovery and Authorization Revocation

Discovery MAY indicate that an endpoint exists even after a particular caller's authorization has been revoked.

Therefore authorization MUST be evaluated at invocation time according to the applicable security policy.


---

87. Discovery Rate Limiting

Discovery authorities SHOULD support rate limiting.

Limits MAY apply to:

registration;

renewal;

query;

event subscription;

health checking.


Rate limiting MUST NOT create ambiguous service identity.


---

88. Discovery Resource Exhaustion

Implementations MUST protect against malicious or accidental discovery amplification.

Examples:

excessive registrations;

huge advertisements;

excessive queries;

excessive metadata;

event storms;

endpoint churn.


Implementations SHOULD impose explicit limits.


---

89. Discovery Advertisement Size

Discovery records SHOULD have bounded size.

Large metadata SHOULD use references rather than unbounded inline data.

An implementation MUST reject records that exceed its configured maximum safely.


---

90. Discovery Privacy

Discovery metadata SHOULD follow data-minimization principles.

A service SHOULD advertise only information necessary for discovery and selection.

Sensitive metadata SHOULD be access-controlled.


---

91. Discovery Auditability

Security-sensitive discovery systems SHOULD record auditable events such as:

registration;

deregistration;

authentication;

authorization;

endpoint change;

security metadata change;

revocation.


Audit mechanisms belong to the applicable observability and security specifications.


---

92. Observability

Discovery implementations SHOULD expose metrics such as:

registrations
deregistrations
queries
query_failures
cache_hits
cache_misses
stale_records
endpoint_selection_failures
health_check_failures
lease_expirations
authorization_failures

The observability model belongs to the observability specifications.


---

93. Distributed Tracing

Discovery operations SHOULD be traceable where distributed tracing is enabled.

The implementation SHOULD correlate:

Discovery
    |
    v
Endpoint Selection
    |
    v
Remote Invocation

Invocation tracing remains governed by the remote-call and observability specifications.


---

94. Deterministic Discovery

For environments requiring reproducibility, discovery MAY operate in deterministic mode.

Deterministic discovery SHOULD define:

authority set;

query;

metadata snapshot;

filtering policy;

ordering;

selection algorithm.


Dynamic data MUST be captured or controlled if reproducibility is required.


---

95. Discovery and Real-Time Systems

Real-time profiles MAY impose strict limits on discovery.

A real-time implementation SHOULD avoid unbounded:

network queries;

registry waits;

DNS resolution;

retries;

health checks;

endpoint selection.


Critical real-time operations SHOULD use prevalidated endpoints where appropriate.


---

96. Embedded Systems

Embedded implementations MAY use static discovery.

Static discovery MAY omit dynamic registry functionality.

However, the resulting endpoint metadata MUST still conform to the applicable ULABI contract.


---

97. Mobile Systems

Mobile implementations MAY experience changing connectivity.

Discovery SHOULD account for:

network transitions;

temporary disconnection;

endpoint migration;

battery/resource limits;

intermittent availability.



---

98. Cloud Systems

Cloud implementations MAY use dynamic registries.

Cloud-specific discovery MUST remain compatible with the ULABI semantic model.

Vendor-specific metadata SHOULD remain extensions rather than replacing ULABI identities.


---

99. Accelerators

Accelerators MAY be discoverable as service endpoints.

Examples:

GPU;

NPU;

FPGA;

quantum processor;

specialized hardware.


The discovered endpoint MUST expose sufficient metadata to distinguish:

Capability
Device
Interface
Endpoint


---

100. Service Discovery Contract

A conforming discovery system MUST provide a semantic equivalent of:

discover(query)
    -> discovery_result

The exact programming API is implementation-specific.

The semantic operation MUST:

1. accept a discovery query;


2. identify candidate services or interfaces;


3. return candidate endpoints;


4. expose sufficient compatibility metadata;


5. expose freshness information where dynamic;


6. apply applicable discovery authorization;


7. preserve endpoint identity;


8. preserve interface identity.




---

101. Registration Contract

A conforming dynamic registry SHOULD provide semantic equivalents of:

register(advertisement)
renew(lease)
deregister(endpoint)

The exact API is implementation-defined.


---

102. Query Contract

A discovery query SHOULD support:

service identity
interface identity
version constraints
profile constraints
capability constraints
locality constraints
transport constraints
security constraints
resource constraints
health constraints
policy constraints

Unsupported query dimensions MUST be reported rather than silently ignored when they are REQUIRED.


---

103. Discovery Response Contract

A discovery response MUST distinguish:

successful discovery with zero endpoints

from:

discovery failure

where the implementation can make that distinction.


---

104. Failure Modes

Service discovery MUST account for:

No Service

No matching service exists.

No Endpoint

Service exists but no endpoint is currently discoverable.

Discovery Timeout

The discovery authority did not respond within the applicable limit.

Discovery Unavailable

The authority cannot currently be reached.

Discovery Unauthorized

The caller lacks permission to obtain the requested information.

Discovery Incompatible

The discovery authority cannot satisfy the query.

Stale Result

Available information has exceeded its validity period.

Conflicting Result

Multiple authorities provide incompatible information.

Invalid Advertisement

An endpoint advertisement fails validation.

Revoked Endpoint

The endpoint was previously valid but has been revoked.


---

105. Recovery Behaviour

Recovery MUST be policy-controlled.

Possible recovery:

Discovery failure
      |
      +-- cached data valid?
      |       |
      |       +-- YES -> use cache
      |
      +-- alternate authority?
      |       |
      |       +-- YES -> query alternate
      |
      +-- static fallback?
      |       |
      |       +-- YES -> use fallback
      |
      +-- otherwise -> escalate

An implementation MUST NOT use an unsafe fallback merely to avoid failure.


---

106. Graceful Degradation

If the preferred endpoint is unavailable, an implementation MAY select a compatible alternative.

The alternative MUST satisfy all mandatory constraints.

Example:

Preferred:
GPU + Streaming + Encryption

Alternative:
CPU + Streaming + Encryption

Allowed:
only if CPU execution is contractually acceptable.

The implementation MUST NOT silently substitute an endpoint that changes required semantics.


---

107. Compatibility Invariants

The following invariants are REQUIRED:

1. Interface identity MUST remain stable across endpoint migration.


2. Endpoint identity MUST distinguish separate endpoint instances.


3. Discovery MUST NOT grant authorization.


4. Discovery MUST NOT grant capabilities.


5. Discovery MUST NOT redefine serialization.


6. Discovery MUST NOT redefine remote invocation.


7. Discovery MUST NOT redefine distributed error semantics.


8. Discovery MUST NOT silently become consensus.


9. Expired dynamic discovery information MUST NOT be treated as permanently valid.


10. Security revocation MUST override stale discovery data.


11. Required compatibility constraints MUST be enforced.


12. Unknown required semantics MUST NOT be ignored.


13. Transport selection MUST remain separate from interface identity.


14. Service identity MUST remain distinct from endpoint location.


15. Reachability MUST NOT imply trust.




---

108. Security Requirements

A production-grade implementation SHOULD provide:

authenticated registration;

authenticated discovery where necessary;

integrity protection;

replay resistance;

authorization;

endpoint verification;

revocation;

metadata confidentiality where necessary;

resource limits;

rate limiting;

auditability.


Security-sensitive deployments MUST use the applicable ULABI security profiles.


---

109. Conformance Requirements

A conforming Service Discovery implementation MUST demonstrate:

Identity

stable service identity;

stable interface identity;

endpoint identity;

instance identity.


Discovery

discovery query;

endpoint discovery;

version discovery;

profile discovery;

capability metadata.


Freshness

TTL or equivalent freshness semantics;

stale-record handling.


Registration

registration;

renewal;

deregistration where dynamic.


Security

required authentication;

authorization separation;

endpoint verification.


Compatibility

version filtering;

profile filtering;

capability filtering;

graceful degradation according to policy.


Failure

discovery timeout;

unavailable authority;

empty result;

stale endpoint;

revoked endpoint.


Selection

eligible endpoint filtering;

policy-controlled selection.



---

110. Conformance Test Matrix

The conformance suite SHOULD include at least:

SD-001 Service identity validation
SD-002 Interface identity validation
SD-003 Endpoint identity validation
SD-004 Static discovery
SD-005 Direct discovery
SD-006 Registry discovery
SD-007 Dynamic discovery
SD-008 Registration
SD-009 Renewal
SD-010 Deregistration
SD-011 Lease expiration
SD-012 TTL expiration
SD-013 Version filtering
SD-014 Profile filtering
SD-015 Capability filtering
SD-016 Locality filtering
SD-017 Transport filtering
SD-018 Security filtering
SD-019 Resource filtering
SD-020 Health filtering
SD-021 Endpoint selection
SD-022 Duplicate endpoint handling
SD-023 Endpoint replacement
SD-024 Service migration
SD-025 Discovery cache
SD-026 Cache invalidation
SD-027 Stale discovery
SD-028 Discovery timeout
SD-029 Discovery authority failure
SD-030 Invalid advertisement
SD-031 Unauthorized registration
SD-032 Unauthorized discovery
SD-033 Revoked endpoint
SD-034 Conflicting authorities
SD-035 Unknown optional metadata
SD-036 Unknown required metadata
SD-037 Graceful degradation
SD-038 Discovery recovery
SD-039 Security metadata integrity
SD-040 Replay protection
SD-041 Rate limiting
SD-042 Resource exhaustion protection
SD-043 Deterministic selection
SD-044 Multi-registry discovery
SD-045 Failure-domain selection
SD-046 Cross-domain discovery
SD-047 Discovery observability
SD-048 Discovery-to-invocation correlation


---

111. Reference Data Model

A reference language-neutral model is:

ServiceIdentity
    namespace
    service_id

InterfaceIdentity
    namespace
    interface_id
    major_version

InstanceIdentity
    instance_id

EndpointIdentity
    namespace
    endpoint_id

Endpoint {
    service
    interface
    instance
    endpoint
    versions
    transports
    locality
    profiles
    capabilities
    security_requirements
    resource_limits
    health
    freshness
    lease
    metadata
}

These are semantic structures.

They MUST NOT require any particular programming language.


---

112. Reference Discovery Flow

Client
  |
  | discovery query
  v
Discovery Authority
  |
  | candidate endpoints
  v
Client
  |
  | validate identity
  v
Compatibility Filter
  |
  | compatible endpoints
  v
Security Filter
  |
  | authorized candidates
  v
Policy Filter
  |
  | eligible endpoints
  v
Endpoint Selection
  |
  v
Remote Call


---

113. Complete Discovery Lifecycle

Service Instance
       |
       v
Registration
       |
       v
Validation
       |
       v
Publication
       |
       v
Discovery
       |
       v
Compatibility Filtering
       |
       v
Security Filtering
       |
       v
Policy Filtering
       |
       v
Endpoint Selection
       |
       v
Invocation
       |
       +---- Health Monitoring
       |
       +---- Lease Renewal
       |
       +---- Capability Changes
       |
       +---- Version Changes
       |
       v
Deregistration / Expiration


---

114. Reference Implementation Responsibilities

A future reference implementation SHOULD provide the following logical modules:

ServiceIdentity
InterfaceIdentity
InstanceIdentity
EndpointIdentity

ServiceAdvertisement
DiscoveryQuery
DiscoveryResult

DiscoveryAuthority
DiscoveryRegistry
DiscoveryClient

RegistrationManager
LeaseManager
DeregistrationManager

DiscoveryCache
FreshnessManager
InvalidationManager

VersionFilter
ProfileFilter
CapabilityFilter
LocalityFilter
TransportFilter
SecurityFilter
ResourceFilter
HealthFilter

EndpointEligibility
EndpointSelector

DiscoveryPolicy
DiscoverySecurity
DiscoveryValidator

DiscoveryHealth
DiscoveryEvents
DiscoveryMetrics
DiscoveryAudit

DiscoveryRecovery
DiscoveryConformance

These are logical modules, not mandatory source-language modules.


---

115. Implementation Independence

ULABI does not prescribe:

Rust;

C;

C++;

Go;

Java;

Python;

Swift;

Kotlin;

Sankofa;

Zamani;

a specific database;

a specific network stack;

a specific operating system.


Any implementation may map these semantic modules onto its own architecture.


---

116. Integration Requirements

This document integrates with:

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

docs/security/security-model.md
docs/security/capability-security.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md

docs/observability/tracing.md
docs/observability/diagnostics.md
docs/observability/telemetry.md

docs/standards/conformance.md
docs/standards/test-suite.md

This document owns service discovery semantics.

Those documents own their respective domains.

No later document should need to redefine service identity, registration, endpoint advertisement, freshness, discovery filtering, or endpoint selection.


---

117. Schema Integration

The future schema layer SHOULD contain definitions equivalent to:

schemas/service-identity
schemas/interface-identity
schemas/instance-identity
schemas/endpoint-identity
schemas/service-advertisement
schemas/discovery-query
schemas/discovery-result
schemas/lease
schemas/freshness
schemas/health-state
schemas/discovery-event

The schemas MUST implement the semantics established by this document.

They MUST NOT create incompatible alternate meanings.


---

118. Test Integration

The future test suite SHOULD contain:

tests/distributed/service_discovery/

with logical groups:

identity/
registration/
discovery/
filtering/
selection/
freshness/
leases/
health/
security/
compatibility/
failure/
recovery/
multi_registry/
observability/


---

119. Conformance Integration

The future conformance suite SHOULD contain:

conformance/distributed/service_discovery/

with tests corresponding to the SD-* test identifiers defined in this document.


---

120. Reference Implementation Integration

A future reference implementation SHOULD contain a discovery subsystem equivalent to:

reference/
└── distributed/
    └── service_discovery/
        ├── identity
        ├── advertisement
        ├── registry
        ├── client
        ├── registration
        ├── leases
        ├── cache
        ├── filtering
        ├── selection
        ├── health
        ├── security
        ├── policy
        ├── recovery
        └── validation

The reference implementation MUST remain subordinate to the specification.

The specification is normative.


---

121. Required Code Modules

When ULABI moves from specification to implementation, the minimum logical code modules are:

service_identity
interface_identity
instance_identity
endpoint_identity

service_advertisement
discovery_query
discovery_result

discovery_authority
discovery_registry
discovery_client

registration
deregistration
lease_manager

freshness_manager
cache
invalidation

version_filter
profile_filter
capability_filter
locality_filter
transport_filter
security_filter
resource_filter
health_filter

eligibility
endpoint_selector

discovery_policy
discovery_validator
discovery_security

health_monitor
discovery_events

discovery_errors
discovery_recovery

discovery_metrics
discovery_tracing
discovery_audit

discovery_conformance

These module names are intentionally language-neutral.


---

122. Required Error Categories

A future implementation SHOULD expose semantic errors equivalent to:

ServiceNotFound
InterfaceNotFound
EndpointNotFound

DiscoveryUnavailable
DiscoveryTimeout
DiscoveryUnauthorized
DiscoveryIncompatible

InvalidAdvertisement
InvalidIdentity
InvalidVersion
InvalidProfile
InvalidCapability

EndpointExpired
LeaseExpired
EndpointRevoked

SecurityRequirementUnsatisfied
CapabilityRequirementUnsatisfied
ResourceRequirementUnsatisfied
LocalityRequirementUnsatisfied

ConflictingDiscovery
StaleDiscovery
DiscoveryRateLimited
DiscoveryResourceExhausted

The final error representation MUST integrate with:

docs/distributed/distributed-errors.md

rather than creating an independent distributed error system.


---

123. Final Normative Rule

The central rule of ULABI Service Discovery is:

> Service discovery identifies potentially usable implementations of a ULABI contract; it does not grant trust, authorization, capabilities, or permission to invoke them.



Therefore:

Discover
   ↓
Validate
   ↓
Check Compatibility
   ↓
Check Security
   ↓
Check Policy
   ↓
Select
   ↓
Negotiate
   ↓
Invoke

No implementation may legally collapse these stages in a way that bypasses required security, compatibility, authorization, or policy controls.


---

124. Conformance Statement

An implementation claiming:

ULABI Distributed Service Discovery

MUST implement the mandatory requirements of this specification.

An implementation MAY implement only a subset of discovery mechanisms, such as static discovery, provided that it clearly declares its supported profile.

It MUST NOT claim support for capabilities it does not implement.

A valid implementation declaration should therefore resemble:

ULABI Core                 ✓
ULABI Distributed          ✓
ULABI Service Discovery    ✓

Discovery Modes:
    Static                 ✓
    Direct                 ✓
    Registry               ✓
    Dynamic                ✓

Features:
    Version Discovery      ✓
    Capability Discovery   ✓
    Lease Management       ✓
    Health Metadata        ✓
    Security Filtering     ✓
    Locality Filtering     ✓
    Graceful Degradation   ✓

This allows independent implementations to demonstrate precisely what they support instead of making an ambiguous claim of "ULABI compatibility."


---

125. Architectural Boundary

This specification establishes the following permanent boundary:

ULABI
                      |
          +-----------+-----------+
          |                       |
     Contract Layer         Discovery Layer
          |                       |
     What interface?        Where is it?
     What semantics?        Which endpoint?
     What version?          Which capabilities?
     What types?            Which transport?
          |                 Which locality?
          |                 Which health?
          |                       |
          +-----------+-----------+
                      |
                Policy/Security
                      |
                   Invocation

ULABI Service Discovery MUST remain independent of any particular:

language;

compiler;

runtime;

operating system;

CPU;

hardware vendor;

cloud provider;

service registry;

networking protocol;

company;

project.


That independence is a normative architectural requirement.

## What should be done in the repository

I would **not** modify `distributed-abi.md`, `remote-calls.md`, `serialization.md`, `distributed-errors.md`, or `consensus-boundaries.md` merely to integrate this file. Those documents already exist and have their own ownership boundaries. The current repository shows all five as substantive documents, while `service-discovery.md` is the placeholder.  

### Phase 1 — independent file

**1. `docs/distributed/service-discovery.md`**

This is the file you should replace now.

It becomes the authoritative source for:

- service identity;
- endpoint identity;
- instance identity;
- registration;
- deregistration;
- discovery;
- discovery queries;
- advertisements;
- leases;
- TTL/freshness;
- endpoint eligibility;
- endpoint selection;
- discovery caching;
- discovery security metadata;
- discovery failure;
- discovery recovery;
- discovery conformance.

**No dependency on future code is required.**

That means once this file is completed, the specification itself is complete and should not need rewriting simply because an implementation is added.

---

## Phase 2 — schemas

After the specification is locked:

```text
schemas/
├── service-identity.*
├── interface-identity.*
├── instance-identity.*
├── endpoint-identity.*
├── service-advertisement.*
├── discovery-query.*
├── discovery-result.*
├── lease.*
├── freshness.*
├── health-state.*
└── discovery-event.*

These are data-contract files, not implementation-specific code.

They consume the semantics from service-discovery.md.


---

Phase 3 — tests

Then:

tests/distributed/service_discovery/
├── identity/
├── registration/
├── discovery/
├── filtering/
├── selection/
├── freshness/
├── leases/
├── health/
├── security/
├── compatibility/
├── failure/
├── recovery/
├── multi_registry/
└── observability/

The tests should implement the SD-001 through SD-048 contract defined above.

This prevents tests from inventing requirements that are absent from the specification.


---

Phase 4 — conformance

Then:

conformance/distributed/service_discovery/
├── identity/
├── registration/
├── discovery/
├── filtering/
├── selection/
├── freshness/
├── leases/
├── security/
├── compatibility/
├── failure/
└── recovery/

These are the tests that an independent ULABI implementation must pass to claim Service Discovery conformance.


---

Phase 5 — reference implementation

Only after the contract and tests exist:

reference/distributed/service_discovery/
├── identity
├── advertisement
├── registry
├── client
├── registration
├── leases
├── cache
├── filtering
├── selection
├── health
├── security
├── policy
├── recovery
└── validation

The implementation should be driven by the specification—not the other way around.


---

Required code-module inventory

For the eventual implementation, I would lock these logical modules into the architecture:

Module	Responsibility

service_identity	Stable service identity
interface_identity	Stable interface identity
instance_identity	Running service instance identity
endpoint_identity	Concrete endpoint identity
service_advertisement	Published endpoint metadata
discovery_query	Structured discovery requests
discovery_result	Discovery responses
discovery_authority	Discovery authority abstraction
discovery_registry	Dynamic registration registry
discovery_client	Client-side discovery
registration	Endpoint registration
deregistration	Endpoint removal
lease_manager	Registration lease lifecycle
freshness_manager	TTL/expiry handling
cache	Cached discovery state
invalidation	Cache invalidation
version_filter	Version compatibility
profile_filter	Profile compatibility
capability_filter	Capability requirements
locality_filter	Locality constraints
transport_filter	Transport constraints
security_filter	Security constraints
resource_filter	Resource constraints
health_filter	Health constraints
eligibility	Determines usable candidates
endpoint_selector	Selects endpoint
discovery_policy	Selection/fallback policy
discovery_validator	Validates advertisements
discovery_security	Discovery security integration
health_monitor	Endpoint health
discovery_events	Dynamic discovery changes
discovery_errors	Maps discovery failures into ULABI errors
discovery_recovery	Safe discovery fallback
discovery_metrics	Metrics
discovery_tracing	Trace integration
discovery_audit	Security/audit events
discovery_conformance	Conformance execution


These should be treated as logical contracts, so a Rust implementation could combine several into one module while a larger implementation could split them into separate files.

Integration dependency order

The clean dependency graph is:

service-discovery.md
        │
        ├── schemas
        │
        ├── tests
        │
        └── conformance
                │
                v
        reference implementation
                │
                ├── discovery_identity
                ├── registration
                ├── registry
                ├── leases
                ├── cache
                ├── filters
                ├── selection
                ├── security
                └── recovery

And externally:

ULABI Core
   │
   ├── Interface Identity
   ├── Versioning
   ├── Types
   └── Capabilities
   │
   v
Distributed ABI
   │
   ├── Service Discovery  ← THIS FILE
   ├── Remote Calls
   ├── Serialization
   ├── Distributed Errors
   └── Consensus Boundaries

This preserves the repository's existing architectural separation rather than creating another giant cross-cutting document. The master design explicitly establishes transport independence, explicit locality, distributed operation as a profile, and the principle that ULABI itself must remain independent of languages and implementations.

Important: I would leave ULABI-DESIGN.md alone for this step. The current architecture already provides enough authority for this specification, and changing the master document merely to add another cross-reference would violate your goal of completing files independently.

