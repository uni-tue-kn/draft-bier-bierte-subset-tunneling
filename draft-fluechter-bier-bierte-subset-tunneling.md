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
However, this encoding leads to scalability challenges, particularly in large network domains which are similar as in BIER.
The size of the BIER-TE domain is constrained by the maximum possible header length, limiting the usability of BIER-TE in large multicast domains.

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

## Limitation of current BIER-TE subsets
BIER-TE packets can only be forwarded over links that are in the same subset as the destination BFERs.
Therefore, all links of the multicast tree from BFIR to BFERs have to be in the same subset.
This is further referred to as 1-connectedness of a subset, meaning that any forwarding node in the subset has at least one path to any other node in the subset.
The BFIR is also regarded as a part of the subset for 1-connectedness.

For 1:1 protection and tree engineering more than one path between two nodes is needed.
When a node or link fails, the remaining subset has to still be 1-connected.
Therefore, a BIER-TE subset has to be at least 2-connected, meaning that there are at least two paths between any two forwarding nodes.

This constraint is hard to fulfill in large multicast domains due to limitations of the maximum size of the BIER-TE bitstring.
It is not necessarily possible to select 2-connected subsets in a large multicast domain that are sufficiently small.

## Proposed BIER-TE Subset Concept
Smaller BIER-TE subsets are only possible if the BFIR does not have to be directly connected to a BIER-TE subset.
Rather, only the BFERs in a subset have to be 2-connected with each other.
This also requires a new way with which the BFIR can forward packets to a subset as it cannot reach them with native BIER-TE anymore.
This draft introduces Subset BFIRs (S-BFIR) which are assigned to each subset.
The S-BFIRs serve as ingress node into the subset and as tunneling endpoint for the BFIR.
The BFIR distributes packets to the subsetws by tunneling a packet to the corresponding S-BFIRs.

## Subset Selection Constraints
For the selection of such a BIER-TE subset, the following constraints MUST be fulfilled:

- Maximum BitStringLength: A subset cannot be larger than the maximum supported BitStringLength in the domain.

- 2-connectedness: To leverage BIER-TE FRR as defined in {{?I-D.draft-eckert-bier-te-frr}} the subset has to be at least 2-connected.

- Backup ingresses: For 1:1 protection, a subset MUST have at least two S-BFIRs. Each serves as a backup S-BFIR if the other fails.

- Virtual links: 2-connectedness may still not be possible for a subset in, e.g., a ring ropology. Therefore, additional virtual links can be assigned between two nodes via a BIER-TE routed connection.

# Subset Tunneling
The BFIR reaches each subset by tunneling packets to the corresponding S-BFIR.
To that end, the BFIR forwarding logic has to be slightly modified from the one defined in {{!RFC8279}}.
When a BFIR receives an IPMC packet, it determines the destination BFERs and their subsets as with standard BIER-TE.
The BFIR clones the packet for each subset and adds the corresponding BIER-TE header to each packet.
The BIER-TE bitstring contains the multicast tree from S-BFIR to the BFERs in the same subset and is computed by the BIER-TE controller.
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
