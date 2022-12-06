# Private State Token Deployment Types

## First-Party vs Third-Party Issuance
Issuers need to make a decision about what sort of information they will use as part of issuing tokens, and where issuances can happen, generally this decision is made based on where they can get good issuance signals, and we see issuers falling into two major categories.

![Issuance Types](https://raw.githubusercontent.com/wicg/trust-token-api/main/assets/deployment_issuance.png)

### First Party Issuance
First-Party issuers are instances where the issuer is the first-party site the user is interacting with, that then decides to perform a Private State Token issuance based on the first-hand information it has about the user based on the first-party interactions it sees. Some examples of this are things like an e-commerce or social media site measuring user interactions and the user having an account to derive the trust signal.

### Third Party Issuance
Third-Party issuers are instances where the actual issuer is not the same entity as the top-level site the user is interacting with, but rather some third-party entity that uses information the first-party is willing to share with it to build up an issuance decision, potentially making issuing decisions across a bunch of different origins. This sort of information sharing is acceptable, since the third-party won't be able to join the information shared with state from other first-parties (https://w3ctag.github.io/privacy-principles/#principle-identity-per-context). These sort of issuers may also use the type of token the user currently has to make decisions about providing other type of tokens when issuing in a different context. Some examples of this are things like CAPTCHA and IVT providers, where they have some sort of relation to the sites they're embedded in and provide IVT/Bot protection if tokens are available, while issuing new tokens based on challenges/data the first-party provides when they're not available. Potentially having some sort of state machine to transition between token metadata as the tokens are redeemed and issued in more locations that can increase the trust signal the issuer is willing to issue tokens to.

## Public vs Private Redemption
On the redemption side, we expect issuers to fall into two categories.

![Redemption Types](https://raw.githubusercontent.com/wicg/trust-token-api/main/assets/deployment_redemption.png)

### Public Redemption
Some issuers may be willing to allow any entity to redeem their tokens, providing a general trust signal that sites can consume to improve the web ecosystem. Particularly things like device attestations (described in https://github.com/WICG/trust-token-api#non-web-sources-of-tokens), about the integrity of the device, are likely to fall into this category, where the value is in attesting to real devices and sites being able to rely on that signal.

### Private Redemption
Most other issuers will fall into a private model, where redemption information will only be provided to their own redeemers or partners that they have a relationship with. This design allows issuers to choose where they are willing to provide trust information and avoid arbitrary sites from using their token. This doesn't affect the amount of information that a website can learn via the Private State Token API.

Within this model, there are a couple of implementation methods. First, the issuer can provide the information necessary to parse and understand the redemption record directly to the partner, allowing them to asynchronously use that information to decide whether the user is 'trusted'. The second option is that the partner sends the redemption record to some endpoint that the issuer provides that then tells the partner what the redemption record means, and whether the user is trusted, certain actions should be allowed or require extra verification. The redeemer can perform this kind of redemption by checking the origin of the request for a redemption or requiring some key that the site has to allow for a redemption.

## Example Deployments

**Third-Party Issuance, Private Redemption**: Most IVT/CAPTCHA companies will likely be using a third-party issuance, private redemption model. This reflects that these companies don't have strong first-party state and usually partner with the publishers that use their services to build up and consume their own signals. Similarly, due to the economic/business needs, these sorts of companies wouldn't want to allow any non-partnered entity from consuming their signal.

**Third-Party Issuance, Public Redemption**: For platform issuers, they will likely be using a third-party issuance (as the non-web source isn't considered first-party to each origin), public redemption model. Platforms are more likely to want to expose their attestation/trust signals to be consumed by anyone in order to promote the ability to protect user safety on their platform.

**First-Party Issuance, Private Redemption**: Larger publishers that have associated but separate partnered sites (news websites) might use a first-party issuance, private redemption model. They are able to establish the initial trust signal based on their primary site, while wanting partners they work with to be able to consume that signal
