---
title: "Extensions to BIER Tree Engineering (BIER-TE) for Large Multicast Domains and 1:1 Protection"
abbrev: "bierte-subset-extensions"
category: info

docname: draft-fluechter-bier-bierte-subset-tunneling-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Bit Indexed Explicit Replication"
keyword:
 - multicast
 - resilience
venue:
  group: "Bit Indexed Explicit Replication"
  type: "Working Group"
  mail: "bier@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/bier/"
  github: "uni-tue-kn/draft-bier-bierte-subset-tunneling"
  latest: "https://uni-tue-kn.github.io/draft-bier-bierte-subset-tunneling/draft-fluechter-bier-bierte-subset-tunneling.html"

author:
 -
    fullname: "Moritz Flüchter"
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: "moritz.fluechter@uni-tuebingen.de"
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: michael.menth@uni-tuebingen.de
 -
    fullname: Toerless Eckert
    organization: Futurewei USA
    email: tte@cs.fau.de

normative:

informative:


--- abstract

BIER-TE extends BIER with tree engineering capabilities and can be supported by almost the same hardware.
However, a bitstring in BIER-TE hold bits for BFERs and links, which effects that the size of supportable networks in terms of BFERs is smaller for BIER-TE than for BIER.
While for BIER, subsets can be utilized to improve scaling to large multicast domains, this mechanism requires adaptation for BIER-TE.
In this document, requirements for BIER-TE subsets are defined, a mechanism is described for how BIER-TE packets can reach their subsets using tunneling, and 1:1 protection for the entire BIER-TE multicast tree is explained.
The concept utilizes egress protection for reaching a subset when the ingress BFIR of the subset fails.

--- middle

# Introduction

Bit Index Explicit Replication with Traffic Engineering (BIER-TE) was introduced in {{!RFC9262}} as an alternative to BIER multicast, offering enhanced capabilities through traffic engineering, or "tree engineering" in this context.
In BIER-TE, the forwarding of multicast traffic is guided by the BIER-TE header that includes bits representing Bit Forwarding Egress Routers (BFERs) and network links.
As a result, the entire multicast distribution tree is encoded directly within the packet header.
This encoding leads to scalability challenges, particularly in large network domains which are similar as in BIER.
However, BIER-TE encodes more information in the bistring and thus scales worse than BIER for large networks.

BIER-TE inherits the concept of subsets from BIER where the network domain can be partitioned into smaller segments.
A BIER-TE subset is a partition of the BIER-TE domain with a unique Set Identifier (SI).
The SI containing the destination BFERs of a packet is indicated in the BIER-TE header as a bitstring.
Further, the bitstring addresses only links and BFERs in that subset.

In this document, requirements for BIER-TE subsets are defined, a mechanism is described for how BIER-TE packets can reach their subsets using tunneling.
Further the application of 1:1 protection for the entire multicast tree is explained.
With these extensions, BIER-TE can scale to large multicast domains.
The extensions depend on well established technologies and do not require any changes to the BIER header or forwarding logic.

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{!RFC9262}} and in {{?RFC8679}}.

Further abbreviations used in this document:

