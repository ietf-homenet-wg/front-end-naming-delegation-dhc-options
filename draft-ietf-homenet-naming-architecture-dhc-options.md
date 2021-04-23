---

title: DHCPv6 Options for Home Network Naming Authority
abbrev: DHCPv6 Options for HNA
docname: draft-ietf-homenet-naming-architecture-dhc-options-11
ipr: trust200902
area: Internet
wg: Homenet
kw: Internet-Draft
cat: std
coding: utf-8

pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  docmapping: yes

author:
      -
        ins: D. Migault
        name: Daniel Migault
        org: Ericsson
        street: 8275 Trans Canada Route
        city: Saint Laurent, QC
        code: 4S 0B6
        country: Canada
        email: daniel.migault@ericsson.com
      -
        ins: R. Weber
        name: Ralf Weber
        org: Akamai
        street:
        city:
        code:
        country:
        email: ralf.weber@akamai.com
      -
        ins: T. Mrugalski
        name: Tomek Mrugalski
        org: Internet Systems Consortium, Inc.
        street: 950 Charter Street
        city: Redwood City
        code: 94063
        country: US
        email: tomasz.mrugalski@gmail.com


--- abstract

This document defines DHCPv6 options so an agnostic Homenet Naming Authority (HNA) can automatically proceed to the appropriate configuration and outsource the authoritative naming service for the home network.
In most cases, the outsourcing mechanism is transparent for the end user.

--- middle


# Terminology

{::boilerplate bcp14}

The reader is expected to be familiar with {{?I-D.ietf-homenet-front-end-naming-delegation}} and its terminology section.

# Introduction

{{?I-D.ietf-homenet-front-end-naming-delegation}} specifies how an entity designated as the Homenet Naming Authority (HNA) outsources a Public Homenet Zone to an Outsourcing DNS Infrastructure (DOI).

This document shows how an ISP can provision automatically the HNA with a specific DOI.
Most likely the DOI will be - at least partly be - managed or provided by its ISP, but other cases may envision the ISP storing some configuration so the homenet becomes resilient to HNA replacement.

The ISP delegates the home network an IP prefix it owns as well as the associated reverse zone.
The ISP is well aware of the owner of that prefix, and as such becomes a natural candidate for hosting the Homenet Reverse Zone - that is the Reverse Distribution Master (RDM) and potentially the Reverse Public Authoritative Servers.

In addition, the ISP often identifies the home network with a name.
In most cases, the name is used by the ISP for its internal network management operations and is not a name the home network owner has registered to.
The ISP may thus leverage such infrastructure and provide the homenet a specific domain name designated as per {{?I-D.ietf-homenet-front-end-naming-delegation}} a Homenet Registered Domain.
Similarly to the reverse zone, the ISP is well aware of who owns that domain name and may become a natural candidate for hosting the Homenet Zone - that is the Distribution Master (DM) and the Public Authoritative Servers.

This document describes DHCPv6 options that enables the ISP to provide the necessary parameters to the HNA, to proceed.
More specifically, the ISP provides the Registered Homenet Domain, necessary information on the DM and the RDM so the HNA can manage and upload the Public Homenet Zone and the Reverse Public Homenet Zone as described in {{?I-D.ietf-homenet-front-end-naming-delegation}}.

The use of DHCPv6 options makes the configuration completely transparent to the end user and provides a similar level of trust as the one used to provide the IP prefix.


# Protocol Overview {#sec-overview}

This section illustrates how a HNA receives the necessary information via DHCPv6 options to outsource its authoritative naming service to the DOI.
For the sake of simplicity, and similarly to {{?I-D.ietf-homenet-front-end-naming-delegation}}, this section assumes that the HNA and the home network DHCPv6 client are collocated on the CPE.
Note also that this is not mandatory and specific communications between the HNA and the DHCPv6 client only are needed.
In addition, this section assumes the responsible entity for the DHCPv6 server is able to configure the DM and RDM.
In our case, this means a Registered Homenet Domain can be associated to the DHCP client.

This scenario has been chosen as it is believed to be the most popular scenario. This document does not ignore scenarios where the DHCP Server does not have privileged relations with the DM or RDM.
These cases are discussed latter in {{sec-scenario}}.
Such scenarios do not necessarily require configuration for the end user and can also be zero-config.

