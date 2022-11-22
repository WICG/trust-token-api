

# Private State Tokens vs Private Access Tokens

**_TL;DR: Private State Tokens (PST) and Private Access Tokens are both anti-fraud focused web APIs that rely on the same underlying protocol (privacypass), but serve different usecases/requirements and have different API shapes. PST serves as a limited, blinded token issued by a website/"issuer", whereas PAT includes an attestation from the platform (iOS) in addition to the signal provided by the issuer. They can both co-exist independently, but there is also potential to merge them into one "Private Tokens" API._**

Private State Tokens (PST) and Private Access Tokens (PAT) have a number of similarities and differences. This document is a very brief overview of the two features.

PST and PAT are both based on versions of the privacypass protocol being developed in the IETF. PST currently is built on top of an earlier version, while PAT is built on top of a version currently undergoing review in the IETF. The plan is to update PST to the latest version of privacypass, and the API surface includes version information to allow for this migration without backwards-compatibility challenges. privacypass is a generic protocol for issuing and redeeming tokens in a privacy-preserving manner to prevent linkability between the issuance and redemption.

PAT is intended as an attestation signal for access purposes that attests to the validity of the device via the platform. Apple's current use of that API issues tokens through the platform (iOS/macOS) and those tokens are then redeemed on resource accesses. Our interpretation of the [documentation](https://developer.apple.com/news/?id=huqjyh7k) is that each token represents that the bearer of the token had an authentic Apple device. PAT's design also moves to a three-party model where at issuance, an attester (Apple) attests to the client's validity, and then communicates to an issuer (Cloudflare/Fastly) to issue the tokens, this is intended to allow for rate-limiting by the (top-level) origin that tokens are used on, without revealing the origin to the attester. PAT usage is currently limited to top-level contexts only.

PST is intended to attest to a signal from one website to another, where a site that might have a greater signal about a user's authenticity (website with login or banking or website which challenged the user with a CAPTCHA or other anti-fraud technology) can issue tokens that are then redeemed on other sites where there's either no signal, or a more limited signal about the user's authenticity. In this model, each token issuer determines what the token itself represents and how it is used as a signal for anti-fraud. PSTs can be issued and/or redeemed on top-level, as well as embedded contexts.


<table>
  <tr>
   <td>
   </td>
   <td>Private Access Tokens
   </td>
   <td>Private State Tokens
   </td>
  </tr>
  <tr>
   <td>Underlying Technology
   </td>
   <td>privacypass (recent version)*
   </td>
   <td>privacypass (earlier version)
   </td>
  </tr>
  <tr>
   <td>Token Issuance Source
   </td>
   <td>Apple/Device
   </td>
   <td>Website (typically anti-fraud service providers)
   </td>
  </tr>
  <tr>
   <td>Redeemer
   </td>
   <td>Website (typically the CDN serving the top-level website)
   </td>
   <td>Top-level website, or third-party embeds (typically websites running their own anti-fraud systems or third-party anti-fraud service providers ie CAPTCHA)
   </td>
  </tr>
  <tr>
   <td>Issuer/Attester Split
   </td>
   <td>Yes, to support rate-limiting by origin, at issuance time PAT requires knowing both the client that is being attested and the origin that the token will eventually be sent to to avoid tracking the client against all the issuers, the issuance is split between an issuer and attester.
   </td>
   <td>No, since tokens are not issued to or rate limited to specific origins, the issuance only needs to learn the client that is being attested, so there is no need for separate attester/issuers.
   </td>
  </tr>
</table>


Given the differences between these two features, they are both useful to the web ecosystem, though some of the similarities between the redemption flow of these features provide a potential place to look at unifying the flows that websites go through to trigger these features, allowing for a shared API surface to redeem different types of tokens.


## privacypass Version

One comment about PST is that it is based on an earlier version of privacypass that doesn't provide a lot of the rate-limiting/public-verifiability features of the current version of privacypass. PAT is updated to the current version as it relies on those features, while PST doesn't need these features, however we plan to eventually update the PST cryptographic protocol to match the version of privacypass that gets standardized as part of the IETF process. We expect the standardization in privacypass to happen between November and Q1 next year, and have left versioning holes in the PST protocol so we can swap primitives and then deprecate the current version once that happens. (\* deprecation on the web platform is usually a time-consuming and drawn out  process, but as the only entities that have to update their code for PST is Chrome and the Issuers, and Issuers must explicitly register and provide a communication channel, we hope we'll be able to work through these direct channels to try to move them forward to newer versions as the needs and features available change)
