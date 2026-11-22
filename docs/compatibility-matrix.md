# ZeroID compatibility matrix

Status: **proposal**

This matrix describes intended behavior during a gradual v1-to-v2 migration. It is not a claim that current legacy clients support v2.

| Consumer | v1 Bitcoin identity | v2 Ethereum identity | Linked v1 + v2 identity |
|---|---|---|---|
| Unmodified classic client | Must continue to work | Not supported unless its existing protocol explicitly supports it | Sees and uses the v1 identity only |
| Unmodified classic ZeroMe | Must continue to work | Must not be assumed to work | Continues using the historical v1 identity |
| Updated ZeroNetX client | Must remain supported | Supported after v2 verifier ships | Can resolve link after validating its attestation |
| Updated ZeroMe | Supported | Supported after application integration | May present one logical profile while preserving original authorship |
| Other updated applications | Per-app trust policy | Per-app trust policy | Per-app trust policy; linking alone grants no permissions |

## Required invariants

1. V1 certificate bytes, provider identity, and verification semantics are not changed by the v2 rollout.
2. A v2 signature is never passed to a v1 verifier as though it were a legacy signature.
3. Legacy applications do not need to fetch or understand v2 data to keep operating with v1.
4. Identity resolution does not rewrite historical authorship, signatures, comments, or posts.
5. A link attestation is not an authorization grant.
6. Each application explicitly chooses which identity versions and providers it trusts.
7. A v2 outage or rollback must not disable v1 authentication.

## Test scenarios

- Existing v1 user logs into an unchanged classic client and classic ZeroMe.
- Existing v1 user logs into an updated client without linking an Ethereum account.
- New v2 user logs into a v2-aware client and updated application.
- A user links v1 and v2 using valid proofs for both.
- Linking fails if either proof is invalid, expired, replayed, or bound to another audience.
- An updated application resolves a valid link but preserves the v1 identity as author of old content.
- An application configured as v1-only rejects v2-only identities.
- An application configured as v2-only rejects v1-only identities.
- Disabling v2 leaves v1 login and certificate verification unchanged.

## Rollout policy

Ship v2 as an additive capability. Keep v1 support enabled for as long as supported applications or users depend on it. Any future v1 retirement requires a separate proposal, published migration window, consumer inventory, and tested recovery plan.
