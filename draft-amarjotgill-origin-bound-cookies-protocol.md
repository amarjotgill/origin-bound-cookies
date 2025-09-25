---
title: "Origin-Bound Cookies"
abbrev: "Origin-Bound Cookies"
category: info

docname: draft-amarjotgill-origin-bound-cookies-protocol-latest
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
  latest: "https://amarjotgill.github.io/origin-bound-cookies/draft-amarjotgill-origin-bound-cookies-protocol.html"

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
 Cookies-Lack-Integrity:
    title: Cookies Lack Integrity Real-World Implications
    date: 12 August 2015
    target: https://www.microsoft.com/en-us/research/wp-content/uploads/2017/01/sec15_cookies-lack-integrity-published.pdf

--- abstract

This draft introduces Origin-Bound Cookies which updates {{COOKIES}} aiming to make cookies more secure by default through binding cookies by port and scheme.

--- middle

# Introduction

Cookies are one of the few components of the web platform that are not scoped to the origin by default. This difference in scoping means that cookies have [weak confidentiality](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-weak-confidentiality) compared with other storage APIs on the web platform.

## Examples

  1. https://somesite.com sets a simple cookie, secret=123456, which contains private information about a user. Information that an attacker wishes to learn. To do so the attacker man-in-the-middles the user, and then tricks them into visiting http://somesite.com (note the insecure scheme). When the user visits that page their browser will send the secret cookie and the attacker can see it.

  2. Similarly, if the attacker has somehow compromised a service running on a different port on the same server, let's say port 345, as https://somesite.com then they could trick the user into visiting https://somesite.com:345, the user's browser will send the secret cookie, and once again the attacker can see it.
  Even more, through the same techniques, an attacker can also modify a user's cookies, sending a Set-Cookie field instead of simply eavesdropping.

All of these examples are possible because cookies by default do not care about the scheme or port of their connection. As long as the host matches the cookie will be accessible.

This document proposes changes to {{COOKIES}} by binding cookies to port and scheme which will prevent against these attacks and greatly improve cookie security.



# Conventions and Definitions

## Terminology

An aliasing cookie refers to a cookie which shares the same cookie-name, cookie-value and sub domain with another cookie but differs via scheme or port.

A domain cookie is a cookie which has the Domain= parameter set in its field.

An origin cookie is a set of cookies which are set on each origin, an origin being the same port and scheme value.


{::boilerplate bcp14-tagged}

# User Agent Requirements

## Origin-Bound Behavior

### Port Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding port to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the port attribute.

{:quote}
> A cookie's port is either null or a 16-bit unsigned integer.  It is initially null.

For port matching algorithms below will be updated to compare integers to ensure port values match.
Pre-existing cookies with unspecified "port" will have a null value. This value will cause the cookie to be treated with legacy behavior.

Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com(:443). It will never be sent to a different port value such as https://example.com:8443.


### Scheme Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding scheme to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the scheme attribute.

{:quote}
> A cookie's scheme is null or a byte sequence.  It is initially null.

To simplify cookie-to-scheme matching, a new scheme matching concept will be introduced. This is necessary because initial WebSocket connections, in addition to HTTP and HTTPS,
can also set and send cookies. To maintain WebSocket compatibility, scheme comparisons will look for "compatible schemes" rather than exact string matches. This will be achieved
by adding a new section under [Section 5.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-subcomponent-algorithms).

 A string A Scheme-Matches a string B if one of the following is true:

1.  A equals B
2.  A and B are both in [“http”, “ws”]
3.  A and B are both in [“https”, “wss”]

Pre-existing cookies with unspecified "scheme" will have a null value. This value will cause the cookie to be treated with legacy behavior.

Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com. It will never be sent to a different scheme value such as http://example.com.

## Storage
We update the storage model in [Section 5.4.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-store-a-cookie) to ensure that port and scheme are being considered when comparing with existing cookies.

Step number 18 of this section will need to be altered to:

{:quote}
> If the user agent's cookie store contains a cookie oldCookie whose name is cookie's name, host is host-equal to cookie's host, host-only is cookie's host-only, path is path-equal to cookie's path, and cookie's scheme is equal to oldCookie's scheme.

A new substep should also be added to step number 18

{:quote}
> 3. If cookie's host-only is set to true and cookie’s port does not equal oldCookie’s port then skip the remaining sub-steps.

Note the addition of checking port and scheme, this will prevent a cookie with differing port or scheme values from overwriting the oldCookie, instead this cookie would be stored as a separate cookie and the oldCookie will not be deleted.
Also note if the domain attribute is set then we will ignore checking via port so it will overwite the oldCookie.

## Retrieve Cookies

