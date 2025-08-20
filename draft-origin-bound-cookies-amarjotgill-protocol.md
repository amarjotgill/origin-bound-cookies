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

This document proposes changes to {{COOKIES}} by bounding cookies to port and scheme which will prevent against these attacks and greatly improve cookie security.



# Conventions and Definitions

## Terminology

An aliasing cookie refers to a cookie which shares the same cookie-name, cookie-value and sub domain with another cookie but differs via scheme or port.

{::boilerplate bcp14-tagged}

# User Agent Requirements

## Origin-Bound Behavior

### Port Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding source-port to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the port attribute.

{:quote}
> A cookie's port. It is initially "unset" and is an integer type.

Then add the concept of port matching which helps to simplify checking if a cookie would match a port value We can do that by adding a new section under [Section 5.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-subcomponent-algorithms).

An integer `port-matches` a given cookie if any of the following conditions are true:

1.  The cookie's `host-only-flag` is `false`.
2.  The integer is equal to the cookie's `port` value.

Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com(:443). It will never be sent to a different port value such as https://example.com:8443.


### Scheme Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding source-scheme to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the source-scheme attribute.

{:quote}
> A cookie's source-scheme. It is initially "unset" and is a string type.

Then add the concept of scheme matching which helps to simplify checking if a cookie would match a scheme value We can do that by adding a new section under [Section 5.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-subcomponent-algorithms).

Schemes are represented as strings, A and B, are considered `scheme-matches` if one of the following is true:

1.  A exactly matches B
2.  A and B are both in [“http”, “ws”]
3.  A and B are both in [“https”, “wss”]

Else they are considered incompatible.


Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com. It will never be sent to a different scheme value such as http://example.com.

## Storage
We update the storage model in [Section 5.4.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-store-a-cookie) to ensure that port and scheme are being considered when comparing with existing cookies.

Step number 18 of this section will need to be altered to:

{:quote}
> 1. If httpOnlyAllowed is false and oldCookie's http-only is true, then return null.
> 2. If cookie's secure is equal to oldCookie's secure, cookie's same-site is equal to oldCookie's same-site, and cookie's expiry-time is equal to oldCookie's expiry-time, then return null.
> 3. If cookie's port or scheme is not equal to oldCookie's port or scheme, then stop here and do not do steps 4 and 5.
> 4. Set cookie's creation-time to oldCookie's creation-time.
> 5. Remove oldCookie from the user agent's cookie store.

Note the addition of step number 3. This step will prevent a cookie with differing port or scheme values from overwriting the oldCookie, instead this cookie would be stored as a separate cookie and the oldCookie will not be deleted.

## Retrieve Cookies

The Retrieve Cookies algorithm will need to be updated in [Section 5.4.5 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-retrieve-cookies).

The following will need to be added to step 2:

1.  port `port-matches` cookie's port.
2.  scheme `scheme-matches` cookie's scheme.

This will ensure that a cookie is only retrieved if the request origin is equal to the cookie's origin

## The Cookie Header Field
The cookie header field outlined in [Section 5.5.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-the-cookie-header-field) will need to be altered.

It will be altered to the following:

{:quote}
>1. Let isSecure be a boolean indicating whether request's URL's scheme is deemed secure, in an implementation-defined manner.
>2. Let host be request's host.
>3. Let path be request's URL's path.
>4. Let httpOnlyAllowed be true.
>5. Let sameSite be a string whose value is implementation-defined, but has to be one of "strict-or-less", "lax-or-less", "unset-or-less", or "none".
>6. let port be request's URL's port.
>7. let scheme be request's URL's scheme.
>8. Let cookies be the result of running Retrieve Cookies given isSecure, host, path, httpOnlyAllowed, sameSite, port, and scheme.
>9. Return the result of running Serialize Cookies given cookies.

Note the major change here is extracting the port and scheme from the request and running the new version of Retrieve Cookies based on that.


## Garbage Collection
The last algorithm that will need to be updated is the Garbage Collection algorithm outlined in [Section 5.4.4 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-garbage-collect-cookies).

To prevent an insecure origin from deleting the cookies of secure origins (of the same eTLD+1) the eviction policy will be modified to prefer non-secure cookies before secure cookies.
In the context of eviction policy secure cookies are cookies that specify the Secure attribute or cookies that are set by a secure scheme.
Similarly, domain cookies on an origin will be preferred for eviction before origin cookies on an origin (of the same scheme://eTLD+1).

New eviction policy (per eTLD+1):

1. Expired cookies (Most preferred to evict)
2. For each {priority, secureness} tuple :
3. {Low, insecure}
4. {Low, secure}
5. {Medium, insecure}
6. {High, insecure}
7. {Medium, secure}
8. {High, secure}
9. Aliased domain cookies (in legacy mode)
10. Aliased origin cookies (in legacy mode)
11. Unique domain cookies
12. Unique origin cookies

Under this policy a unique high priority secure origin cookie would be the least preferred to evict. The tuple ordering prevents any insecure cookies from evicting medium or high
priority secure cookies (and is inherited from the legacy behavior).

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
Flexible Cookies Using the Domain Attribute
A cookie that includes a Domain attribute is called a "domain cookie" and has more relaxed restrictions. Specifically, it can be sent to:

Any host that matches the specified domain.

Any port on those hosts.

Importantly, the cookie remains bound to the scheme (e.g., https) of the site that set it. This behavior allows developers to opt-out of the stricter default protections, which is useful when a cookie needs to be shared across multiple subdomains or ports.

The "Shadowing" Security Issue
This flexibility creates a potential vulnerability where a secure "origin cookie" (one without a Domain attribute) could be overridden or "shadowed" by a less-secure domain cookie.

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
