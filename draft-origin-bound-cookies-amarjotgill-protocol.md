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

{::boilerplate bcp14-tagged}

# Server Requirements

## Origin-Bound Behavior

### Port Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding source-port to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the source-port attribute.

{:quote}
> A cookie's source-port. It is initially "unset".

Then add the concept of port matching which helps to simplify checking if a cookie would match a port value We can do that by adding a new section under [Section 5.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-subcomponent-algorithms).

An integer `port-matches` a given cookie if any of the following conditions are true:

1.  The cookie's `host-only-flag` is `false`.
2.  The integer exactly matches the cookie's `port` value.

Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com(:443). It will never be sent to a different port value such as https://example.com:8443.


### Scheme Bound

First, alter [Section 5.1.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-cookie-struct) by adding source-scheme to the Cookie Struct, this would be necessary to keep track of the port the cookie was set on.

Below is the definition of the source-scheme attribute.

{:quote}
> A cookie's source-scheme. It is initially "unset".

Then add the concept of scheme matching which helps to simplify checking if a cookie would match a scheme value We can do that by adding a new section under [Section 5.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-subcomponent-algorithms).

A scheme `scheme-matches` a given cookie if any of the following conditions are true:

1.  The cookie's `host-only-flag` is `false`.
2.  The scheme exactly matches the cookie's `scheme` value.

Example:

The new behavior will behave as the following.

A cookie set by origin https://example.com will only ever be sent to https://example.com. It will never be sent to a different scheme value such as http://example.com.

### Storage
Altering the storage model in [Section 5.4.3 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-store-a-cookie) will also be neccessary this is due to the fact that is the newCookie and oldCookie's scheme and port do not exact match then instead of overwriting the new cookie is stored as a seperate cookie.

Step number 18 of this section will need to be altered to:

{:quote}
> 1. If httpOnlyAllowed is false and oldCookie's http-only is true, then return null.
> 2. If cookie's secure is equal to oldCookie's secure, cookie's same-site is equal to oldCookie's same-site, and cookie's expiry-time is equal to oldCookie's expiry-time, then return null.
> 3. If cookie's port or scheme is not equal to oldCookie's port or scheme, then stop here and do not do steps 4 and 5.
> 4. Set cookie's creation-time to oldCookie's creation-time.
> 5. Remove oldCookie from the user agent's cookie store.

Note the addition of step number 3. This step will prevent a cookie with differing port or scheme values from overwriting the oldCookie, instead this cookie would be stored as a separate cookie and the oldCookie will not be deleted.

### Retrieve Cookies 

The Retrieve Cookies algorithm will need to be updated in [Section 5.5.4 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-retrieve-cookies)

The following will need to be an addition to step 2:

1. scheme `scheme-matches` cookie's scheme.
2. port `port-matches` cookie's port.

This will ensure that a cookie is only retrieved if the origin is the same as the one it was set on.

### The Cookie Header Field
The cookie header field outlined in [Section 5.5.2 of COOKIES](https://httpwg.org/http-extensions/draft-ietf-httpbis-layered-cookies.html#name-the-cookie-header-field) will need to be altered.

The following will need to be altered:

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

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