| Abbreviation |  Meaning                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|   S-BFIR    |  Subset Bit Forwarding Ingress Router |  This document
{: #table_abbrev title="Abbreviations."}

# BIER-TE Subsets
In this section, limitations of the current BIER-TE subset definition are described.
Based on these shortcomings, a new BIER-TE subset concept is proposed.

## Need for BIER-TE Subsets
BIER subsets can be constructed by arbitrarily selecting BFERs in the domain.
The BIER BFIR can always reach any BFER, no matter which subset they are located in.
This does not hold for BIER-TE subsets as the BIER-TE header also encodes the links to traverse in the bitstring.
Therefore, the BFERs in a subset have to be sufficiently connected by links in the same subset.
BFIRs that are not part of such a subset cannot send packets to the BFERs in the subset directly.
Rather, they have to tunnel packets to a BFR in the subset which then applies BIER-TE forwarding again.

## Subsets for BIER-TE
The following constraints MUST be fullfilled by BIER-TE subsets.
A BIER-TE subset MUST be at least one-connected, i.e., there has to be at least one path from any BFR to any BFER in the subset.
Further, the combined number of BFERs and links in a subset MUST NOT be larger than the maximum supported BitStringLength.
There is also the posbility that a one-connected domain does not contain suitable one-connected subsets.
To solve this, tunnels can be assigned as virtual links in the subset.
These tunnels leverage the routing layer and may forward the packet over links that are not in the subset.

Every BFR that can reach any BFER in a subset over links of the subset is considered a part of the subset.
These BFRs may serve as ingress nodes for the subset and tunneling endpoints to deliver packets to the BFERs of a subset.
BFRs that receive tunneled packets from a BFIR are referred to as Subset BFIRs (S-BFIR).

## Subsets for BIER-TE with 1:1 Protection
Subsets for BIER-TE with 1:1 protection have to be at least two-connected, i.e., there have to be at least two paths from any BFR to any BFER in the subset.
When a link or node failes, the remaining subset is still one-connected.
Further, for every S-BFIR in the subset, one or more backup S-BFIRs have to be assigned.
When an S-BFIR fails, the backup S-BFIRs serve as alternative ingress to the subset and ensure that the packets reach the BFERs.

# Subset Tunneling
The BFIR reaches each subset by tunneling packets to any S-BFIR in the subset.
To that end, the BFIR forwarding logic has to be slightly modified from the one defined in {{!RFC8279}}.
When a BFIR receives an IPMC packet, it determines the destination BFERs and their subsets as with standard BIER-TE.
The BFIR clones the packet for each subset and adds the corresponding BIER-TE header to each packet.
The BIER-TE bitstring contains the multicast tree from S-BFIR to the BFERs in the same subset.
Then, the BFIR adds an outer tunneling header that reaches one of the S-BFIRs of the destination subset.
When an S-BFIR receives such a tunneled packet, it removes the tunnel header and applies BIER-TE forwarding.
Used tunneling protocols SHOULD support traffic engineering, e.g., MPLS TE or SRv6.
Further, they SHOULD support FRR and egress protection mechanisms for 1:1 protection.

# 1:1 Protection
The protection of BIER-TE with subsets is split into three sections: within the subset, at the subset ingress and during tunneling.

## Intra-Subset Protection
Protection against failure inside a subset is achieved with existing BIER-TE FRR mechanisms.
There are currently two drafts for BIER-TE FRR {{?I-D.draft-eckert-bier-te-frr}} and {{?I-D.draft-chen-bier-te-frr}}.
Both define mechanisms for link and node protection and propose BIER-TE-in-BIER-TE tunneling to reroute packets around a failure.
Therefore, either of these drafts may be used for the FRR protection mechanisms.

## Subset Ingress Protection
When an S-BFIR fails, the packet has to be rerouted to a backup S-BFIR.
To protect against this failure, the tunneling protocol must support an egress protection mechanism.
If MPLS is used for the tunnel, then the MPLS Egress Protection Framework {{?RFC8679}} can be used.
For SRv6 egress protection there is a similar draft {{?I-D.draft-ietf-rtgwg-srv6-egress-protection}}.

When the penultimate tunneling hop detects the failure of the S-BFIR, it becomes the PLR.
On a detected failure, the PLR determines a backup S-BFIR of the same subset.
If there are multiple backup S-BFIR, the PLR may select a backup S-BFIR based on different criteria, e.g., the closest backup S-BFIR.
The PLR uses the egress protection mechanism of the tunneling protocol to reroute the packet to the backup S-BFIR.

The backup S-BFIR recognizes its role as backup ingress based on the tunneling protocol header.
It removes the tunneling header and applies BIER-TE FRR with node protection.
This ensures that the packet is forwarded to the correct next hops of the failed S-BFIR.
It also modifies the BIER-TE bitstring to prevent loops in the subset.
After the backup S-BFIR processing, standard BIER-TE forwarding is applied.

## Subset Tunneling Protection
The tunnel between BFIR and S-BFIR may be protected using existing FRR 1:1 protection mechanisms.
RSVP-TE with FRR {{?RFC8796}} can be used for MPLS and {{?I-D.draft-ietf-rtgwg-segment-routing-ti-lfa}} for FRR protection in SRv6.

# Examples
In this section, we present two exemplary scenarios.
One for the subset tunneling and one for the ingress protection mechanism.

## Subset Tunneling
An example topology using BIER-TE subset tunneling is illustrated in {{fig-subset-tunneling}}.
The BFIR determines that the packet needs to be delivered to BFERs 1 and 2.
It recognizes that the two BFERs lie within two different subsets and clones the packet.
The first packet receives a BIER-TE header with the bits set for link 0 and BFER 1 and is tunneled to the S-BFIR of subset 1.
The second packet recieves a BIER-TE header with the bits set for link 1 and BFER 2 and is tunneled to subset 2.
When the packet arrives at each S-BFIR, the S-BFIR removes the MPLS tunnel header.
For subset 1, it then matches the BIER bitstring and forwards the packet over link 0.
The S-BFIR of subset to forwards the packet over link 1.

~~~~
{::include ./drawings/subset-tunneling.txt}
~~~~
{: #fig-subset-tunneling title="Example BIER-TE network with two subsets and S-BFIRs."}

## Ingress Protection with MPLS Tunneling
{{fig-ingress-protetion}} illustrates the concept of ingress protection with MPLS.
When the tunneled BIER-TE packet arrives at the LSR B, it detects that S-BFIR A failed.
Using MPLS egress protection, the PLR (LSR B) switches the label to the context label C' which indicates the backup path to S-BFIR'.
S-BFIR' recognizes its role as protector by matching the context label to the failed ingress node.
Then, it applies the BIER-TE FRR node protection handling, unsets the bit for link 1 and sets the bit for link 0.
This way, the packet reaches the BFR and can be forwarded with standard BIER-TE.


~~~~
{::include ./drawings/ingress-protection.txt}
~~~~
{: #fig-ingress-protetion title="Example BIER-TE network with a single subset and two S-BFIRs."}


# Security Considerations

The security issues discussed in {{?RFC8279}}, {{?RFC9262}}, and {{?RFC8679}} apply to this document.


# IANA Considerations

This document has no IANA actions.


--- back
