---
title: "Extensions to BIER Tree Engineering (BIER-TE) for Large Multicast Domains and 1:1 Protection"
abbrev: "bierte-subset-tunneling"
category: std

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
However, a bitstring in BIER-TE hold bits for BFERs and links, which effects that the size of supportable topologies in terms of BFERs is smaller for BIER-TE than for BIER.
While for BIER, subsets can be utilized to improve scaling to large multicast domains, this mechanism requires adaptation for BIER-TE.
In this document, requirements for BIER-TE subsets are defined, a mechanism is described for how BIER-TE packets can reach their subsets using tunneling, and 1:1 protection for the entire BIER-TE multicast tree is explained.
The concept utilizes egress protection for reaching a subset when the ingress BFIR of the subset fails.

--- middle

# Introduction

Bit Index Explicit Replication with Traffic Engineering (BIER-TE) was introduced in {{!RFC9262}} as an extension to the BIER archtecture, offering enhanced capabilities through traffic engineering, or "tree engineering" in this context.
In BIER-TE, the forwarding and replication of multicast traffic is guided by the BIER-TE header that includes bits representing Bit Forwarding Egress Routers (BFERs) and links.
As a result, the entire multicast distribution tree is encoded directly within the packet header.
This encoding leads to scalability challenges, particularly in large networks, which are similar as in BIER.
However, BIER-TE encodes more information in the bistring and thus scales worse than BIER for large networks.

BIER-TE inherits the concept of subsets from BIER where the network domain can be partitioned into smaller sub-topologies.
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
BIER subsets can be constructed by selecting a number of arbitrary BFERs within the BIER domain. The number of BFERs of each subset must not exceed the bitstring length and the BFERs of all subsets should cover all BFERs of the entire domain. This rather unconstraint selection of BFERs works well because a packet sent by a BIER BFIR reaches any BFER over the routing underlay, no matter which subset the BFER belongs to.

In contrast, with BIER-TE, a BFER can be reached over a path that is encoded in the bitstring. Therefore, also links need to be part of the subset. Moreover, within a very large ring or a line, a BFIR may be so far away from a BFER that the links of the subset do not suffice to define a path from the BFIR to that BFER.

Therefore, a different approach is needed for subsets in BIER-TE. First, a subset also needs links in additions to BFERs which are encoded in the bitstring. The number of BFERs and links within the subset MUST NOT exceed the size of the bitstring. Moreover, links and BFERs need to be connected so that a BIER-TE packet at a source of any link in the subset can reach all BFERs within the subset. The BFRs belonging to the links are also part of the subset but are not represented by a bit in the bitstring unless they constitute BFERs. Thus, a subset needs to be one-connected. That is, the mentioned property is fulfilled unless one link or BFR fails.
A BFIR may need to send a packet to a specific BFER, but it does not need to be part of the subset the BFER belongs to. It can however use unicast tunneling to direct packets to some BFR within the subset. That BFR is denoted as S-BFIR (subset BFIR). There may be one or many S-BFIRs for a subset.

Subsets for BIER-TE with 1:1 protection have to be at least two-connected, i.e., the remaining subset is one-connected if a link or BFR fails. That is, in case of a failure, there is still a path from any BFR to any BFER within the subset. Further, for every S-BFIR in the subset, one or more backup S-BFIRs have to be assigned. When an S-BFIR fails, the backup S-BFIRs serve as alternative ingress to the subset and ensure that the packets reach the BFERs.

A domain may be one- or two connected, but it may be difficult or even impossible to find a desired number of subsets with that property. Then, virtual links may be defined as tunnels whose paths extend over links outside the subset.


