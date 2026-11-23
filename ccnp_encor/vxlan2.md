# Table of contents

- [Table of contents](#table-of-contents)
- [Overlay Network](#overlay-network)
  - [Underlay Network vs Overlay Network](#underlay-network-vs-overlay-network)
  - [Overlay Networking Technologies](#overlay-networking-technologies)
- [VXLAN Introduction](#vxlan-introduction)
  - [Key Takeaways](#key-takeaways)
  - [VNI (VXLAN Network Identifier)](#vni-vxlan-network-identifier)
  - [VTEP (VXLAN Tunnel Endpoint - VXLAN Gateway)](#vtep-vxlan-tunnel-endpoint---vxlan-gateway)
  - [Control Plane](#control-plane)


# Overlay Network

## Underlay Network vs Overlay Network

- The **underlay network** is typically **a L3 Routed Fabric** with **Routing Protocols** such as OSPF, IS-IS, or BGP

    - Its only purpose is to be a transport network for the overlay network

    - The underlay network also requires an increased MTU because of encapsulation overhead

- The **overlay network** is the virtual (logical) network that is built on top of the underlay network

    - This is usually done using tunneling
    
    - Overlay networks can provide Layer 2 (bridging) or Layer 3 (routing) services over the underlay network

## Overlay Networking Technologies

- VLAN

- GRE

- VPN

- DMVPN

- VXLAN

- LISP

- SD-WAN


# VXLAN Introduction

## Key Takeaways

- VXLAN uses a **24-bit VNI** (VXLAN Network Identifier), allowing **~16 million VXLANs**

- VTEP devices (VXLAN Tunnel Endpoints) **encapsulate and de-encapsulate L2 traffic**, available as **software (hypervisor-based)** or **hardware (switch/router-based)**

- L3 Underlay Network **eliminates spanning tree**, enabling **ECMP load balancing** on redundant links

- Four control plane options: Static unicast, multicast flood-and-learn, MP-BGP EVPN, and VXLAN with LISP

- **UDP port 4789** is the official VXLAN port

## VNI (VXLAN Network Identifier)

- It **identifies the VXLAN** and has a similar function as the VLAN ID for regular VLANs

## VTEP (VXLAN Tunnel Endpoint - VXLAN Gateway)

- It is the device that’s responsible for encapsulating and de-encapsulating layer 2 traffic

- The VTEP comes in two forms: **Software (host-based)** and **Hardware (gateway)**

    - Software: Hypervisors like VMWare’s ESXi or Microsoft’s Hyper-V. These hypervisors use **virtual switches**, and some of them support VXLAN

    - Hardware
    
        - A **Router, Switch, or Firewall** which supports VXLAN

        - We also call a hardware VTEP a **VXLAN Gateway** because it combines a regular VLAN and VXLAN segment into a single layer 2 domain
        
        - Some switches have VXLAN support with ASICs, offering better VXLAN performance than a software VTEP

- Each VTEP has two interfaces types

    - VTEP IP Interface: connects the VTEP to the underlay network with **a unique IP address**. This interface **encapsulates and de-encapsulates** Ethernet frames

    - VNI Interface: a virtual interface that keeps network traffic separated on the physical interface. Similar to an SVI interface

    - A VTEP can have multiple VNI interfaces, but they associate with the same VTEP IP interface

    - Example

        ![VTEP and VNI](./images/vtep_vni.png)

## Control Plane

- Each VTEP has a **VXLAN mapping (forwarding) table** that **maps a destination MAC address to a remote VTEP IP address**

- The control plane helps VTEP devices learn MAC addresses

- 4 control plane solutions

    - VXLAN with static unicast VXLAN tunnels

    - VXLAN with multicast underlay

    - VXLAN with MP-BGP EVPN (a popular choice in data centers)

    - VXLAN with LISP (a popular choice for campus networks)