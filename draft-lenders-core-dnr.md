---
title: "Discovery of Network-designated OSCORE-based Resolvers: Problem Statement"
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

informative:
  RFC7252: coap
  RFC7228: constr-nodes
  RFC7301: alpn
  RFC7858: dot
  RFC7959: coap-block
  RFC8323: coap-tcp
  RFC8484: doh
  RFC8613: oscore
  RFC9250: doq
  RFC9203: ace-oscore
  RFC9528: edhoc
  I-D.amsuess-core-coap-over-gatt: coap-gatt
  I-D.ietf-ace-edhoc-oscore-profile: ace-edhoc
  I-D.ietf-core-dns-over-coap: doc
  I-D.ietf-core-transport-indication: coap-indication
  I-D.ietf-core-coap-dtls-alpn: coap-dtls-alpn
  lwm2m:
    title: White Paper – Lightweight M2M 1.1
    author:
      org: OMA SpecWorks
    date: 2018-10
    target: https://omaspecworks.org/white-paper-lightweight-m2m-1-1/

--- abstract

This document states problems when designing DNS SVCB records to discover endpoints that communicate over
Object Security for Constrained RESTful Environments (OSCORE) {{-oscore}}.
As a consequence of learning about OSCORE, this discovery will allow a host to learn both CoAP servers and DNS over CoAP resolvers that use OSCORE to encrypt messages and Ephemeral Diffie-Hellman Over COSE (EDHOC) {{-edhoc}} for key exchange.
Challenges arise because SVCB records are not meant to be used to exchange security contexts, which is required in OSCORE scenarios.

--- middle

# Introduction

The discovery of Internet services can be facilitated by the Domain Name System (DNS).
To discover services of the constrained Internet of Things (IoT) using the DNS, two challenges must be solved.
First, the discovery of a DNS resolver that supports DNS resolution based on secure, IoT-friendly protocols&mdash;otherwise the subsequent discovery of IoT-tailored services would be limited to resolution protocols conflicting with constrained resources.
Second, the discovery of an IoT-friendly service beyond the DNS resolution.

{{-svcb}} specifies the "SVCB" ("Service Binding") DNS resource record to lookup information needed to connect to a network service. Service Parameters (SvcParams) carry
that information within the SVCB record.

The discovery of DNS resolvers can be enabled by the DNS itself {{-svcb-for-dns}}, {{-ddr}} or, in a local network, by Router Advertisements and DHCP {{-dnr}}.
In all theses cases, the SvcParams is used, but supports only DNS transfer based on Transport Layer Security (TLS), namely DNS over TLS (DoT) {{-dot}}, DNS over HTTPS (DoH) {{-doh}}, and DNS over Dedicated QUIC (DoQ) {{-doq}}.
The use of DoT, DoH, or DoQ, however, is not recommended in IoT scenarios.

DNS over CoAP {{-doc}} provides a solution for encrypted DNS in constrained environments.
The Constrained Application Protocol (CoAP) {{-coap}} is mostly agnostic to the transport layer CoAP can be transported over UDP, TCP, or WebSockets {{-coap-tcp}}, and even less common transports such as Bluetooth GATT {{-coap-gatt}} or SMS {{lwm2m}} are discussed.
{{-coap-indication}} covers the selection of different CoAP transports using SVCB records.

CoAP offers three security modes:

- **No Security:** This plain CoAP mode does not support any encryption. It
  is not recommended when using {{-doc}} but inherits core CoAP features
  such as block-wise transfer {{-coap-block}} for datagram-based
  segmentation.  Such features are beneficial in constrained settings even
  without encryption.
- **Transport Security:** CoAP may use DTLS when transferred over UDP {{-coap}} and TLS when transferred over TCP {{-coap-tcp}}.
- **Object Security:** Securing content objects can be achieved using
  OSCORE {{-oscore}}. OSCORE can be used either as an alternative or in
  addition to transport security.

  OSCORE keys have a limited lifetime and need to be set up.
  Keys can be established through an EDHOC key exchange {{-edhoc}},
  received from an ACE Authorization Server (AS, as described in the ACE OSCORE profile {{-ace-oscore}}),
  or through a combination of those (established with an EDHOC peer whose public key is confirmed by an AS, using the ACE EDHOC profile {{-ace-edhoc}}).

The SVCB-based discovery of a CoAP service in mode "no security" is covered in {{-coap-indication}}, and a CoAP service in the mode "transport security" in {{-coap-dtls-alpn}}.
The discovery of CoAP services in mode "object security" is not specified.
To guide future specifications, this document clarifies aspects when using SVCB in the context of CoAP and object security.

