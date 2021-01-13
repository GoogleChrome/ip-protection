# Gnatcatcher <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Cuban_Gnatcatcher._Polioptila_lembeyei_._Endemic_-_Flickr_-_gailhampshire.jpg/320px-Cuban_Gnatcatcher._Polioptila_lembeyei_._Endemic_-_Flickr_-_gailhampshire.jpg" height="50px">


**Global Network Address Translation Combined with Audited and Trusted CDN or HTTP-Proxy Eliminating Reidentification**

## Background

We have previously proposed two separate solutions for IP Blindness: Near-Path NAT and Willful IP Blindness


### Near-Path NAT

Explained [here](near_path_nat.md). 

The fundamental benefit of this proposal is that no server side changes are necessary to guarantee universal IP address privacy.  This makes roll-out easy for sites and users.  As the explainer describes, the client IP address and port tuple visible to first-party servers is constant thus facilitating pre-existing anti-abuse mechanisms.  Abuse prevention for third-party resources requires changes and additional complexity however, for example shifting from reliance on client IP addresses as conveying trust to other trust-conveyance mechanisms like [Trust Tokens](https://github.com/WICG/trust-token-api).


### Willful IP Blindness

Explained [here](willful_ip_blindness.md)

This proposal facilitates use of IP addresses by anti-abuse layers while preventing reidentification of users by their IP address in application serving layers. Audits are used to enforce the division between the two layers.  These audits are challenging and complicated to complete and can require sites to rearchitect to ensure the division between the two layers is defined, enforced and visible to auditors.  Roll-out is hence more cumbersome.


## Proposal: Why not both? <img src="https://i.imgur.com/c7NJRa2.gif" height="50px">

We can get the strength of both by using the Near-Path NAT (or some other proxying solution) for any connection that isn’t participating in Willful IP Blindness. This means at a baseline users' IP addresses will be hidden, but that sites can do the extra work to attest that they aren’t misusing IP addresses if they would like to have direct connections.

Put another way, Near-Path NAT can fill the role of the blinding reverse proxy described in Willful IP Blindness for the majority of the web. There does exist a small subset of web service providers who need additional control (mainly 3rd party service providers for whom the stable first party IP address mappings are not a sufficient anti-abuse mechanism) who can choose to go through an audit process such that the client can make a direct connection assured that the IP address won’t be used for tracking purposes.
