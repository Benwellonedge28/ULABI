# ULABI Capability Lifecycle

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Authorization](authorization.md), [Runtime resource management](../runtime/resource-management.md), [Security model](security-model.md)

## Scope

Describe scoped host grants used by components to access a resource or operation.

## Contract

- A capability binds issuer, holder or delegation scope, resource, allowed operations, generation and validity. Use an opaque handle or a profile-validated capability representation.
- Derivation can narrow scope or validity. Delegation requires explicit permission and cannot grant rights the parent lacks. Validate holder, generation, scope, expiry and revocation at use.
- Associate each live grant with revocation and cleanup policy. On restart or rollback, reissue only currently authorized grants. Serialized handles use a transport-defined identity and must be validated before use.

## Implementation sequence

1. Issue the grant after authorization and record its scope. Resolve handles only through the trusted capability authority.
2. On revoke, invalidate affected grants and derived grants according to policy, then reconcile in-flight operations and associated resources.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Derived grant requests a wider scope | Deny issuance. |
| Expired grant is replayed | Deny the protected action. |
| Old component handle is used after restart | Reject the stale generation. |
