---
title: "Network+: Cabling & Topology"
last_modified_at: 2019-10-24T16:20:02-05:00
categories:
  - Network+
tags:
  - Topology
  - Cabling
---

## Topology
A network topology is defined by the lines and nodes that make up the network. This can be further delineated into two twpes of topology: `physical` vs `logical` topology.

Physical topology is the geometric pattern of the connected nodes of a network, whereas logical topology defines the pattern of data transfer between network nodes.

### Physical topologies

![image](/assets/images/network-topology.png "A sample of physical network topologies")

1. Bus topology (dropdowns)
2. Ring topology (Token ring, developed by IBM)
3. Star topology (rare/old)
4. Hybrid topology (Star-Bus)
5. Mesh topology (Wireless: fully-meshed vs partially-meshed)

The hybrid star-bus topology is very often implemented... 

![image](/assets/images/star_network_topology_ani.gif "Star-Bus topology")

## Cabling
### Coaxial cabling
Coaxial cable has two conductors. Center copper conductor, second tubular conductor. 
Common axis as both conductors share same axis, same centerpoint.

Radio Grade (RG ratings) defines the thickness ofconductors, insulation and shielding.

RG-58 (50 Ohm) Older cable for networking
BNC Connector

RG-59 (75 Ohm) common in cable modems (older)
Threaded Connector (F-type connector, FC-type connector)

RG-6 (75 Ohm) common in cable modems (modern)
Thicker thab RG-59
 
### Twisted pair cabling
Twisting allows signal to propogate further down a piece of copper wire.
Unshielded twisted pair (UTP) cabling (no metal protective sheath).

![image](/assets/images/utp.jpg "UTP wire. 4 pairs")

Not strong or durable due to absence of sheath but is very cheap.
Solid core versus stranded core.
When used in network environemt it tends to be 4-twisted pairs.
We use a RJ-45 connector
Two standards for how the wires are connected to a connector.
1. EIA/TIA 568A standard
2. EIA/TIA 568B standard

![image](/assets/images/tia-eia-568-standards.jpg "UTP wire. 4 pairs")

Shielded twisted pair (STP) cabling (metal protective sheath).

### Cat Ratings
UTP gas evolved to work for faster and faster network speeds.
Cat ratings have a different number of twists per inch. Kevlar for pulling.

CAT-3 (10 Mbps)
CAT-5 (100 Mbps, @100m)
CAT-5e (100-1000 Mbps, @100m)
CAT-6: (1 Gbps, @100m)
CAT-6a: (10 Gbps, @100m)
CAT-7: (10 Gbps, @100m, each pair shielded)

### Fiber Optic Cabling
Multi-mode (light from LEDS) cable is mostly orange
Single mode (lasers) cable is mostly yellow

Duplex cables

Connectors

### Fire ratings


### Legacy Network Connections
