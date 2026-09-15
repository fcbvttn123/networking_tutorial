# WAN

## Leased Line (old days)

- Offices are connected to Service Provider via **Leased Line (serial)**

- Service Provider is connected to the Data Center

## Ethernet (modern)

- Offices are connected to Service Provider via **Ethernet (fiber)**

## CATV and DSL

- They are technologies used by consumers for home internet access

- DSL - Digital Subscriber Line


# Leased Line

- A leased line is a dedicated physical link, connecting 2 sites

- It uses **serial connection** (PPP or HDLC encapsulation - **not Ethernet**)

- In modern days, these serial leased lines are replaced by **Ethernet WAN Technologies**

    - Leased Lines have higher cost, installation lead time, slower speed


# MPLS VPN (private WAN technology)

## MPLS Overview

- Multi Protocol Label Switching

- Similar to the Internet, service provider's MPLS networks are shared infrastructures

- MPLS is used by service provider only

- MPLS uses **labels** to forward traffic, not IP Addresses

- The **label switching** term in MPLS allows VPN by the user of **labels**

    - The **label** makes sure customer traffics are not mixed together

## CE, PE and P

- Important Terms

    - CE Router (Customer Edge Router) - connected to PE Router

    - PE Router (Provider Edge Router)

    - P Router (Provider Core Router)

- When the PE routers receive frames from CE Routers, they add a label to the frame

    - This label is placed between L2 and L3 => L2.5 Protocol

    - These labels (**not the destination IP**) are then used to make forwarding decisions within the ISP network

- **CE Routers don't use MPLS**, only PE/P routers

## L3 MPLS VPN

- When using a Layer 3 MPLS VPN, the CE and PE routers peer using OSPF, for example, to share routing information

- For example, Office A's CE will peer with one PE, and Office B's CE will peer with the other PE

- Office A's CE will learn about Office B's routes via this OSPF peering, and Office B's CE will learn about Office A's routes too

## L2 MPLS VPN

- When using a Layer 2 MPLS VPN, the CE and PE routers do not form peerings

- The service provider network is entirely transparent to the CE routers

- It is like the two CE routers are directly connected

- Their WAN interfaces will be in the same subnet

- If a routing protocol is used, the two CE routers will peer directly with each other
