# WAN

## Leased Line (old days)

- Offices are connected to Service Provider via **Leased Line (serial)**

- Service Provider is connected to the Data Center

## Ethernet (modern)

- Offices are connected to Service Provider via **Ethernet (fiber)**

## CATV, DSL and Modem

- They are technologies used by consumers for home internet access

- DSL - Digital Subscriber Line

    - A DSL Modem (modulator-demodulator) is used to convert data into a format to be sent over the phone lines

    - The modem can be integrated into the router, or a separate device

- CATV is used for TV Service and can be used to provide internet access


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


# Internet VPNs

## Sire-to-site VPN using IPsec

- GRE over IPsec

    - IPsec VPN doesn't support broadcast or multicast traffic (e.g., OSPF)

    - GRE creates tunnels like IPsec, but doesn't encrypt the original packet => **GRE over IPsec** is the solution

    - The packet is encapsulated by a **GRE Header** and a **new IP Header**, then the GRE Packet is encrypted and encapsulated within an **IPsec VPN Header** and **new IP Header**

- DMVPN

    - A Cisco solution which allows routers to dynamically create a full mesh of IPsec tunnels without manually configure every single tunnel

    - Step 1: configure VPN for each **spoke router** to the **hub router**

    - Step 2: the hub router gives each spoke router information about how to form an IPsec tunnel with other spoke routers to create a full mesh topology

## Remote-access VPNs using TLS


# Configure GRE Tunnels

```bash
# configure tunnel source (inside interface, not WAN), tunnel destination (R2 WAN Interface) and tunnel IP (IP for the tunnel only - /30 network, not internal LANs)
R1(config)# interface tunnel 0
R1(config-if)# tunnel source g0/0/0
R1(config-if)# tunnel destination 200.0.0.2
R1(config-if)# ip address 192.168.1.1 255.255.255.252
R1(config-if)# do show ip interface brief # view the tunnel interface

# configure static route to form the tunnel with R2
R1(config)# ip route 0.0.0.0 0.0.0.0 <R1 WAN Interface>
R1(config)# do show ip route # you should see a route: C  192.168.1.0 /30 is directly connected, Tunnel0

# configure OSPF to share internal subnets
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.1 0.0.0.0 area 0
R1(config-router)# network 10.0.1.1 0.0.0.0 area 0
```