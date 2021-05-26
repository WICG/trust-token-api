<<<<<<< HEAD
# TrustTokenV3 Issuer Protocol

*Version 3 is a working version and is subject to change.*

This document documents the cryptographic protocol for the "TrustTokenV3PMB" and "TrustTokenV3VOPRF" experimental version of Trust Token. An issuer needs to support maintaining a set of keys and a key commitment endpoint, as well as implementing the Issue and Redeem cryptographic functions to sign and validate Trust Tokens. Experimental versions of Trust Token are not intended to be backwards-compatible with each other and will undergo rapid design/implementation changes during the experiment timeframe. Note that there is a distinct different between the issuer protocol version and the cryptographic protocol version, with the latter describing the underlying cryptographic primitives used in the issuer protocol.
=======
# TrustTokenV2 Issuer Protocol

This document documents the cryptographic protocol for the "TrustTokenV2PMB" and "TrustTokenV2VOPRF" experimental version of Trust Token. An issuer needs to support maintaining a set of keys and a key commitment endpoint, as well as implementing the Issue and Redeem cryptographic functions to sign and validate Trust Tokens. Experimental versions of Trust Token are not intended to be backwards-compatible with each other and will undergo rapid design/implementation changes during the experiment timeframe.
>>>>>>> origin/v2changes

