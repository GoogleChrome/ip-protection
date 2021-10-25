# Proposed Willful IP Blindness (WIB) Principles

## Introduction

<p>For motivation, goals, non-goals, and general description of the proposed mechanism of Willful IP Blindness (WIB), please refer to <a href="willful_ip_blindness.md">the explainer</a>.
</p>
<p>This document proposes a set of delineations between conforming and nonconforming IP address uses, such that website operators and other interested parties can make consistent assessments and representations about a serving infrastructure’s conformance to WIB. 
</p>
<p>This document is intended to promote discussion in the ecosystem given the engineering lift required to adapt to the proposed WIB principles. Ideal principles would offer meaningful privacy guarantees to users without creating unnecessary burdens or breakage for website operators. Additionally, while we have suggested <a href="willful_ip_blindness.md#policy-enforcement-mechanisms">potential enforcement mechanisms</a> in the explainer, we welcome thoughts on how the industry might adopt these WIB principles.
</p>

## Scope

Willful IP Blindness (WIB) is one of two systems for IP address privacy proposed in <a href="https://github.com/bslassey/ip-blindness">Gnatcatcher</a>. While <a href="near_path_nat.md">Near-path NAT</a> is Chrome’s proposed technical approach for IP address privacy, we recognize that there are certain use cases where a broad technical approach is too limiting. We propose this principle-based approach to maintain the use of IP addresses for these operationally necessary uses.  We expect that only a small subset of web service providers who need additional control (mainly third-party service providers for whom the stable first-party IP address mappings are not a sufficient anti-abuse mechanism) will utilize WIB, and thus choose to adhere to the principles described below.  The vast majority of web service providers will instead utilize <a href="near_path_nat.md">Near-path NAT</a> for IP address privacy.

## Overview

The simplest way to demonstrate a website is willfully blinding itself to IP addresses is to demonstrate that IP addresses are not stored in ephemeral or persistent storage; however, it’s impossible to successfully operate even the simplest website in this manner.  This document starts by delineating some IP address uses that are considered conforming and do not represent a breach of WIB principles.  In general, these use cases can be compared against <a href="https://github.com/michaelkleber/privacy-model">a Privacy Model for the Web</a> when evaluating their intentions.

## Nonconforming Uses of IP Addresses

This document proposes limiting the use of IP addresses as global static identifiers to only the  purposes of routing traffic, regulatory and legal conformance, preventing abuse, and rare issue investigation as defined by the <a href="proposed_willful_ip_blindness_principles.md#conforming-uses-of-ip-addresses">Conforming Uses of IP Addresses section</a>.  For example, using a client’s IP address as a global static identifier to perform web-wide tracking for purposes other than abuse prevention is nonconforming.

## Conforming Uses of IP Addresses

### Conforming use #1: Routing traffic

IP addresses are necessary to route traffic across the internet, in particular from the client to the appropriate server and back to the client.  We can further divide this use into two subcategories:

#### Routing response traffic back to the client

A serving stack must keep track of ongoing TCP, TLS, and HTTP connections to clients so responses can be sent back to clients.  This requires keeping track of client IP addresses in the serving stack layers responsible for TCP, TLS, and HTTP connection termination.  Other layers (e.g. application layer) of the serving stack should not use IP addresses to keep track of client connections or sessions, and instead use an uncorrelated identifier.  Browsers are responsible for partitioning sockets and connections per site, so these should represent uncorrelated identifiers.

#### Routing client traffic to the appropriate server

Selecting which particular server should respond to a user’s request may require the user’s IP address.  This may be required for legal reasons, peering configuration and/or agreements, or performance considerations (e.g. load balancing or latency reduction).  Additional verification may be required to ensure that information derived from the IP address isn’t used for cross-site user identification purposes.

### Conforming use #2: Regulatory and legal compliance

Ensuring a website conforms to local regulatory and legal requirements means the website needs to know which country, and in some cases state (e.g. CCPA), client traffic originates from.  Generally, the country and state can be derived from the client’s IP address.  To facilitate this regulatory and legal conformance, WIB allows for inferring geolocation from IP addresses down to country level and sub-country <a href="https://en.wikipedia.org/wiki/Administrative_division">administrative divisions</a> (e.g. states within countries) of at least 500,000 people (500,000 people covers all US states).  Use of more accurate geolocation information from IP addresses is subject to the <a href="proposed_willful_ip_blindness_principles.md#additional-uses-of-ip-addresses">Additional uses of IP addresses section</a> below.  <b>General use of country level geolocation information from IP addresses conforms to WIB principles</b> due to the many far reaching implications of country-level regulations.

