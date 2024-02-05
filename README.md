# IP Protection (formerly known as Gnatcatcher)

## Introduction

As browser vendors make efforts to provide their users with additional privacy, a user’s IP address continues to make it feasible to associate their activities across origins when that otherwise wouldn’t be possible. This information can be combined over time to create a unique, persistent user profile and track a user’s activity across the web. This represents a threat to their privacy. Moreover, unlike with third-party cookies, there is no straightforward way for users to opt out of this kind of covert tracking.

However, in addition to being used as a possible tracking vector, IP addresses have been, and will continue to be, instrumental in routing traffic, preventing fraud and abuse, and performing other important functions for network operators and domains.

Therefore, any privacy solution for protecting IP addresses must account for both user privacy and the safety and functionality of the web. This proposal initially focuses on efforts where IP addresses are most likely to be used as a vector for tracking in a third-party context. 

IP Protection will evolve and broaden over time in conjunction with ecosystem changes to continue to protect users’ privacy from cross-site tracking.


## Cross-site tracking and the role of IP addresses
There are various definitions for “tracking” used in the web ecosystem. We will initially follow Mozilla’s definition for cross-site tracking, as it has served as a starting point for other browser policies.

Mozilla defines [tracking](https://wiki.mozilla.org/Security/Anti_tracking_policy#Tracking_Definition) as “...the collection of data regarding a particular user's activity across multiple websites or applications (i.e., first parties) that aren’t owned by the data collector, and the retention, use, or sharing of data derived from that activity with parties other than the first party on which it was collected.”

Browsers are moving against cross-site tracking. For Chrome, this move [includes phasing out third-party cookies and limiting fingerprinting](https://privacysandbox.com/open-web/), while ensuring the web stays healthy and vibrant. One way to limit fingerprinting is by limiting sources of identifiable information such as IP addresses.

An IP address is an effective cross-site identifier, as it is highly unique, relatively stable, and easy to collect. The use of IP addresses by websites and services are also not detectable by the browser. Therefore limiting access to IP addresses is important to prevent methods of cross-site tracking beyond third-party cookies.

With this in mind, Chrome will first focus on third parties identified as potentially using IP addresses for web-wide cross-site tracking. We’ll explore using methods similar to other browsers, including using lists that identify these third parties.

Chrome’s evolved focus on cross-site tracking has emerged from feedback received on the Gnatcatcher proposal. Chrome wants to focus on behaviors that are most likely to be using IP addresses for tracking users across sites in ways that might not align with user expectations of privacy. Chrome will work with the ecosystem to help preserve privacy while not bwhile still maintaining anti abuse use cases.



## Proposal
Chrome is reintroducing a proposal to protect users against cross-site tracking via IP addresses. This proposal involves the use of a privacy proxy that masks IP addresses for qualifying traffic as described previously.

**Goals**

- To improve user privacy by protecting users’ IP addresses from being used as a tracking vector. 
- To minimize disruption to the normal operations of servers, including the use of IP addresses for anti-abuse by first party sites, until there are alternative mechanisms in place.

### Privacy Proxy

#### Core requirements

- The destination origin doesn’t see the client’s original IP address.
- Google can’t see the origin that clients interact with.
- No single proxy can see the origins that clients interact with and the clients' original IP address.
- IP addresses of the proxies cannot be used as stable identifiers.
- We are using a list-based approach and only domains on the list in a third-party context will be impacted. More information below.

To meet these requirements, this proposal prioritizes routing eligible third-party traffic through two proxies. To identify which third-party traffic goes through the proxies, IP Protection is using a list-based approach. Connections to origins that are on the list but are accessed in a first-party context will not be proxied through this service.  For example, if an analytics company is on the list of domains and a user navigates directly to the site, that site will still be able to observe the user’s IP address, instead of the proxied IP address. However, if that domain on the list loads in a third-party context, the connection will be proxied and the user's IP address will not be visible to the site.

For proxied third-party traffic, the DNS will be resolved at the second proxy.

Chrome is taking the following steps to prevent user identifiers, including IP address, from being linked to origin-bound traffic:

- IP Protection will use CONNECT and CONNECT-UDP (MASQUE) to forward traffic. There is an end-to-end encrypted tunnel via TLS or QUIC from Chrome to the destination server. Separate connections will use different IP addresses from the proxies.

- We are using two proxies for improved privacy. A second proxy (proxyB) will be run by an external CDN, while Google runs the first proxy (proxyA). This ensures that neither proxy can see both the client IP address and the destination. This means that the Google proxy cannot link clients to the destination origins they’re visiting. CONNECT and CONNECT-UDP support chaining of proxies. Connections through the proxies are encrypted multiple times to prevent Google from being able to access browsing data. In particular, the connection client-website is end-to-end encrypted, and so are the client-proxyA and client-proxyB connections. Because of this, the proxyA (operated by Google) will only be able to see the client IP address but won't be able to know which website is visited. The proxyB (operated by an external CDN) will be able to see the hostname of the website, but it won't know which client IP is accessing it. Neither proxy can see the URL nor the data due to the end-to-end encryption. 

- Chrome will employ an [RSA blind signature scheme](https://datatracker.ietf.org/doc/draft-hendrickson-privacypass-public-metadata/) between client authentication and proxy usage to isolate client identity from all proxy servers. This will include strict and open source bounds on what metadata is shared with proxies during this authentication process. Chrome is committed to ensuring a user cannot be uniquely identified via their unblinded token or associated authentication metadata.

#### Anti-abuse

There are several anti-abuse concerns for proxied third party traffic:

- defensibility of the proxy, a compromised proxy may be used to deploy attacks 
- disruption of existing DoS defenses
- disruption of existing defenses for fraud and invalid traffic detection

To limit abuse of the proxy, we are considering the following non-exhaustive set of anti-abuse protections:

- user authenticates to the proxy
  - this will require a user account
  - auth tokens will be issued and redeemed at the proxy
- proxy shouldn’t be able to correlate traffic to user account
  - blinded signatures will be used
- limit abuse with harvesting of auth tokens
  - rate limit tokens per account
  - token expiry

In addition to preventative measures, we are also looking for opportunities to allow websites to report DoS and other abuse. Additionally, we are actively exploring new anti-abuse defenses to enable third party services to prevent abuse and fraud.

#### IP geolocation

IP-based geolocation is used by a swath of services within proxied third-party traffic, to comply with local laws and regulations and serve content that is relevant to users. Use cases include content localization, local cache assignment, and geo-targeting for ads. To support these needs but with privacy controls in place, the proxy will assign IP addresses that represent the user’s coarse location, including country. For more information, read the [IP Geolocation Explainer.](https://github.com/GoogleChrome/ip-protection/blob/master/Explainer-IP-Geolocation.md)

### Availability 
Chrome will initially launch IP Protection as an user opt-in setting for users in specific regions, understanding that this could be a significant change for how some companies rely on IP addresses, and seeking to minimize disruption as the ecosystem adjusts. We plan to initially deploy IP Protection to Chrome on Android and Desktop platforms. It will be possible for users and enterprise-managed versions of Chrome to disable IP Protection.

### Longer term
Long term solutions will evolve and will be shaped in conjunction with the ecosystem.
We will collaborate with ISPs, CDNs, third parties, and destination sites towards the end-state of privacy proxies for the web. For instance, ISPs and CDNs are well suited to operate privacy proxies.

As IP Protection evolves, we believe policy will have a part in the overall solution to address circumvention by websites. As we move forward with the Privacy Sandbox project, we'll develop the policy and seek input from the ecosystem. Our intent for a policy within the proposal will be to encourage web services to be accountable for the usage and sharing of client IP addresses, given the sensitivity of IP as an identifying data point. By creating transparency around the use of IP addresses, we hope to promote industry accountability for how IP addresses are accessed and used in the web ecosystem.

We welcome feedback on this proposal, especially with regard to some of the open questions we are considering.