This document uses TLS presentation language (https://tools.ietf.org/html/rfc8446#section-3) for structures and serialization.

## Public Issuer Interfaces

This section describes the public issuer interfaces that an issuer will need to support.

### Issuer Key Commitments

A Trust Token issuer should have an endpoint at a publicly accessible secure URL (HTTPS) that serves the current key commitments used in the Trust Token protocol. Requests to this endpoint should result in a JSON response of the following format:


```
Key commitment result
{
<<<<<<< HEAD
  <protocol_version>: {
    "protocol_version": <protocol version as a string, "TrustTokenV3PMB" or "TrustTokenV3VOPRF" for this>,
    "id": <key commitment identifier, as a monotonically increasing integer>
    "batchsize": <batch size>,
    "keys": {
      <keyID>: { "Y": <base64-encoded TrustTokenPublicKey>,
                         "expiry": <key expiry, encoded as a string representation of
                                    an integer timestamp in microseconds since the Unix
                                    epoch> },
      <keyID>: { "Y": <base64-encoded public key>,
                         "expiry": <key expiry, encoded as a string representation of
                                    an integer timestamp in microseconds since the Unix
                                    epoch> }, ...
    }
  },
  ...
=======
  "protocol_version": <protocol version, TrustTokenV2PMB or TrustTokenV2VOPRF for this>,
  "id": <key commitment identifier, as a monotonically increasing integer>
  "batchsize": <batch size>,
  "srrkey": <base-64 encoded SRRVerificationKey, in 32-byte RFC8032 encoding>,
  <keyID>: { "Y": <base64-encoded TrustTokenPublicKey>,
                     "expiry": <key expiry, encoded as a string representation of
                                an integer timestamp in microseconds since the Unix
                                epoch> },
  <keyID>: { "Y": <base64-encoded public key>,
                     "expiry": <key expiry, encoded as a string representation of
                                an integer timestamp in microseconds since the Unix
                                epoch> }, ...
>>>>>>> origin/v2changes
}
```

The commitment is a dictionary keyed by protocol version where each value is a self-contained key commitment for that version (for
current protocol versions, the protocol_version field is also included as a field of the individual commitment itself). To support compatibility with prior
key commitment formats, unknown fields in the top level dictionary should be ignored. The `id` field is used as a unique identifier to identify each key commitment
and to allow comparing the freshness of key commitments (larger values indicate a newer key commitment).

### Issuing Tokens

To support the issuance of tokens made via Trust Token calls by the client, server paths that support token issuance should parse the "Sec-Trust-Token" header as a base64 binary blob. This decoded binary blob should be interpreted as a **Trust Token Issuance Request** and passed into the _Issue_ crypto protocol along with the Issuance metadata based on the rest of the request. The result of _Issue_ (a **Trust Token Issuance Response**) should be base64 encoded and returned via the "Sec-Trust-Token" header in the HTTP response.


#### Issuance Metadata

<<<<<<< HEAD
As part of an issuance, two forms of metadata can be embedded into the token. Public metadata is embedded via the choice of key which is used as part of the issuance process and is passed into the _Issue_ method as the keypair selection. Private metadata is embedded via a cryptographically hidden bit in the signed token itself and is passed into the _Issue_ method as the private metadata boolean. TrustTokenV3VOPRF supports up to 6 buckets (keypairs) of public metadata and no private metadata, while TrustTokenV3PMB supports up to 3 buckets (keypairs) of public metadata and one bit of private metadata.
=======
As part of an issuance, two forms of metadata can be embedded into the token. Public metadata is embedded via the choice of key which is used as part of the issuance process and is passed into the _Issue_ method as the keypair selection. Private metadata is embedded via a cryptographically hidden bit in the signed token itself and is passed into the _Issue_ method as the private metadata boolean. TrustTokenV2VOPRF supports up to 6 buckets (keypairs) of public metadata and no private metadata, while TrustTokenV2PMB supports up to 3 buckets (keypairs) of public metadata and one bit of private metadata.
>>>>>>> origin/v2changes


### Redeeming Tokens

To support the redemption of tokens by the client, server paths that support token redemption should parse the "Sec-Trust-Token" header as a base64 binary blob. This decoded binary blob should be interpreted as a **Trust Token Redemption Request** and passed into the _Redeem_ crypto protocol. The result of _Redeem_ (a **Trust Token Redemption Response**) should be base64 encoded and returned via the "Sec-Trust-Token" header in the HTTP response. Additionally, the issuer should keep track of redeemed tokens to prevent token reuse from malicious clients.


#### Redemption Metadata

At redemption time, the token is decoded and provided via the redemption API. The issuer can then choose to encode this in the Redemption Record in whatever way it prefers.
<<<<<<< HEAD

#### Request Signing
=======
>>>>>>> origin/v2changes

In Version 3, the algorithm used for [request signing](https://github.com/WICG/trust-token-api#extension-trust-bound-keypair-and-request-signing) is [`ecdsa_secp256r1_sha256`](https://tools.ietf.org/html/rfc8446#section-4.2.3).

<<<<<<< HEAD

## TrustTokenV2PMB Crypto Protocol

This Trust Token crypto protocol is based on the PMBTokens design in https://eprint.iacr.org/2020/072 (appendix H) using P-384. This crypto protocol is used in both the V2 and V3 Trust Token protocol versions.  The necessary keys and function mappings are described below.
=======
## TrustTokenV2PMB Crypto Protocol

This Trust Token crypto protocol is based on the PMBTokens design in https://eprint.iacr.org/2020/072 (appendix H) using P-384. The necessary keys and function mappings are described below.
>>>>>>> origin/v2changes

### Keys

The Trust Token protocol primarily requires keypairs consisting of each public metadata bucket:

*   TrustTokenSecretKey/TrustTokenPublicKey - A Trust Token keypair used to sign and verify Trust Tokens.


#### TrustTokenSecretKey/TrustTokenPublicKey

These are keys used in Trust Token consisting of elliptic curve scalars and points. All scalars and points are sized based on the curve choice (P-384). Up to 3 keys may be configured in parallel in the key commitment.

Each keypair consist of the following:


```
opaque ECPoint<1..2^16-1>; // X9.62 Uncompressed point.
opaque Scalar<Ns>; // big-endian bytestring

struct {
  // Corresponding to the FALSE private metadata bit (x0, y0).
  Scalar x0;
  Scalar y0;
  // Corresponding to the TRUE private metadata bit (x1, y1).
  Scalar x1;
  Scalar y1;
  // Corresponding to the token validity (x˜, y˜).
  Scalar xs;
  Scalar ys;
} TrustTokenSecretKey;

struct {
  ECPoint pub0; // Corresponding to the FALSE private metadata bit.
  ECPoint pub1; // Corresponding to the TRUE private metadata bit.
  ECPoint pubs; // Corresponding to the token validity check.
} TrustTokenPublicKey;
```

### Serialization/Hashing

For the PMBTokens functions, the following serialization schemes and hashes are used internally using draft 07 of the hash-to-curve specification (https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07):


`Ht(t)` is defined to be P384_XMD:SHA-512_SSWU_RO_ with a big-endian bytestring input of `t` and a dst of "PMBTokens Experiment V2 HashT".

`Hs(T, s)` is defined to be P384_XMD:SHA-512_SSWU_RO_ with a bytestring input of `T` with the following content and a dst of "PMBTokens Experiment V2 HashS".

```
struct {
  ECPoint t;
  opaque s[Nn];
} T;
```

The hash-to-curve document does not define hash to scalars, so `Hc(x)` is defined to be the output of the [hash_to_field](https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07#section-5.2) function with the following parameters:

* `DST` is "PMBTokens Experiment V2 HashC"
* `F`, `p`, and `m` are defined according to the finite field `GF(r)`, where `r` is the order of P-384. Note this is a different modulus from `hash_to_field` as used in P384_XMD:SHA-512_SSWU_RO_.
* `L` is 72, derived based on P384_XMD:SHA-512_SSWU_RO_'s security parameter `k` (192), and `p` defined above.
* `expand_message` uses the corresponding function from P384_XMD:SHA-512_SSWU_RO_.

When used in the DLEQ proof, `Hc` takes the following input.

```
struct {
  uint8 label[6] = "DLEQ2\0";
  ECPoint X;
  ECPoint T;
  ECPoint S;
  ECPoint W;
  ECPoint K0;
  ECPoint K1;
} DLEQInput; // Used for DLEQ proof.
```

When used in the DLEQOR proof, `Hc` takes the following input.

```
struct {
  uint8 label[8] = "DLEQOR2\0";
  ECPoint X0;
  ECPoint X1;
  ECPoint T;
  ECPoint S;
  ECPoint W;
  ECPoint K00;
  ECPoint K01;
  ECPoint K10;
  ECPoint K11;
} DLEQORInput; // Used for DLEQOR proof.
```

When used in the DLEQ batching step, `Hc` is called once for each `e_i` output, with the following input. The `index` field contains `i`.

```
struct {
  uint8 label[11] = "DLEQ Batch\0";
  ECPoint pubs;
  ECPoint pub0;
  ECPoint pub1;
  ECPoint T'0;
  ECPoint S'0;
  ECPoint Wp0;
  ECPoint Wsp0;
  ...
  ECPoint T'n;
  ECPoint S'n;
  ECPoint Wpn;
  ECPoint Wspn;
  uint16 index;
} DLEQBatchInput; // Used for Batch proof.
```

### Issue Function

The _Issue_ function corresponds to the **AT.Sig** stage of the PMBTokens protocol.

Inputs:

*   count (the number of tokens being requested, from the IssueRequest)
*   nonces (the list of nonces to sign)
*   secretKey (the secret key that should be used to sign this request, determined by the public metadata)
*   publicKey (the public key corresponding to the secretKey)
*   keyID (the ID corresponding to this key)
*   privateMetadata (the boolean value corresponding to the private metadata value)
*   toIssue (number of tokens to issue)

Outputs:

*   issued (the number of tokens being issued)
*   keyID (the key used to sign these tokens)
*   signed (the list of signed nonces)
*   cs, us, vs, c0, u0, v0, c1, u1, v1 (the list of scalars corresponding to the zero-knowledge proof)

```
Issue:
  issued = min(count, toIssue)
  xb,yb = secretKey.x1, secretKey.y1
  if privateMetadata == 0:
    xb,yb = secretKey.x0, secretKey.y0
  signed = []
  T,S,W,Ws = [], [], [], []
  for i in 0..issued:
    T' = nonces[i]
    s ←$ {0, 1}λ
    S' = Hs(T', s)
    Wp = xb*T' + yb*S'
    Wsp = secretKey.xs*T' + secretKey.ys*S'
    T,S,W,Ws += T', S', Wp, Wsp
    signed += SignedNonce{s, Wp, Wsp}
  proof = DLEQbatched.P((publicKey,T,S,W,Ws),(secretKey.xs, secretKey.ys, xb, yb))
  return (issued, keyID, signed, proof)

DLEQbatched.P((X,T,S,W,Ws),(xs, ys, xb, yb)):
  e0,...,em = Hc(X,T,S,W,Ws)
  Tt = Sum(ej*Tj) // over j from 0 to m
  St = Sum(ej*Sj) // over j from 0 to m
  Wt = Sum(ej*Wj) // over j from 0 to m
  Wst = Sum(ej*Wsj) // over j from 0 to m
  dleqProof = DLEQ2.P((X,T,S,Ws), (xs,ys)) // cs,us,vs
  dleqorProof = DLEQOR2.P((X,T,S,W), (xb,yb)) // c0,c1,u0,u1,v0,v1
  return dleqProof + dleqorProof
```

Input Serialization:

The Trust Token Issuance Request contains an `IssueRequest` structure defined below.

```
struct {
  uint16 count;
  ECPoint nonces[count];
} IssueRequest;
```

Output Serialization:

The Trust Token Issuance Response contains an `IssueResponse` structure defined below.

```
struct {
  opaque s<Nn>; // big-endian bytestring
  ECPoint Wp;
  ECPoint Wsp;
} SignedNonce;

struct {
  Scalar cs;
  Scalar us;
  Scalar vs;
  Scalar c0;
  Scalar c1;
  Scalar u0;
  Scalar u1;
  Scalar v0;
  Scalar v1;
} DLEQProof;

struct {
  uint16 issued;
  uint32 key_id = keyID;
  SignedNonce signed[issued];
  opaque proof<1..2^16-1>; // Length-prefixed form of DLEQProof.
} IssueResponse;
```

### Redeem Function

The _Redeem_ function corresponds to the **AT.VerValid** and **AT.ReadBit** stage of the PMBTokens protocol.

Inputs:

*   token (The trust token to redeem)
*   client\_data (the client data sent as part of the redemption request to include in the SRR)
*   secretKey (the secret key that should be used to sign this request, determined by the public metadata)
*   keys (dictionary from known key IDs to secret/public keys)

Outputs:

*   public_metadata (the ID of the keypair used to generate this token)
*   private_metadata (the bit value of the private metadata, if aplicable)

```
Redeem:
  secretKey, publicKey = keys[token.key_id]
  T = Ht(token.nonce)
  if token.Ws != secretKey.xs*T+secretKey.ys*token.S:
    return 0 // Invalid validity verification.
  W0 = secretKey.x0*T+secretKey.y0*token.S
  W1 = secretKey.x1*T+secretKey.y1*token.S
  // These equalities are timing-sensitive as they could be a timing side-channel.
  isW0 = constantTimeEquality(tokenW, W0)
  isW1 = constantTimeEquality(tokenW, W1)
  if !(isW0 ^ isW1):
    return 0 // Internal Error
  privateMetadata = isW1
  return (token.key_id, privateMetadata)
```

Input Serialization:

The Trust Token Redemption Request contains a `RedemptionRequest` structure as defined below.

```
struct {
  uint32 key_id;
  opaque nonce<nonce_size>;
  ECPoint S;
  ECPoint W;
  ECPoint Ws;
} Token;

struct {
  opaque token<1..2^16-1>; // Bytestring containing a serialized Token struct.
  opaque client_data<1..2^16-1>;
  uint64 redemption_time;
} RedeemRequest;
```


Output Serialization:

The Trust Token Redemption Response contains a `RedemptionResponse` structure as defined below.

```
struct {
  opaque rr<1..2^16-1>;
<<<<<<< HEAD
} RedeemResponse;
```

## TrustTokenV2VOPRF Crypto Protocol

This Trust Token crypto protocol is based on the VOPRF design in https://datatracker.ietf.org/doc/draft-irtf-cfrg-voprf/. This crypto protocol is used in both the V2 and V3 Trust Token protocol versions.

### Keys

The Trust Token protocol primarily requires keypairs consisting of each public metadata bucket:

*   TrustTokenSecretKey/TrustTokenPublicKey - A Trust Token keypair used to sign and verify Trust Tokens.


#### TrustTokenSecretKey/TrustTokenPublicKey

These are keys used in Trust Token consisting of elliptic curve scalars and points. All scalars and points are sized based on the curve choice (P-384). Up to 6 keys may be configured in parallel in the key commitment.

Each keypair consist of the following:


```
opaque ECPoint<1..2^16-1>; // X9.62 Uncompressed point.
opaque Scalar<Ns>; // big-endian bytestring

struct {
  Scalar x;
} TrustTokenSecretKey;

struct {
  ECPoint pub;
} TrustTokenPublicKey;
```

### Serialization/Hashing

For the VOPRF functions, the following serialization schemes and hashes are used internally using draft 07 of the hash-to-curve specification (https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07):


`H2C(t)` is defined to be P384_XMD:SHA-512_SSWU_RO_ with a big-endian bytestring input of `t` and a dst of "TrustToken VOPRF Experiment V2 HashToGroup".

The hash-to-curve document does not define hash to scalars, so `H2S(x)` is defined to be the output of the [hash_to_field](https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07#section-5.2) function with the following parameters:

* `DST` is "TrustToken VOPRF Experiment V3 HashToScalar"
* `F`, `p`, and `m` are defined according to the finite field `GF(r)`, where `r` is the order of P-384. Note this is a different modulus from `hash_to_field` as used in P384_XMD:SHA-512_SSWU_RO_.
* `L` is 72, derived based on P384_XMD:SHA-512_SSWU_RO_'s security parameter `k` (192), and `p` defined above.
* `expand_message` uses the corresponding function from P384_XMD:SHA-512_SSWU_RO_.

When used in the DLEQ proof, `H2S` takes the following input.

```
struct {
  uint8 label[6] = "DLEQ2\0";
  ECPoint X;
  ECPoint T;
  ECPoint W;
  ECPoint K0;
  ECPoint K1;
} DLEQInput; // Used for DLEQ proof.
```

When used in the DLEQ batching step, `H2C` is called once for each `e_i` output, with the following input. The `index` field contains `i`.

```
struct {
  uint8 label[11] = "DLEQ BATCH\0";
  ECPoint pub;
  ECPoint BT0;
  ECPoint Z0;
  ...
  ECPoint Tn;
  ECPoint Zn;
  uint16 index;
} DLEQBatchInput; // Used for Batch proof.
```

### Issue Function

The _Issue_ **Blind**/**Evaluate**/**Unblind** stages of the VOPRF protocol.

Input Serialization:

The Trust Token Issuance Request contains an `IssueRequest` structure defined below.

```
struct {
  uint16 count;
  ECPoint nonces[count];
} IssueRequest;
```

Output Serialization:

The Trust Token Issuance Response contains an `IssueResponse` structure defined below.

```
struct {
  opaque s<Nn>; // big-endian bytestring
  ECPoint W;
} SignedNonce;

struct {
  Scalar c;
  Scalar u;
  Scalar v;
} DLEQProof;

struct {
  uint16 issued;
  uint32 key_id = keyID;
  SignedNonce signed[issued];
  opaque proof<1..2^16-1>; // Length-prefixed form of DLEQProof.
} IssueResponse;
```

### Redeem Function

The _Redeem_ function corresponds to the **VerifyFinalize** stage of the VOPRF protocol.

Input Serialization:

The Trust Token Redemption Request contains a `RedemptionRequest` structure as defined below.

```
struct {
  uint32 key_id;
  opaque nonce<nonce_size>;
  ECPoint W;
} Token;

struct {
  opaque token<1..2^16-1>; // Bytestring containing a serialized Token struct.
  opaque client_data<1..2^16-1>;
  uint64 redemption_time;
} RedeemRequest;
```


Output Serialization:

The Trust Token Redemption Response contains a `RedemptionResponse` structure as defined below.

```
struct {
  opaque rr<1..2^16-1>;
} RedeemResponse;
```

## Version History

V3 uses [`ecdsa_secp256r1_sha256`](https://tools.ietf.org/html/rfc8446#section-4.2.3) as the signing algorithm for request signing and updates the key commitment format to nest it in a version-keyed dictionary (and to move the set of keys to their own dictionary within the commitment). The `send-redemption-record` API is also updated to support sending multiple redemption records from different issuers with differing signing algorithms for future compatibility.

V2 introduces two protocol versions, each supporting a different arrangement of public and private metadata. It also enables the issuer to structure the Redemption Record as they choose, and removes the signing requirement.
=======
} RedeemResponse;
```

## TrustTokenV2VOPRF Crypto Protocol

This Trust Token crypto protocol is based on the VOPRF design in https://datatracker.ietf.org/doc/draft-irtf-cfrg-voprf/.

### Keys

The Trust Token protocol primarily requires keypairs consisting of each public metadata bucket:

*   TrustTokenSecretKey/TrustTokenPublicKey - A Trust Token keypair used to sign and verify Trust Tokens.


#### TrustTokenSecretKey/TrustTokenPublicKey

These are keys used in Trust Token consisting of elliptic curve scalars and points. All scalars and points are sized based on the curve choice (P-384). Up to 6 keys may be configured in parallel in the key commitment.

Each keypair consist of the following:


```
opaque ECPoint<1..2^16-1>; // X9.62 Uncompressed point.
opaque Scalar<Ns>; // big-endian bytestring

struct {
  Scalar x;
} TrustTokenSecretKey;

struct {
  ECPoint pub;
} TrustTokenPublicKey;
```

### Serialization/Hashing

For the VOPRF functions, the following serialization schemes and hashes are used internally using draft 07 of the hash-to-curve specification (https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07):


`H2C(t)` is defined to be P384_XMD:SHA-512_SSWU_RO_ with a big-endian bytestring input of `t` and a dst of "TrustToken VOPRF Experiment V2 HashToGroup".

The hash-to-curve document does not define hash to scalars, so `H2S(x)` is defined to be the output of the [hash_to_field](https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07#section-5.2) function with the following parameters:

* `DST` is "TrustToken VOPRF Experiment V2 HashToScalar"
* `F`, `p`, and `m` are defined according to the finite field `GF(r)`, where `r` is the order of P-384. Note this is a different modulus from `hash_to_field` as used in P384_XMD:SHA-512_SSWU_RO_.
* `L` is 72, derived based on P384_XMD:SHA-512_SSWU_RO_'s security parameter `k` (192), and `p` defined above.
* `expand_message` uses the corresponding function from P384_XMD:SHA-512_SSWU_RO_.

When used in the DLEQ proof, `H2S` takes the following input.

```
struct {
  uint8 label[6] = "DLEQ2\0";
  ECPoint X;
  ECPoint T;
  ECPoint W;
  ECPoint K0;
  ECPoint K1;
} DLEQInput; // Used for DLEQ proof.
```

When used in the DLEQ batching step, `H2C` is called once for each `e_i` output, with the following input. The `index` field contains `i`.

```
struct {
  uint8 label[11] = "DLEQ BATCH\0";
  ECPoint pub;
  ECPoint BT0;
  ECPoint Z0;
  ...
  ECPoint Tn;
  ECPoint Zn;
  uint16 index;
} DLEQBatchInput; // Used for Batch proof.
```

### Issue Function

The _Issue_ **Blind**/**Evaluate**/**Unblind** stages of the VOPRF protocol.

Input Serialization:

The Trust Token Issuance Request contains an `IssueRequest` structure defined below.

```
struct {
  uint16 count;
  ECPoint nonces[count];
} IssueRequest;
```

Output Serialization:

The Trust Token Issuance Response contains an `IssueResponse` structure defined below.

```
struct {
  opaque s<Nn>; // big-endian bytestring
  ECPoint W;
} SignedNonce;

struct {
  Scalar c;
  Scalar u;
  Scalar v;
} DLEQProof;

struct {
  uint16 issued;
  uint32 key_id = keyID;
  SignedNonce signed[issued];
  opaque proof<1..2^16-1>; // Length-prefixed form of DLEQProof.
} IssueResponse;
```

### Redeem Function

The _Redeem_ function corresponds to the **VerifyFinalize** stage of the VOPRF protocol.

Input Serialization:

The Trust Token Redemption Request contains a `RedemptionRequest` structure as defined below.

```
struct {
  uint32 key_id;
  opaque nonce<nonce_size>;
  ECPoint W;
} Token;

struct {
  opaque token<1..2^16-1>; // Bytestring containing a serialized Token struct.
  opaque client_data<1..2^16-1>;
  uint64 redemption_time;
} RedeemRequest;
```


Output Serialization:

The Trust Token Redemption Response contains a `RedemptionResponse` structure as defined below.

```
struct {
  opaque rr<1..2^16-1>;
} RedeemResponse;
```
>>>>>>> origin/v2changes
