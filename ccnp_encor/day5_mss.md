# TCP Segment Review

- The min size of a TCP Header is **20 bytes**

- The max size of a TCP Header is **60 bytes** - which includes up to **40 bytes of optional fields** (such as TCP timestamps or window scaling)


# What TCP MSS is

- Maximum Segment Size

- It is the max of **TCP Payload** a host can receive in a single TCP Segment

- MSS **doesn't include the TCP Header**

- The MSS is specified in the `Options` field of the TCP Header - not one of the fields of the **fixed 20-byte TCP Header**

- **If MSS is specified**, the length of the TCP Header will be greater than 20 bytes

- **If a host doesn't specify its MSS in the first SYN**, it is default to 536 bytes for IPv4 and 1220 bytes for IPv6

- If **Segment + IP Header > IP MTU**, fragmentation will occur

- **Router and MSS**

    - Routers don't look at Layer 4 information when forwarding packets

    - However, routers can also be end hosts for network traffic (participate in TCP connections with other hosts)

    - Example: when connecting to a router with SSH, your computer establishes a TCP connection with the router

    - Cisco IOS devices specify an MSS of **1460** for devices in the same network, **536** for devices in another network

    - Change MSS value: `R1(config)# ip tcp mss 250`


# MSS and MTU

- **IP headers and TCP headers can be larger than 20 bytes** (if IP and/or TCP options are included), which could result in packets larger than the MTU with an MSS of 1460

- **PMTUD allows a host to determine the maximum packet size** that can be used along a path

    - It doesn't always work. For example, for security reasons firewalls might block ICMP traffic

- Routers can be configured to automatically **adjust MSS values** on all `TCP SYN` messages passing through the router


# MSS Adjustment

- MSS Adjustment allows a router to adjust the MSS of `TCP SYN` messages passing through it

- It is configured on a per-interface basis

- If a `TCP SYN` message passing through an interface (ingress or egress), configured with MSS Adjustment, has an MSS value higher than the value configured with the `ip tcp adjust-mss` command, the router will adjust the MSS down

- If the MSS value in the `TCP SYN` message is lower than the configured value, the router will not adjust it

- Configuration

    ```bash
    R1(config)# interface tunnel0
    R1(config-if)# ip tcp adjust-mss 1436 
    R1(config-if)# end
    ```

- Check the example of this video at **20:37**: `https://www.youtube.com/watch?v=wSTBrYzbXqs&list=PLxbwE86jKRgOb2uny1CYEzyRy_mc-lE39&index=13`