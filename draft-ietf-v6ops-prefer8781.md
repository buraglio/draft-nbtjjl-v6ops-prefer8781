---
title: "Recommendations for Discovering IPv6 Prefix Used for IPv6 Address Synthesis"
abbrev: "Prefer RFC8781"
category: info

docname: draft-ietf-v6ops-prefer8781-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: v6ops
keyword:
 - IPv6
 - DNS64
 - PREF64
 - NAT64
 - CLAT
venue:
  group: v6ops
  type: Working Group
  mail: v6ops@ietf.org
  arch: https://datatracker.ietf.org/wg/v6ops/about/
  github: "buraglio/draft-nbtjjl-v6ops-prefer8781"
  latest: "https://github.com/buraglio/draft-nbtjjl-v6ops-prefer8781"

author:
 -
    fullname: Nick Buraglio
    organization: Energy Sciences Network
    email: "buraglio@forwardingplane.net"
 -
    fullname: Tommy Jensen
    email: "tojens.ietf@gmail.com"
 -
    fullname: Jen Linkova
    organization: Google
    email: "furry13@gmail.com"
normative:
  RFC7050:
  RFC8781:

informative:
  RFC4861:
  RFC4862:
  RFC6105:
  RFC6052:
  RFC6144:
  RFC6146:
  RFC7225:
  RFC7915:
  RFC6147:
  RFC6877:
  RFC7051:
  RFC8880:
  RFC9463:
  RFC9499:
  RFC8305:
  I-D.ietf-v6ops-claton:

--- abstract

On networks providing IPv4-IPv6 translation (RFC7915), hosts and other endpoints need to know the IPv6 prefix(es) used for translation (the NAT64 prefix, RFC6052). This document provides guidelines for NAT64 prefix discovery, specifically recommending obtaining the NAT64 prefix from Router Advertisement option (RFC8781) when available.

--- middle

# Introduction

Devices translating between IPv4 and IPv6 packet headers [RFC7915] use a NAT64 prefix to map IPv4 addresses into the IPv6 address space, and vice versa.
When a network provides NAT64, it is advantageous for endpoints to acquire the network's NAT64 prefixes (PREF64).
Discovering the PREF64 enables endpoints to:

  * Implement the customer-side translator (CLAT) function of the 464XLAT architecture [RFC6877].
  * Translate IPv4 literals to IPv6 literals (Section 7.1 of [RFC8305]).
  * Perform local DNS64 [RFC6147] functions.
  * Support applications relying on IPv4 address referral (Section 3.2.2 of [RFC7225]).

Dynamic PREF64 discovery is useful to keep the NAT64 prefix configuration up-to-date, particularly for unmanaged endpoints or endpoints which move between networks.
[RFC7050] introduces the first DNS64-based mechanism for PREF64 discovery based on [RFC7051] analysis.
However, subsequent methods have been developed to address [RFC7050] limitations.

For instance, [RFC8781] defines a Neighbor Discovery [RFC4861] option for Router Advertisements (RAs) to convey PREF64 information to hosts.
This approach offers several advantages (Section 3 of [RFC8781]), including fate sharing with other host network configuration parameters.

Due to fundamental shortcomings of the [RFC7050] mechanism ({{issues}}), [RFC8781] is the preferred solution for new deployments.
Implementations should strive for consistent PREF64 acquisition methods.
The DNS64-based mechanism of [RFC7050] should be employed only when RA-based PREF64 delivery is unavailable, or as a fallback for legacy systems incapable of processing the PREF64 RA Option.

# Terminology

DNS64: a mechanism for synthesizing AAAA records from A records, defined in [RFC6147].

NAT64: a mechanism for translating IPv6 packets to IPv4 packets and vice versa.  The translation is done by translating the packet headers according to the IP/ICMP Translation Algorithm defined in [RFC7915]. NAT64 translators can operate in stateful ([RFC6144]) or stateless mode (e.g. customer-side translator, CLAT, [RFC6877]).
This document uses "NAT64" as a generalized term for a translator which uses the stateless IP/ICMP translation algorithm defined in [RFC7915] and operates within a framework for IPv4/IPv6 translation described in [RFC6144].

PREF64 (or Pref64::/n, or NAT64 prefix): An IPv6 prefix used for IPv6 address synthesis and for network addresses and protocols translation from IPv6 clients to IPv4 servers using the algorithm defined in [RFC6052].

Router Advertisement (RA): A packet used by Neighbor Discovery [RFC4861] and SLAAC to advertise the presence of the routers, together with other IPv6 configuration information.

SLAAC:  StateLess Address AutoConfiguration, [RFC4862]

{::boilerplate bcp14-tagged}

# Recommendations for PREF64 Discovery

## Deployment Recommendations for Endpoints

