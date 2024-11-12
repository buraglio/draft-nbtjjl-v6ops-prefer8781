---
title: "Discouraging use of RFC7050 for Discovery of IPv6 Prefix Used for IPv6 Address Synthesis"
abbrev: "Discouraging RFC7050"
category: std

docname: draft-nbtjjl-dnsop-prefer8781-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: dnsop
keyword:
 - IPv6
 - DNS64
venue:
  group: dnsop
  type: Working Group
  mail: dnsop@ietf.org
  arch: https://datatracker.ietf.org/wg/dnsop/about/
  github: "buraglio/draft-nbtjjl-dnsop-prefer8781"
  latest: "https://github.com/buraglio/draft-nbtjjl-dnsop-prefer8781"

author:
 -
    fullname: Nick Buraglio
    organization: Energy Sciences Network
    email: "buraglio@forwardingplane.net"
 -
    fullname: Tommy Jensen
    organization: Microsoft
    email: "tojens.ietf@gmail.com"
 -
    fullname: Jen Linkova
    organization: Google
    email: "furry13@gmail.com"
normative:
  RFC7050:

informative:
  RFC4861:
  RFC4862:
  RFC6105:
  RFC6146:
  RFC6145:
  RFC6147:
  RFC6877:
  RFC8781:
  RFC8880:
  RFC9463:
  RFC9499:

--- abstract

RFC7050 describes a method for detecting the presence of DNS64 and for learning the IPv6 prefix used for protocol translation (RFC6145). This methodology depends on the existence of a well-known IPv4-only fully qualified domain name "ipv4only.arpa.". Because newer methods exist that lack the requirement of a higher level protocol, instead using existing operations in the form of native router advertisements, discovery of the IPv6 prefix used for protocol translation using RFC7050 should be discouraged. RFC7050 MAY only be used if other methods (such as RFC8781]) can not be used.

--- middle


# Introduction

The DNS-based mechanism defined in [RFC7050] was the very first mechanism available for nodes to discover the PREF64 information.
However since the publication of RFC7050, other methods have been developed to address some of [RFC7050] limitations.

For example, [RFC8781] describes a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) to hosts. This approach has the advantage of using the same communication channel IPv6 clients use to discover other network configurations such as the network's default route. This means network administrators can secure this configuration along with other configurations IPv6 requires using a single approach such as RA Guard [RFC6105].

Taking into account some fundamental flaws of [RFC7050] mechanism, it seems desirable to prefer [RFC8781] for all new deployments and for implementations to use consistent methods to obtain PREF64 information. RFC 7050 should be used only in cases where there is an inability to offer PREF64 information with a router advertisement, and as a fallback for legacy systems incapable of processing those RA options.


# Conventions and Definitions

NAT64: a mechanism for translating IPv6 packets to IPv4 packets and vice versa.  The translation is done by translating the packet headers according to the IP/ICMP Translation Algorithm defined in [RFC6145].

DNS64: a mechanism for synthesizing AAAA records from A records, defined in [RFC6147].

Router Advertisement: A packet used by Neighbor Discovery [RFC4861] and StateLess Address AutoConfiguration (SLAAC, [RFC4862]) to advertize the presence of the routers, together with other IPv6 configuration information.

PREF64 (or NAT64 prefix): An IPv6 prefix used for IPv6 address synthesis and for network addresses and protocols translation from IPv6 clients to IPv4 servers, [RFC6146].

{::boilerplate bcp14-tagged}

# Existing issues with RFC 7050

DNS-based method of discovering the NAT64 prefix introduces some challenges, which make this approach less preferable than most recently developed alternatives (such as PREF64 RA option, [RFC8781]).
This section outlines the key issues, associated with [RFC7050].

## Dependency on Network-Provided Recursive Resolvers

Fundamentally, the presence of the NAT64 and the exact value of the prefix used for the translation are network-specific attributes.
Therefore, to discover the PREF64 the device needs to use the DNS resolvers provided by the network.
If the device is configured to use other recursive resolvers, its name resolution APIs and libraries are required to recognize 'ipv4only.arpa.' as a special name and give it special treatment.
This issue and remediation approach are discussed in [RFC8880].
However, it has been observed that not all [RFC7050] implementations support [RFC8880] requirements for special treatment of 'ipv4only.arpa.'.
As a result, configuring such systems to use resolvers other than the one provided by the network might break the PREF64 discovery, leading to degraded user experience.

## Network Stack Initialization Delay

