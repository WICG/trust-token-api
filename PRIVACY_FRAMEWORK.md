# Trust Token Privacy Framework


## Overview/Goals
The Trust Token API provides a mechanism that allows origins to transfer a small number of bits of information to cross-site/third-party contexts, in a manner that prevents linkability to the user’s identity on the same origin when in first-party context. In conformance with the [privacy principles](https://w3ctag.github.io/privacy-principles/#principle-prevent-cross-partition-recognition) that we envision for the web, we must ensure that the mechanism cannot be misused to form cross-site tracking vectors that allows identification of individual users at scale. In order to achieve this goal, this document discusses a framework for technical mitigations, and policy-based usage recommendations.
The mitigations are generally split between those enforced on the Issuers themselves, those that the Trusted Intermediary needs to enforce, those at issuance time and those enforced at redemption time. This document is intended to be a general framework for when developing policies.

Some example limits are described below, though exact values will need to be determined as the ecosystem surrounding Trust Token continues to evolve and change.

## Preventing Malicious use of Tokens
Issuers may choose to issue tokens based on their own logic, and for the most part UAs don't have visibility into the token meaning and how issuance decisions are made, so can't enforce the particular purpose a token might be used for. For instance, issuers may use tokens to track specific sets of users or to build bitmasks across multiple tokens. In order to mitigate this, the UA should implement a number of limits to combat these uses.

### Limiting amount of Information, instead of type of information stored in Tokens
Since different issuers will store different information within their tokens based on their business logic for determining who gets issued tokens, instead of trying to limit the type of information stored in the tokens, the Trust Token API limits the amount of information (treating them as authenticated cryptographically-limited cross-site storage in terms of privacy leakage/entropy). This limitation is provided through the choice of Trust Token version and the configurations that are supported by the UA.

### Example Information Entropy Limit
Initial experimentation and versions of the Trust Token API are limited to a total of log2(6) bits (\~2.6 bits, 3 public metadata values and one private metadata bit) of data, representing some combination of metadata values and keys available to be used. This choice is made to minimize the amount of data available in a token while still allowing the token to be flexible enough to provide a range of values (for instance representing varying levels of trust/distrust).

## Prevent Fingerprint Accumulation via Token Aggregation
Malicious sites/issuers can try to attack this API by trying to redeem many different tokens and use the results as a sort of bit mask to fingerprint the user. To mitigate this, UAs should place limits on the number of tokens that can be redeemed in a specific [redemption context](https://github.com/ietf-wg-privacypass/base-drafts/blob/master/draft-ietf-privacypass-architecture.md#redemption-contexts-redemption-contexts). On the web platform, these redemption contexts map to the storage sharded by the top-level site. These limits will need to be maintained across the lifetime of the top-level origins state since any redemption results can be stored in first-party storage and then paired with future aggregations.

### Example Limits on Redemption
To allow for sites to consume multiple types of trust signals and to be able to migrate between different sources of trust over time, initially each site will be allowed to redeem tokens from two different sources (issuers or number of different tokens redeemed) over a period of time. Depending on the use cases that use Trust Tokens, the UA may want to increase this limit or tie it into something like a privacy budget.

## Prevent personalized Key Commitments
A key requirement of Trust Token (and the underlying Privacy Pass) is a way of ensuring that different clients all receive the same set of key commitments at a particular point in time, to prevent malicious issuers from presenting different keys to different users. This is provided by some sort of trusted intermediary (key commitment service) that is responsible for verifying the key commitments clients see are identical. Generally trusted intermediaries should allow any issuer that abides by their key rotation policy and uptime requirements. Similar to the CT ecosystem, intermediaries should have published requirements for inclusion to avoid any cherry picking of supported issuers. UAs should also consider allowing for multiple intermediaries to be used to avoid a single intermediary being a centralized single point of failure.

## Limit Key Rotation Frequency
Trust Token issuers need to rotate their keys regularly both for the security of the key material used, as well as to invalidate old tokens issued under previous epochs (to avoid large pools of tokens being gathered by malicious parties and to update issuance logic). However, each key rotation introduces additional bits of entropy as the user repeatedly visits the site that can be used to identify individual users by issuers, the key that the client generates tokens against could reveal at what point in time the client interacted with the issuer, allowing the issuer to track clients based on what epoch of tokens they use. In order to allow for the former while defending against the latter, under normal operation, the trusted intermediary should enforce maximum key rotation frequencies. The choice of these limits should be based on the security requirements for the key lifetime compared to the lifetime of first-party state that might link issuances at different keys on each site.

### Emergency Key Rotation Mechanism
Beyond the normal key rotation limits, the trusted intermediary will need to establish a mechanism for allowing ‘emergency’ key rotations in the case that an issuer’s key material is lost or stolen. This policy should be such that issuers can keep running secure systems as necessary but can’t maliciously expose their keys repeatedly to bypass the normal key rotation limits.

### Example Limit
Key rotations will be allowed at a 60 day cadence, in keeping with best practices while limiting the aggregate information leakage over time. Issuers are allowed to perform an emergency key rotation, but must publish an incident report similar to those required by the CAB Baseline Requirements.
