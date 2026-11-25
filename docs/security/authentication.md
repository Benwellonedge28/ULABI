# ULABI Authentication at Boundaries

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Authorization](authorization.md), [Cryptography](cryptography.md), [Distributed calls](../distributed/remote-calls.md)

## Scope

Establish the identity of a caller or peer where the selected profile requires authentication.

## Contract

- Keep artifact identity separate from caller and session identity. Bind verified credentials to a principal, peer instance, audience and validity interval. Verify replay protection for remote authentication according to the profile.
- Use authenticated host context for local runtime identities and the profile-approved identity mechanism for remote peers. Validate issuer trust, credential freshness and revocation before forming the principal.
- Expose a principal and evidence reference to authorization while withholding raw credential material from logs and downstream components.

## Implementation sequence

1. Select a profile-approved verifier and issuer set for each logical endpoint. Validate context and credential, then attach the principal to the session.
2. On expiration or revocation, deny new work and handle admitted work according to profile policy. Emit a safe diagnostic with a stable failure category.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Credential targets another service | Reject the authentication attempt. |
| Credential is valid but has expired | Require fresh authentication before new work. |
| Component signature is valid but caller has no identity | Apply the operation’s authentication requirement independently. |
