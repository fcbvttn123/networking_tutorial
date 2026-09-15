# How multicast works

- Multicast group creation

    - A multicast application uses a special multicast IP address

    - IPv4 multicast addresses range from: `224.0.0.0 - 239.255.255.255`

    - One address represents a group rather than a specific device

- Receivers join the group

    - Devices that want the stream tell their local router: "I am interested in multicast group 239.1.1.1"

    - They typically use the IGMP in IPv4

    - Routers keep track of which multicast groups have subscribers

- The sender sends packets to the multicast address and **does not need to know who the receivers are**


# Multicast routing

- Routers use multicast routing protocols to determine where to forward multicast traffic

- Common protocols include PIM (Protocol Independent Multicast)

    - PIM Sparse Mode (most common)

    - PIM Dense Mode

- These protocols build the multicast forwarding tree so traffic reaches only networks with interested receivers


# How Switches track multicasting-joined hosts

- SWs learn which hosts want multicast traffic by using **IGMP Snooping** to listen in on control messages sent between hosts and routers

- How IGMP Snooping Works

    - When a host wants to join a multicast group, it sends an **IGMP Membership Report** (an IGMP Join)

    - A Layer 2 SW, with IGMP snooping enabled, **snoops** or **intercepts** these traffic messages passing through its ports

    - **Building a table**: The switch records the multicast group address and maps it directly to the specific switch port where the host is connected

    - When multicast data arrives, the switch sends the stream only to the ports that requested it

- Without IGMP snooping, a switch cannot learn multicast addresses from source fields, because **multicast addresses are never used as a source MAC**


# How Routers learn multicast paths

- Topology

    ```bash
    Receiver A ---- R3
                    \
                    R2 ---- R1 ---- Source
                    /
    Receiver B ---- R4
    ```

    - Receiver A and Receiver B join group `239.1.1.1` using `IGMP` 

    - Their local routers (R3 and R4) learn: **Group 239.1.1.1 exists on LAN interface**

    - At this point, only R3 and R4 know there are interested receivers

- Next: routers tell each other using `PIM`

    - R3 and R4 effectively tells upstream routers: "I need traffic for group 239.1.1.1"

    - This information propagates toward the source

        ```bash
                Source
                  |
                 R1
                  |
                 R2
                /  \
              R3    R4
              |      |
         Group members
        ```

    - The routers then build a multicast distribution tree

    - R1 learns: `239.1.1.1` forward to R2

    - R2 learns: `239.1.1.1` forward to R3 and R4

- Compare it to normal routing

    - For unicast routing, a router typically stores: `Destination: 10.10.10.0/24, Next-hop: R5`

    - For multicast, it stores something more like: `Group: 239.1.1.1, Incoming interface: Gi0/0, Outgoing interfaces: Gi0/1 Gi0/2`