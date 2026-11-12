# Enterprise Network Design (1.1.a): 2-tier, 3-tier, fabric, and cloud

## Two-tier architecture (Collapsed Core)

- Access Layer

    - Connect end-user devices: Desktop computers, IP phones, APs, Printers, Servers

    - Typical functions: VLAN assignment, L2 switching, PoE, Port security, QoS marking and classification, ACLs

- Collapsed Core Layer

    - Typical functions: Inter-VLAN routing, Default gateway for VLANs, L3 routing, Route summarization, Policy enforcement, Redundancy, High-speed connectivity between access switches

## Three-tier architecture (Core, Distribution, Access)

- Access Layer

- Distribution Layer

- Core Layer

    - Its primary responsibility is to transport traffic between distribution blocks as quickly and reliably as possible

    - Avoid putting unnecessary complex policies or services in the core

## Fabric architecture

- What it is

    - Fabric architecture is a modern networking design that **uses an underlying physical network to create a logical network on top of it**

    - Instead of relying entirely on traditional VLANs and Layer 2 connectivity, fabric technology uses an overlay network to provide connectivity, segmentation, and policy enforcement

    - Cisco's implementation of campus fabric architecture is called **SD-Access** (Software-Defined Access)

- Underlay vs. Overlay

    - Underlay Network: the physical network infrastructure

        - Physical switches and routers

        - IP addressing and routing
        
        - Provides IP connectivity between **fabric nodes**

    - Overlay Network: the logical network built on top of the underlay

        - Virtual network connectivity

        - Logical segmentation

        - Encapsulation and tunneling

        - Policy-based communication

- Important fabric components

    - Fabric Edge Node: connects endpoints (workstation, server, IP phone, printer, APs, IoT device) to the fabric

    - Control Plane Node: maintains endpoint location and reachability information

    - Border Node: connects the fabric to external networks

    - Fabric Wireless: integrates wireless clients into the fabric

    - Fabric Underlay: provides physical IP connectivity

    - Fabric Overlay: provides logical connectivity and segmentation

## Cloud architecture

- Public cloud model

- Private cloud model

- Hybrid cloud model