### Conforming use #3: Abuse prevention

Successfully operating a website requires protecting it from abuse, and in many cases IP address is the only way to differentiate abusive traffic and allow it to be blocked or ignored.  The following are conforming uses of IP addresses for abuse prevention:

#### Denial of Service (DoS) prevention

Unfortunately, DoS attacks are commonplace on the internet, and hence protecting websites from DoS attacks is a fundamental need of websites.  Prompt response to DoS attacks is critical, so frequently DoS prevention is implemented in levels of the serving stack that are close to the client (e.g. at the CDN level) to prevent allowing DoS attacks to consume more costly resources deeper in the serving stack.  WIB was developed to facilitate using IP addresses at the CDN level for DoS prevention while providing a delineation whereby IP addresses need not be propagated to the application layer of the serving stack.

#### Botnet, SPAM, and invalid traffic detection and attack prevention

IP addresses are critical to detect, identify, block, and prevent infected devices, botnets, command and control servers, account hijacking, cookie theft, spam, and invalid traffic which would otherwise erode the web’s functionality and ecosystem’s success.  IP addresses can also be used to facilitate account recovery.

<p><b>Note:</b> While IP addresses remain a critical piece of abuse prevention today, their use still poses a potential privacy risk to users.  Anti-abuse mechanism implementers need to plan for long term migration to privacy preserving mechanisms like <a href="https://github.com/WICG/trust-token-api">Trust Tokens</a> or <a href="proposed_willful_ip_blindness_principles.md#domain-partitioning-ip-addresses">domain partitioned IP addresses</a> to reduce this potential privacy risk.
</p>

### Conforming use #4: Rare issue investigation

In what should be extremely rare cases, it may be necessary to use IP addresses to investigate particular issues, such as debugging a functionality problem or a performance problem, or a problem that presents only for a particular IP address or tiny set of IP addresses (e.g. traffic internal to an organization).  Using IP addresses to identify problematic traffic should only be used as a last resort after all other traffic identification mechanisms are exhausted, in other words a “break glass” situation.  “Extremely rare” should be no more than 0.05% of unique site visitors.  Any access of IP addresses in this case must be logged and include sufficient justification.

## Additional Uses of IP Addresses 