When using SLAAC ([RFC4862]), an IPv6 host usually needs just one Router Advertisement (RA, [RFC4861]) packet to obtain all information required to complete the host's network configuration.
For an IPv6-only host, the PREF64 information is essential, especially if the host implements the customer-side translator (CLAT) ([RFC6877]).
The mechanism defined in [RFC7050] implies that the PREF64 information is not bundled with all other network configuration parameters provided by RAs, and can only be obtained after the host has configured the rest of its IPv6 stack.
Therefore, until the process described in Section 3 of [RFC7050] is completed, the CLAT process can not start, which negatively impacts IPv4-only applications which have already started.

## Inflexibility

Section 3 of [RFC7050] requires that the node SHALL cache the replies received during the PREF64 discovery and SHOULD repeat the discovery process ten seconds before the TTL of the Well-Known Name's synthetic AAAA resource record expires.
As a result, once the PREF64 is discovered, it will be used until the TTL expired, or until the node disconnects from the network.
There is no mechanism for an operator to force the PREF64 rediscovery on the node without disconnecting the node from the network.
If the operator needs to change the PREF64 value used in the network, they need to proactively reduce the TTL value returned by the DNS64 server.
This method has two significant drawbacks:

*  Many networks utilize external DNS64 servers and therefore have no control over the TTL value.
*  The PREF64 changes need to be planned and executed at least TTL seconds in advance. If the operator needs to notify nodes that a particular prefix must not be used (e.g. during a network outage or if the nodes learnt a rogue PREF64 as a result of an attack), it might not be possible without interrupting the network connectivity for the affected nodes.


## Security Implications

As discussed in Section 7 of [RFC7050], the DNS-based PREF64 discovery is prone to DNS spoofing attacks.
In addition to creating a wider attack surface for IPv6 deployments, [RFC7050] has other security challenges worth noting to justify declaring it legacy.

### Definition of secure channel {#secure-channel-def}

[RFC7050] requires a node's communication channel with a DNS64 server to be a "secure channel" which it defines to mean "a communication channel a node has between itself and a DNS64 server protecting DNS protocol-related messages from interception and tampering." This need is redundant when another communication mechanism of IPv6-related configuration, specifically Router Advertisements, can already be defended against tampering by RA Guard [RFC6105]. Requiring nodes to implement two defense mechanisms when only one is necessary when [RFC8781] is used in place of [RFC7050] creates unnecessary risk.

### Secure channel example of IPsec

One of the two examples [RFC7050] defines to qualify a communication channel with a DNS64 server is the use of an "IPsec-based virtual private network (VPN) tunnel". As of the time of this writing, this is not supported as a practice by any common operating system DNS client. While they could, there have also since been multiple mechanisms defined for performing DNS-specific encryption such as those defined in [RFC9499] that would be more appropriately scoped to the applicable DNS traffic. These are also compatible with encrypted DNS advertisement by the network using Discovery of Network-designated Resolvers [RFC9463] that would ensure the clients know in advance that the DNS64 server supported the encryption mechanism.

### Secure channel example of link layer encryption

The other example given by [RFC7050] that would allow a communication channel with a DNS64 server to qualify as a "secure channel" is the use of a "link layer utilizing data encryption technologies". As of the time of this writing, most common link layer implementations use data encryption already with no extra effort needed on the part of network nodes. While this appears to be a trivial way to satisfy this requirement, it also renders the requirement meaningless since any node along the path can still read the higher-layer DNS traffic containing the translation prefix. This seems to be at odds with the definition of "secure channel" as explained in {{Section 2.2 of RFC7050}}.

# Recommendations for PREF64 Discovery

## Deployment Recommendations

Operators deploying NAT64 networks SHOULD provide PREF64 information in Router Advertisements as per [RFC8781].

## Clients Implementation Recommendations

Clients SHOULD obtain PREF64 information from Router Advertisements as per [RFC8781] instead of using [RFC7050] method.
In the absense of the PREF64 information in RAs, a client MAY choose to fall back to RFC7050.

# Security Considerations

Obtaining PREF64 information from Router Advertisements improves the overall security of an IPv6-only client as it mitigates all attack vectors related to spoofed or rogue DNS response, as discussed in Section 7 of [RFC7050].
Security considerations related to obtaining PREF64 information from RAs are discussed in Section 7 of [RFC8781].

# IANA Considerations

It is expected that there will be a long tail of both clients and networks  still relying on [RFC7050] as a sole mechanism to discover PREF64 information.
Therefore IANA still need to maintain "ipv4only.arpa." as described in [RFC7050] and this document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
