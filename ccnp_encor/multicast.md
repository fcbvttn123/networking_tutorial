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