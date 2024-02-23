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
  RFC7301: alpn
  RFC8613: oscore
  RFC9460: svcb
  RFC9461: svcb-for-dns
  RFC9462: ddr
  RFC9463: dnr
  I-D.ietf-core-dns-over-coap: doc
  I-D.ietf-core-oscore-edhoc: edhoc

informative:
  RFC7228: constr-nodes
  RFC7858: dot
  RFC7959: coap-block
  RFC8323: coap-tcp
  RFC8484: doh
  RFC9250: doq
  RFC9203: ace-oscore
  I-D.amsuess-core-coap-over-gatt: coap-gatt
  I-D.ietf-ace-edhoc-oscore-profile: ace-edhoc
  I-D.ietf-core-href: cri
  lwm2m:
    title: White Paper – Lightweight M2M 1.1
    author:
      org: OMA SpecWorks
    date: 2018-10
    target: https://omaspecworks.org/white-paper-lightweight-m2m-1-1/

--- abstract

This document discusses problems and proposes potential solutions for discovery of encrypted DNS
configuration of DNS over CoAP resolvers over DNS and in a local network using Router Advertisements
or DHCP. It discusses discovery of CoAP transfer protocols, security modes, and configuration for
said security modes.

--- middle

# Introduction

{{-svcb-for-dns}}, {{-ddr}} and {{-dnr}} introduced ways to discover the encrypted DNS configuration
of resolvers, both over DNS and in a local network using Router Advertisements or DHCP.
They use SVCB records or their SvcParams definitions to carry the information on a resolver.
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
- **Transport Security:** CoAP may use DTLS for when transferred over UDP {{-coap}} and TLS when
  transferred over TCP {{-coap-tcp}}.
- **Object Security:** Application-layer based object encryption within CoAP based on OSCORE
  {{-oscore}}. OSCORE can be either used as an alternative or in addition to transport security.

  OSCORE keys are not usable indefinitely and need to be set up,
  for example through an EDHOC key exchange {{-edhoc}},
  which may use credentials from trusted ACE Authorization Server (AS)
  as described in the ACE EDHOC profile {{-ace-edhoc}}.
  As an alternative to EDHOC,
  keys can be set up by such an AS as described in the ACE OSCORE profile {{-ace-oscore}}.

For a DoC server to be discoverable via Discovery of Designated Resolvers (DDR) {{-ddr}} and
Discovery of Network-designated Resolvers (DNR) {{-dnr}}, both transfer protocol and type and
parameters for the security parameter need to be provided in the SvcParams field of these
mechanisms, which this document will discuss.

# Terminology

The terms “DoC server” and “DoC client” are used as defined in {{-doc}}.

The term “constrained node” is used as defined in {{-constr-nodes}}.

SvcParams denotes the field in either DNS SVCB/HTTPS records as defined in {{-svcb}}, or DHCP and RA
messages as defined in {{-dnr}}. SvcParamKeys are used as defined in {{-svcb}}.

{::boilerplate bcp14-tagged}

# Problem Space

The first and most important question to ask for the discoverability of DoC resolvers is if and what
new SvcParamKeys need to be defined.

{{-svcb}} defines the “alpn” key, which is used to identify the protocol suite of a service binding
using its Application-Layer Protocol Negotiation (ALPN) ID {{-alpn}}. While this is useful to
identify classic transport layer security, the question is raised if this is needed or even helpful
for when there is only object security. There is an ALPN ID for CoAP over TLS that was defined in
{{-coap-tcp}} but it is not advisable to use the same ALPN ID for CoAP over DTLS. Object security
may be selected in addition to transport layer security, so defining an ALPN ID for each
combination might not be viable or scalable. For some ways of setting up object security, additional information is
needed for the establishment of an encryption context and for authentication with an authentication
server (AS). Orthogonally to the security mechanism, the transfer protocol needs to be established.

Beyond the SvcParamKeys, there is the question of what the field values of the Encrypted DNS Options defined
in {{-dnr}} might be with EDHOC or ACE EDHOC. While most fields map,
“authentication-domain-name” (ADN) and its corresponding ADN length field may not matter in ACE driven cases.

Out of scope of this document are related issues adjacent to its problem space.
they are listed both for conceptual delimitation,
and to aid in discussion of more comprehensive solutions:

* There is ongoing work in addressing the trouble created by CoAP using a diverse set of URI schemes
  in the shape of `coap+...`, such as `coap+tcp` {?I-D.ietf-core-transport-indication}.
  The creation of URI authority values that express the content of SVCB records together with IP literals
  is part of the solution space that will be explored there.