Endpoints SHOULD attempt to obtain PREF64 information from RAs per [RFC8781] instead of using [RFC7050] method.
In the absence of the PREF64 information in RAs, an endpoint MAY choose to fall back to the mechanism defined in RFC7050.
This recommendation to prefer the [RFC8781] mechanism over one defined in [RFC7050] is consistent with Section 5.1 of [RFC8781].

## Deployment Recommendations for Operators

Network operators deploying NAT64 SHOULD provide PREF64 information in Router Advertisements per [RFC8781].

### Mobile Network Considerations

While [RFC8781] support is widely integrated into modern operating systems on mobile endpoints, equipment deployed in mobile network environments often lacks abilities to include the PREF64 Option into RAs.
Therefore, the immediate deployment and enablement of PREF64 by mobile operators may not currently be feasible and the recommendations outlined in this document are not presently applicable to mobile network operators.
These environments are encouraged to incorporate [RFC8781] when made practical by infrastructure upgrades or software stack feature additions.

### Migration Considerations

Transitioning from the [RFC7050] heuristic to using the [RFC8781] approach might require a period of time where both mechanisms coexist.
How long this may take depends on the endpoint footprint, particularly the presence and number of endpoints running outdated operating systems, which do not support [RFC8781].
Operators are advised to take those factors into account prior to removing support for the [RFC7050] heuristic, noting that it is still safe to add support for the [RFC8781] approach since endpoints that support it will always prefer it over [RFC7050] if they follow RFC requirements.

Migrating away from DNS64-based discovery also reduces dependency on DNS64 in general, thereby eliminating DNSSEC and DNS64 incompatibility concerns (Section 6.2 of [RFC6147]).

# Existing Issues with RFC7050 {#issues}

DNS-based discovery the NAT64 prefix introduces some challenges, which make this approach less preferable than latest developed alternatives (such as PREF64 RA Option, [RFC8781]).
This section outlines the key issues, associated with [RFC7050], with a focus on those not discussed in [RFC7050] or in the analysis of solutions for hosts to discover NAT64 prefix ([RFC7051]).

Signalling PREF64 in RA option addresses all issues outlined in this section (see Section 3 of [RFC8781] for details).

## Dependency on Network-Provided Recursive Resolvers

Fundamentally, the presence of the NAT64 and the exact value of the prefix used for the translation are network-specific attributes.
Therefore, [RFC7050] requires the endpoint discovering the prefix to use the DNS64 resolvers provided by the network.
If the device or an application is configured to use other recursive resolvers or runs a local recursive resolver, the corresponding name resolution APIs and libraries are required to recognize 'ipv4only.arpa.' as a special name and give it special treatment.
This issue and remediation approach are discussed in [RFC8880].
However, it has been observed that very few [RFC7050] implementations support [RFC8880] requirements for special treatment of 'ipv4only.arpa.'.
As a result, configuring such systems and applications to use resolvers other than the one provided by the network breaks the PREF64 discovery, leading to degraded user experience.

VPN applications may override the endpoint's DNS configuration, for example, by configuring enterprise DNS servers as the node's recursive resolvers and forcing all name resolution through the VPN.
These enterprise DNS servers typically lack DNS64 functionality and therefore cannot provide information about the PREF64 used within the local network.
If the VPN is configured in so-called "split tunneling" mode (when only a subset of network traffic is routed into the VPN tunnel), endpoints may not discover the necessary PREF64, which negatively impacts their connectivity on IPv6-only networks.

If both the network-provided DNS64 and the endpoint's resolver happen to utilize the Well-Known Prefix (64:ff9b::/96, [RFC6052]), the endpoint would end up using a PREF64 that's valid for the current network.
However, if the endpoint changes its network attachment, it can't detect if the new network lacks NAT64 entirely or uses a network-specific NAT64 prefix (NSP, [RFC6144]).

Signalling PREF64 in RA option decouples the PREF64 discovery from the host's DNS resolvers configuration.

## Network Stack Initialization Delay

When using SLAAC, an IPv6 host typically requires a single RA to acquire its network configuration.
For IPv6-only endpoints, timely PREF64 discovery is critical, particularly for those performing local DNS64 or NAT64 functions, such as CLAT ([RFC6877]).
Until a PREF64 is obtained, the endpoint's IPv4-only applications and communication to IPv4-only destinations are impaired.
The mechanism defined in [RFC7050] does not bundle PREF64 information with other network configuration parameters, and requires at least one round-trip time (to send a DNS request and receive a response) after the network stack configuration is completed.

Advertising PREF64 in RA, on the other hand, elminates the period when the host obtains IPv6 addresses and default routers, but no PREF64.

## Latency in Updates Propagation

