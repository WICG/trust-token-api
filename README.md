# Private State Token API Explainer

This document is an explainer for a potential future web platform API that allows propagating trust across sites, using the [Privacy Pass](https://privacypass.github.io) protocol as an underlying primitive.

This API was formerly called the Trust Token API and the repository and API surfaces are in the process of being updated with the new name.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Motivation](#motivation)
- [Overview](#overview)
- [Potential API](#potential-api)
  - [Trust Token Issuance](#trust-token-issuance)
  - [Trust Token Redemption](#trust-token-redemption)
  - [Forwarding Redemption Attestation](#forwarding-redemption-attestation)
  - [Extension: Private Metadata](#extension-private-metadata)
- [Privacy Considerations](#privacy-considerations)
  - [Cryptographic Property: Unlinkability](#cryptographic-property-unlinkability)
    - [Key Consistency](#key-consistency)
    - [Potential Attack: Side Channel Fingerprinting](#potential-attack-side-channel-fingerprinting)
  - [Cross-site Information Transfer](#cross-site-information-transfer)
    - [Mitigation: Dynamic Issuance / Redemption Limits](#mitigation-dynamic-issuance--redemption-limits)
    - [Mitigation: Allowed/Blocked Issuer Lists](#mitigation-allowedblocked-issuer-lists)
    - [Mitigation: Per-Site Issuer Limits](#mitigation-per-site-issuer-limits)
  - [First Party Tracking Potential](#first-party-tracking-potential)
- [Security Considerations](#security-considerations)
  - [Trust Token Exhaustion](#trust-token-exhaustion)
  - [Double-Spend Prevention](#double-spend-prevention)
- [Future Extensions](#future-extensions)
  - [Publicly Verifiable Tokens](#publicly-verifiable-tokens)
  - [Request mechanism not based on `fetch()`](#request-mechanism-not-based-on-fetch)
  - [Optimizing redemption RTT](#optimizing-redemption-rtt)
- [Appendix](#appendix)
  - [Sample API Usage](#sample-api-usage)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Motivation

The web ecosystem relies heavily on building trust signals to detect fraudulent or spammy actors. One common way this is done is via tracking an individual browser’s activity across the web, usually via associating stable identifiers across sites.

Preventing fraud is a legitimate use case that the web should support, but it shouldn’t require an API as powerful as a stable, global, per-user identifier. In third party contexts, merely segmenting users into trusted and untrusted sets seems like a useful primitive that also preserves privacy. This kind of fraud protection is important both for CDNs, as well as for the ad industry which receives a large amount of invalid, fraudulent traffic.

Segmenting users into very coarse sets satisfies other use-cases as well. For instance, sites could use this as a set inclusion primitive in order to ask questions like, “do I have identity at all for this user?” or even do non-personalized cross-site authentication ("Is this user a subscriber?").


## Overview

This API proposes a new per-origin storage area for “Privacy Pass” style cryptographic tokens, which are accessible in third party contexts. These tokens are non-personalized and cannot be used to track users, but are cryptographically signed so they cannot be forged.

When an origin is in a context where they trust the user, they can issue the browser a batch of tokens, which can be “spent” at a later time in a context where the user would otherwise be unknown or less trusted. Crucially, the tokens are indistinguishable from one another, preventing websites from tracking users through them.


## Potential API


### Trust Token Issuance

When an issuer.example context wants to provide tokens to a user (i.e. when the user is trusted), they can use a new Fetch API with the trustToken parameter:


```
fetch('<issuer>/<issuance path>', {
  trustToken: {
    type: 'token-request',
    issuer: <issuer>
  }
}).then(...)
```


This API will invoke the [Privacy Pass](https://privacypass.github.io) Issuance protocol:

*   Generate a set of nonces.
*   Blind them and attach them (in a Sec-Trust-Token header) to the HTTP request
*   Send a POST to the provided endpoint

When a response comes back with blind signatures in a Sec-Trust-Token response header, they will be unblinded, stored, and associated with the unblinded nonces internally in the browser. The pairs of nonces and signatures are trust tokens that can be redeemed later. Raw tokens are never accessible to JavaScript. The issuer can store a limited amount of metadata in the signature of a nonce by choosing one of a set of keys to use to sign the nonce and providing a zero-knowledge proof that it signed the nonce using a particular key or set of keys. The browser will verify the proof and may choose to keep or drop the token based on other metadata constraints and limits from the UA. Additionally, the issuer may include an optional `Sec-Trust-Token-Clear-Data` header in the response to indicate to the UA that it should discard all previously stored tokens. If the value of the header is `all`, then all previously stored tokens should be discarded before the newly issued tokens are stored. Other values in the header should be ignored.

### Trust Token Redemption


When the user is browsing another site (publisher.example), that site (or issuer.example embedded on that site) can optionally redeem issuer.example tokens to learn something about the trust of a user. One way to do this would be via new APIs:


```
document.hasTrustToken(<issuer>)
```


This returns whether there are any valid trust tokens for a particular issuer, so that the publisher can decide whether to attempt a token redemption.


```
fetch('<issuer>/<redemption path>', {
  trustToken: {
    type: 'token-redemption',
    issuer: <issuer>,
    refreshPolicy: {none, refresh}
  }
}).then(...)
```


If there are no tokens available for the given issuer, the returned promise rejects with an error. Otherwise, it invokes the PrivacyPass redemption protocol against the issuer, with the token (potentially, if specified by an extension, along with associated redemption metadata) attached in the `Sec-Trust-Token` request header. The issuer can either consume the token and act based on the result, optionally including a Redemption Record (RR) in the `Sec-Trust-Token` response header to provide a redemption attestation to forward to other parties. Additionally, the issuer may include the `Sec-Trust-Token-Lifetime` header in the response to indicate to the UA how long (in seconds) the RR should be cached for. When `Sec-Trust-Token-Lifetime` header value is invalid (too large, a negative number or non-numeric), UA should ignore the `Sec-Trust-Token-Lifetime` header. When `Sec-Trust-Token-Lifetime` header value is zero UA should treat the record as expired. In case of multiple `Sec-Trust-Token-Lifetime` headers, UA uses the last one. If `Sec-Trust-Token-Lifetime` header is omitted, the lifetime of the RR will be tied to the lifetime of the Trust Token verification key that confirmed the redeemed token's issuance.
The RR is HTTP-only and JavaScript is only able to access/send the RR via Trust Token Fetch APIs. It is also cached in new first-party storage accessible only by these APIs for subsequent visits to that first-party. The RR is treated as an arbitrary blob of bytes from the issuer, that may have semantic meaning to downstream consumers.

UA stores the RR obtained from the initial redemption. A publisher site can query whether a valid RR exists for a specific issuer using following API.

```
document.hasRedemptionRecord(<issuer>)
```

This returns whether there are any valid RRs from the given issuer.


To mitigate [token exhaustion](#trust-token-exhaustion), a site can only redeem tokens for a particular issuer if they have no cached non-expired RRs from that issuer or if they are the same origin as the issuer and have set the `refresh` parameter.


### Forwarding Redemption Attestation

Redemption Records are only accessible via a new option to the Fetch API:


```
fetch(<resource-url>, {
  ...
  trustToken: {
    type: 'send-redemption-record',
    issuers: [<issuer>, ...]
  }
  ...
});
```


The RRs will be added as a new request header `Sec-Redemption-Record`. The header contains a list of issuer and redemption record pairs corresponding to each requested redemption record. This option to Fetch is only usable in the top-level document. If there are no RRs available, the request header will be empty.


### Extension: Trust Token Versioning

In order to allow multiple versions of Trust Token to be supported in the ecosystem, issuers include the version of the protocol (i.e. "TrustTokenV1") in their key commitments via the "protocol_version" field, and that is included in Trust Token requests via the Sec-Trust-Token-Version header. Trust Token operations should not be performed with issuers configured with an unknown protocol version.

In addition to the core cryptographic layer, signed requests' formats (see the next section) might change from version to version. In order to make adapting to these changes easier, we could employ a mechanism like the Sec-Trust-Token-Version header, or an addition to the requests' payloads, to tell consumers the version of the client that generated the request.


### Extension: Metadata

In addition to attesting trust in a user, an issuer may want to provide a limited amount of metadata in the token (and forward it as part of the RR) to provide limited additional information about the token.

This small change opens up a new application for Privacy Passes: embedding small amounts of information along with the token and RR. This increases the rate of cross-site information transfer somewhat, but introduces some new use-cases for trust tokens.

Once the metadata has been passed along to the redemption request, the issuer can include the value in some form in the RR for downstream partneres to read.

#### Extension: Public Metadata

Some information about the token can be publicly visible by the client. Issuers could use this limited information to run A/B experiments or other comparisons against different trust metrics, so they can iterate on and improve their token issuing logic.

This can be managed by assigning different keys in the key commitment to have different labels, indicating a different value of the public metadata. The client and issuer would be able to determine what the value of the public metadata is based on which key is used to sign at issuance time.

### Extension: Private Metadata

Other information about the token may need to be shared with themselves (on redemption) and other partners (via the RR) without revealing the metadata to the client. This could be used as a negative indicator of trust or other limited information that the client shouldn't know about. Private metadata makes it possible to mask a decision about whether traffic is fraudulent, and increase the time it takes to reverse-engineer detection algorithms. This is because distrusted clients would still be issued tokens, but with the private distrusted bit set.

This can be managed using the [PMBTokens construction](https://eprint.iacr.org/2020/072.pdf) that combines two different entwined secret keys being used to indicate the value of the metadata. At redemption time, the issuer can then check which of the two keys was used to retrieve the value of the private metadata.


### Extension: iframe Activation

Some resources requests are performed via iframes or other non-Fetch-based methods. One extension to support such use cases would be the addition of a `trustToken` attribute to iframes that includes the parameters specified in the Fetch API. This would allow an RR to be sent with an iframe by setting an attribute of `trustToken="{type:'send-redemption-record',issuer:<issuer>,refreshPolicy:'refresh'}"`.

## Privacy Considerations


### Cryptographic Property: Unlinkability

In Privacy Pass, tokens have an unlinkability property: token issuance cannot be linked to token redemption.

The privacy of this API relies on that fact: the issuer is unable to correlate its issuances on one site with redemptions on another site. If the issuer gives out N tokens each to M users, and later receives up to N*M requests for redemption on various sites, the issuer can't correlate those redemption requests to any user identity (unless M = 1). It learns only aggregate information about which sites users visit.

However, there are a couple of finer points that need to be considered to ensure the underlying protocol remains private.

#### Key Consistency

If the server uses different values for their private keys for different clients, they can de-anonymize clients at redemption time and break the unlinkability property. To mitigate this, the [Privacy Pass](https://privacypass.github.io) protocol should ensure that issuers publish a public key commitment list, verifies it is small (e.g. max 3 keys), and verifies consistency between issuance and redemption.


#### Potential Attack: Side Channel Fingerprinting

If the issuer is able to use network-level fingerprinting or other side-channels to associate a browser at redemption time with the same browser at token issuance time, privacy is lost. Importantly, the API itself has not revealed any new information, since sites could use the same technique by issuing GET requests to the issuer in the two separate contexts anyway.


### Cross-site Information Transfer

Trust tokens transfer information about one first-party cookie to another, and we have cryptographic guarantees that each token only contains a small amount of information. Still, if we allow many token redemptions on a single page, the first-party cookie for user U on domain A can be encoded in the trust token information channel and decoded on domain B, allowing domain B to learn the user's domain A cookie until either 1p cookie is cleared. Separate from the concern of channels allowing arbitrary communication between domains, some identification attacks---for instance, a malicious redeemer attempting to learn the exact set of issuers that have granted tokens to a particular user, which could be identifying---have similar mitigations.

#### Mitigation: Dynamic Issuance / Redemption Limits

To mitigate this attack, we place limits on both issuance and redemption. At issuance, we require [user activation](https://html.spec.whatwg.org/multipage/interaction.html#activation) with the issuing site. At redemption, we can slow down the rate of redemption by returning cached Redemption Records when an issuer attempts too many refreshes (see also the [token exhaustion](#trust-token-exhaustion) problem). These mitigations should make the attack take a longer time and require many user visits to recover a full user ID.


#### Mitigation: Allowed/Blocked Issuer Lists

To prevent abuse of the API, browsers could maintain a list of allowed or disallowed issuers. Inclusion in the list should be easy, but should mean agreeing to certain policies about what an issuer is allowed to do. An analog to this is the [baseline requirements](https://cabforum.org/baseline-requirements/) in the CA ecosystem.


#### Mitigation: Per-Site Issuer Limits

The rate of identity leakage from one site to another increases with the number of tokens redeemed on a page. To avoid abuse, there should be strict limits on the number of token issuers contacted per top-level origin (e.g. 2); this limit should apply for both issuance and redemption; and the issuers used on an origin should be persisted in browser storage to avoid excessively rotating issuers on subsequent visits. This should also apply for `document.hasTrustToken`, as the presence of tokens can be used as a tracking vector.


### First Party Tracking Potential

Cached RRs and their associated browser public keys have a similar tracking potential to first party cookies. Therefore these should be clearable by browser’s existing Clear Site Data functionality.

The RR and its public key are untamperable first-party tracking vectors. They allow sites to share their first-party user identity with third parties on the page in a verifiable way. To mitigate this potentially undesirable situation, user agents can request multiple RRs in a single token redemption, each bound to different keypairs, and use different RRs and keypairs when performing requests based on the third-party or over time.

In order to prevent the issuer from binding together multiple simultaneous redemptions, the UA can blind the keypairs before sending them to the issuer. Additionally, the client may need to produce signed timestamps to prevent the issuer from using the timestamp as another matching method.


## Security Considerations


### Trust Token Exhaustion

The goal of a token exhaustion attack is to deplete a legitimate user's supply of tokens for a given issuer, so that user is less valuable to sites who depend on the issuer’s trust tokens.

We have a number of mitigations against this attack:



*   Issuers issue many tokens at once, so users have a large supply of tokens.
*   Browsers will only ever redeem one token per top-level page view, so it will take many page views to deplete the full supply.
*   The browser will cache RRs per-origin and only refresh them when an issuer iframe opts-in, so malicious origins won't deplete many tokens. The “freshness” of the RR becomes an additional trust signal.
*   Browsers may choose to limit redemptions on a time-based schedule, and either return cached RRs if available, or require consumers to cache the RR.
*   Issuers will be able to see the Referer, subject to the page's referrer policy, for any token redemption, so they'll be able to detect if any one site is redeeming suspiciously many tokens.

When the issuer detects a site is attacking its trust token supply, it can fail redemption (before the token is revealed) based on the referring origin, and prevent browsers from spending tokens there.


### Double-Spend Prevention

Issuers can verify that each token is seen only once, because every redemption is sent to the same token issuer. This means that even if a malicious piece of malware exfiltrates all of a users' trust tokens, the tokens will run out over time. Issuers can sign fewer tokens at a time to mitigate the risk.


## Future Extensions


### Publicly Verifiable Tokens

The tokens used in the above design require private key verification, necessitating a roundtrip to the issuer at redemption time for token verification and RR generation. Here, the unlinkability of a client’s tokens relies on the assumption that the client and issuer can communicate anonymously (i.e. the issuer cannot link issuance and redemption requests from the same user via a side channel like network fingerprint).

An alternative design that avoids this assumption is to instead use _publicly verifiable_ tokens (i.e. tokens that can be verified by any party). It is possible to instantiate these tokens using blind signatures as well, achieving the [same unlinkability properties](#cryptographic-property-unlinkability) of the existing design. These tokens can be spent without a round trip to the issuer, but it requires either decentralized double spend protection, or round trips to a centralized double-spend aggregator.


### Request mechanism not based on `fetch()`

A possible enhancement would be to allow for sending Redemption Records (and signing requests using the trust keypair) for requests sent outside of `fetch()`, e.g. on top-level and iframe navigation requests. This would allow for the use of the API by entities that aren't running JavaScript directly on the page, or that want some level of trust before returning a main response.


### Optimizing redemption RTT

If the publisher can configure issuers in response headers (or otherwise early in the page load), then they could invoke a redemption in parallel with the page loading, before the relevant `fetch()` calls.

### Non-web sources of tokens

Trust token issuance could be expanded to other entities (the operating system, or native applications) capable of making an informed decision about whether to grant tokens. Naturally, this would need to take into consideration different systems' security models in order for these tokens to maintain their meaning. (For instance, on some platforms, malicious applications might routinely have similar privileges to the operating system itself, which would at best reduce the signal-to-noise ratio of tokens created on those operating systems.)

## Appendix


### Sample API Usage


```
areyouahuman.example - Trust Token Issuer
coolwebsite.example - Publisher Top-Level Site
foo.example - Site requiring a Trust Token to prove the user is trusted.

```


1.  User visits `areyouahuman.example`.
1.  `areyouahuman.example` verifies the user is a human, and calls `fetch('areyouahuman.example/get-human-tokens', {trustToken: {type: 'token-request', issuer: 'areyouahuman.example'}})`.
    1.  The browser stores the trust tokens associated with `areyouahuman.example`.
1.  Sometime later, the user visits `coolwebsite.example`.
1.  `coolwebsite.example` wants to know if the user is a human, by asking `areyouahuman.example` that question, by calling `fetch('areyouahuman.example/redeem-human-token', {trustToken: {type: 'token-redemption', issuer: 'areyouahuman.example'}})`.
    1.  The browser requests a redemption.
    1.  The issuer returns an RR (this indicates that `areyouahuman.example` at some point issued a valid token to this browser).
    1.  When the promise returned by the method resolves, the RR can be used in subsequent resource requests.
1.  Script running code in the top level `coolwebsite.example` document can call `fetch('foo.example/get-content', {trustToken: {type: 'send-redemption-record', issuer: 'areyouahuman.example'}})`
    1.  The third-party receives the RR, and now has some indication that `areyouahuman.example` thought this user was a human.
    1.  The third-party responds to this fetch request based on that fact.

### Sample Redemption Record Format
Different issuers/ecosystems should specify their own Redemption Record format so downstream consumers are able to parse it. One potential structure is:


```
{
  Redemption timestamp,
  ClientData: {
    Publisher Origin
  },
  Metadata: {
    Trust Token Key ID
  },
  Signature of the above verifiable by well-known public key of the issuer,
  SigAlg: <Well known algorithm identifier used to sign this RR>,
  optional expiry timestamp
}
```


The redemption timestamp is included to prove the freshness of the RR. The `ClientData` comes from the client as part of the redemption request and includes the publisher the redemption occurred on, allowing an issuer to bind a RR to a particular redeeming origin. The `Metadata` includes the key ID, so that consumers of the RR can query the issuer what different metadata values mean.
