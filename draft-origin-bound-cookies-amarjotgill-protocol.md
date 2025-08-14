---
title: "Origin-Bound Cookies"
abbrev: "OBC"
category: info

docname: draft-origin-bound-cookies-amarjotgill-protocol-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "amarjotgill/origin-bound-cookies"
  latest: "https://amarjotgill.github.io/origin-bound-cookies/draft-origin-bound-cookies-amarjotgill-protocol.html"

author:
 -
    fullname: "Amarjot Gill"
    organization: Google LLC
    email: "amarjotgill@google.com"

normative:

informative:

...

--- abstract

This draft introduces Origin-Bound Cookies which aims to make cookies more secure by default through binding cookies by port and scheme.

--- middle

# Introduction

Cookies are one of the few components of the web platform that are not scoped to the origin by default. This difference in scoping means that cookies have weaken confidentiality and integrity compared with other storage APIs on the web platform.

## Examples

https://somesite.com sets a simple cookie, secret=123456, which contains private information about a user. Information that an attacker wishes to learn. To do so the attacker man-in-the-middles the user, and then tricks them into visiting http://somesite.com (note the insecure scheme). When the user visits that page their browser will send the secret cookie and the attacker can see it.

Similarly, if the attacker has somehow compromised a service running on a different port on the same server, let's say port 345, as https://somesite.com then they could trick the user into visiting https://somesite.com:345, the user's browser will send the secret cookie, and once again the attacker can see it.
Even more, through the same techniques, an attacker can also modify a user's cookies, sending a Set-Cookie field instead of simply eavesdropping.

All of these examples are possible because cookies by default do not care about the scheme or port of their connection. As long as the host matches the cookie will be accessible.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
