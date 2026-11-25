# ULABI Cryptographic Profiles

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Authentication](authentication.md), [Secure loading](secure-loading.md), [Versioning](../../ULABI-VERSIONING.md)

## Scope

Define how profiles select and manage cryptographic mechanisms for identity, integrity and protected transport.

## Contract

- Use a reviewed cryptographic provider and profile-defined algorithms and parameters. Record the policy revision, key role, key ID and signature or channel context. Resolve algorithm agility and post-quantum options through published profiles.
- Bind signatures to the exact authenticated bytes and purpose. A verifier checks trusted keys, expiry and revocation under the selected policy. Reject unsupported mandatory algorithms and unauthorized downgrade.
- Keep key material in an authorized provider. Support key rotation with explicitly bounded overlap. Use approved profile modes and keep keys for separate purposes distinct.

## Implementation sequence

1. Load the permitted policy and resolve keys by role. Validate the canonical representation before verification and record a safe result with the policy revision.
2. Add fixtures for valid signatures, altered content, untrusted keys, expired keys and prohibited algorithms. Exercise key rotation for each supported profile.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| A previously valid signature references a revoked key | Apply the policy’s explicit revocation rule. |
| Peer offers only a prohibited algorithm | Reject establishment of the protected binding. |
| Signed metadata is altered | Fail integrity verification before loading. |
