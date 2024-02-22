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
  RFC7252: coap
  RFC8613: oscore
  RFC9460: svcb
  RFC9461: svcb-for-dns
  RFC9462: ddr
  RFC9463: dnr
  I-D.ietf-core-dns-over-coap: doc
  I-D.ietf-core-oscore-edhoc: edhoc

informative:
  RFC7858: dot
  RFC7959: coap-block
  RFC8323: coap-tcp
  RFC8484: doh
  RFC9250: doq
  I-D.amsuess-core-coap-over-gatt: coap-gatt
  lwm2m:
    title: White Paper – Lightweight M2M 1.1
    author:
      org: OMA SpecWorks
    date: 2018-10
    target: https://omaspecworks.org/white-paper-lightweight-m2m-1-1/
  I-D.ietf-ace-edhoc-oscore-profile: ace-edhoc
  RFC9203: ace-oscore

--- abstract

TODO Abstract


--- middle

# Introduction

{{-svcb-for-dns}}, {{-ddr}} and {{-dnr}} introduced ways to discover the encrypted DNS configuration
of resolvers, both over DNS and in a local network using Router Advertisements or DHCP.
They use SVCB records or their SvcParam definitions to carry the information on a resolver.
However, so far only DNS transfer protocols based on Transport Layer Security (TLS) were accounted
for, namely DNS over TLS (DoT) {{-dot}}, DNS over HTTPS (DoH) {{-doh}}, and DNS over Dedicated QUIC
(DoQ) {{-doq}}. This document aims to bridge this gap for DNS over CoAP (DoC) {{-doc}}.

DoC provides a solution for encrypted DNS in constrained environments, i.e., where the usage of DoT,
DoH, DoQ or similar TLS-based solutions typically are not possible.
The Constrained Application Protocol (CoAP) {{-coap}}, the transfer protocol for DoC, is mostly
agnostic to the transport layer, i.e., it can be transported over UDP, TCP, or WebSockets
{{-coap-tcp}}, and even more obscure transports such as Bluetooth GATT {{-coap-gatt}} or SMS
{{lwm2m}} are discussed.
CoAP comes with 3 security modes that would need to be covered by the SvcParams:

- **No Security:** No encryption, just plain CoAP. While not recommended with {{-doc}}, this mode
  provides CoAP features, otherwise not present in classic DNS over UDP, such as
  block-wise transfer {{-coap-block}} for datagram-based segmentation.
- **Transport Security:** CoAP may use DTLS for when transfered over UDP {{-coap}} and TLS when
  transfered over TCP {{-coap-tcp}}.
- **Object Security:** Application-layer based object encryption within CoAP based on OSCORE
  {{-oscore}}. OSCORE can be either used as an alternative or in addition to transport security.

  OSCORE keys are not usable indefinitely and need to be set up,
  for example through an EDHOC key exchange {{-edhoc}},
  which may use credentials from trusted authorization server (AS)
  as described in the ACE EDHOC profile {{-ace-edhoc}}.
  As an alternative to EDHOC,
  keys can be set up by such an AS as described in the ACE OSCORE profile {{-ace-oscore}}.

For a DoC server to be discoverable via DDR {{-ddr}} and DNR {{-dnr}}, both transfer
protocol and type and parameters for the security parameter need to be provided in the SvcParams
field of these mechanisms, which this document will discuss.

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
    /* or ace-edhoc?, also encoded as a numeric value */,
    objectsecurity="edhoc"
    docpath="/dns",
    port=61616,
    oauth-aud="dns.example.com",
    oauth-scope="resolve DNS"/* should this be expressed at all? */,
    oauth-as="coap://as.example.com" /* encoded as a CRI? */
~~~~~~~

# Security Considerations

TODO Security


# IANA Considerations

TODO IANA Considerations


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
