# Contents

- [Contents](#contents)
- [Classic STP](#classic-stp)
  - [How STP Works](#how-stp-works)
  - [BPDU (Bridge Protocol Data Unit)](#bpdu-bridge-protocol-data-unit)
  - [Root Bridge Election (Bridge ID)](#root-bridge-election-bridge-id)
  - [STP Port Roles](#stp-port-roles)
  - [Root Port Election](#root-port-election)
  - [Designated Port Election](#designated-port-election)
  - [Port States](#port-states)
  - [Port State Progression](#port-state-progression)
  - [Timers](#timers)
- [Verification Commands](#verification-commands)
  - [`show spanning-tree` (Check STP status)](#show-spanning-tree-check-stp-status)
  - [`show spanning-tree interface <interface_id>` (See STP details for an interface)](#show-spanning-tree-interface-interface_id-see-stp-details-for-an-interface)
  - [`show spanning-tree detail` (Look for STP topology changes)](#show-spanning-tree-detail-look-for-stp-topology-changes)
- [RSTP](#rstp)
  - [Differences](#differences)
  - [RSTP Cost](#rstp-cost)
  - [Port States](#port-states-1)
  - [Port Roles](#port-roles)
  - [Why sending BPDUs in Discarding (and DP) is critical](#why-sending-bpdus-in-discarding-and-dp-is-critical)
  - [RSTP Security \& Protection Features](#rstp-security--protection-features)
  - [When a new SW is added](#when-a-new-sw-is-added)


# Classic STP

## How STP Works

- Classic STP follows four major steps:

  - Elect the Root Bridge

  - Elect Root Ports

  - Elect Designated Ports

  - Block redundant paths: ports that aren't RP and DP become NDP and remain non-forwarding

## BPDU (Bridge Protocol Data Unit)

- Classic STP primarily uses two BPDU types:

  - **Configuration BPDU (Hello Packet):** A multicast network packet used to elect a Root Bridge, discover neighbors, and continuously monitor the network topology to prevent loops

  - **Topology Change Notification (TCN) BPDU:** Notifies the network of a topology change

- The Root Bridge generates a Configuration `BPDU` every Hello Time interval (default: 2 seconds)

  - Non-root switches do not generate `BPDUs` on their own timer

  - They wait to receive a `BPDU` on their Root Port, update the message age and cost, and then relay (forward) out all Designated Ports

  - As long as the relayed `BPDUs` arrive before the **Max Age timer** expires (default: 20 seconds), non-root SWs stay quiet and simply pass them down the tree

- When the switches boot up, the SBID and the RBID is identical on each single switch in the network

## Root Bridge Election (Bridge ID)

- Every switch has a Bridge ID (BID)

- The switch with the **lowest BID** wins the Root Bridge election

- The Bridge ID consists of:

  - **Bridge Priority: Default 32768**

  - Extended System ID: VLAN ID in Cisco PVST+

  - MAC Address: Switch MAC address used in the BID

- The comparison happens in this order:

  - Lowest Bridge Priority: most switches have the same BID - 32768

  - Lowest MAC address if priorities are equal

## STP Port Roles

- Root Port (RP):	Best path from a non-root switch to the Root Bridge

  - Every non-root switch has exactly one Root Port per spanning-tree instance

  - A Root Bridge does not have a Root Port

- Designated Port (DP):	Best forwarding port for a particular segment

- Non-designated Port (NDP): Port that is blocked to prevent loops

## Root Port Election

- A Root Port is the port with the **lowest total path cost** to the Root Bridge

- STP uses **path cost** to determine the best route

- Classic short **path cost values**

  - 10 Mbps: 100

  - 100 Mbps: 19

  - 1 Gbps: 4

  - 10 Gbps: 2

- If two paths have the same cost, STP uses tie-breakers in this order:

  - Lowest upstream (sender) Bridge ID

  - Lowest upstream (sender) Port ID

  - Lowest local Port ID

## Designated Port Election

- A Designated Port (DP) is selected for every **network segment**

- The switch, advertising the best path toward the Root Bridge, wins the Designated Port election for that segment

- The comparison follows these principles: Lowest Root Path Cost, Lowest Bridge ID, Lowest Port ID

- The Root Bridge always has its active ports as Designated Ports

## Port States

| State | Forwards frames | Learns MAC addresses |
| --- | --- | --- |
| Blocking | No | No |
| Listening | No | No |
| Learning | No | Yes |
| Forwarding | Yes | Yes |

## Port State Progression

- The normal transition when a blocked port is selected to become a forwarding port is: Blocking → Listening → Learning → Forwarding

## Timers

- Hello: 2 seconds - Interval between configuration BPDUs from the Root Bridge

- Forward Delay: 15 seconds	- Duration of Listening and Learning states

- Max Age:	20 seconds - How long received STP information remains valid


# Verification Commands

## `show spanning-tree` (Check STP status)

```bash
VLAN0001
  Spanning tree enabled protocol rapid-pvst
  Root ID    Priority    32769
             Address     0011.2233.4455
             Cost        4
             Port        24(GigabitEthernet0/24)

  Bridge ID  Priority    32769
             Address     00aa.bbcc.ddee

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.1    P2p
Gi0/2               Desg FWD 4         128.2    P2p
Gi0/24              Root FWD 4         128.24   P2p
```

## `show spanning-tree interface <interface_id>` (See STP details for an interface)

```bash
Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001             Root FWD 4         128.24   P2p

Port 24 (GigabitEthernet0/24) of VLAN0001 is root forwarding
  Port path cost 4, Port priority 128, Port Identifier 128.24
  Designated root has priority 32769, address 0011.2233.4455
  Designated bridge has priority 32769, address 0011.2233.4455
  Designated port id is 128.24
```

## `show spanning-tree detail` (Look for STP topology changes)

```bash
VLAN0001 is executing the ieee compatible Spanning Tree protocol
  Bridge Identifier has priority 32768, sysid 1, address 00aa.bbcc.ddee
  Topology change flag not set, detected flag not set
  Number of topology changes 12 last change occurred 00:05:31 ago
          from GigabitEthernet0/24
  Times:  hold 1, topology change 35, notification 2
```


# RSTP

## Differences

- Every switch generates its own BPDUs every Hello Time interval (2 seconds), containing its current view of the topology. Switches do not wait for the Root Bridge to send a BPDU first

- If a switch misses BPDUs from a neighbor for 3 consecutive Hello intervals (6 seconds), it immediately considers neighbor connectivity lost 

    - and recalculates its topology, without waiting up to 20 seconds for a Max Age timer to expire

## RSTP Cost

- 10 Mbps = 2,000,000

- 100 Mbps = 200,000

- 1 Gbps = 20,000

- 10 Gbps = 2000

- 100 Gbps = 200

## Port States

- Discarding: the port drops data frames and does not learn MAC addresses, but it listens to BPDUs, and it can **send BPDUs in DP role**

- Learning: port processes BPDUs and builds the MAC address table, but does not forward user traffic yet

- Forwarding

- In RSTP, a new port **does not use a fixed timer** to stay in the **discarding** and **learning** states

  - Instead, it transitions almost instantly (usually within milliseconds to a few seconds) using a **handshaking mechanism** called the **proposal/agreement process**

  - No Fixed Timers: Unlike classic STP (which waits 30 seconds for listening and learning), RSTP relies on an active handshake

## Port Roles

- RP and DP are the same with Classic STP

- NDP is split into 2 roles: Alternate Port and Backup Port

    - **Alternate Port** immediately takes over if the local Root Port fails

    - **Backup Port** is only in use with a hub, because hub is not in use today, the backup port is rare

## Why sending BPDUs in Discarding (and DP) is critical

- When a new point-to-point link brings up a connection between two SWs, both ports start in **Designated Discarding**

- SW A sends a BPDU with the Proposal flag set while still in the **Discarding** state

- SW B receives it, syncs its local ports, and responds with an Agreement

- If ports in the Discarding state couldn't send BPDUs, SWs could never negotiate the **Proposal/Agreement handshake** to transition safely to **Forwarding**

## RSTP Security & Protection Features

- Edge Ports: `spanning-tree portfast`

- BPDU Guard: `spanning-tree portfast bpduguard default`

- Root Guard: `spanning-tree guard root`

- Loop Guard: protect against unidirectional link failures by blocking ports that stop receiving expected BPDUs instead of transitioning them to Forwarding

## When a new SW is added

- A new SW A sends a BPDU to SW B with the **Proposal** bit set, offering to make its connecting port a DP

- SW B recognizes SW A's superior Bridge ID and accepts SW A as the new Root Bridge

- The port connected to SW A immediately becomes SW B's new RP (but in discarding state)

- To guarantee no temporary loops are created, SW B performs a **Sync**

  - SW B immediately puts all non-edge DP into the Discarding state (blocking user traffic)

  - Edge ports are not affected and stay Forwarding

- Once SW B's non-edge ports are blocked and isolated, SW B sends an **Agreement** back to SW A

- Upon receiving the Agreement, SW A transitions its port to Forwarding immediately (sub-second, no 30-second timers)

- SW B's new RP also becomes Forwarding immediately (no 30-second timers)

- SW B now sends a Proposal out its downstream ports to the next SW (SW C)

- SW C performs a Sync on its ports, sends an Agreement back to SW B, and unblocks its link

- As soon as SW B receives the Agreement from SW C, SW B's downstream port transitions immediately from Discarding to Forwarding