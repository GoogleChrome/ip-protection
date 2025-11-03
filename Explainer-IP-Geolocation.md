Some Privacy Sandbox technologies are being phased out. Please see our
[Update on Plans for Privacy Sandbox Technologies](https://privacysandbox.com/news/update-on-plans-for-privacy-sandbox-technologies/).

[Privacy Sandbox feature status](https://privacysandbox.google.com/overview/status)
provides more information about the status of individual APIs and platform features.

This repository will be archived and no longer updated.

# Explainer: IP Geolocation Approach for IP Protection

## Context and Goals

Location can be a key identifier for understanding what content is relevant for users. As Chrome prepares to [offer users an informed choice on third-party cookies](https://developers.google.com/privacy-sandbox/cookies/prepare/overview), the Chrome team wants to ensure that we are taking appropriate steps to improve privacy on the web while also maintaining key use cases. We have [shared our proposal](https://github.com/GoogleChrome/ip-protection) for masking IP addresses for certain third party domains on the web. This comes with additional implications, like how IP addresses are used to understand the location of [GitHub - GoogleChrome/ip-protection](https://github.com/GoogleChrome/ip-protection) web users. This document outlines our initial proposal for how IP Protection will implement IP geolocation mappings.

It’s worth noting that IP geolocation—with or without IP Protection—only provides approximate and coarse location information and that there are ways in which users can obfuscate this data. With this proposal, we aim to improve user privacy while maintaining most of the existing uses of IP geolocation as a coarse location signal. Addressing pre-existing accuracy issues of IP geolocation is not a goal and generally, this proposal will inherit the prior limitations of IP as a source of location information.

## How IP geolocation information will be shared

Geo assignments of the IP addresses exposed by IP Protection are shared publicly via a geofeed file. Our geofeed can be found [here](https://www.gstatic.com/ipprotection/geofeed). The geofeed uses the format defined in [[RFC 8805](https://datatracker.ietf.org/doc/html/rfc8805)], and provides city-level mappings. These city-level mappings correspond to top cities, each representing a geographic area around that city.

## Defining and Dividing Geographies

We have divided the entire geographic area where IP Protection might be available into areas with large enough populations of Internet users to ensure individual users remain anonymous. We aimed to define areas where we observe at least one million users over a two week period across Google properties, which we use as a proxy for the number of Internet users in that region. Note that this estimation can differ significantly from census population data or other sources due to a range of factors, for example the presence of temporary visitors or if a person uses multiple digital profiles or accounts. For example, in the US this leads to a subdivision of the country into ~700 geographic areas. Since the U.S. has approximately 330 million people, this would equate to roughly 470,000 people per geo on average.

Each geographic area is represented in the geofeed by its most populous city. We have aimed to preserve the most popular cities across the globe by Internet population, in order to maximize the utility of the IP geolocation data while improving user privacy. Note that, since we aim for the areas to have a minimum size in terms of Internet users, the areas will be geographically smaller in size in very densely populated areas and larger in sparse areas.

Note that the assigned IP geolocation for a user would maintain country borders to our best knowledge based on the user’s original IP. For example, a user that appears in Windsor (Canada) according to their original IP address would not be assigned to Detroit (US), despite the geographical proximity. This rule applies for all countries, including those that may have a total population below the established threshold.

!["The map is subdivided into areas delimited by the blue boundaries in the US and green boundaries in Canada. Users within a certain area will be assigned an IP address that is mapped to the top city of that area, marked with the pin. An area will never cross a country border."](./images/geo-example.png)

_Image 1. Illustrative mapping of Detroit area. The image shows how country boundaries with Canada are preserved while also indicating unique Geos for certain regions._



## Country, State, and Sub-Country Mapping

As mentioned earlier, IP geolocation can be useful as a coarse location signal but it is not exactly precise and there have always been mechanisms for users to obfuscate this data. As a result, IP geolocation can’t be used to guarantee country, state or other sub-country divisions with 100% accuracy. This limitation is also true of IP Protection, and it remains the responsibility of companies to ensure they are meeting any applicable regulations or other obligations in each jurisdiction.

Having said that, we have designed our geo mappings to provide best effort accuracy in terms of country and region mappings, while preserving privacy by ensuring sufficiently large geographic areas at the sub-country level.

As illustrated in the chart above, our geo mappings would preserve country borders to our best knowledge based on users’ original IP addresses. In addition to the country, users will be assigned to an IP that represents the top city in their geographic area, which can then be mapped to a state in the US or relevant region in other countries. These sub-country mappings would also generally hold, although some inaccuracies should be expected, particularly near the regional borders.

## Assigning Users to Geos

Users will be assigned to a geographic area based on their pre-proxy IP address representing their current location. At any time, a geo area will only be used if we can ensure sufficient anonymity for the users who are assigned to it. That is, the IPs representing that geo are being used by a sufficiently large number of users to prevent IP-based tracking. If this condition isn’t met, the user will be defaulted to a less granular geo. We don’t expect this to occur frequently, except potentially in initial rollout phases.

## Preserving Privacy and Utility

We have designed our IP geolocation approach to balance privacy expectations while trying to preserve the maximum utility for the various uses of IP geolocation, such as ads personalization, analytics or compliance.

In order to ensure a highly privacy-preserving approach, we propose a threshold of one million unique web cookies (over a two week period) to determine the geographic areas that users will be mapped to. We estimate that, at this size, the regions are big enough to preserve privacy such that individual users can’t be tracked or identified based on the IPs that are being assigned to their requests when using the IP Protection system. We’ve also taken into account location privacy considerations, such that the precision revealed by IP geolocation aligns with users’ expectations, even in densely populated areas.


## Have feedback?

We welcome your feedback on this proposal. Please use the following links to provide your input in our GitHub repository:
* [Impacts of proposed IP geolocation granularity](https://github.com/GoogleChrome/ip-protection/issues/3)
* [Impacts of proposed IP geolocation granularity for regulatory & contractual use cases](https://github.com/GoogleChrome/ip-protection/issues/2)

