# Table of contents

- [Table of contents](#table-of-contents)
- [1.1.a Enterprise Network Design: 2-tier, 3-tier, fabric, and cloud](#11a-enterprise-network-design-2-tier-3-tier-fabric-and-cloud)
  - [Two-tier architecture (Collapsed Core)](#two-tier-architecture-collapsed-core)
  - [Three-tier architecture (Core, Distribution, Access)](#three-tier-architecture-core-distribution-access)
  - [Fabric architecture](#fabric-architecture)
  - [Cloud architecture](#cloud-architecture)
- [1.1.b HA techniques: redundancy, FHRP, and SSO](#11b-ha-techniques-redundancy-fhrp-and-sso)
- [1.2 SD-WAN](#12-sd-wan)
  - [What it is](#what-it-is)
  - [Components](#components)
  - [Underlay and Overlay](#underlay-and-overlay)
  - [OMP (Overlay Management Protocol)](#omp-overlay-management-protocol)
  - [TLOC (Transport Locator)](#tloc-transport-locator)


# 1.1.a Enterprise Network Design: 2-tier, 3-tier, fabric, and cloud

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


# 1.1.b HA techniques: redundancy, FHRP, and SSO

- SSO allows a standby supervisor to take over immediately when the active supervisor fails, without requiring a full device reboot

- Used on: Catalyst 4500, Catalyst 6500, Catalyst 6800, Catalyst 9400, Catalyst 9500, Catalyst 9600

- Traditional Supervisor Failover

    - Active Supervisor fails

    - Switch reloads

    - Protocols restart

    - Users disconnected

    - Downtime can be minutes

- With SSO

    - Active Supervisor fails

    - Standby Supervisor becomes active

    - Traffic continues

    - Downtime often measured in milliseconds to a few seconds

- In Cisco networking, a supervisor (often called a **Supervisor Engine** or **Supervisor Module**) is the component that acts as the brain of a chassis switch

    - The supervisor is responsible for the **switch's control plane and management functions**

    - Supervisor Module: handles routing, STP, management, config, protocols

    - Line Cards: provide the physical ports (line card 1, line card 2, line card 3)

    - A Catalyst 9600 may contain: supervisor 1, 48-port line card, 24-port 40G line card

- How SSO Works

    - Two supervisors exist: Active Supervisor & Standby Supervisor

    - The standby continuously synchronizes: Interface state, VLAN information, STP information, Routing information, FHRP state

    - The standby always has a current copy

- SSO States

    - ACTIVE: fully synchronized, can immediately assume control, preferred state

    - STANDBY Hot: fully synchronized, can immediately assume control, preferred state

    - STANDBY COLD: present but not synchronized, longer failover time

- NSF (Nonstop Forwarding)

    - SSO is usually paired with NSF

    - SSO protects Control Plane by switching supervisors

    - NSF protects Data Plane by continuing packet forwarding during routing reconvergence

    - Example: `ospf` runs supervisor failure

        - Without NSF: `ospf` restarts, routes lost, traffic interrupted

        - With NSF: packets continue forwarding, neighbor relationships preserved

- SSO + NSF Operation

    - Active supervisor fails

    - Standby supervisor becomes active

    - FIB remains available

    - Traffic forwarding continues

    - Routing protocols recover


# 1.2 SD-WAN

## What it is

- Traditional WAN with **MPLS**

    - Historically, companies connected sites using expensive MPLS circuits

        ```bash
        Branch A ---- MPLS ---- HQ
        Branch B ---- MPLS ---- HQ
        ```

    - MPLS is expensive

    - Adding new sites takes weeks or months

    - Cloud applications (Microsoft 365, Salesforce, Webex) often take inefficient paths through HQ

- Traditional WAN with **IPSec VPN**

    - Each branch would build an IPsec VPN tunnel to HQ

    - All traffic must go through the HQ first

    - With 100 branches, we have hundreds of VPN tunnels, routing complexity, manual firewall configuration

    - Every new branch requires: firewall setup, tunnel creation, routing updates, testing

    - With SD-WAN, we add a new site `Site ID = 400`, the router connects to `vBond`, authenticates, downloads config and builds tunnels automatically

- Cisco SD-WAN Idea

    - SD-WAN separates Control Plane and Data Plane

    - Control Plane: the intelligence that decides which routes to use, which apps get priority, security policies, segmentation

        - In Cisco SD-WAN, the vSmart controller is responsible for distributing routing and policy information to the SD-WAN devices

    - Data Plane: responsible for actually moving network traffic

        - It is the router which receives the packets and forwards those packets through the selected WAN connection

    - Traditional routers combine everything:

        - The same physical switch performs two jobs

        - Control plane: Makes decisions about where traffic should go. For example, it runs STP, OSPF, and learns routing information

        - Data plane: Forwards packets based on its MAC address table or routing table

## Components

- `vManage` (management plane): the Web GUI used for management

- `vSmart` (control plane): the software-based control plane, which acts as the "brain" of the overlay network fabric

- `vBond` (orchestration plane): authenticates devices, helping them discover and join the overlay fabric

- `vEdge`/`cEdge` (data plane): routers that actually forward user and application data packets

## Underlay and Overlay

- Underlay is the real transport network: internet, MPLS, LTE, 5G

- Overlay is the logical SD-WAN tunnels built, over transports, between sites. Even though packets travel across the Internet underneath

## OMP (Overlay Management Protocol)

- OMP is Cisco's proprietary SD-WAN routing protocol

- It's like BGP for the SD-WAN fabric

- OMP advertises 

    - Routes (networks available at each site)
    
    - TLOCs - Transport Locator Information (how to reach a WAN Edge)

    - Service Routes - Information about services such as: Firewalls, IDS/IPS, Cloud Security Services

## TLOC (Transport Locator)