Section 3 of [RFC7050] states: "The node SHALL cache the replies it receives during the Pref64::/n discovery procedure, and it SHOULD repeat the discovery process ten seconds before the TTL of the Well-Known Name's synthetic AAAA resource record expires."
As a result, once a PREF64 is discovered, it will be used until the TTL expired, or until the node disconnects from the network.
There is no mechanism for an operator to force the PREF64 rediscovery on the node without disconnecting the node from the network.
If the operator needs to change the PREF64 value used in the network, they need to proactively reduce the TTL value returned by the DNS64 server.
This method has two significant drawbacks:

*  Many networks utilize external DNS64 servers and therefore have no control over the TTL value, if the PREF64 needs to be changed or withdrawn.
*  The PREF64 changes need to be planned and executed at least TTL seconds in advance. If the operator needs to notify nodes that a particular prefix must not be used (e.g. during a network outage or if the nodes learnt a rogue PREF64 as a result of an attack), it might not be possible without interrupting the network connectivity for the affected nodes.

Mechanism defined in [RFC8781] allows to notify hosts about PREF64 changes immidiately, by sending an RA with updated information.

## Multihoming Implications

Section 3 of [RFC7050] requires a node to examine all received AAAA resource records to discover one or more PREF64s and to utilize all learned prefixes.
However, this approach presents challenges in some multihomed topologies where different DNS64 servers belonging to different ISPs might return different PREF64s.
In such cases, it is crucial that traffic destined for synthesized addresses is sent to the correct NAT64 and the source address selected for those flows belongs to the prefix from that ISP's address space.
In other words, the node needs to associate each discovered PREF64 with upstream information, including the IPv6 prefix and default gateway.
Currently, there is no reliable way for a node to map a DNS64 response (and the prefix learned from it) to a specific upstream in a multihoming scenario.
Consequently, the node might inadvertently select an incorrect source address for a given PREF64 and/or send traffic to the incorrect uplink.

Advertising PREF64 in RAs allows hosts to track which PREF64 was advertised by which router and use that information to select the correct nexthop.
Section 8 of [I-D.ietf-v6ops-claton] discusses this scenario in more details.

## Security Implications

As discussed in Section 7 of [RFC7050], the DNS-based PREF64 discovery is prone to DNS spoofing attacks.
In addition to creating a wider attack surface for IPv6 deployments, [RFC7050] has other security challenges, which are discussed below.

### Definition of Secure Channel {#secure-channel-def}

[RFC7050] requires a node's communication channel with a DNS64 server to be a "secure channel" which it defines to mean "a communication channel a node has between itself and a DNS64 server protecting DNS protocol-related messages from interception and tampering."
This need is redundant when another communication mechanism of IPv6-related configuration, specifically RAs, can already be defended against tampering, for example by enabling RA-Guard [RFC6105].
Requiring nodes to implement two defense mechanisms when only one is necessary when [RFC8781] is used in place of [RFC7050] creates unnecessary risk.

### Secure Channel Example of IPsec

One of the two examples that [RFC7050] defines to qualify a communication channel with a DNS64 server is the use of an "IPsec-based virtual private network (VPN) tunnel". As of the time of this writing, this is not supported as a practice by any common operating system DNS client. While they could, there have also since been multiple mechanisms defined for performing DNS-specific encryption such as those defined in [RFC9499] that would be more appropriately scoped to the applicable DNS traffic. These are also compatible with encrypted DNS advertisement by the network using Discovery of Network-designated Resolvers [RFC9463] that would ensure the clients know in advance that the DNS64 server supported the encryption mechanism.

### Secure Channel Example of Link Layer Encryption

The other example given by [RFC7050] that would allow a communication channel with a DNS64 server to qualify as a "secure channel" is the use of a "link layer utilizing data encryption technologies". As of the time of this writing, most common link layer implementations use data encryption already with no extra effort needed on the part of network nodes. While this appears to be a trivial way to satisfy this requirement, it also renders the requirement meaningless since any node along the path can still read the higher-layer DNS traffic containing the translation prefix. This seems to be at odds with the definition of "secure channel" as explained in {{Section 2.2 of RFC7050}}.

# Security Considerations

Obtaining PREF64 information using RAs improves the overall security of an IPv6-only endpoint as it mitigates all attack vectors related to spoofed or rogue DNS response, as discussed in Section 7 of [RFC7050].
Security considerations related to obtaining PREF64 information from RAs are discussed in Section 7 of [RFC8781].

# IANA Considerations

This document does not introduce any IANA considerations.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the following people for their valuable contributions: Mike Bishop, Mohamed Boucadair, Lorenzo Colitti, Tom Costello, Charles Eckel, Susan Hares, Nick Heatley, Gabor Lencse, Ted Lemon, David Lou, Peter Schmitt, Éric Vyncke, Chongfeng Xie.
