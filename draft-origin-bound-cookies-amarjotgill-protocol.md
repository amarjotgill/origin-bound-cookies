---
title: "Origin-Bound Cookies"
abbrev: "Origin-Bound Cookies"
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

 COOKIES:
    title: Cookies HTTP State Management Mechanism
    date: 14 May 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-httpbis-layered-cookies/


informative:

--- abstract

This draft introduces Origin-Bound Cookies which updates {{COOKIES}} aiming to make cookies more secure by default through binding cookies by port and scheme.

--- middle

# Introduction

Cookies are one of the few components of the web platform that are not scoped to the origin by default. This difference in scoping means that cookies have weaken confidentiality and integrity compared with other storage APIs on the web platform.

## Examples

  1. https://somesite.com sets a simple cookie, secret=123456, which contains private information about a user. Information that an attacker wishes to learn. To do so the attacker man-in-the-middles the user, and then tricks them into visiting http://somesite.com (note the insecure scheme). When the user visits that page their browser will send the secret cookie and the attacker can see it.

  2. Similarly, if the attacker has somehow compromised a service running on a different port on the same server, let's say port 345, as https://somesite.com then they could trick the user into visiting https://somesite.com:345, the user's browser will send the secret cookie, and once again the attacker can see it.
  Even more, through the same techniques, an attacker can also modify a user's cookies, sending a Set-Cookie field instead of simply eavesdropping.

All of these examples are possible because cookies by default do not care about the scheme or port of their connection. As long as the host matches the cookie will be accessible.

This document proposes changes to {{COOKIES}} by bounding cookies to port and scheme.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Server Requirements

## Origin-Bound Behavior

### Port Bound

First, alter Section 5.1.2 of {{COOKIES}} by adding source-port to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Then add the concept of port matching which helps to simplify checking if a cookie would match a port value We can do that by adding a new section under Section 5.3 of {{COOKIES}}

    5.3.5 Port matching
    
    An integer port-matches a given cookie if any of the following
    conditions are true:
    
    1. The cookie's host-only-flag is false.
    
    2. The integer exactly matches the cookie's port value.

Example:

The new behavior will behave as the following.
A cookie set by origin https://example.com will only ever be sent to https://example.com(:443). It will never be sent to a different port value such as https://example.com:8443.


### Scheme Bound
TODO

### Shadowing Domain Cookies

TODO

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
