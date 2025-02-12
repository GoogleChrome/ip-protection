# IP Protection

## Introduction

IP Protection is a feature that limits availability of a user’s original IP address in third party contexts in Incognito mode, enhancing Incognito's protections against cross-site tracking when users choose to browse in this mode.

IP addresses are essential to the basic functioning of the web, notably for routing traffic and to prevent fraud and spam. However, like third-party cookies, they can also be used for tracking. For Chrome users who choose to browse in Incognito mode, we wanted to provide additional control over their IP address, without breaking essential web functionality.  

To strike this balance between protection and usability, this proposal focuses on limiting the use of IP addresses in a third-party context. To that end, this proposal uses a list-based approach, where only domains on the masked domain list (MDL) in a third-party context will be impacted. 

As mentioned, the scope of this proposal is limited to Chrome’s Incognito mode.

## Proposal
Chrome is introducing an updated proposal to protect users’ IP addresses on qualifying traffic when browsing in Incognito mode. This protection applies to domains on the MDL in a third-party context, when users are signed into their Google account in the Chrome browser before starting an Incognito session.

**Goals**

- To improve user privacy by protecting users’ IP addresses in Incognito mode.
- To minimize disruption to the normal operations of servers, including the use of IP addresses for anti-fraud and anti-spam use cases, until there are alternative mechanisms in place.

### Privacy Proxy

#### Core requirements

