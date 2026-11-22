# IGMP

## IGMPv1

- It uses 2 messages: `Membership Report` and `Membership Query`

    - Hosts send a membership report to the group address to join

    - The router will periodically send a membership query to destination `224.0.0.1 ` (all hosts multicast group address)
    
        - Hosts that receive this message will respond with a membership report to tell the router that they are still interested in receiving the multicast traffic
        
        - When the router receives the membership report, its expiry timer will be refreshed

- Problem: hosts don't report router when they are no longer interested in multicast group. The router will keep forwarding multicast traffic to the hosts until the timer expires

    ```bash
    R1#show ip igmp groups 239.1.1.1
    IGMP Connected Group Membership
    Group Address    Interface                Uptime    Expires   Last Reporter   Group Accounted
    239.1.1.1        GigabitEthernet0/1       19:22:56  00:01:07  192.168.1.101
    ```

## Configuration

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