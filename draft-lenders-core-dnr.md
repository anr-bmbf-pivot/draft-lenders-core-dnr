---
title: "Discovery of Network-designated CoRE Resolvers"
abbrev: "CoRE DNR"
category: info

docname: draft-lenders-core-dnr-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Constrained RESTful Environments"
keyword:
 - CoRE
 - CoAP
 - DoC
 - DNR
 - SVCB
venue:
  group: "Constrained RESTful Environments"
  type: "Working Group"
  mail: "core@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/core/"
  github: "anr-bmbf-pivot/draft-lenders-core-dnr"
  latest: "https://anr-bmbf-pivot.github.io/draft-lenders-core-dnr/draft-lenders-core-dnr.html"

author:
 -  fullname: Martine Sophie Lenders
    org: TUD Dresden University of Technology
    abbrev: TU Dresden
    street: Helmholtzstr. 10
    city: Dresden
    code: D-01069
    country: Germany
    email: martine.lenders@tu-dresden.de
 -  name: Christian Amsüss
    email: christian@amsuess.com
 -  fullname: Thomas C. Schmidt
    organization: HAW Hamburg
    email: t.schmidt@haw-hamburg.de
 -  name: Matthias Wählisch
    org: TUD Dresden University of Technology & Barkhausen Institut
    abbrev: TU Dresden & Barkhausen Institut
    street: Helmholtzstr. 10
    city: Dresden
    code: D-01069
    country: Germany
    email: m.waehlisch@tu-dresden.de

normative:
  RFC9460: svcb
  RFC9461: svcb-for-dns
  RFC9462: ddr
  RFC9463: dnr
  I-D.ietf-core-dns-over-coap: doc

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction

## Problems

TODO transform into sentences:

- What do we need to find DoC using SvcParams {{-svcb}} {{-dnr}}?
- Do we need sepcific ALPNs?
    - What should the fields in {{-dnr}} RA / DHCP options give us?
      (authentication-domain-name may not matter in EDHOC/ACE-OSCORE setup, but audience value for
      ACE authorization server and the servers address might)
- How do we signal ALPN-equivalent information when there is not “the one” transport layer security?
- Can we indicate the transport (CoAP over UDP/TCP/etc.) orthogonally from (object) security mechanism (EDHOC, ACE-OSCORE, ...)
- TBD but might be out-of-scope:
    - replace coap+... URI schemas with hostname literals
    - Increased RA size / fragmentation

# Terminology

{::boilerplate bcp14-tagged}

# Solution Sketches

What should the fields in RA / DHCP option give us?

## Unencrypted DoC
TBD: Does DNR allow it?

~~~~~~~~
authenticator-domain-name:
    (I'm leaving it empty b/c there is no use for it)
ipv6-address: ...
svcb-params:
    coaptransport="coap-over-tcp",
    docpath="/dns",
    port=61616
~~~~~~~~

## DoC over DTLS
TBD:
- Not even a problem, just register the relevant ALPN.
- Trigger in CoRE, should be separate ALPN to “coap” (CoAP over TLS) ... and bikeshed value “co”, “COAP”, “cod”, ...
- `docpath` if not given = `""` (cmp. URI-Path option in CoAP)?
  > a la ‘When using the SVCB method for obtaining a DoC server (eg. because querying _dns or
  > because it comes in DNR), the server MUST set docpath unless it is empty, in which case the
  > client MUST assume docpath=“”’ to avoid implying an empty docpath even in places where no DoC is
  > done

~~~~~~~~
authenticator-domain-name: dns.example.com
ipv6-address: ...
svcb-params: alpn="cod"/*TBD*/,docpath="/dns"
~~~~~~~~


## DoC over OSCORE using EDHOC
- In a “web-browser style” (tell the device which name to authenticate, and it’ll do the cert
  validation)

~~~~~~~~
authenticator-domain-name: dns.example.com
ipv6-address: ...

svcb-params:
    coaptransport="coap-over-tcp",
    objectsecurity="edhoc",
    docpath="/dns",
    port=61616
~~~~~~~~

## DoC over ACE-OSCORE
- ACE to authenticate server (not necessarily the client)

~~~~~~~~
authenticator-domain-name:
    (leaving empty b/c there is no use for it)
ipv6-address: ...
svcb-params:
    coaptransport="coap-over-tcp" /* encoded as a numeric value */,
    objectsecurity="edhoc"/* or ace-edhoc?, also encoded as a numeric value */,
    docpath="/dns",
    port=61616,
    oauth-aud="dns.example.com",
    oauth-scope="resolve DNS"/* should this be expressed at all? */,
    oauth-as="coap://as.example.com" /* encoded as a CRI? */
~~~~~~~

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
