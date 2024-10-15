# Extensions to BIER Tree Engineering (BIER-TE) for Large Multicast Domains and 1:1 Protection

# Abstract
BIER-TE extends BIER with tree engineering capabilities while still supporting the same hardware.

However, there is currently no appropriate concept to scale BIER-TE to large multicast networks.

This document defines extensions to BIER-TE for large multicast domains and 1:1 protection.
It leverages the well established mechanisms MPLS, MPLS egress protection and BIER-TE FRR.

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

This document introduces the STARC (Subset Tunneling and Resilience Concept) extension for BIER-TE, designed to improve its scalability and performance in large multicast domains. 
STARC reduces BIER-TE subsets to only include links that connect BFERs with one another. 
Each subset MUST be assigned one or more Subset Bit Forwarding Ingress Routers (S-BFIRs), which act as ingress nodes for the subset. 
The Bit Forwarding Ingress Router (BFIR) tunnels multicast packets to an S-BFIR, where the packets are decapsulated and forwarded using standard BIER-TE mechanisms. 
In the event of an S-BFIR node failure, MPLS egress protection can be employed to switch traffic to a backup S-BFIR, ensuring robust network operations.


# Subset Tunneling and Resilience Concept for BIER-TE

In this section, we describe the concept of subset tunneling for BIER-TE and a 1:1 ingress protection mechanism.

## Subset Tunneling



## Processing

## 1:1 Protection

## Subset Selection Constraints

# Example
In this section, we present two exemplary scenarios.
One for the subset tunneling and one for the ingress protection mechanism.

## Subset Tunneling
An example topology using BIER-TE subset tunneling is illustrated in Figure 4.
The BFIR determines that the packet needs to be delivered to the subset SI:1 and BFERs 1 and 2.
It adds the BIER-TE header to the packet and sets the bits for links 0,1,2 and BFERs 1,2.
Then, the BFIR tunnels the packet over MPLS to the S-BFIR of subset SI:1.
When the packet arrives at the S-BFIR, the S-BFIR removes the MPLS tunnel header.
It then matches the BIER bitstring and forwards the packet over link 0.
BFR A matches the bits of links 1,2 and forwards the packet to both BFER 1 and 2.

|                    BIER-TE domain                            |
                            |    BIER-TE subset (SI:1)         |
                                             /───1─── BFER 1
                                            /         
                                           / 
BFIR ───A─── LSR ───B─── S-BFIR A ───0─── BFR A ───2─── BFER 2
                                           \
                                            \
                                             \───3─── BFER 3
Figure 4: Example BIER-TE network with a single subset and S-BFIR.

## Ingress Protection
Figure 5 illustrates the concept of ingress protection.
When the tunneled BIER-TE packet arrives at the LSR B, it detects that S-BFIR A failed.
Using MPLS egress protection, it switches the label to the context label C' which indicates the backup path to S-BFIR B.
S-BFIR B recognizes its role as protector by matching the context label to the failed ingress node.
Then, it applies the BIER-TE FRR node protection handling, unsets the bit for link 1 and sets the bit for link 0.
This way, the packet reaches BFR A and can be forwarded with standard BIER-TE.

|                    BIER-TE domain                            |
                                        | BIER-TE subset (SI:1)|
                               /--C'-- S-BFIR B -- 0 -- \
                              /                          \
                             /                            \
BFIR ───A─── LSR A ───B─── LSR B  ───C─── S-BFIR A ───1─── BFR A
                        (PLR)
Figure 5: Example BIER-TE network with a single subset and two S-BFIRs.




