# ZeroID v1 to v2 migration plan

Status: **proposal / documentation only**  
Scope: preserve the Bitcoin-based ZeroID deployment while designing an Ethereum-capable identity protocol.

## Goals

1. Introduce an Ethereum-backed identity method without invalidating existing Bitcoin-backed identities.
2. Keep legacy ZeroNet sites, including ZeroMe deployments that trust the existing ZeroID certificate authority, working without requiring an immediate site upgrade.
3. Make migration opt-in and cryptographically prove control of both the legacy and new identity.
4. Separate identity proof, certificate issuance, identity linking, and application trust policy.
5. Avoid changing historical posts, comments, signatures, usernames, or archived user records as a side effect of migration.

## Non-goals for this proposal

- No implementation of Ethereum signature verification yet.
- No change to the existing ZeroID site address, signing keys, certificate format, or published user databases.
- No automatic conversion of Bitcoin addresses into Ethereum addresses.
- No forced migration, username reassignment, or retirement date for v1.
- No claim that a linked v1 and v2 identity have identical application-level permissions. Each application retains its own authorization policy.

## Existing deployment: baseline observations

The repository currently identifies itself as the ZeroID site at `http://127.0.0.1:43110/zeroid.bit`. Its root `content.json` declares the `zeroid.bit` domain and includes large legacy datasets at `data/users.json` and `data/users_archive.json`, alongside the site frontend.

These published files are compatibility-sensitive. Before implementation, audit the complete current authentication flow, certificate serialization and verification, signing-key custody, API calls, and ZeroNet/ZeroNetX client behavior. This document deliberately does not assume that a new Ethereum signature can be placed into a legacy certificate field.

## Proposed identity model

Treat an identity as a tuple, not as a bare address:

- namespace / provider identifier;
- authentication method and protocol version;
- chain namespace and chain ID when applicable;
- account or key identifier;
- username claim, where applicable.

The v1 Bitcoin identity remains a first-class legacy identity. The v2 Ethereum identity is a distinct identity with its own proof and certificate. A v2 identity must never be inferred from, or silently substituted for, a v1 address.

## Migration flow: explicit identity linking

1. The user starts a link operation from an authenticated session.
2. The service issues a random, single-use challenge bound to the intended v1 identity, the proposed v2 identity, the operation, audience, and expiry.
3. The user proves control of the v1 identity using the exact legacy proof mechanism supported by the deployed client.
4. The user proves control of the v2 account by signing a domain-separated structured message. EIP-712 is a candidate format, subject to implementation review.
5. The service verifies both proofs, checks nonce consumption and identity/username conflicts, and records an immutable link event.
6. The service issues a link attestation. It does not rewrite or replace either identity's original certificate.
7. Applications may resolve the link, but must retain the original identity as the author of historical content.

If the legacy key is unavailable, do not treat knowledge of a username or possession of an existing profile URL as proof of ownership. Any recovery process must be separately specified, auditable, and explicitly weaker than dual-proof linking.

## Compatibility strategy

### Legacy clients and sites

- Preserve the v1 certificate bytes and verification semantics.
- Keep the existing v1 provider and signing behavior available.
- Do not require legacy sites to understand Ethereum, EIP-712, or link attestations.
- Do not repurpose a v1 provider identifier to mean v2.

### Updated clients and sites

- Support v1 and v2 through separate verification adapters.
- Resolve linked identities only when a valid link attestation is available.
- Preserve the original author identifier on historical records.
- Make trust policy explicit: v1-only, v2-only, either, or either with a link requirement.

A legacy client cannot be made to verify a new cryptographic scheme merely by changing ZeroID server code. If the classic client only understands its existing certificate signature format, v2-aware client support is a separate compatibility milestone.

## Rollout stages

1. **Inventory and fixtures:** document current API, certificate format, signatures, trusted-provider configuration, and client versions. Preserve real, non-secret legacy fixtures.
2. **Protocol specification:** settle canonical identity identifiers, challenge encoding, certificate and link-attestation schemas, expiry, revocation, and error behavior.
3. **Isolated v2 verifier:** implement and test Ethereum proof verification without changing v1 paths.
4. **Linking service:** add dual-proof linking, replay protection, conflict handling, and audit records.
5. **Client compatibility:** add v2 verification to an updated client while retaining v1 support.
6. **Application adoption:** update ZeroMe and other applications independently, with explicit trust-policy tests.
7. **Operational rollout:** enable v2 registration gradually, monitor failures, publish recovery procedures, and retain v1 support while legacy dependants exist.

## Required security properties

- Cryptographically secure, unpredictable, single-use nonces.
- Challenge binding to protocol, operation, audience, identities, expiry, and chain ID.
- Domain separation between environments and networks.
- Strict chain ID validation; never accept a signature from an unintended chain context.
- Replay protection and duplicate-link prevention.
- Canonical encoding and strict input validation.
- Explicit key rotation, certificate expiry, revocation, and recovery semantics.
- No private keys or signing secrets in the repository, site data, logs, or client-visible responses.
- No trust escalation merely because two identities are linked.

## Acceptance criteria for the migration design

- Existing v1 certificates remain verifiable by the unchanged legacy path.
- Existing ZeroMe-style consumers can continue using v1 without a v2 dependency.
- An invalid, expired, replayed, or cross-domain challenge is rejected.
- A link cannot be created without valid proof of both identities, except through a separately approved recovery flow.
- Historical content retains its original author identity and signature.
- Updated applications can distinguish identity resolution from authorization.
- A failed v2 rollout can be disabled without modifying v1 data.

## Open questions before implementation

- Which exact legacy client versions and authentication methods must be supported?
- What is the deployed v1 challenge/signature format, including any message-prefix or address-derivation rules?
- Is v2 limited initially to Ethereum mainnet, or should the protocol accept any explicitly configured EVM chain?
- Who operates the v2 certificate authority, and how are its signing keys protected and rotated?
- Should a username be globally unique across v1 and v2, or scoped by provider/namespace?
- What is the canonical representation of a linked identity in ZeroMe and other applications?
- Which ZeroNetX changes are required for v2 certificates to be consumed safely?

## Implementation guardrail

Do not begin destructive data migration or change `content.json`, legacy databases, certificate issuance, or production signing keys until the inventory, protocol specification, and compatibility tests are reviewed.
