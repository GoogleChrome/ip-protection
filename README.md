## Willful IP Blindness

## Introduction

IP addresses by their nature provide a unique identifier for a client such that they can be found and routed to over the open internet. In the face of growing concerns about privacy, this feature of the protocol has turned into a bug in that if an IP address is stable over a period of time (which it often is), it can be used to identify users across first party websites (which runs afoul of Michael Kleber’s proposed [Privacy Model for the Web](https://github.com/michaelkleber/privacy-model)). Since it is a passive source of information, the [Privacy Budget](https://github.com/bslassey/privacy-budget) explainer describes that it must be considered a consumed source of identifying information for all sites and automatically deducted from the budget. This is particularly problematic for APIs which may be large sources of information in themselves (such as [FLoC](https://github.com/jkarlin/floc)) as the combination would exceed any reasonable budget. For websites that need more API access in the face of the Privacy Budget than what remains after IP address access is deducted, we need a way for them to opt out of being exposed to this information.

## Goals

The goal of this is to provide a mechanism for HTTP applications to blind themselves to IP address and other identifying network information such as connection pooling behavior and advertise that fact to clients such that they can change their behaviour accordingly.

Some applications will be able to live behind an IP-blinded endpoint and some will not.  A goal of this explainer is to start a conversation about what uses could live with IP-blinding — and for use cases that could _not_ live with it, what relaxation of blinding could meet those uses without allowing full IP-based tracking.

It is also a goal of this proposal to not interfere with the normal operations of servers, including the use of IP address as bot, DoS and SPAM detection.

## Non-goals

It is not a goal of this proposal to provide universal IP privacy. 

## Proposed Mechanism

Consider the high-level division of an HTTP serving stack into its serving infrastructure layer and its application layer.  The serving infrastructure has access to the IP address and other network identifiers, but not the application data while the application has access to the application data but not the network identifiers. In this way, the two sets of information cannot be combined to produce a durably identifying fingerprint. 

One could imagine making this cut at other layers in the typical path between a client and the http application such as at the [network operator](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.310.8986&rep=rep1&type=pdf), [edge cache](https://blog.cloudflare.com/1111-warp-better-vpn/) or [client](https://brave.com/vpn0-a-privacy-preserving-distributed-virtual-private-network/).

We propose introducing a signed attestation (perhaps in the form of an HTTP header) that advertises the fact that a server masks IP addresses and other identifying network information from the application layer of the services that it hosts. A CDN could offer this as a feature to the services they host such that the hosted service can access more APIs than would otherwise be available because their Privacy Budget hasn’t been exhausted on the IP address. Similarly, reverse proxies could provide the same service without the burden of hosting the services.

## Policy Enforcement Mechanisms

### Audit

Optionally, the certificate extension can include a signed assertion from an organization that asserts that the server operator is in compliance with the agreed upon policies and verifies that through some audit mechanism. Clients can choose to only trust the assertions of certificate extensions with signatures of organizations that they trust.

### Spot checking

A public CDN or reverse proxy could be spot checked periodically. Anyone can set up an instance on a public CDN and test to see if it reveals network identifiers. This could be done by an auditor as part of its normal checks or crowd-sourced from volunteers. CDNs advertising IP privacy and found to be exposing network identifiers can be block-listed such that their privatization assertion is ignored and the content they host have the IP address accounted for in their privacy budget.

## Open questions

### Geo-IP

IP is used by many services to give roughly geo-located content and to obey local laws and regulations. There are a couple options. There is already the[ Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API), but we probably don’t want to condition people on granting high precision geolocation to most sites. Presumably better, since this is the removal of previously passively available information, geo-ip-granularity location could be made available through a client hint, which also allows the sites consuming the information to be tracked, measured and potentially denied. Alternatively, if policy allows it, the privatizer could provide geo-IP information to the hosted services.  Providing country-level location information to the hosted service would allow services to conform to local laws while at the same time not providing significant identifying information.

### DoS and Fraud Prevention

Since IP is used as a signal for DoS and Fraud detection and prevention it is important to think through the implications of removing that signal for some use cases. To start, this proposal is opt-in, so services that rely on IP as a key signal do not need to opt into being blinded. Clearly some services will want to use APIs that reveal large amounts of entropy and also prevent DoS and Fraud though. By design under this proposal, IP is still available to part of the serving stack such that DoS protections can be implemented that still fully incorporate IP address. Similarly, IP signals will be available to the blinder to be incorporated into fraud prevention. Additionally, [Trust Tokens](https://github.com/WICG/trust-token-api) could be used to use trust derived from a first party notion of identity in a third party context.
