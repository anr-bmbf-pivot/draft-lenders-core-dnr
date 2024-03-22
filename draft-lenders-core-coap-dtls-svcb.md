---
title: "Service Binding and Parameter Specification for CoAP over (D)TLS "
abbrev: "CoRE SVCB"
category: std

docname: draft-lenders-core-coap-dtls-svcb-latest
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
 - SVCB
venue:
  group: "Constrained RESTful Environments"
  type: "Working Group"
  mail: "core@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/core/"
  github: "anr-bmbf-pivot/draft-lenders-core-coap-dtls-svcb"
  latest: "https://anr-bmbf-pivot.github.io/draft-lenders-core-coap-dtls-svcb/draft-lenders-core-dnr.html"

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
  RFC9460: svcb
  RFC9463: dnr
  I-D.ietf-core-dns-over-coap: doc

informative:
  RFC8323: coap-tcp
  I-D.amsuess-core-coap-over-gatt: coap-gatt
  lwm2m:
    title: White Paper – Lightweight M2M 1.1
    author:
      org: OMA SpecWorks
    date: 2018-10
    target: https://omaspecworks.org/white-paper-lightweight-m2m-1-1/

--- abstract

This document specifies the usage of "SVCB" ("Service Binding") DNS resource
records for the discovery of transport secured CoAP services.

--- middle

# Introduction

{{-svcb}} specifies the "SVCB" ("Service Binding") DNS resource records to lookup information on
how to communicate with a service. Service Parameters (SvcParams) are used to
carry that information. This document specifies how to lookup information, or
SvcParams, on CoAP services that are secured by transport security, namely TLS and DTLS. These
SvcParams can also be used to discover DNS over CoAP (DoC) servers (see
{{-doc}}) that use TLS and DTLS to secure their messages.

Future work may also provide guidance on how to discover CoAP services that secure their messages
using OSCORE or use transport layers other than TCP or UDP (see, e.g.,
{{coap-gatt}} or {{lwm2m}}). They are, however, out of bounds for this document.

# Terminology

SvcParams denotes the field in either DNS SVCB/HTTPS records as defined in {{-svcb}}, or DHCP and RA
messages as defined in {{-dnr}}.

{::boilerplate bcp14-tagged}

# Application-Layer Protocol Negotiation (ALPN) IDs

{{-svcb}} defines the "alpn" key, which is used to identify the protocol suite of a service binding
using its Application-Layer Protocol Negotiation (ALPN) ID {{-alpn}}. There is
an ALPN ID for CoAP over TLS that was defined in {{-coap-tcp}}. As using the
same ALPN ID for different transport layers is not recommended, an ALPN for
CoAP over DTLS has also been registered in {{iana}}. To discover CoAP services
that secure their messages with TLS or DTLS, these ALPN IDs can be used in the
same manner as for any other service secured with transport layer security, as
described in {{-svcb}}. Other authentication mechanisms are currently out of scope.

# Constructing a CoAP URI from a SVCB Resource Record
TBD Christian

# Security Considerations

Any security considerations on SVCB resource records (see {{-svcb}}), also apply to this document.

# IANA Considerations {#iana}

## TLS ALPN for CoAP

The following entry has been added to the
"TLS Application-Layer Protocol Negotiation (ALPN) Protocol IDs" registry,
which is part of the "Transport Layer Security (TLS) Extensions" group.

* Protocol: CoAP (over DTLS)
* Identification sequence: 0x63 0x6f ("co")
* Reference: {{-coap}} and \[this document\]

Note that {{-coap}} does not prescribe the use of the ALPN TLS extension during connection the DTLS handshake.
This document does not change that, and thus does not establish any rules like those in {{Section 8.2 of -coap-tcp}}.


--- back

# Change Log


# Acknowledgments
{:numbered="false"}

TODO acknowledge.
