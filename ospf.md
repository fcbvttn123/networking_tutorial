# How **link-state routing protocols** work

- Link-state protocols use the **SPF algorithm** which was developed by **Edsger Dijkstra**

- The routers that share a link will **recognize the neighboring routers** and **form relationships**

- When this relationship has been formed, they will **share their directly connected routes** with each other

- The neighbors that receive this information will then **propagate it to other neighbors**

- When all the neighbors know all the routes, each router will use the information to create a **MAP** to all the destinations in the networks

- When this map has been created, the **SPF (Shortest Path First) algorithm**, is run to determine which the best route to a particular remote network is




# OSPF packets types

## Hello

- They are the **first messages** that are sent by routers that have been configured with OSPF

- They use the **multicast IP address** specially **reserved for OSPF** which is `224.0.0.5`

- Sent at **10-second interval**

- Purpose: **discover neighbors** and **maintain relationships**

## DBD (Database Description)

- This packet is **a list which contains a summary of routes** that have been learnt by a particular router in the routing domain

- The router that receives this packet, checks the list against its own link-state database, to discover any missing routes

## LSR (Link-state request)

- When a router discovers that it is missing some routes as a result of the information contained in a DBD packet it has received

- It sends this packet to the router that informed it of the missing routes, requesting more detailed information on the missing routes

- This is done so that it can update its link-state database with these missing routes

## LSU (Link-State Update)

- This packet is sent by a router that has information on any missing routes

- It contains the next-hop information and the cost to reach the particular route that **was requested using an LSR**

## LSAck (Link-State Acknowledgment)

- This is a packet that is sent to confirm that a router has received an LSU




# Basic OSPF Config

## Enable OSPF: `R1(config)# router ospf <process_id>`

- The process ID is locally signifcicant (between 1 and 65535)

- Neighboring routers do not need this number to match

## Advertise Routes: `R1(config-router)# network <network_address> <wildcard_mask> area <area_ID>`

- If we use one area, we must use the backbone area `0`

- The `network` command advertises the **directly connected networks** that we want to participate in OSPF

## OSPF Router ID: `R1(config-router)# router-id <unique_ip_address>`

- The Router ID is a way to name each router in the routing domain

- If not configured, it selects the highest IP of the **loopback interface** or **active physical interface**

- Example: `R1(config-router)# router-id 1.1.1.1`

- Configure Loopback Interface

    ```bash
    Router(config)# interface loopback 0
    Router(config-if)# ip address 1.1.1.1 255.255.255.255
    Router(config-if)# exit
    Router(config)# router ospf 1
    Router(config-router)# router-id 1.1.1.1
    ```

## Verifying OSPF operation

```bash
show ip ospf
show ip ospf neighbor
show ip ospf interface
show ip ospf database
```