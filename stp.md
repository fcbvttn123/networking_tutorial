# Contents

- [Contents](#contents)
- [BPDU (`hello` message)](#bpdu-hello-message)
- [STP Convergence Process](#stp-convergence-process)
- [Port Types](#port-types)
  - [Root Port](#root-port)
  - [Designated Ports](#designated-ports)
  - [Non-designated Ports](#non-designated-ports)
- [Port States](#port-states)
  - [Blocking State](#blocking-state)
  - [Listening State (Forward Delay Timer)](#listening-state-forward-delay-timer)
  - [Learning State (Forward Delay Timer)](#learning-state-forward-delay-timer)
  - [Forwarding State](#forwarding-state)
- [STP Timers](#stp-timers)
  - [Hello Time](#hello-time)
  - [Max Age](#max-age)
  - [Forward Delay](#forward-delay)
- [Root Bridge Election](#root-bridge-election)
- [STP Cost](#stp-cost)
- [Verification Commands](#verification-commands)
  - [`show spanning-tree` (Check STP status)](#show-spanning-tree-check-stp-status)
  - [`show spanning-tree interface <interface_id>` (See STP details for an interface)](#show-spanning-tree-interface-interface_id-see-stp-details-for-an-interface)
  - [`show spanning-tree detail` (Look for STP topology changes)](#show-spanning-tree-detail-look-for-stp-topology-changes)
- [Todo: RP, NDP \& DP Selection Rule](#todo-rp-ndp--dp-selection-rule)
- [RSTP](#rstp)




# BPDU (`hello` message)

- The `BPDU` (Bridge Protocol Data Unit) is broadcast by the switches

- The `BPDU`, contains

    - Root Bridge ID
    
    - Sender Bridge ID
    
    - Root Path Cost

    - Port ID = Port ID + Port Number `Interface: g0/1, Role: root, Sts: FWD, Cost 4, Prio.Nbr: 128.1, Type: P2p`

    - Expiration Fields

        - Message Age
        
            - The amount of time that has elapsed since the Root Bridge originally generated the `BPDU`
            
            - The Root sets this to 0 seconds, and each switch that relays the `BPDU` increments this value (typically by adding 1 second or adding the propagation delay)

        - Max Age: The maximum allowed age for a `BPDU` (default is 20 seconds)

- When the switches boot up, the SBID and the RBID is **identical** on each single switch in the network

    - The switches then broadcast the `BPDU` frame with this information to the neighbors

    - No `BPDU` Loop because switches consume `BPDUs` locally instead of flooding them

- If a SW receives a Hello `BPDU` on an interface, it knows that interface is connected to another SW (routers, PCs, etc. do not use STP) so they do not send Hello BPDUs

- The Root Bridge generates a Configuration `BPDU` every Hello Time interval (default: 2 seconds)

    - Non-root switches do not generate `BPDUs` on their own timer

    - They wait to receive a `BPDU` on their Root Port, update the message age and cost, and then relay (forward) out all Designated Ports

    - As long as the relayed `BPDUs` arrive before the **Max Age timer** expires (default: 20 seconds), non-root SWs stay quiet and simply pass them down the tree




# STP Convergence Process

- Initialization: When **a switch boots** up or **a link goes up**, all active ports start in the Blocking State by default

    - When the switches boot up, the BID and the RBID (of the BPDU sent) is **identical** on each single switch in the network

- Role Election: While in this initial blocking phase, the switches exchange BPDUs to elect the Root Bridge and decide Port Roles

- When a new switch is added to the topology

    - A new SW never stays silent. The moment it comes up, it:

    - Sends its own BPDUs (claiming “I am the root”)

    - Receives BPDUs from existing switches

    - Compares them

    - Updates its role accordingly

- When a Computer is connected

    - The SW sends BPDUs and receives none — but it does NOT “wait for confirmation.”

    - It becomes a DP by default because **it is the only bridge on that LINK**

- When the RP is dead

    - NDP Transitions to Listening

    - Receiving BPDUs -> Learning State




# Port Types

## Root Port

- All the ports, connecting to the root bridge, on non-bridge switches are root ports

- Forwarding State

## Designated Ports

- All ports on the Root Bridge

- Other ports which are not NDP

- There is only one DP for each **network segment** (the physical link between two switches)

- Forwarding State

## Non-designated Ports

- Blocked State




# Port States

## Blocking State

- The port only receives the BPDU frames

## Listening State (Forward Delay Timer)

- The ports in this state can send and receive BPDU frames

- This state has 15 seconds (we can change this duration) to process BPDUs it receives

    - The port in this state DOESN'T learn MAC addresses

    - The switch uses the BPDU information to determine whether the port should be RP, DP, or NDP

## Learning State (Forward Delay Timer)

- The main purpose of this state is to learn MAC addresses before forwarding normal traffic

## Forwarding State

- The port can process normal traffic




# STP Timers

## Hello Time

- What it is: How often the Root Bridge sends BPDUs

- Default is 2 seconds

## Max Age

- What it is: How long a SW will keep a BPDU before considering it invalid

- Default is 20 seconds 

- If a SW does not hear from the root for 20 seconds, it recalculates the topology

## Forward Delay

- What it is: How long a port stays in Listening and Learning states

- Default is 15 secs




# Root Bridge Election

- The root bridge election is determined by the **Bridge ID** (contained inside BPDUs)

- `BID = Bridge Priority + VLAN ID + MAC Address`

    - By default: Bridge Priority is 32768 on all switches

    - So, the MAC Address is often the tie breaker, **the lowest MAC wins**

- A switch identifies itself as the Root Bridge if it has the lower MAC address and Priority




# STP Cost

- 10Gps: 2

- 1Gps: 4

- 100Mps: 19




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




# Todo: RP, NDP & DP Selection Rule




# RSTP

- Every switch generates its own BPDUs every Hello Time interval (2 seconds), containing its current view of the topology. Switches do not wait for the Root Bridge to send a BPDU first

- If a switch misses BPDUs from a neighbor for 3 consecutive Hello intervals (6 seconds), it immediately considers neighbor connectivity lost 

    - and recalculates its topology—without waiting up to 20 seconds for a Max Age timer to expire