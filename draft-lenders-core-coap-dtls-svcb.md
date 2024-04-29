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

A CoAP client that obtains an SVCB record during the discovery of some service can construct a CoAP request message to interact with.
Through the rules of {{Section 6.5 of RFC7252}} and {{Section 5.10.2 of RFC7252}}
(or the equivalent sections for other transports as indicated by the ALPN, eg. {{Section 8.7 of ?RFC8323}}),
those options can be composed into a URI,
but contain more information than that.

Requests are initially attempted according to the priorities of the records
following {{Section 2.4.1 of RFC9460}}.
Once a request is successful, subsequent requests follow that resolution path
until the service can not be reached,
at which point the complete discovery may be reinitiated,
respecing any rate limits and cachable information along the way.

## Input data

As its input, this algorithm takes a SVCB DNS resource record set (RRs),
consisting of information about

a. the host name that was queried
b. the service name that was queried,
c. whether a / which non-default service port was queried,
- a set of SVCB records, each containing
  c. a priority,
  d. a target name,
  e. service parameters, and
  f. additional information about the target (e.g. in the form of AAAA records)

In a DNS request, items a, b and c are grouped using the Attrleaf naming pattern as described in {{Section 2.3 of RFC9460}}.

The algorithm may also be started from equivalent data.
For example, discovery of a network-designated resolver (DNR, {{-dnr}}) through DHCPv6
implies that the service name that was queried was "dns" (with no queried port number),
that the target name was the queried host name ("."),
and contains a service priority,
an authentication-domain-name (used as the host name, and by the previous implication, the target),
IPv6 addresses (equivalent to AAAA records for the target)
and service parameters.

Records are required to contain information about the protocol through which they are used.
Currently, the only way this information can be transported is through the ALPN inside the service parameters.
Future specifications may describe other per-record parameters,
or use service names for which a default transport is specified.
Records with no usable transport (lack of indication of unknown ALPN values) MUST be ignored.

## Request construction

The steps through which the CoAP request is constructed are as follows:

* Select a CoAP transport based on per-record information, falling back to a default for the named service if one is defined.

  In particular, select CoAP over UDP if the ALPN is "co", and CoAP over TCP if the ALPN is "coap".
  Note that the use of ALPNs in the presence of a default transport for the service is also influenced by the no-default-alpn parameter as per {{Section 7.1 of RFC9460}}.

  If multiple transports are available in a single record,
  transports tried in sequence, following the rules of {{Section 7.1.2 of RFC9460}}.

* Send the request to the address indicated in additional information about the target,
  e.g. from a AAAA record associated with the target or an ipv6hint in the service parameters,
  falling back to an attempt to resolve the names.

* If a port is indicated in the service parameters, use that as the destination port;
  otherwise, the destination port is the default port of the transport.

* If the queried service name indicates a particular scheme, set that scheme in the Proxy-Scheme option.

* Set the queried host name as the Uri-Host option.
  The option should be elided if that value is the default value on that transport.

* If the queried port differs from either the destination port or the used protocol's default port,
  set the Uri-Port option to the queried port.

  \[ There may need to be a rule that sets the queried port to the default port of the scheme implied by the service name if there is one;
  that is best handle when better understanding other use cases. \]

* If the queried service specifies an option through which a particular resource can be set,
  add a Uri-Path option for each value in that list.
  It is recommended that such options are serialized as a CBOR sequence of text strings.

Note that when services names that do not indicate a particular scheme (such as "dns")
are used with multiple ALPNs,
create implicit URI aliasing between the two URIs those requests can be composed into.

## Example

A host receives an IPv6 Route Advertisement (RA) for Encrypted DNS as described in {{Section 6 of -dnr}}.
The RA contains the priority 1, authenticated-domain-name "dns1.example.com", IPv6 address 2001:db8:53::1 and service parameters alpn="coap",port="61616",keycpa9=\143dns\146server (where keycpa9 may also written as docpath="dns","server" depending on whether that option defines a zone file format).

The Encrypted DNS RA is implied to be a response to an SVCB request to "\_dns.dns1.example.com".
The "dns" service has no implied default transport for CoAP, and the "docpath" is the service parameter that provides path construction for the request path.

A DNS request is therefore sent by establishing a CoAP over TLS connection
with 2001:db8:53::1 on port 61616.
The name "dns1.example.com" is sent through the SNI extension.
The individual request is then sent as a FETCH request
with two Uri-Path options "dns" and "server",
and a payload according to the DNS over CoAP specification.

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
