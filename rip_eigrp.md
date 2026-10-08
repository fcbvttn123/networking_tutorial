# RIP

- IGP - Distance Vector Protocol

- Multicast IP `224.0.0.9`

- Use **hop count** as metric (**bandwidth is irrelevant**). The maximum hop count is **15**

- A RIP-enabled Router **forms adjacencies with its neighbors**

- 3 versions: RIPv1, RIPv2 used for IPv4, RIPng used for IPv6

    - RIPv1 is old and has a lot of problems

- 2 message types

    - **Request**: ask RIP-enabled neighbor routers to send their routing table

    - **Response**: to send the routing table to neighbor routers

- By default, RIP-enabled routers share their routing table every **30 seconds**

- Configuration

    - Enable RIPv2

        ```bash
        R1(config)# router rip
        R1(config-router)# version 2
        R1(config-router)# no auto-summary
        ```

    - The `R1(config-router)# network 10.0.0.0` command: advertise routes

        - The RIP network command is **classful**, it will **automatically convert to classful network**

        - Even if you enter the command `network 10.0.12.0`, it will be converted to `10.0.0.0` (a class A network)

        - But for advertisement, the router still advertises `10.0.12.0/30` and `10.0.13.0/30`, NOT `10.0.0.0/8`, RIPv1 doesn't do this

    - Passive Interface: `R1(config-router)# passive-interface g2/0`

    - Advertise a default route: `R1(config-router)# default-information originate`


# EIGRP

- Cisco proprietary, only partly published

- IGP - **Distance Vector** Protocol

- Multicast IP `224.0.0.10`

- Enable RIPv2

    ```bash
    R1(config)# router eigrp 1
    R1(config-router)# no auto-summary
    R1(config-router)# network 10.0.0.0
    R1(config-router)# network 172.16.1.0 0.0.0.15
    R1(config-router)# passive-interface g2/0
    ```