The scenario considered in this section is as follows:

1. The HNA is willing to outsource the Public Homenet Zone or Homenet Reverse Zone and configures its DHCP Client to include in its Option Request Option (ORO) the Registered Homenet Domain Option (OPTION_REGISTERED_DOMAIN), the Distribution Master Option (OPTION_DIST_MASTER) and the Reverse Distribution Master Option (OPTION_REVERSE_DIST_MASTER) option codes.

2. The DHCP Server responds to the HNA with the requested DHCPv6 options based on the identified homenet.
The DHCP Client transmits the information to the HNA.

3. The HNA is able to get authenticated by the DM and the RDM. The HNA builds the Homenet Zone ( resp. the Homenet Reverse Zone) and proceed as described in {{?I-D.ietf-homenet-front-end-naming-delegation}}.
The DHCPv6 options provide the necessary and non optional parameters described in section 14 of {{?I-D.ietf-homenet-front-end-naming-delegation}}.
The HNA MAY set complement the configurations with additional parameters.
Section 14 of {{?I-D.ietf-homenet-front-end-naming-delegation}} describes such parameters that MAY take a default value.

# Payload Description {#sec-format}

This section details the payload of the DHCPv6 options.

## Registered Homenet Domain Option {#o_rd}

The Registered Domain Option (OPTION_REGISTERED_DOMAIN) indicates the FQDN associated to the home network.


