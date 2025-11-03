Some Privacy Sandbox technologies are being phased out. Please see our
[Update on Plans for Privacy Sandbox Technologies](https://privacysandbox.com/news/update-on-plans-for-privacy-sandbox-technologies/).

[Privacy Sandbox feature status](https://privacysandbox.google.com/overview/status)
provides more information about the status of individual APIs and platform features.

This repository will be archived and no longer updated.

# Self-Review Questionnaire: Security and Privacy
*Last updated: April 18, 2025*

**1\. What information does this feature expose, and for what purposes?**

It doesn’t expose any new information. Instead, it masks the source IP for eligible 3rd-party
traffic (i.e.,
[Masked Domain List](https://github.com/GoogleChrome/ip-protection/blob/main/Masked-Domain-List.md))
to improve user privacy.

**2\. Do features in your specification expose the minimum amount of information necessary to
implement the intended functionality?**

Yes

**03\. Do the features in your specification expose personal information, personally-identifiable
information (PII), or information derived from either?**

No

**04\. How do the features in your specification deal with sensitive information?**

N/A

**05\. Does data exposed by your specification carry related but distinct information that may not
be obvious to users?**

No

**06\. Do the features in your specification introduce state that persists across browsing
sessions?**

No

**07\. Do the features in your specification expose information about the underlying platform to
origins?**

No

**08\. Does this specification allow an origin to send data to the underlying platform?**

No

**09\. Do features in this specification enable access to device sensors?**

No

**10\. Do features in this specification enable new script execution/loading mechanisms?**

No

**11\. Do features in this specification allow an origin to access other devices?**

No

**12\. Do features in this specification allow an origin some measure of control over a user agent's
native UI?**

No

**13\. What temporary identifiers do the features in this specification create or expose to the
web?**

None

**14\. How does this specification distinguish between behavior in first-party and third-party
contexts?**

This feature only proxies [certain third-party traffic](https://github.com/GoogleChrome/ip-protection/blob/main/Masked-Domain-List.md)
(when in a first- or third-party context), if the user is in Incognito mode.


**15\. How do the features in this specification work in the context of a browser’s Private Browsing
or Incognito mode?**

The feature only works when in Incognito mode.

**16\. Does this specification have both "Security Considerations" and "Privacy Considerations"
sections?**

N/A

**17\. Do features in your specification enable origins to downgrade default security protections?**

No

**18\. What happens when a document that uses your feature is kept alive in BFCache (instead of
getting destroyed) after navigation, and potentially gets reused on future navigations back to the
document?**

Nothing special.

**19\. What happens when a document that uses your feature gets disconnected?**

Nothing special.

**20\. Does your spec define when and how new kinds of errors should be raised?**

No. There are no new relevant errors.

**21\. Does your feature allow sites to learn about the user's use of assistive technology?**

No

**22\. What should this questionnaire have asked?**

Did you discover https://github.com/w3c/security-questionnaire/blob/main/questionnaire.markdown
before or after manually copying question headers from https://w3c.github.io/security-questionnaire/?