# Terminology

The terms "DoC server" and "DoC client" are used as defined in {{-doc}}.

The terms "constrained node" and "constrained network" are used as defined in {{-constr-nodes}}.

SvcParams denotes the field in either DNS SVCB/HTTPS records as defined in {{-svcb}}, or DHCP and RA
messages as defined in {{-dnr}}. SvcParamKeys are used as defined in {{-svcb}}.

{::boilerplate bcp14-tagged}

# Problem Space

The first and most important point of discussion for the discoverability of CoAP is if and what
new SvcParamKeys need to be defined and what is already there.

{{-svcb}} defines the "alpn" key, which is used to identify the protocol suite of a service binding
using its Application-Layer Protocol Negotiation (ALPN) ID {{-alpn}}. While this is useful to
identify classic transport layer security, the question is raised if this is needed or even helpful
for when there is only object security. There is an ALPN ID for CoAP over TLS that is defined in
{{-coap-tcp}}. As using the same ALPN ID for different transport layers is not recommended, another
ALPN ID for CoAP over DTLS is introduced in {{-coap-dtls-alpn}}. Object security may be
selected in addition to transport layer security or without it. Additionally, different
CoAP transports can be selected, which may be orthogonal to the transport security.
For instance, DTLS can be used over transports other than UDP. The selection of CoAP transport
protocols will be covered in future versions of {{-coap-indication}}. Defining an ALPN ID for each
combination of object security, mode of transport layer security, and transport protocol might not
be viable or scalable. For some ways of setting up object security, additional information is
needed, such as an establishment options for an encryption context with EDHOC or an authentication
server (AS) with ACE.

Beyond the SvcParamKeys, there is the question of what the field values of the Encrypted DNS Options
defined in {{-dnr}} might be with EDHOC or ACE EDHOC. While most fields map,
"authentication-domain-name" (ADN) and its corresponding ADN length field may not matter
when authentication is driven by Authorization for Constrained Environments (ACE) {{-ace-oscore}}
{{-ace-edhoc}}.

# Objectives

SVCB records are not meant and should not be used to exchange security contexts, so this eliminates
scenarios that use pre-shared keys with OSCORE. This leaves 2 base scenarios for OSCORE, which may
occur in combination, with scenarios using transport security, or alternative transport protocols:

- DoC over OSCORE using EDHOC, and
- DoC using any ACE profile that eventually produces an OSCORE context.

We mostly need to answer the question for additional SvcParamKeys. {{-svcb}} defines the keys
"mandatory", "alpn", "no-default-alpn", "port", "ipv4hint", and "ipv6hint".
Additionally, {{-doc}} defines "docpath" which carries the path for the DNS resource at the DoC
server as a CBOR sequence.

Since "alpn" is needed for transport layer security, the type of object security (OSCORE using
EDHOC, OSCORE using ACE, OSCORE using EDHOC using ACE), needs to be conveyed in a different
SvcParamKey. The semantics and necessacity of the authenticator-domain-name field in {{-dnr}} needs
to be evaluated in each case.

When using ACE, more SvcParamKeys might be needed, such as the OAuth audience, the scope or the
authentication server URI.

Defining these SvcParamKeys, including their value formats and spaces, as well as the behavior
definition for authenticator-domain-name field, shall be part of future work.

# Security Considerations

TODO Security

# IANA Considerations {#iana}

This document has no IANA considerations.

--- back

# Change Log

## Since [draft-lenders-core-dnr-03]
- Update {{-coap-dtls-alpn}} reference

## Since [draft-lenders-core-dnr-02]

- Forward reference to upcoming changes in {{-coap-indication}} updated

## Since [draft-lenders-core-dnr-01]

- Remove parts specified in {{-coap-indication}}
- Remove parts specified in {{-coap-dtls-alpn}}
- Remove solution sketches, set objectives to solve problem space

## Since [draft-lenders-core-dnr-00]

- IANA has processed the "co" ALPN and it is now added to the registry

[draft-lenders-core-dnr-00]: https://datatracker.ietf.org/doc/html/draft-lenders-core-dnr-00
[draft-lenders-core-dnr-01]: https://datatracker.ietf.org/doc/html/draft-lenders-core-dnr-01
[draft-lenders-core-dnr-02]: https://datatracker.ietf.org/doc/html/draft-lenders-core-dnr-02
[draft-lenders-core-dnr-03]: https://datatracker.ietf.org/doc/html/draft-lenders-core-dnr-03

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
