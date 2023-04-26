# [WIP] Private State Tokens Issuer Registration

> :warning: This is a WIP Issuer Registration process, a separate repository will be made with the final policy for registering with Chrome.

With the [Private State Tokens](https://developer.chrome.com/en/docs/privacy-sandbox/trust-tokens/) API, a limited amount of information can be conveyed from one browsing context to another (for example, across sites) to help combat fraud, without passive tracking. This readme describes the process of how websites can issue Private State Tokens.
To apply for an issuer (and its key commitments) to be included within Chrome, the issuer website’s operator must open a new [issue](https://github.com/GoogleChrome/private-tokens/issues/new) on [this repository](github.com/googlechrome/private-tokens) using the “New Issuer” template which specifies:


*   Issuer name - Human-readable name representing the issuer website.
*   The [origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin) that is used for the issuing service (scheme, host, port). The scheme must be HTTPS.
*   An email or email alias that is monitored by the issuer’s operator for issues regarding the issuer website.
*   A public HTTPS endpoint that responds to key commitment requests as described in the [specification](https://wicg.github.io/trust-token-api/#issuer-public-keys).
*   A description of the issuer, describing the intended purpose of the tokens.
*   Acceptance of the [current Disclosure and Acknowledgement](#disclosure-and-acknowledgement) to be an issuer website.

Once an endpoint has been verified (to make sure that the endpoint responds with an [appropriate JSON dictionary](https://wicg.github.io/trust-token-api/#issuer-public-keys)), it will be merged into this repository and Chrome server-side infrastructure will begin fetching those keys roughly at an hourly rate and eventually distributing those keys to Chrome instances. Key commitments are only allowed to change every 60 days, and any rotation faster than that will be ignored.

Issuer entries in this repository will have an expiration date 6 months after the issuer request has been filed, issuers will need to refile before the 6 months elapse to ensure that Chrome continues to receive their key commitments. This expiration is to ensure that issuers are still active and acknowledge the latest specification and PST versions. Expired issuer entries will result in invalid tokens. To avoid interruption, it is recommended that you mark your calendars to refresh your configuration 2 weeks before it expires.

If there’s an emergency key loss/compromise, issuers will need to set a key of “emergency” in their key commitment. This may happen in the event that their company has been hacked and their keys have been compromised. To perform another “emergency” rotation, they will be required to first publicize a notification about the incident by opening a new issue with the appropriate issue template on this repository (this requirement is inspired  by the incident notification requirements in the [CA ecosystem](https://www.mozilla.org/en-US/about/governance/policies/security-group/certs/policy/)), on why they needed to do an emergency key rotation. Issuers who have not yet issued an incident notification and had it acknowledged may continue performing normal frequency key rotations but will be unable to perform additional “emergency” rotations. Key rotations out of the normal cycle can allow malicious issuers to gain additional information about a user based on the timing of their key rotations, so incident notification requirement is intended to prevent issuers from abusing the mechanism and rotating keys too frequently.

### Disclosure and Acknowledgement
By registering as an issuer, you acknowledge the following:


1. I understand the technical restrictions on key rotation frequency of 60 days in the PST API.2. I understand that attempts to bypass these restrictions may result in my issuer registration being revoked.3. I understand that my issuer registration will be valid for a period of six months after the key commitment is accepted, and that I will need to re-register in this repository following that six-month period.
---
Request Template:

**Summary:** Private State Token Issuer Request - {Issuer Name}

**Description:**

{Issuer Name}

[Origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin): {Origin}

Contact: {Contact Email}

Key Commitment Endpoint URL: {Issuer Key Commitment Endpoint}

Purpose: {Issuer purpose}

Disclosure & Acknowledgement: {Disclosure Version this request acknowledges}


4. I understand the technical restrictions on key rotation frequency of 60 days in the PST API.5. I understand that attempts to bypass these restrictions may result in my issuer registration being revoked.6. I understand that my issuer registration will be valid for a period of six months after the key commitment is accepted, and that I will need to re-register in this repository following that six-month period.