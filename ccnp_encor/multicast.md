# IGMP

## IGMPv1

- It uses 2 messages: `Membership Report` and `Membership Query`

    - Hosts send a membership report to the group address to join

    - The router will periodically send a membership query to destination `224.0.0.1` (all hosts multicast group address)
    
        - Hosts that receive this message will respond with a membership report to tell the router that they are still interested in receiving the multicast traffic
        
        - When the router receives the membership report, its expiry timer will be refreshed

- Problem: hosts don't report router when they are no longer interested in multicast group. The router will keep forwarding multicast traffic to the hosts until the timer expires

    ```bash
    R1#show ip igmp groups 239.1.1.1
    IGMP Connected Group Membership
    Group Address    Interface                Uptime    Expires   Last Reporter   Group Accounted
    239.1.1.1        GigabitEthernet0/1       19:22:56  00:01:07  192.168.1.101
    ```

- Configuration

    ![IGMPv1 Topology](./images/igmp_v1.png)

    ```bash
    # configure R1
    R1(config)#ip multicast-routing
    R1(config)#interface GigabitEthernet 0/1
    R1(config-if)#ip igmp version 1
    R1(config-if)#ip pim sparse-mode

    # configure host
    H1(config)#interface GigabitEthernet 0/1
    H1(config-if)#ip igmp version 1
    H1(config-if)#ip igmp join-group 239.1.1.1
    ```

## IGMPv2

- In IGMPv1, hosts stop listening to the multicast group address without reporting to the router

- New features of IGMPv2

    - Leave group messages: when a host no longer wants to listen to a multicast group address then it will report to the router that it has stopped listening

    - Group-specific membership query: When the router receives a leave group message, it will send this query to check if there are still any hosts interested in receiving the multicast traffic

    - MRT (Maximum Response Time) field: a new field in query messages. It specifies how much time hosts have to respond to the query

    - Querier election process: when there are two routers in the same subnet, then only one of them should send query messages

- Report Suppression: a mechanism where hosts listening to the same multicast group cancel their own membership report if they hear another host report for that group first

    - A multicast router sends a membership query to the subnet, including a MRT field

    - Each host subscribed to a multicast group sets a random delay timer up to the MRT

    - The host whose timer expires first sends a Membership Report to the multicast group address

    - Other hosts on the same segment receive this report while their timers are still running, stop their timers, and stay silent

    - The router only needs confirmation that at least one active member remains on the subnet to keep forwarding multicast traffic for that group

- The Leave Mechanism

    ```bash
    # R2 leaves the multicast group
    H2(config)#interface GigabitEthernet 0/1
    H2(config-if)#no ip igmp join-group 239.1.1.1

    # R1 receives the leave msg of H2 (.102), sends a query to the group, receives response from H1 (.101)
    R1#
    IGMP(0): Received Leave from 192.168.1.102 (GigabitEthernet0/1) for 239.1.1.1
    IGMP(0): Send v2 Query on GigabitEthernet0/1 for group 239.1.1.1
    IGMP(0): Received v2 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1

    # H1 leaves the multicast group
    H1(config)#interface GigabitEthernet 0/1
    H1(config-if)#no ip igmp join-group 239.1.1.1
    H1#
    IGMP(0): IGMP delete group 239.1.1.1 on GigabitEthernet0/1
    IGMP(0): Send Leave for 239.1.1.1 on GigabitEthernet0/1

    # R1 receives the leave msg of H1 (.101), sends a query to the group, receives no responses, deletes the group from the multicast routing table
    R1#
    IGMP(0): Received Leave from 192.168.1.101 (GigabitEthernet0/1) for 239.1.1.1
    IGMP(0): Send v2 Query on GigabitEthernet0/1 for group 239.1.1.1
    IGMP(0): MRT delete GigabitEthernet0/1 for (*,239.1.1.1) by 0
    ```

## IGMPv3

- IGMP version 3 adds support for **source filtering**

- Each group a host joins also has a `mode`

    - Include: this is a list of source addresses that we accept multicast traffic from

    - Exclude: this is a list of source addresses that we refuse to accept multicast traffic from

    - Example

        ```bash
        R1#show ip igmp groups 239.1.1.1 detail
        Interface:      GigabitEthernet0/1
        Group:          239.1.1.1
        Flags:
        Uptime:         00:12:27
        Group mode:     INCLUDE
        Last reporter:  192.168.1.101
        Group source list: (C - Cisco Src Report, U - URD, R - Remote, S - Static,
                            V - Virtual, M - SSM Mapping, L - Local,
                            Ac - Channel accounted towards access control limit)
        Source Address   Uptime    v3 Exp   CSR Exp   Fwd  Flags
        1.1.1.1          00:12:27  00:02:27  stopped   Yes  R
        ```

- The router **maintains multicast membership information per host** and can account for different source-filtering requirements

    - Suppose two hosts want to receive traffic for multicast group `239.1.1.1`
    
    - Host A wants traffic from all sources EXCEPT `Source 10.1.1.5`, host B wants all traffic INCLUDING `Source 10.1.1.5`

    - The router maintains separate membership records for each host, rather than applying one source filter to the entire multicast group

- The destination of the `membership report` is `224.0.0.22`, NOT the group address like IGMPv1 and IGMPv2

    - For example, a host wants to join multicast group `239.1.1.1`

    - The host sends to the destination IP `224.0.0.22`, and the group record is `239.1.1.1`

    - It is `239.1.1.1` for IGMPv1 and IGMPv2

- Configuration

    ```bash
    H1(config)#interface GigabitEthernet 0/1
    H1(config-if)#ip igmp version 3
    H1(config-if)#ip igmp join-group 239.1.1.1 source 1.1.1.1

    H1#
    IGMP(0): WAVL Insert group: 239.1.1.1 interface: GigabitEthernet0/1 Successful
    IGMP(0): Create source 1.1.1.1
    IGMP(0): Building v3 Report on GigabitEthernet0/1
    IGMP(0): Add Group Record for 239.1.1.1, type 5
    IGMP(0): Send unsolicited v3 Report with 1 group records on GigabitEthernet0/1
    IGMP(0): Building v3 Report on GigabitEthernet0/1
    IGMP(0): Add Group Record for 239.1.1.1, type 5
    IGMP(0): Send unsolicited v3 Report with 1 group records on GigabitEthernet0/1
    ```