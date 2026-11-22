# ZeroID v2 protocol draft

Status: **early design draft; not an implementation specification**

This document proposes the shape of an Ethereum-capable ZeroID protocol. Field names and encodings are provisional and must be reviewed before code is written.

## Design principles

- Keep v1 Bitcoin authentication separate and operational.
- Treat Ethereum account ownership as a proof method, not as a replacement for the ZeroID certificate authority.
- Bind every proof to a specific protocol, operation, audience, identity, chain, and nonce.
- Use deterministic, documented serialization.
- Keep authentication, certificate issuance, identity linking, and application authorization distinct.

## Identity descriptor

A v2 identity descriptor should identify the provider, authentication method, chain namespace, chain ID, and account. For EVM networks, CAIP-2 / CAIP-10 identifiers are candidates for canonical chain and account references.

Illustrative descriptor (not a finalized schema):

```json
{
  "protocol": "zeroid",
  "version": 2,
  "provider": "zeroid.eth",
  "method": "eip712",
  "chain": "eip155:1",
  "account": "eip155:1:0x..."
}
```

The provider label above is illustrative. Do not assume that a DNS name, ENS name, or provider identifier is available or controlled until governance and naming are decided.

## Challenge structure

A challenge should include at least:

- protocol name and version;
- operation, such as registration or identity-link;
- audience / service identifier;
- random nonce;
- issued-at and expiry;
- intended chain ID;
- exact identity descriptor;
- optional request identifier for audit correlation.

The verifier must reject unknown versions, malformed encodings, expired challenges, reused nonces, wrong audiences, wrong chains, and mismatched identities.

## Ethereum proof

EIP-712 typed structured data is a candidate for wallet signing because it provides domain separation and structured fields. The final type definitions, domain fields, wallet compatibility requirements, and signature validation rules must be fixed in the implementation specification.

Do not use personal-message signing and EIP-712 interchangeably. The verifier must know exactly which scheme was requested and reject signatures from other schemes.

For contract accounts, decide whether ERC-1271 validation is in scope. If supported, specify the chain and block context used for validation and how historical verification behaves. If not supported, state clearly that only externally owned accounts are accepted in the initial release.

## Certificate

A v2 certificate should be a versioned, canonical object issued by the ZeroID authority after successful proof verification. It should include:

- certificate schema version;
- issuer/provider identifier;
- subject identity descriptor;
- username claim, if issued;
- issued-at and expiry;
- certificate identifier;
- key / issuer identifier for signature verification;
- status or revocation reference.

The issuer signature authenticates the certificate. It is not a substitute for the user's Ethereum ownership proof, and the user's wallet signature is not a substitute for the issuer's certificate signature.

The v2 certificate must not be serialized into a legacy v1 signature field unless a client protocol explicitly defines and safely verifies that representation.

## Identity-link attestation

A link attestation binds one exact legacy identity descriptor to one exact v2 identity descriptor. It must reference both proof transcripts or their cryptographic digests, the challenge identifier, issuer, issuance time, and status information.

A link means that the linking procedure accepted proof of control of both identities at the time of linking. It does not mean that the two keys are interchangeable, that historical content was signed by both, or that every application must merge their permissions.

## Canonicalization and storage

Before implementation, define canonical JSON or another deterministic encoding, Unicode normalization rules for usernames, address casing/checksum rules, and hash/signature preimages. Never sign arbitrary JSON text whose key ordering or whitespace may vary.

Store link records append-only where practical. Define correction, revocation, and supersession events rather than silently rewriting history. Do not publish private challenge material or signing secrets.

## Open decisions

- EIP-712 type definitions and domain fields.
- EOA-only versus ERC-1271 contract-account support.
- Supported chain IDs and chain configuration governance.
- Certificate signature algorithm and key-rotation model.
- Certificate expiry, revocation, and offline verification behavior.
- Canonical username rules and collision handling across v1/v2.
- Link revocation and unlinking semantics.
- Whether the v2 authority is centralized, threshold-controlled, or otherwise governed.

No production implementation should rely on this draft until these decisions are resolved and interoperability fixtures exist.