The Retrieve Cookies algorithm will need to be updated in [Section 5.4.5 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-retrieve-cookies).

Retrieve Cookies will now take a port and scheme value.

The following will need to be added to step 2:

1.  port `port-matches` cookie's port, if the cookie’s domain attribute is set or port value is null then we can ignore this step.
2.  scheme `scheme-matches` cookie's scheme, if the cookie’s scheme value is null then we can ignore this step.

This will ensure that a cookie is only retrieved if the request origin is equal to the cookie's origin, while minimizing disruption to users.
Allowing current sites to continue working as-is, as old cookies are replaced with newer ones, the new cookies will be origin-bound.

## Cookie Store Eviction
 The last algorithm that will need to be updated is the Cookie Store Eviction algorithm outlined [Section 5.2.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-remove-excess-cookies-for-a).

Step 2 will need to be updated to the following:

{:quote}
> Sort all insecureCookies by last access time (earliest to latest), prioritizing domain cookies first, followed by origin cookies.

Step 4 will also need to be updated to the following:

{:quote}
> Sort all secureCookies by last access time (earliest to latest), prioritizing domain cookies first, followed by origin cookies.

Updating these steps will ensure that domain cookies for each origin are deleted before any other cookie.


## Requirements Specific to Non-Browser User Agents

### The Cookie Header Field
The cookie header field outlined in [Section 5.5.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-the-cookie-header-field) will need to be altered.

It will be altered to the following:

{:quote}
>1. Let isSecure be a boolean indicating whether request's URL's scheme is deemed secure, in an implementation-defined manner.
>2. Let host be request's host.
>3. Let path be request's URL's path.
>4. Let httpOnlyAllowed be true.
>5. Let sameSite be a string whose value is implementation-defined, but has to be one of "strict-or-less", "lax-or-less", "unset-or-less", or "none".
>6. let port be request's URL's port,this can be unspecified.
>7. let scheme be request's URL's scheme, this can be unspecified.
>8. Let cookies be the result of running Retrieve Cookies given isSecure, host, path, httpOnlyAllowed, sameSite, port, and scheme.
>9. Return the result of running Serialize Cookies given cookies.

Note the major change here is extracting the port and scheme from the request and running the new version of Retrieve Cookies based on that.

# Security Considerations

Origin-Bound Cookies is considered net positive for security, the following would be mitigated by it.

Weak integrity: This occurs when an insecure site, controlled by a network attacker, sets a malicious cookie that is then sent to the secure version of that site. OBC binds
cookies to their setting origin, preventing such malicious cookies from being accessed by a different origin.

Weak confidentiality: This refers to an attacker reading sensitive user data set by a secure site, which is unintentionally sent to an insecure site. OBC ensures that cookies are
only accessible by the same origin that set them, preventing this type of data leakage.


One security concern is how to handle clients that do not support OBC, this  is handled by the following, when a cookie has a valid Domain attribute specified (hereby called a
“domain cookie”) that cookie has relaxed bindings. Namely, the cookie may be sent to:
 1. any host which domain matches the Domain value (This is unchanged from legacy behavior)
 2. any port value

Importantly a domain cookie is still bound to the scheme of its setting origin.

This behavior allows developers to opt-out of the stronger protections of an origin cookie which can help with compatibility for usages that need a particular cookie available
across hosts and/or ports.


## Shadowing Cookies

A cookie that includes a Domain attribute is called a "domain cookie" and has more relaxed restrictions. Specifically, it can be sent to:

Any host that matches the specified domain.

Any port on those hosts.

Importantly, the cookie remains bound to the scheme (e.g., https) of the site that set it. This behavior allows developers to opt-out of the stricter default protections, which is useful when a cookie needs to be shared across multiple subdomains or ports.

The "Shadowing" Security Issue as described in {{Cookies-Lack-Integrity}}
This flexibility creates a potential vulnerability where a secure "origin cookie" (one without a Domain attribute) could be overridden or "shadowed" by a less-secure domain cookie.
Removal for Secure could be a possible future consideration for the HTTP WG.


Here's an example:

A user visits https://trusted.example.com, which sets a secure cookie:

Set-Cookie: trustedValue=1234

Later, the user visits https://evil.example.com, which sets a conflicting domain cookie:

Set-Cookie: trustedValue=evil1234; Domain=example.com

The next time the user visits https://trusted.example.com, the browser will send both cookies. The server might incorrectly process the malicious evil1234 cookie, leading to potential security problems.

The Solution:

To preserve the security benefits of origin cookies, this design disallows a domain cookie from shadowing an origin cookie. This ensures that the more specific and secure cookie cannot be overridden by a less secure one.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
