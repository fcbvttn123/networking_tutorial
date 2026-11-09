# BGP Characteristics

- **Path Vector Protocol**

- Routers form **neighborships**

    - Routers don't send `hello` packet to a multicast address like OSPF and EIGRP

    - The Neighbor IP is explicitly configured - **no dyanmic neighbor discover**

- A **TCP Port 179** Session is established between neighbors

- **NLRI** (Network Layer Reachability Information): router advertisement

    - It is the block of data that represents **a set of reachable IP prefixes** (or network destinations) **advertised between BGP peers**

    - `NLRI` entry typically consists of a prefix (the network address, such as `192.168.1.0` or `10.0.0.0`) and a length (the CIDR network mask, such as `/24` or `/8`). For example, `192.168.1.0/24 = NLRI`

    - How it is sent: `NLRIs` are carried inside BGP `UPDATE` messages. When a BGP router wants to tell its neighbors about a new network it can reach—or withdraw a network that went down—it includes the corresponding NLRI in an `Update` message

    - Path Attributes: An NLRI doesn't travel alone; it is bundled with a set of path attributes (such as `AS_PATH`, `NEXT_HOP`, and `LOCAL_PREF`) that describe how to get to that network and policies for choosing the best rout

- **Path Attributes**: BGP advertises a collection of Path Attributes used for Path Selection


# iBGP Full Mesh

## What it is and its problem

![iBGP Full Mesh Topology](./images/ibgp_full_mesh.png)

- An iBGP full mesh is a network design in which every internal BGP (iBGP) router within the same AS establishes a direct BGP peering session with every other iBGP router

- It has a Scaling Problem: `n(n-1)/2` -> 50 routers require 1,225 sessions

- Solutions: **BGP Route Reflectors** and **BGP Confederations**

## BGP Confederation

![BGP Confederation](./images/bgp_confederation.png)

- A BGP confederation is **a scaling mechanism** that divides a single large AS into multiple smaller, interconnected sub-ASes to avoid the heavy overhead of a full-mesh iBGP peering requirement

- Sub-AS Division: The large AS is broken down into smaller sub-ASes, typically assigned private AS numbers (ranging from 64,512 to 65,535)

- Routers inside the same sub-AS use standard iBGP rules

- Connections between different sub-ASes inside the confederation use confederation eBGP sessions. These sessions behave much like eBGP — retaining next-hop behavior and avoiding the iBGP full-mesh constraint — while preserving special attributes like the confederation sequence

## BGP Route Reflectors

![BGP RR](./images/bgp_route_reflector.png)

- A BGP Route Reflector (RR) is a designated router in an Internal BGP (iBGP) network that redistributes (reflects) routing updates between iBGP peers


# Neighbor Formation and States

## Neighbor Formation

- Idle State

- 3-way TCP Handshake

- Active State

- Open Sent State: R1 to R2

- Open Confirm State: R2 to R1

- Established State

## Messages After The Neighbor Formation

- Open Message: includes BGP Version Number, Local AS Number, Hold Time, BGP Router ID, Optional Parameters

- Update Message: contain withdrawn routes, path attributes and NLRI

- Notification Message: contains an error code, an error subcode and information about the error

- Default Timers: Keepalive - 60 seconds, Holdtime 180 seconds