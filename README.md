# IP Protection (formerly known as Gnatcatcher)

## Introduction

As browser vendors make efforts to provide their users with additional privacy, the user’s IP address continues to make it feasible to associate users’ activities across origins that otherwise wouldn’t be possible. This information can be combined over time to create a unique, persistent user profile and track a user’s activity across the web, which represents a threat to their privacy. Moreover, unlike with third-party cookies, there is no straightforward way for users to opt out of this kind of covert tracking. 

In addition to being used as a possible tracking vector, IP addresses have been and will continue to be instrumental in routing traffic, preventing fraud and abuse, and performing other important functions for network operators and domains. 

Therefore, any IP address privacy solution must account for both user privacy and the safety and functionality of the web. This proposal initially focuses on efforts where IP addresses are most likely to be used as a vector for tracking in a third-party context. 

IP Protection will evolve and broaden over time in conjunction with ecosystem changes to continue to protect users’ privacy from cross-site tracking.


## Cross-site tracking and the role of IP addresses
There are various definitions for “tracking” used in the web ecosystem. We will initially use Mozilla’s definition for cross-site tracking as  it has served as an inspiration for other browser policies. 

Mozilla defines [tracking](https://wiki.mozilla.org/Security/Anti_tracking_policy#Tracking_Definition) as “...the collection of data regarding a particular user's activity across multiple websites or applications (i.e., first parties) that aren’t owned by the data collector, and the retention, use, or sharing of data derived from that activity with parties other than the first party on which it was collected.”

Browsers are moving against cross-site tracking. For Chrome, this [includes phasing out third-party cookies and limiting fingerprinting](https://privacysandbox.com/open-web/), while ensuring the web stays healthy and vibrant. One way to limit fingerprinting is by limiting sources of identifiable information such as IP addresses.

An IP address is an effective cross-site identifier as it is highly unique, relatively stable, cheap to collect and the applications of IP addresses by websites are not detectable by the browser. Therefore limiting access to IP addresses is important to prevent other methods of cross-site tracking beyond third-party cookies.

Based on how impactful IP addresses are for tracking, it would make sense to first focus on third parties identified as potentially using IP addresses for web-wide cross-site tracking. We’ll explore leveraging methods similar to other browsers and existing lists that identify these third parties. 

Chrome’s evolved focus on third-party tracking has emerged from feedback received on the Gnatcatcher proposal. Chrome wants to focus on behaviors that are most likely to be using IP for tracking users across sites in ways that might not align with user expectations of privacy. Chrome will work with the ecosystem to help preserve privacy while not breaking key uses on the web.



## Proposal
Chrome is reintroducing a proposal to protect users against cross-site tracking via IP addresses. This proposal is a privacy proxy that anonymizes IP addresses for qualifying traffic as described above.

**Goals**

- To improve user privacy by protecting users’ IP addresses from being used as a tracking vector. 
- To minimize disruption to the normal operations of servers, including the use of IP addresses for anti-abuse by first party sites, until there are alternative mechanisms in place.

### Privacy Proxy

#### Core requirements

- the destination origin doesn’t see the client’s original IP address
- the proxy and network intermediaries are not privy to the contents of the traffic.

To meet these requirements, this proposal prioritizes proxying eligible third-party traffic through the Privacy Proxy. 

This will use CONNECT and CONNECT-UDP (with MASQUE), to forward traffic. There is an end-to-end encrypted tunnel via TLS, from Chrome to the destination server.

We are considering using 2 hops for improved privacy. A second proxy would be run by an external CDN, while Google runs the first hop. This ensures that neither proxy can see both the client IP address and the destination. CONNECT & CONNECT-UDP support chaining of proxies.

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

#### GeoIP

IP-based geolocation is used by a swath of services within proxied third party traffic to comply with local laws & regulation and serve content that is relevant to users, such as: content localization (e.g. language), local cache assignment, and geo targeting for ads.
To support these needs, the Privacy Proxy will assign IP addresses that represent the user’s coarse location, including country.

### Longer term
Long term solutions will evolve and will be shaped in conjunction with the ecosystem.
We will collaborate with ISPs, CDNs, third parties, and destination sites towards the end-state of privacy proxies for the web. For instance, ISPs and CDNs are well suited to operate privacy proxies.

As IP Protection evolves, we believe policy will have a part in the overall solution to address circumvention by websites. When needed, we'll develop the policy and seek input from the ecosystem. Our intent for a policy within the proposal will be to encourage web services to be accountable for the usage and sharing of client IP addresses given the sensitivity of IP as an identifying data point. By creating  transparency around the use of IP addresses, we hope to promote industry accountability regarding how IP addresses are accessed and used in the web ecosystem.

We welcome feedback on this proposal, especially with regard to some of the [open questions](https://github.com/spanicker/ip-blindness/issues?q=is%3Aissue+is%3Aopen+label%3A%22open+question%22) we are considering.