* Route Advertisements (RAs) as used in {{-dnr}} can easily exceed the link layer fragmentation threshold of constrained networks.
  The presence of DNR information in an RA can contribute to that issue.

# Solution Sketches

To answer the raised questions, we first look at the general case then 4 base scenarios, from which
other scenarios might be a combination of:

- Unencrypted DoC,
- DoC over TLS/DTLS,
- DoC over OSCORE using EDHOC, and
- DoC over OSCORE using ACE-EDHOC.

In the general case, we mostly need to answer the question for additional SvcParamKeys. {{-svcb}}
defines the keys “mandatory”, “alpn”, “no-default-alpn”, “port”, “ipv4hint”, and “ipv6hint” were
defined. Additionally, {{-svcb-for-dns}} defines “dohpath” which carries the URI template for the
DNS resource at the DoH server in relative form.

For DoC, the DNS resource needs to be identified as, so a corresponding “docpath” key should be
provided that provides either a relative URI or CRI {{-cri}}. Since the URI-Path option in CoAP may
be omitted (defaulting to the root path), this could also be done for the “docpath”.

## Unencrypted DoC {#sec:solution-unencrypted}
While unencrypted DoC is not recommended by {{-doc}} and might not even be viable using DDR/DNR, it
provides additional benefits not provided by classic unencrypted DNS over UDP, such as segmentation
block-wise transfer {{-coap-block}}. However, it provides the simplest DoC configuration and thus is
here discussed.

At minimum for a DoC server a way to identify the following keys are required. “docpath” (see
above), an optional “port” (see {{-svcb}}), the IP address (either with an optional
“ipv6hint”/“ipv4hint” or the respective IP address field in {{-dnr}}), and a yet to be defined
SvcParamKey for the CoAP transfer protocol, e.g., “coaptransfer”. The latter can be used to identify
the service binding as a CoAP service binding.

The “authenticator-domain-name” field should remain empty as it does not serve a purpose without
encryption.

See this example for the possible values of a DNR option:

~~~~~~~~
authenticator-domain-name: ""
ipv6-address: <DoC server address>
svc-params:
 - coaptransfer="tcp"
 - docpath="/dns"
 - port=61616
~~~~~~~~

## DoC over TLS/DTLS {#sec:solution-tls}
In addition to the SvcParamKeys proposed in {{sec:solution-unencrypted}}, this scenario needs the
“alpn” key. While there is a “coap” ALPN ID defined, it only identifies CoAP over TLS {{-coap-tcp}}.
As such, a new ALPN ID for CoAP over DTLS is required.

See this example for the possible values of a DNR option:

~~~~~~~~
authenticator-domain-name: "dns.example.com"
ipv6-address: <DoC server address>
svc-params:
 - alpn="co" /*TBD*/
 - docpath="/dns"
~~~~~~~~

Note that “coaptransfer” may not necessarily be needed, as it is implied by the ALPN ID.

## DoC over OSCORE using EDHOC
While the “alpn” SvcParamKey is needed for the transport layer security (see {{sec:solution-tls}}),
we can implement a CA-style authentication with EDHOC when using object security with OSCORE using
the authenticator-domain-name field.

A new key SvcParamKey “objectsecurity” identifies the type of object security, using the value
"edhoc" in this scenario.

See this example for the possible values of a DNR option:

~~~~~~~~
authenticator-domain-name: "dns.example.com"
ipv6-address: <DoC server address>
svc-params:
 - coaptransfer="udp",
 - objectsecurity="edhoc",
 - docpath="/dns",
 - port=61616
~~~~~~~~

## DoC over ACE-OSCORE
Using ACE, we require an OAuth context to authenticate the server in addition to the
“objectsecurity” key. We propose three keys “oauth-aud” for the audience, “oauth-scope” for the
OAuth scope, and “auth-as” for the authentication server. “oauth-aud” should be the valid domain
name of the DoC server, “oauth-scope” a list of identifiers for the scope, and “oauth-as” a valid
URI or CRI.

TBD: should oauth-scope be expressed at all?

Since authentication is done over OAuth and not CA-style, the “authenticator-domain-name” is not
needed. There might be merit, however, to use it instead of the “oauth-aud” SvcParamKey.

See this example for the possible values of a DNR option:

~~~~~~~~
authenticator-domain-name: ""
ipv6-address: <DoC server address>
svc-params:
 - coaptransfer="tcp"
 - objectsecurity="edhoc" /* TBD: or ace-edhoc? */
 - docpath="/dns",
 - port=61616,
 - oauth-aud="dns.example.com",
 - oauth-scope="resolve DNS"
 - oauth-as="coap://as.example.com"
~~~~~~~

# Security Considerations

TODO Security


# IANA Considerations

TODO IANA Considerations


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