- Destination origins on the masked domain list don’t see the client’s original IP address.
- Google can’t see the origin that clients interact with.
- No single proxy can see the origins that clients interact with and the clients' original IP address.
- IP addresses of the proxies cannot be used for cross-site identification.
- We are [using a list-based approach](#identifying-domains-and-the-masked-domain-list-mdl) and only domains on the list in a third-party context will be impacted.

To meet these requirements, this proposal routes connections via a two-hop proxy system that masks the user's original IP address and exposes a different masked IP address to domains. The masked IP address retains IP-based geolocation information down to a user’s coarse location (including country), but it can't be used to track an individual user across websites over time.

On a technical level, the IP Protection proxy infrastructure is designed to ensure that none of the entities operating the system can access both a user’s original IP address and the domain their traffic is being sent to. As a result, the entities operating the system aren't gaining access to users' traffic information. This privacy property is achieved by leveraging a two-hop proxy architecture and a blinded authentication scheme. Google will run the first proxy, and external CDNs will run the second proxy. This implementation ensures that Google's proxy can only view a user’s IP address but not the destination domain, while the CDN proxy can only view the destination domain but not the user’s original IP address, and that neither proxy can associate this data to the user's account used to grant access to the feature.

For proxied third-party traffic, the DNS will be resolved at the second proxy.

Chrome is taking the following steps to prevent user identifiers, including IP address, from being linked to origin-bound traffic:

- IP Protection will use CONNECT and CONNECT-UDP (MASQUE) to forward traffic. There is an end-to-end encrypted tunnel via TLS or QUIC from Chrome to the destination server. Separate connections will use different IP addresses from the proxies.

- We are using two proxies for improved privacy. A second proxy (proxyB) will be run by an external CDN, while Google runs the first proxy (proxyA). We then leverage two additional layers of HTTPS encryption, one at each proxy, to ensure that neither proxy can see both the client IP address and the destination. The two additional layers of encryption are from the QUIC proxy tunnels that Chrome uses to connect through each proxy server. When the request supports HTTPS the flow is the following: There is an encrypted QUIC tunnel between the client and proxyA; through it, there is another QUIC tunnel between the client and proxy B; through that tunnel, there is an end-to-end HTTPS connection between the client and the website, protected by QUIC or TLS. This end-to-end encryption prevents both proxies from seeing any browsing contents including the destination URL. The client-proxyB encryption prevents the proxyA from seeing the hostname of the website. The proxyB cannot see the IP address of the client because, from its perspective, all client traffic is coming from the proxyA. These nested tunnels are established using CONNECT and CONNECT-UDP. Both sets of proxies are run close to users to minimize latency to the user. 

- Chrome will employ an [RSA blind signature scheme](https://datatracker.ietf.org/doc/draft-hendrickson-privacypass-public-metadata/) between client authentication and proxy usage to isolate client identity from all proxy servers. This will include open source bounds on what metadata is shared with proxies during this authentication process. This metadata is used to provide the information that is necessary for the basic operation of the proxy B, which includes: (i) approximate token expiration to detect expired tokens and, (ii) IP Geolocation to assign the appropriate egress IP. Chrome is committed to ensuring a user cannot be uniquely identified via their unblinded token or associated authentication metadata.

### IP geolocation
IP-based geolocation is used by a swath of services within proxied third-party traffic, to comply with local laws and regulations and serve content that is relevant to users. Use cases include content localization, local cache assignment, and geo-targeting for ads. To support these needs but with privacy controls in place, the proxy will assign IP addresses that represent the user’s coarse location, including country. For more information, read the [IP Geolocation Explainer](https://github.com/GoogleChrome/ip-protection/blob/master/Explainer-IP-Geolocation.md).

As described in the IP Geolocation explainer, this architecture sets a user's approximate IP Geolocation by assigning an appropriate block based on the user's non-proxied IP address.

To accomplish this, Google will purchase IPv4 and IPv6 blocks, and defer BGP control of the blocks to external CDNs that Google has partnered with.

### Availability of the IP Protection feature
IP Protection will be available for users in Chrome’s Incognito mode only, on Android and Desktop platforms. Users will have the ability to disable IP Protection. For enterprise-managed versions of Chrome, IP Protection can be enabled, but it will be off by default.

The feature will be initially available in certain regions, and we plan to expand the availability over time. IP Protection will launch no sooner than May 2025.

### Identifying domains and the Masked Domain List (MDL)
IP Protection will use a list-based approach to determine which network traffic should be proxied. Domains that are on the list will be proxied when they appear in a third-party context (for example if the domain is embedded within another website). If that domain is accessed in a first-party context it will receive the original unmasked IP address. Domains that are not on the list will be unaffected in either third- or first-party contexts. This applies equally to Google-owned and non-Google-owned domains.

#### The Masked Domain List Criteria
We’ve developed the following criteria to identify which domains should be on the Masked Domain List (MDL) and therefore receive masked IP addresses.

**MDL Inclusion Criteria**

The MDL will be comprised of domains that fulfill the following criteria:

  **The domain is embedded as a third-party domain** and therefore in a position to collect information about a user or their device across multiple sites that aren’t owned by the data collector ([see below for how we plan to identify ownership](#first-party-vs-third-party-determination)).

  In addition, the domain also meets at least one of the following criteria:

  - The domain serves one of the following business purposes:
    - Serving of ads
    - Targeting of ads
    - Measuring ad effectiveness
    - Collection of user data for ads, commerce or marketing related activities

    These business purposes have been selected because they indicate a heightened risk that an embedded domain could have a business incentive to collect users’ activity across sites for commercial purposes.

  OR
  - The domain collects user or device information in a way that appears likely to support re-identification of users or devices across contexts.

While the criteria should be seen as largely stable and durable, the MDL itself is subject to ongoing evolution and change. This is driven by the refinement of detection mechanisms and the dynamic emergence and disappearance of domains that meet these criteria.

#### Composition of the Masked Domain List
Google has partnered with [Disconnect.me](http://Disconnect.me), a prominent internet privacy leader who also collaborates with other web browsers. Chrome will leverage Disconnect.me to identify domains that align with Chrome’s established criteria. Additionally, Chrome has developed a methodology to identify widely used JavaScript functions that provide consistent outputs from stable and high-entropy web APIs and can therefore be used to construct high entropy probabilistic identifiers. These functions are then detected when they are loaded on websites in a third-party context, resulting in a list of domains that serve scripts with these characteristics that become part of the MDL and are therefore proxied. The detection pipeline that looks for these patterns of API misuse considers all domains, including Google’s own domains.

#### Publication of the Masked Domain List
The MDL ([initial version](https://github.com/GoogleChrome/ip-protection/blob/master/Masked-Domain-List.md)) will be hosted on GitHub. Periodically, domains may be added or removed based on the fingerprinting detection system and updates to Disconnect's published list. Chrome will also remove domains that have successfully obtained an appeal. The published MDL will be the latest version used by Chrome.

#### Appeals
We recognize the importance of implementing an appeals process for our list-based approach. Appeals permit companies to make a claim that their domain on the MDL does not meet the inclusion criteria and ought to be removed, thereby allowing that domain to continue to receive users' original IP addresses in a third-party context in Incognito. Before launching IP Protection, we will establish the appeals process to ensure companies have adequate opportunity to seek an appeal and receive a decision. The appeals process is being designed to align with governance principles for Privacy Sandbox under discussion with the UK’s Competition and Markets Authority ([see CMA's 2024 Q3 report](https://www.gov.uk/cma-cases/investigation-into-googles-privacy-sandbox-browser-changes#q2q3-2024)).

#### First-party vs Third-party determination
IP Protection will apply to the domains on the MDL only when accessed in a third-party context in Incognito, thus we must have a mechanism to determine what is first-party and what is third-party on a contextual basis.  If the domain for a resource on a page matches the top level domain, that will be considered first-party context, even if the domain itself is on the MDL.  However, simply using domain structures is insufficient as a mismatch does not inherently mean the context is third-party. Websites are often constructed using a modular approach, where various resources are provided by different domains, even if they are operated by the same company. While masking IP addresses across multiple domains enhances user privacy, it offers limited privacy gain when those domains are service domains under common ownership or, to a certain extent, if they have an affiliation that is clearly presented to users. Additionally, it imposes an unnecessary burden for web developers operating such websites. As such, Privacy Sandbox needs to provide some model that reduces the burden where reasonable to do so.

Given this is a similar problem statement to that which [Related Website Sets](https://developers.google.com/privacy-sandbox/3pcd/related-website-sets) (RWS) was created to address, IP Protection will build upon RWS as a means of determining the boundaries between first- and third-party contexts such that domains on the list that appear embedded on a website in the same Related Website Set will be treated as first-party context.

For example, consider an analytics company that operates a domain named “B”. This domain serves as both its top-level domain and as an embedded domain for gathering data related to its analytics business, which has led Domain B to be included on the MDL. Domain B may be encountered in three different situations:
  1. Domain “B” loads in another top level domain (e.g. domain “A”) with which it does not share a RWS. The connection to B will be proxied and the user's IP address will not be visible to the site.
  2. A user navigates to domain “B” (either by typing the domain directly into the browser or by navigating through a link). In this case, B will be able to observe the user’s original IP address, instead of the proxied IP address.
  3. Domain “B” loads in another top level domain (e.g. domain “C”) that belongs to the same Related Website Set. B will continue to receive the user's original IP address.

![Three side‐by‐side diagrams illustrate how “Site B” receives either a masked or original IP address depending on the context: Left diagram (light gray): “Site A” embeds “Site B.” Because B is on the masked domain list (MDL) and is loaded in a third‐party context, it only sees a masked IP. Center diagram (light yellow): “Site B” as a first‐party site sees the user’s original IP Right diagram (darker yellow): “Site C” embeds “Site B.” Both are in the same Related Website Set (RWS), so “Site B” gets the original IP.](./ipp-mdl-example.png)

Similar to how browsers handle third-party cookie blocking, IP Protection considers all subdomains of a [registrable domain](https://web.dev/articles/url-parts#registrable-domain) on the MDL as part of the same domain and will therefore also receive the same treatment. Additionally, any subdomains under domains in the [private section of the Public Suffix List](https://publicsuffix.org/list/public_suffix_list.dat#:~:text=//%20%3D%3D%3DBEGIN%20PRIVATE%20DOMAINS%3D%3D%3D) would be considered third-party to each other unless they are in the same RWS; since such a domain would become an "eTLD", and hence each subdomain is considered its own registrable domain. RWS will be honored for IP Protection in Incognito independent of whether RWS is applicable to third-party cookie blocking in Incognito.

To reduce potential disruptions to websites and services, until companies create Related Website Sets, Chrome will temporarily employ a best effort approach to deduce domain ownership leveraging an entity mapping created by [Disconnect.me](http://Disconnect.me). Additionally, in cases where a company has not previously submitted a RWS and our deduced approach contains errors, the company has the option to submit a RWS to rectify any necessary corrections. 

### Anti-Fraud and Anti-Spam Strategy and Implementation
Chrome recognizes that fighting fraud and spam is critical to keeping a healthy and safe online ecosystem for users, publishers and advertisers, and that, today, IP addresses play a critical role in accomplishing this, whether it is preventing fraudulent commercial transactions or spam at scale. 

Internet proxies provide users with increased anonymity online. For this same reason, proxies are useful to potential attackers that want to conceal their activities, such as executing a Denial-Of-Service (DOS) attack. Proxy defensibility measures are the set of tactics that IP Protection will employ to decrease the risk of our proxies being leveraged by potential attackers and they include:

  A. Rate-limiting access to the proxies<br>
  B. Limiting issuance of authentication tokens<br>
  C. Reporting of fraudulent behavior<br>
  D. Probabilistic Reveal Tokens<br>

**A. Rate-limiting access to the proxies**

IP Protection will use client authentication to limit the ability of bad actors to leverage the proxies to amplify attacks on services on the Masked Domain List. Therefore, IP Protection will only be available to users that have been authenticated using the Google account they're signed in with in the Chrome browser prior to opening a new Incognito window.

It's important to note that, unlike a majority of services that use authentication, IP Protection does it in a way that prevents any collection of data about that user's activity tied to their account as a result of their use of the feature. To achieve this, Chrome will employ an [RSA blind signature scheme](https://datatracker.ietf.org/doc/html/draft-amjad-cfrg-partially-blind-rsa-01). This design ensures that the proxies cannot link the traffic that they're handling to a user's account, neither the one operated by Google nor the one operated by the CDN.

**B. Limiting issuance of authentication tokens**

Since tokens are blinded to prevent linkability back to a client, it is theoretically possible to transfer tokens to different clients or use tokens to appear as multiple indistinguishable clients. To limit token transfer or harvesting, there will be a maximum quota of tokens issued per user per day and tokens will be relatively short-lived. Additionally, proxies will limit how much network traffic can be generated per token.

IP Protection will aim to provide most users with a sufficient number of tokens to proxy all their traffic to domains in the MDL. In practice, this means that users with average traffic patterns will get enough tokens to have their IP be masked every time, but users with unusually high activity or that show other indicators of fraud risk may experience limited access.

In the event a user has no tokens, the requests to domains in the MDL will be routed directly, without proxying. Token quotas may change over time in response to reported or observed patterns of fraudulent activity.

**C. Reporting of fraudulent behavior**

In addition to preventative measures, we will provide a way for websites to report DoS or other fraudulent behavior.

Defensibility is a constant need and an ever-changing landscape as threats evolve. We expect to remain vigilant and evolve our tactics as necessary to limit fraud and spam through the IP proxies. 

**D. Probabilistic Reveal Tokens**

As an additional measure to ensure businesses can monitor the amount of fraud on their systems, respond to emerging fraudulent behavior, and provide feedback on future design iterations, we will add a delayed mechanism to access a random sample of IP addresses called **Probabilistic Reveal Tokens (PRT)**. PRTs will be included on proxied requests in a new HTTP header added by Chrome for domains that enable PRTs. This PRT can, after a delay, be decrypted using a key issued by Google. The PRT will contain the non-proxy IP in a small percent of tokens issued. This PRT will be rotated for each new first-party load within a given incognito session. We will publish more detailed information on PRTs at a future time.

### Ecosystem Engagement
Over the years of developing this proposal in the Privacy Sandbox, we've actively sought and encouraged [ecosystem participation and feedback](https://github.com/GoogleChrome/ip-protection/issues). We want to ensure that the ecosystem has sufficient time to provide input and feedback before implementing IP Protection in Incognito. Questions on Github have and continue to act as a platform for collecting this feedback during the development phase. This feedback will be considered for refining our proposal and shaping the roadmap for IP Protection.

### IP Protection and VPNs
IP Protection and VPN services have distinct purposes and features. While a VPN encrypts all user traffic, IP Protection selectively routes third-party traffic through proxies to prevent online tracking. Unlike VPNs, which often allow users to choose a virtual location, IP Protection reflects the user's actual country and nearest metropolitan area. IP Protection is designed to complement active VPN services, ensuring that both can function simultaneously on the same device without interference.