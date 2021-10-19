# October 18, 2021

## Logistics

9-10AM PDT / 4-5PM GMT

## Agenda

* Overview from facilitators (5 minutes)
* Introductions (5 minutes)
* Presentations on anti-fraud/IVT problem space (15 minutes)
  * CAPTCHAs/Bot Detection/Website Fraud (Kevin Gibbons & Michael Ficarra, Shape Security
  * Accounts Safety (Daniel Margolis, Google)
  * Hoarding Attacks in Privacy Pass (Sofia Celi, Cloudflare)
* Brief overview on existing work (5 minutes)
* Open discussion (25 minutes)
* Next Steps (5 minutes)


### Existing Work
* [Privacy Pass](https://github.com/ietf-wg-privacypass/base-drafts/)
* [Trust Token API](https://github.com/WICG/trust-token-api)
* [Privacy Proxy, Private Access Tokens](https://github.com/tfpauly/privacy-proxy)


## Slides
* [Facilitator Slides](https://docs.google.com/presentation/d/1TBT2QWL28UAI2J_o5UoE_RlcHP9sVITpGDG6HMPdDBI/edit?usp=sharing)
* [CAPTCHAs/Bot Detection/Website Fraud](https://docs.google.com/presentation/d/1Yfmx6u7pa4b8X_iQHE2WNuqEZySLZKKSsrI2rzUp6Go/edit)

## Minutes/Notes
### Introductions
Kevin Gibbons: I'll try attaching text-to-speech to Google Docs  
Kaustubha Govind: Please review W3C code of ethics and professional conduct.  Raise your hand in Zoom for queueing.  We will record the presentation (\~first half) of the meeting, and make it available for the general public.  If the presenter of any of the three prepared talks objects, please let us know.  Later Q&A will not be recorded.  
…: W3C anti-harassment statement, no harassment.  This is a shared learning space, we are here to learn, let's all approach with curiosity.  
…: Please keep speaking to one person at a time, organizers will call on people to speak.  Take space & make space — want to be sure we hear many voices  
…: "Oops and Ouch": apologize and move forward if you make a mistake ("oops"; if you feel you've been hurt, call an "ouch".  Impact is much more important than intent.  
…: Please thumbs-up on Zoom to agree  
…: There are 81 participants, so please sign up in the Google Doc for introductions.  
…: I'm an Eng Manager in Google Chrome, objective is to tackle the problem of tracking on the web.  Reason for the session: We're thinking, along with other browsers, about how we can clamp down on the problems of privacy, but we've been hearing that some of the work impacts anti-fraud work on the web as well.  Trust Tokens were a part of offering tools to address this, but does not solve the problem on its own.  Plan to talk about existing work in the space, align on potential solutions.  
…: Introductions: Please update your Zoom name to reflect preferred pronouns; add why you're here to the list of participants section in (this) Google Doc  

### CAPTCHAs/Bot Detection/Website Fraud (Kevin Gibbons & Michael Ficarra, Shape Security)
Kevin Gibbons: I'm here with Michael Ficarra, here from Shape Security, to share our experience in fraud prevention  
…: Shape is involved in preventing certain types of fraud based in automation: Credential stuffing and checking credit card and gift card numbers.  Not involved in ads-related stuffd.  
…: We check all the traffic coming from a single user, checking whether they are performing specific actions too frequently, even when spread across multiple IP addresses.  
…: aims at making actions economically infeasible, not impossible.  
…: Core thing that makes this possible: Difficult to represent yourself as many different actors, so that you can block based on unreasonable volume of actions  
…: Other vendors in this space.  Any time you log into your bank, if you're not solving a CAPTCHA, your bank is using one of these vendors for this task.  
…: We're using browser technologies as part of solving this bucketing problem.  Fingerprinting is part of it.  We recognize things like trying to mask yourself, disagreement between multiple signs that show attempts to screen system.  
…: Not about cross-site tracking!  We only care about the stuff happening on a single origin too frequently.  
…: Cannot just use lack of history — people come in from libraries with a fresh profile between users, for example  
… can't just use IPs; fraud uses residential VPNs and botnets  
… CAPTCHA is cheap to get around, and not accessible enough to gate essential services like banks  
… Trust Tokens only convey trust from one place to another; we are often the first point of contact, no prior history to fall back on.  

### Accounts Safety (Daniel Margolis, Google)

Daniel Margolis: Lead on account safety team at Google.  A lot of my story is very similar to Kevin's.  
…: Gaining access to accounts as bulk account creation, and protection against taking over other people's accounts.  
…: Used for bulk activities like spam, and used for account recovery — gmail account is often recovery route for forgotten password  
…: What we do today includes bad or likely low-reputation sources of traffic.  This becomes more difficult as browsers change  
…: The things we use to know that we do trust you generally rely on browser mechanisms designed for that purpose, eg 1p cookies  
…: But ways to identify low-trust traffic are under the most threat by browser changes.  If we know you're you and have had the account for a long time, we want to use that to make it easier to get you doing your business  
…: rely on signals that make it more difficult to track people in the future.  Apple Private Relay, etc.  
…: New device flow: when you start signing into a new device you just bought, your setting it up from the same IP as your previous devices is important signal  
…: There are short-term cases where privacy is pitted against some of our ability to recognize that your access is normal.  In the long run, a lot of what we're relying on today for privacy and security wasn't intended for it, and we do have an opportunity to replace dual-use technologies (with both privacy risk and integrity benefit) with tech that is privacy-by-design and also security-by-design.  
…: Cookies being stolen is a great example, where cookies were not designed for secure storage, lots of potential here.  


### Hoarding attacks in Privacy Pass (Sofia Celi, Cloudflare)
Sofia Celi, Cloudflare: Privacy Pass protocol and hoarding attacks  
Privacy Pass: tokens we give out when you solve CAPTCHA, to help avoid needing to ask for another solve too often  
…: Abuse risk: hoarding or farming attack — attacker hoards tokens for a long time, then spend them all at once, to overwhelm a service, DOS  
…: More complex attack: hoarding of cookies — successful use of tokens turns them into trusted 1p cookies, avoiding future scrutiny.  Hoarder exchanges tokens for cookies, and if you can shuffle tokens between people, then can distribute so that cookies are associated with many IPs.  
…: usually looks like a big spike in Privacy Pass usage, lots of tokens being generated for a single client and then they don't get redeemed at all.  The tokens are all issued to a single IP address, then distributed to a botnet later.  
…: Mitigations: Blocking?  Not recommended — signals to the attacker that they have been found out, change to another device  
…: Can limit tokens issued to a single party — same problem.  
…: Fast verification times to avoid DOS?  In practice this isn't the problem.  
…: Require redemption prior to issuing more?  Regrades experience of honest participants, and attacker can realize it has happened and dodge again  
…: Key rotation, limits how long tokens can be hoarded for.  Rotating long-term key has drawbacks  
…: Use of metadata: PO-PRF: Token contains metadata, but tokens must remain un-linkable, don't want to increase linkability.  

### Overview of Existing Work
Steven: Privacy Pass, Trust Tokens. UA-Client Hints isn’t super-trusted  in getting information about the client. Hoping to get ideas of work for exploration, what can be useful.   

### Open Discussion
Questions from presentations  
Jeff Jaffe: Q: re first CAPTCHA presentation and anti-fraud: End beneficiary is a bank or some other organization that relies on anti-fraud techniques, not sure they are adequately represented in our community.  Can we reach out to the end beneficiaries to participate in these discussions?  
Kevin Gibbons: In practice, all of these banks to have anti-fraud teams, but they lean heavily on vendors — they don't have relevant experience, they are paying us (Shape) to have the experience for them. Hoping Shape will be involved with these conversations going forward.  
Jeff: But they have to be convinced that the community's solution meets their needs, so no substitute for the real end user being involved.  

Lisa Seeman: Need to remember people with cognitive impairments or learning disabilities.  Attempts to make things more secure often make them less accessible.  For example people with early-stage dementia who write passwords and tape them to their computer.  Agency workers are often asked to help with CAPTCHA puzzles.  Also risk of caretakers taking things that don't belong to them.  The better the "security", the more likely that a care working is involved in the transaction, makes it more likely for that trust fraud(?) to occur.  
…: There is a small task force on cognitive and learning disabilities in WAI, happy to explore assumptions about user needs.  
Sofia: I like what you're saying!  What kind of other abilities the attacker can have is a big part of our concern.  

Rolf Lindemann: Many drivers to use of CAPTCHAs today are sign-up scenarios.  Reason is that we use weak authentication mechanisms today.  Web Authentication already proposed in W3C, no need to think about CAPTCHAs from a security perspective, but at least brute-force attacks are no longer possible.  If we fix the problem at the core, we might reduce the need for CAPTCHAs significantly, instead of piling up more work-arounds.  We're talking about a second-level counter-measure; should instead go back to the root problem.  
…: The spec supports attestation, so the RP already understands whether there is a real person from a WebAuthn assertion.  There are alternatives to fix the problem at the core to render CAPTCHAs unnecessary in many, though not all, situations.  

Discussion Questions: 


Steven Valdez (Google), pre-seeded Q: What kind of anti-fraud/anti-abuse system do you work on? What technologies do your systems currently rely on, that may be indistinguishable from tracking? (third-party cookies, fingerprinting, client IP addresses, etc). Do you need to identify legitimate clients, or abusive clients? Is it sufficient to detect these actors in the aggregate, or do you need to identify individual clients?  

Karan Khanna: PM at Microsoft working on Ad Fraud (Apps fraud?) to working on diff technologies to address challenges, trust tokens, etc. Wanted to raise another challenge: we were specifically looking at cross-site understanding of token spend patterns. TT allow for issuance and redemption without user tracking; but fraud can occur between, eg. TT issued to legit user but bot on their computer redeems it. We have a proposal to capture issue and redemption statistics in a privacy-conscious way. Will link in chat.    

Brad Chen: My role at Google is in counter-abuse technology group. Broad responsibility against abuse we see. What kinds of tech we need. If you approach counter-abuse with signals, and reduce signals, we become reactive and “clean up the mess” rather than proa active. We’d rather become proactive, see problems before they escalate. Requires detecting stat anomalies by monitoring a broad range of signals. We’re gluttons for data. Some of the privacy approaches are regrettable, assuming that all data collection is malicious data-hoarding. Prevents us in counter-abuse from doing the best job. Forces us into poverty of signals, and then end up requiring everyone to log in, whcih I think is a bad direction for the web. We want to provide great experience for users who are anonymous, but can't do that if we can’t distinguish them from bad users and bots. Then the world ends up fragmenting into walled gardens. Interest in follow up with others interested in these problems.   

Per Bjorke: Ad fraud at Google . It’s an adversarial space, continual arms-race with bad actors. We need as many signals as possible to create obfuscation. If you have few signals, bad actors know which they need to fake. Richness in signals makes it more expensive for bad actors to know which they need to fake. Keep in mind with regard to TT, it’s only a communication of a level of trust in the browser; it doesn’t detect abuse, still rely on signals to know whether to issue tokens. Do you identify legit or illegit clients? Both.  

Rob Polansky, Akamai: Both anti-automation and anti-fraud products related to ATO. We leverage 1p cookies, cooperating with our customers to help identify repeat users, ID the right kinds of traffic to allow a particular customer. Sometimes SDKs, APIs, not just a browser. Native mobile apps. Both legit and abusive clients, depending on what organizational customer. Some want to disallow Tor, others say “they need to make purchases too.” Eye of the beholder. CAPTCHA is useful in some places, but not against manual fraud.   

Steven V: in the GDoc, we have a poll re next steps. Please +1 below. We’ll be on irc in #antifraud https://irc.w3.org #antifraud  



### Pre-seeded question ideas that we did not reach during the session:
* Are shorter-lived/ephemeral identifiers useful for combating fraud?
* Is detecting fraud after-the-fact valuable?
* Are you able to iterate/experiment with nascent technology or primarily looking to engage once things have been standardized?


### Poll
Q1: What are your preferences for next steps?
* Another one-off session (such as this one) - 0
* A sustained form, or series of sessions - 10
* Form a W3C Community Group - 19
* No next steps needed - 1


### Future Areas of Interest
* Low-friction risk assessment techniques during payment flows; see for example draft [EMV 3-D Secure requirements for risk assessment](https://github.com/w3c/wpsig/blob/gh-pages/3ds-reqs.md)
* Using WebAuthn as CAPTCHA replacement - in some use cases (e.g. user sign-up or sign-in, payments)
* Cross-site understanding of Trust Token spend patterns to mitigate fraudulent usage of tokens: [MSEdgeExplainers/IssuerRedemptionStatistics.md](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/TrustTokenExtensions/IssuerRedemptionStatistics.md)