~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   OPTION_REGISTERED_DOMAIN    |         option-len            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                   Registered Homenet Domain                   /
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{:#fig-rhd title="Registered Domain Option"}

* option-code (16 bits): OPTION_REGISTERED_DOMAIN, the option code for the Registered Homenet Domain  (TBD2).

* option-len (16 bits): length in octets of the option-data field as described in {{!RFC8415}}.

* Registered Homenet Domain (variable): the FQDN registered for the homenet encoded as described in section 10 of {{!RFC8415}}.


## Distribution Master Option {#o_dm}

The Distributed Master Option (OPTION_DIST_MASTER) provides the HNA to FQDN of the DM as well as the transport protocol for the transaction between the HNA and the DM.

~~~
 0                   1                        2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      OPTION_DIST_MASTER       |          option-len           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Supported Transport       |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
|                                                               |
/                   Distribution Master  FQDN                   /
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{:#fig-dm title="Distribution Master Option"}


* option-code (16 bits): OPTION_DIST_MASTER, the option code for the DM Option (TBD3).

* option-len (16 bits): length in octets of the option-data field as
described in {{!RFC8415}}.

* Supported Transport (16 bits): defines the supported transport by the DM.
Each bit represents a supported transport, and a DM MAY indicate the support of multiple modes.
The bit for DNS over TLS {{!RFC7858}} MUST be set.

* Distribution Master FQDN (variable): the FQDN of the DM encoded as described in section 10 of {{!RFC8415}}.

### Supported Transport {#sec-st}

The Supported Transport filed of the DHCPv6 option indicates the supported transport protocol.
Each bit represents a specific transport mechanism. The bit sets to 1 indicates the associated transport protocol is supported.
The corresponding bits are assigned as described in {{tab-st}}.

~~~
Bit | Transport Protocol | Reference
----+--------------------+-----------
 0  | DNS over TLS       | This-RFC
1-15| unallocated        |
~~~
{:#tab-st title="Supported Transport" }

* DNS over TLS: indicates the support of DNS over TLS as described in {{!RFC7858}}.

## Reverse Distribution Master Server Option

The Reverse Distribution Master Server Option (OPTION_REVERSE_DIST_MASTER) provides the HNA to FQDN of the DM as well as the transport protocol for the transaction between the HNA and the DM.

~~~
 0                   1                        2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| OPTION_REVERSE_DIST_MASTER    |          option-len           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Supported Transport       |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
|                                                               |
/               Reverse Distribution Master FQDN                /
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

~~~
{: #fig-rdm title="Reverse Distribution Master Option"}



* option-code (16 bits): OPTION_REVERSE_DIST_MASTER, the option code for the Reverse Distribution Master Option (TBD4).

* option-len (16 bits): length in octets of the option-data field as described in {{!RFC8415}}.

* Supported Transport (16 bits): defines the supported transport by the DM.
Each bit represents a supported transport, and a DM MAY indicate the support of multiple modes. The bit for DoT MUST be set.

* Reverse Distribution Master FQDN (variable): the FQDN of the RDM encoded as described in section 10 of {{!RFC8415}}.

# DHCP Behavior {#sec-dhcp-behavior}

## DHCPv6 Server Behavior

Sections 17.2.2 and 18.2 of {{!RFC8415}} govern server operation in regards to option assignment.
As a convenience to the reader, we mention here that the server will send option foo only if configured with specific values for foo and if the client requested it.
In particular, when configured the DHCP Server sends the Registered Homenet Domain Option, Distribution Master Option, the Reverse Distribution Master Option when requested by the DHCPv6 client by including necessary option codes in its ORO.


## DHCPv6 Client Behavior

The DHCPv6 client sends a ORO with the necessary option codes: Registered Homenet Domain Option, Distribution Master Option, the Reverse Distribution Master Option.

Upon receiving a DHCP option described in this document in the Reply
message, the HNA SHOULD proceed as described in {{I-D.ietf-homenet-front-end-naming-delegation}}.


## DHCPv6 Relay Agent Behavior {#sec-dhcp-relay}

There are no additional requirements for the DHCP Relay agents.

# IANA Considerations {#sec-iana}

IANA is requested to assign the following new DHCPv6 Option Codes in the registry maintained in: https://www.iana.org/assignments/dhcpv6-parameters/dhcpv6-parameters.xhtml#dhcpv6-parameters-2.

~~~
Value Description                   Client ORO     Singleton Option
TBD1  OPTION_REGISTERED_DOMAIN      Yes            Yes
TBD2  OPTION_DIST_MASTER            Yes            Yes
TBD3  OPTION_REVERSE_DIST_MASTER    Yes            Yes
~~~

IANA is requested to maintain a new number space of Supported Transport parameter in the Distributed Master Option (OPTION_DIST_MASTER) or the Reverse Distribution Master Server Option (OPTION_REVERSE_DIST_MASTER). The different parameters are defined in {{tab-st}} in {{sec-st}}.
Future code points are assigned under Specification Required as per {{!RFC8126}}.

# Security Considerations {#security-considerations}

The security considerations in {{!RFC2131}} and {{!RFC8415}} are to be considered.
The use of DHCPv6 options provides a similar level of trust as the one used to provide the IP prefix.
The link between the HNA and the DHCPv6 server may benefit from additional security for example by using {{?I-D.ietf-dhc-sedhcpv6}}.

# Acknowledgments

We would like to thank Marcin Siodelski and Bernie Volz for their comments on the design of the DHCPv6 options.
We would also like to thank Mark Andrews, Andrew Sullivan and Lorenzo Colliti for their remarks on the architecture design. The designed solution has been largely been inspired by Mark Andrews's document {{?I-D.andrews-dnsop-pd-reverse}} as well as discussions with Mark.
We also thank Ray Hunter for its reviews, its comments and for suggesting an appropriated terminology.

# Contributors

The co-authors would like to thank Chris Griffiths and Wouter Cloetens that provided a significant contribution in the early versions of the document.


--- back

# Scenarios and impact on the End User {#sec-scenario}

This section details various scenarios and discuss their impact on the end user.
This section is not normative and limits the description of a limited scope of scenarios that are assumed to be representative. Many other scenarios may be derived from these.

# Base Scenario {#sec-scenario-1}

The base scenario is the one described in {{sec-overview}} in which an ISP manages the DHCP Server, the DM and RDM.

The end user subscribes to the ISP (foo), and at subscription time registers for example.foo as its Registered Homenet Domain example.foo.

In this scenario, the DHCP Server, DM and RDM are managed by the ISP so the DHCP Server and as such can provide authentication credentials of the HNA to enable secure authenticated transaction with the DM and the Reverse DM.

The main advantage of this scenario is that the naming architecture is configured automatically and transparently for the end user.
The drawbacks are that the end user uses a Registered Homenet Domain managed by the ISP and that it relies on the ISP naming infrastructure.

## Third Party Registered Homenet Domain {#scenario-2}

This section considers the case when the end user wants its home network to use example.com not managed by her ISP (foo) as a Registered Homenet Domain.
This section still consider the ISP manages the home network and still provides example.foo as a Registered Homenet Domain.

When the end user buys the domain name example.com, it may request to redirect the name example.com to example.foo using static redirection with CNAME {{!RFC2181}}, {{!RFC1034}}, DNAME {{!RFC6672}} or CNAME+DNAME {{?I-D.sury-dnsext-cname-dname}}.

This configuration is performed once when the domain name example.com is registered.
The only information the end user needs to know is the domain name assigned by the ISP.
Once this configuration is done no additional configuration is needed anymore.
More specifically, the HNA may be changed, the zone can be updated as in {{sec-scenario-1}} without any additional configuration from the end user.

The main advantage of this scenario is that the end user benefits from the Zero Configuration of the Base Scenario {{sec-scenario-1}}.
Then, the end user is able to register for its home network an unlimited number of domain names provided by an unlimited number of different third party providers.
The drawback of this scenario may be that the end user still rely on the ISP naming infrastructure.
Note that the only case this may be inconvenient is when the DNS Servers provided by the ISPs results in high latency.


## Third Party DNS Infrastructure {#scenario-3}

This scenario considers that the end user uses example.com as a Registered Homenet Domain, and does not want to rely on the authoritative servers provided by the ISP.

In this section we limit the outsourcing to the DM and Public Authoritative Server(s) to a third party.
The Reverse Public Authoritative Server(s) and the RDM remain managed by the ISP as the IP prefix is managed by the ISP.

Outsourcing to a third party DM can be performed in the following ways:

1. Updating the DHCP Server Information. One can imagine a GUI interface that enables the end user to modify its profile parameters. Again, this configuration update is done once-for-ever.

2. Upload the configuration of the DM to the HNA. In some cases, the provider of the CPE hosting the HNA may be the registrar and provide the CPE already configured. In other cases, the CPE may request the end user to log into the registrar to validate the ownership of the Registered Homenet Domain and agree on the necessary credentials to secure the communication between the HNA and the DM. As described in {{?I-D.ietf-homenet-front-end-naming-delegation}}, such settings could be performed in an almost automatic way as to limit the necessary interactions with the end user.

## Multiple ISPs {#scenario-4}

This scenario considers a HNA connected to multiple ISPs.

Suppose the HNA has been configured each of its interfaces independently with each ISPS as described in {{sec-scenario-1}}.
Each ISP provides a different Registered Homenet Domain.

The protocol and DHCPv6 options described in this document are fully compatible with a HNA connected to multiple ISPs with multiple Registered Homenet Domains.
However, the HNA should be able to handle different Registered Homenet Domains.
This is an implementation issue which is outside the scope of the current document.

If a HNA is not able to handle multiple Registered Homenet Domains, the HNA may remain connected to multiple ISP with a single Registered Homenet Domain.
In this case, one entity is chosen to host the Registered Homenet Domain.
This entity may be one of the ISP or a third party.
Note that having multiple ISPs can be motivated for bandwidth aggregation, or connectivity fail-over.
In the case of connectivity fail-over, the fail-over concerns the access network and a failure of the access network may not impact the core network where the DM Server and Public Authoritative Primaries are hosted.
In that sense, choosing one of the ISP even in a scenario of multiple ISPs may make sense.
However, for sake of simplicity, this scenario assumes that a third party has been chosen to host the Registered Homenet Domain.
Configuration is performed as described in {{scenario-2}} and {{scenario-3}}.

With the configuration described in {{scenario-2}},  the HNA is expect to be able to handle multiple Homenet Registered Domain, as the third party redirect to one of the ISPs Servers.
With the configuration described in {{scenario-3}}, DNS zone are hosted and maintained by the third party.
A single DNS(SEC) Homenet Zone is built and maintained by the HNA.
This latter configuration is likely to match most HNA implementations.


The protocol and DHCPv6 options described in this document are fully compatible with a HNA connected to multiple ISPs.
To configure or not and how to configure the HNA depends on the HNA facilities.
{{sec-scenario-1}} and  {{scenario-2}} require the HNA to handle multiple Registered Homenet Domain, whereas {{scenario-3}} does not have such requirement.


