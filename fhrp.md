# What is FHRP

- First Hop Redundancy Protocol: `HSRP` (Cisco), `VRRP` (Standard)

- Provide **gateway redundancy** so end devices always have a working default gateway, even if one router fails

- Packet Types

    - **HSRP** uses three specific message types, sent via multicast

        - Hello (Op Code 0): Sent periodically by routers to announce their presence, operational state, and priority

        - Coup (Op Code 1): Sent by a standby router when it wants to take over the active role from a lower-priority active router

        - Resign (Op Code 2): Sent by the active router when it is shutting down or when a higher-priority router takes over
    
    - **VRRP** is simpler and relies on a single packet type

        - Advertisement (Type 1): Sent periodically by the master router to broadcast its priority and state to backup routers
        
        - If backups stop receiving this packet, a new master is elected



# How FHRP Works

- Two or more routers share a **Virtual IP** that end clients use as their default gateway

- Each physical router has its own real IP, but they agree to present a single virtual gateway

- Client devices (PC, laptop) learn the **Virtual MAC** (associated with the **Virtual IP**) from ARP replies -> Only the **Active Router** uses this MAC

- End clients save the Virtual MAC address associated with the Virtual IP (Default Gateway) in their ARP table

- Networking devices store BOTH the Physical MACs and the Virtual MAC in their MAC address table




# How a Switch learns both VMAC and PMAC

## Physical MAC

- The routers continuously exchange FHRP hello messages (e.g., multicast `224.0.0.2` or `224.0.0.102`)

- These `hello` frames are sourced from the router's actual physical interface MAC

- Thus, the switch learns which switchport leads to the physical MAC of Router A and Router B

## Virtual MAC

- The Active/Master router also sends periodic frames or gratuitous ARPs sourced from the Virtual MAC address

- The switch records this Virtual MAC on the active router's switchport

## Switch MAC Address Table Example

```bash
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
10      0011.2233.44aa    DYNAMIC     Gi0/1   <-- Router A Physical MAC
10      0011.2233.44bb    DYNAMIC     Gi0/2   <-- Router B Physical MAC
10      0000.0c07.ac01    DYNAMIC     Gi0/1   <-- FHRP Virtual MAC (Points to Active Router)
```




# Failover Process

- If the **Active Router** fails, the **Standby Router** becomes Active

- New Active router sends a **Gratuitous ARP**

    - This Gratuitous ARP updates ARP Tables of all end devices

    - It informs switches to update their MAC Address Table

        - Although the Virtual MAC does NOT change, what does change is which physical router owns that Virtual MAC

        - When we do `tracert` on a client, it still shows the physical MAC

        - Without gratuitous ARP, the SW will still believe the Virtual MAC is on the old Router port




# Pre-emption

- Preempt allows a higher-priority router (often the “main router”) to take back the Active role after coming back online

- Without `preempt`, the router that becomes Active during a failure will stay Active even when the preferred router returns




# VRRP Config

```bash
# master
Router-1(config)# interface GigabitEthernet0/0/1
Router-1(config-if)# ip address 192.168.10.2 255.255.255.0
Router-1(config-if)# no shutdown
# enable VRRP Group 10 and assign the Virtual IP
Router-1(config-if)# vrrp 10 ip 192.168.10.1
# increase Priority above default (100) so it becomes Master
Router-1(config-if)# vrrp 10 priority 120
# enable Preemption (Enabled by default in VRRP, good practice to enforce)
Router-1(config-if)# vrrp 10 preempt
# set Hello / Advertisement timers (Optional, default is 1 second)
Router-1(config-if)# vrrp 10 timers advertise 1

# standby
Router-2(config)# interface GigabitEthernet0/0/1
Router-2(config-if)# ip address 192.168.10.3 255.255.255.0
Router-2(config-if)# no shutdown
# join the same VRRP Group 10 with the same VIP
Router-2(config-if)# vrrp 10 ip 192.168.10.1
# keep default priority (100)
Router-2(config-if)# vrrp 10 priority 100
Router-2(config-if)# vrrp 10 preempt

# if Router-1's WAN uplink (Gi0/0/0) drops, you want it to decrease its priority so Router-2 takes over as Master automatically
Router-1(config)# track 1 interface GigabitEthernet0/0/0 line-protocol
Router-1(config-if)# interface GigabitEthernet0/0/1
Router-1(config-if)# vrrp 10 track 1 decrement 30

# verification
Router-1# show vrrp brief
Interface          Grp  Pri Time  Own Pre State   Master addr     Group addr
Gi0/0/1            10   120 3609   N   Y  Master  192.168.10.2    192.168.10.1 
# details
Router-1# show vrrp 10
GigabitEthernet0/0/1 - Group 10
  State is Master
  Virtual IP address is 192.168.10.1
  Virtual MAC address is 0000.5e00.010a
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 120
    Track object 1 state UP decrement 30
  Master Router is 192.168.10.2 (local), priority is 120
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec
```




# Hello Message

- These packets are transmitted out of the router's Layer 3 Interface (which has FHRP enabled) using multicast addresses

    - **VRRP** uses `224.0.0.18`

    - **HSRP** uses `224.0.0.102`

- `HSRP` and `GLBP` use **UDP** as their transport layer protocol, while `VRRP` uses **Raw IP** (IP Protocol = 112) instead

- The `hello` packet are sent periodically: 1 second for VRRP, 3 seconds for HSRP

- If 10 seconds (HSRP - Hold Time is 10 seconds) pass with no `hello` in HSRP, `standby` becomes Active




# VRRP in Wireshark

```bash
# Advertisement
Internet Protocol Version 4
    Source: 192.168.10.2
    Destination: 224.0.0.18
    Protocol: VRRP (112)
Version: 3
Type: Advertisement
Virtual Router ID: 10 # VRRP Group 10
Priority: 120
Advertisement Interval: 1 second # how often Master sends advertisements
IP Address Count: 1
Virtual IP Address: 192.168.10.1
```