# Subset Tunneling
Any BFIR reaches each subset by tunneling packets to any S-BFIR in the subset.
To that end, the BFIR forwarding logic has to be slightly modified from the one defined in {{!RFC8279}}.
When a BFIR receives an IPMC packet, it determines the destination BFERs and their subsets as with standard BIER-TE.
The BFIR clones the packet for each subset and adds the corresponding BIER-TE header to each packet.
The BIER-TE bitstring contains the multicast tree from S-BFIR to the BFERs in the same subset.
Then, the BFIR adds an outer tunneling header that reaches one of the S-BFIRs of the destination subset.

~~~~
{::include ./drawings/subset-tunneling-concept.txt}
~~~~
{: #fig-subset-tunneling-concept title="Example BIER-TE network with a one-connected BIER-TE subset, S-BFIR, and MPLS tunnel."}

When an S-BFIR receives such a tunneled packet, it removes the tunnel header and applies BIER-TE forwarding.
The tunneling protocols used SHOULD support path steering, e.g., MPLS TE or SRv6.
Further, they SHOULD support FRR and egress protection mechanisms for 1:1 protection.


# 1:1 Protection
The protection of BIER-TE with subsets is split into three sections: during tunneling, at the subset ingress, and within the subset.

## Subset Tunneling Protection
The tunnel between BFIR and S-BFIR may be protected using existing FRR 1:1 protection mechanisms.
RSVP-TE with FRR {{?RFC8796}} can be used for MPLS and {{?I-D.draft-ietf-rtgwg-segment-routing-ti-lfa}} for FRR protection in SRv6.

## Subset Ingress Protection
When an S-BFIR fails, the packet has to be rerouted to a backup S-BFIR.
To protect against this failure, the tunneling protocol must support an egress protection mechanism.
If MPLS is used for the tunnel, then the MPLS Egress Protection Framework {{?RFC8679}} can be used.
For SRv6 egress protection there is a similar draft {{?I-D.draft-ietf-rtgwg-srv6-egress-protection}}.

When the penultimate tunneling hop detects the failure of the S-BFIR, it becomes the PLR.
On a detected failure, the PLR determines a backup S-BFIR of the same subset.
If there are multiple backup S-BFIR, the PLR may select a backup S-BFIR based on different criteria, e.g., the closest backup S-BFIR.
The PLR uses the egress protection mechanism of the tunneling protocol to reroute the packet to the backup S-BFIR.

~~~~
{::include ./drawings/ingress-protection-concept.txt}
~~~~
{: #fig-ingress-protection-concept title="MPLS example of a backup S-BFIR and a one-connected subset."}

The backup S-BFIR recognizes its role as backup ingress based on the tunneling protocol header.
It removes the tunneling header and applies BIER-TE FRR {{!I-D.draft-eckert-bier-te-frr}} with node protection.
This ensures that the packet is forwarded to the correct next hops of the failed S-BFIR.
It also modifies the BIER-TE bitstring to prevent loops in the subset.
After the backup S-BFIR processing, standard BIER-TE forwarding is applied.

## Intra-Subset Protection
Protection against failures inside a subset is achieved with existing BIER-TE FRR mechanisms.
There are currently two drafts for BIER-TE FRR {{!I-D.draft-eckert-bier-te-frr}} and {{!I-D.draft-chen-bier-te-frr}}.
Both define mechanisms for link and node protection and propose BIER-TE-in-BIER-TE tunneling to reroute packets around a failure.
Therefore, either of these drafts may be used for the FRR protection mechanisms.

# Examples
In this section, we present two exemplary scenarios.
One for the subset tunneling and one for the ingress protection mechanism.

## Subset Tunneling
An example topology using BIER-TE subset tunneling is illustrated in {{fig-subset-tunneling}}.
The BFIR determines that the packet needs to be delivered to BFERs 1 and 2.
It recognizes that the two BFERs lie within two different subsets and clones the packet.
The first packet receives a BIER-TE header with the bits set for link 0 and BFER 1 and is tunneled to the S-BFIR of subset 1.
The second packet receives a BIER-TE header with the bits set for link 1 and BFER 2 and is tunneled to subset 2.
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
