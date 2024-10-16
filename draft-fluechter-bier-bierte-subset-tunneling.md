---
title: "Subset Tunneling and Resilience Concept for BIER-TE"
abbrev: "STARC"
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

normative:

informative:


--- abstract

BIER-TE extends BIER with tree engineering capabilities while still supporting the same hardware.
However, there is currently no appropriate concept to scale BIER-TE to large multicast networks.

This document defines the Subset Tunneling and Resilience Concept (STARC) for BIER-TE for large multicast domains and 1:1 protection.
It leverages the well established mechanisms MPLS, MPLS egress protection, and BIER-TE FRR.


--- middle

# Introduction

Bit Index Explicit Replication with Traffic Engineering (BIER-TE) was introduced as an alternative to BIER multicast, offering enhanced capabilities through traffic engineering, or "tree engineering" in this context. 
In BIER-TE, the forwarding of multicast traffic is guided by the BIER-TE header, which includes bits that represent both Bit Forwarding Egress Routers (BFERs) and network links.
As a result, the entire multicast distribution tree is encoded directly within the packet header.
However, this encoding leads to scalability challenges, particularly in large network domains. 
The size of the BIER-TE domain is constrained by the maximum possible header length, limiting the usability of BIER-TE in large multicast domains.

BIER-TE inherits the concept of subsets from BIER, where the network domain can be partitioned into smaller segments. 
A BIER-TE subset consists of one or more BFERs and the links connecting the BFERs with the BFIR.
BIER-TE subsets have to be at least 1-connected, since packets can only be forwarded over links in the subset indicated in the BIER-TE header.
For 1:1 protection and tree engineering, the subsets has to be at least 2-connected.

Therefore, BIER-TE subsets depend on the domain topology and cannot be selected arbitrarily.
The constraints for the current subsets definition are complex and make a subset selection algorithm difficult.
Further, the selection of a subset greatly influences the possible paths for tree engineering.

This document introduces the STARC (Subset Tunneling and Resilience Concept) for BIER-TE, designed to improve its scalability and performance in large multicast domains. 
STARC reduces BIER-TE subsets to only include links that connect BFERs with one another. 
Each subset MUST be assigned one or more Subset Bit Forwarding Ingress Routers (S-BFIRs), which act as ingress nodes for the subset. 
The Bit Forwarding Ingress Router (BFIR) tunnels multicast packets to an S-BFIR, where the packets are decapsulated and forwarded using standard BIER-TE mechanisms. 
In the event of an S-BFIR node failure, MPLS egress protection can be employed to switch traffic to a backup S-BFIR.
The extensions depend on well established technologies and do not require any changes to the BIER header or forwarding logic.

# Subset Tunneling and Resilience Concept for BIER-TE
In this section, we describe the concept of subset tunneling for BIER-TE and a 1:1 ingress protection mechanism.

## Subset Tunneling
A BIER-TE subset with subset tunnelign support consists of only BFERs and links connecting the BFERs.
The resulting subset MUST be at least 1-connected and MUST not be connected to a BFIR.
One or more BFRs that are connected to at least one of the subset links can then be assigned as Subset-BFIR (S-BFIR).

The proposed extension modifies the forwarding behaviour of a BFIR as specified in {{?RFC8279}} after the BIER-TE header is added.
Instead of forwarding the packet directly using BIER, the BFIR tunnels the packet to the S-BFIR of each destination subset over MPLS.
When an S-BFIR receives such an MPLS packet, it removes the tunnel header and resumes standard BIER forwarding.

## 1:1 Protection
For BIER-TE subset tunneling, there are three protection cases: the MPLS tunnel, the subset ingress and BIER forwrading within the subet.

### MPLS tunnel
The MPLS tunnel between BFIR and S-BFIR may use any protection mechanism such as RSVP-TE with FRR {{?RFC8796}}.

### Subset Ingress Protection
The ingress of a subset is protected using a combination of MPLS egress protection {{?RFC8679}} and BIER-TE FRR {{?I-D.draft-eckert-bier-te-frr}}.
The last LSR before an S-BFIR becomes the PLR when an S-BFIR fails.
It then uses MPLS egress protection to reroute the packet to a backup S-BFIR of the same subset which serves as protector.
The protector identifies the failed S-BFIR by matching on the context label.
It removes the MPLS tunnel header and applies the node protection mechanism of BIER-TE FRR with the failed node being the original S-BFIR.
This way, each next hop of the failed S-BFIR recieve the BIER-TE packet for forwarding.
After the protector S-BFIR handling, standard BIER forwarding resumes.

### Forwarding inside Subset
Within a BIER-TE subset, the standard BIER-TE FRR meachanism as proposed in {{?I-D.draft-eckert-bier-te-frr}} should be applied.

## Subset Selection Constraints
For the selection of a BIER-TE subset, the following constraints MUST be fulfilled:

- Bitstring max length: A subset cannot be larger than the maximum supported bistring length in the domain.

- 2-connectedness: To leverage BIER-TE FRR as defined in {{?I-D.draft-eckert-bier-te-frr}} the subset has to be at least 2-connected.
  
- Backup ingresses: For 1:1 protection, a subset hast to have at least two S-BFIRs. Each serves as a backup S-BFIR if the other fails.
  
- Virtual links: If 2-connectedness is not possible for a subset due to the topology, then virtual links over the routing underlay must be assigned.


# Examples
In this section, we present two exemplary scenarios.
One for the subset tunneling and one for the ingress protection mechanism.

## Subset Tunneling
An example topology using BIER-TE subset tunneling is illustrated in {{fig-subset-tunneling}}.
The BFIR determines that the packet needs to be delivered to the subset SI:1 and BFERs 1 and 2.
It adds the BIER-TE header to the packet and sets the bits for links 0,1,2 and BFERs 1,2.
Then, the BFIR tunnels the packet over MPLS to the S-BFIR of subset SI:1.
When the packet arrives at the S-BFIR, the S-BFIR removes the MPLS tunnel header.
It then matches the BIER bitstring and forwards the packet over link 0.
BFR A matches the bits of links 1,2 and forwards the packet to both BFER 1 and 2.

~~~~
{::include ./drawings/subset-tunneling.txt}
~~~~
{: #fig-subset-tunneling title="Example BIER-TE network with a single subset and S-BFIR."}

## Ingress Protection
{{fig-ingress-protetion}} illustrates the concept of ingress protection.
When the tunneled BIER-TE packet arrives at the LSR B, it detects that S-BFIR A failed.
Using MPLS egress protection, it switches the label to the context label C' which indicates the backup path to S-BFIR B.
S-BFIR B recognizes its role as protector by matching the context label to the failed ingress node.
Then, it applies the BIER-TE FRR node protection handling, unsets the bit for link 1 and sets the bit for link 0.
This way, the packet reaches BFR A and can be forwarded with standard BIER-TE.


~~~~
{::include ./drawings/ingress-protection.txt}
~~~~
{: #fig-ingress-protetion title="Example BIER-TE network with a single subset and two S-BFIRs."}


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