Apart from the conforming uses listed above, a site may choose to access IP addresses at a finer-grained level (e.g., geographical advertisement targeting that is more specific than what is covered by <a href="proposed_willful_ip_blindness_principles.md#conforming-use-2-regulatory-and-legal-compliance">Conforming Use #2</a>).  In this case, the site must communicate the level of identifiability of these additional uses to the browser, which can adjust its behavior.

## Methods of Conformance

There are various ways that a website can ensure that it is using IP addresses only for the conforming uses listed above.  This section seeks to define common methods of conformance.  If IP addresses are not propagated to some or all parts of a serving stack, there is no reason for further analysis of those parts; other parts should be evaluated to determine that IP addresses are only used for <a href="proposed_willful_ip_blindness_principles.md#conforming-uses-of-ip-addresses">Conforming Use Cases</a> or <a href="proposed_willful_ip_blindness_principles.md#additional-uses-of-ip-addresses">Additional Uses of IP Addresses</a>.

### Not storing or propagating IP addresses

<p>Simply not storing or propagating IP addresses is the easiest way to ensure WIB conformance; however, as outlined in the <a href="proposed_willful_ip_blindness_principles.md#conforming-uses-of-ip-addresses">Conforming Uses of IP Addresses</a> section, there are common cases where IP addresses are necessary for basic website operation.  The hope of WIB is that these conforming uses can happen at the serving stack layer close to the client (e.g. CDN layer), and IP addresses can be blocked from propagating to the application layer.  This blocking can be as simple as not including <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded">the Forwarded header</a> on requests from the CDN layer to the application layer.
</p>
<p>This isn’t to say CDNs can do as they please with IP addresses; obviously cross-site identity joining with IP addresses is not conforming.  This is to say if a CDN conforms internally to WIB and does not propagate IP addresses to application backends they serve, then no knowledge of those application backends is necessary to confirm that the backends are WIB conformant.
</p>
<p>Blocking propagation of IP addresses is the simplest and easiest method of conformance, but propagation of IP addresses that are deterministically encrypted is feasible as per the <a href="proposed_willful_ip_blindness_principles.md#domain-partitioning-ip-addresses">Domain partitioning IP addresses</a> rules or when their later use is subject to <a href="proposed_willful_ip_blindness_principles.md#ip-address-access-controls">IP address access controls</a>.
</p>

### Separating identity from IP addresses

When evaluating storage of IP addresses to determine whether such storage could be used to facilitate cross-site identity joins using IP addresses, it’s important to remember these joins can only be exploited when the IP addresses are stored along with a particular site’s user identifier.  For example, storing IP addresses and the corresponding client’s favorite color doesn’t provide a particularly good method of tracking users across websites.  However, storing a client’s IP address and first-party site identifier does.  One way to conform to WIB is to make sure that if IP addresses are ever logged, they are not stored along with other identifying information.

### Domain partitioning IP addresses

The goal of WIB is to prevent cross-site tracking by using IP addresses as cross-site identity join keys.  Other than <a href="proposed_willful_ip_blindness_principles.md#not-storing-or-propagating-ip-addresses">blocking IP addresses from propagating to layers of the serving stack</a>, another alternative is to encrypt IP addresses along with the hostname of the served site before propagating it to other layers of the serving stack.  For example, if 1.2.3.4 browses to http://a.com which references http://b.com/image.jpg, when serving http://b.com/image.jpg the CDN layer could encrypt “a.com,1.2.3.4” and pass the resulting ciphertext to the application layer. The application layer wouldn't be able to use the ciphertext as a cross-site identity join key because it’s specific to a.com.  Note that the hostname of the site (a.com), not the hostname of the subresource (b.com), is used in the encryption because the same subresource can appear on multiple sites.  If deterministic encryption is used, the ciphertext could potentially be used for things like denial-of-service detection to detect repeated requests from the same IP address.  The encryption keys used must be protected with access controls similar to <a href="proposed_willful_ip_blindness_principles.md#ip-address-access-controls">those used for unencrypted IP addresses</a>.

#### Cross-browser-profile or cross-account reidentification must be limited to conforming uses

While domain partitioned IP addresses prevent cross-site re-identification, they do not protect against the ability to re-identify users on a single site across different browser profiles (e.g. when a user opens an incognito browsing window) or across different accounts (e.g. when a user signs out and signs in with a different account). Note that re-identification of users on a single site across different browser profiles or across different accounts must be limited to the Conforming Uses described above (e.g., abuse prevention).


### IP address access controls

Website operators must implement internal controls to ensure that out-of-band access to the IP address conforms to WIB principles.  In order to indicate this method of conformance, access to logs containing IP addresses must have proper access controls in place:
1. Any person with access to the log must acknowledge and agree to only using the log’s information for a conforming purpose.
2. Any computer program with access to the log must be reviewed to ensure its output/effects are limited to conforming purposes.
<p>ISO 27001 or SOC 2 conformance can help ensure appropriate access controls are in place.
</p>

## Additional Considerations

### Completeness

As per <a href="https://github.com/bslassey/ip-blindness">Gnatcatcher</a>, connections to fetch subresources on a web page that are not considered conforming to WIB principles will be routed through the <a href="near_path_nat.md">Near-Path NAT</a> (or some other proxying solution), thus allowing a web page to conform to WIB while all of its subresources may not.  However, for a web server to be considered conforming to WIB, all other servers with which the IP address is shared must also conform.  The server sharing the IP address should verify WIB conformance of other servers in a method akin to how a browser verifies server WIB conformance. The list of separately WIB conforming entities with which an IP address is shared should be made public. For example, <a href="https://www.iab.com/guidelines/real-time-bidding-rtb-project/">OpenRTB</a> is a protocol that can share IP addresses. For OpenRTB to conform to WIB principles, all downstream participants would also need to conform to WIB principles with respect to the IP addresses received via OpenRTB.
