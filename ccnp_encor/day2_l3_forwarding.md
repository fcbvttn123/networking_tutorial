# The `method` column of the command `show ip interfaces brief`

```bash
Switch# show ip interfaces brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  192.168.1.2     YES manual up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     10.10.10.1      YES manual up                    up      
GigabitEthernet0/3     unassigned      YES unset  administratively down down    
GigabitEthernet0/4     unassigned      YES unset  down                  down
```

- Method `manual`: the IP has been manually set since the last boot

- Method `unset`: the IP hasn't been configured, the router boots using the default config (not the `startup-config`)

    - In the first boot, the device doesn't have the `startup-config` file, that file only exists after the command `do write`

- After rebooting the device, all methods are changed to `NVRAM` - the IP config is learned from the `startup-config` in the `NVMRAM`


# Directed Broadcast

- A Broadcast IP Address is the last IP address of a subnet

- A Directed Broadcast is a packet which has the **Destination IP** as the last IP of the **other subnet**

- Example

    - Imagine your computer is on the network `192.168.1.0/24`

    - **Local Broadcast:** you send a packet to `192.168.1.255`

    - **Directed Broadcast:** you send a packet to `10.20.30.255` (DON'T specify the subnet mask)

        - The router accepts the packet, routes it across the network **just like a normal unicast packet**, and delivers it to the `10.20.30.0/24` network

        - Once it arrives, the local router on that subnet removes the L2 destination, and relize this address is the last address of the subnet
        
        - It broadcasts the packet out to every host (`10.20.30.1` through `10.20.30.254`)

- How routers handle directed broadcast packets

    - If the router is not the owner (**Transit Router**)

        - If a router receives a directed broadcast packet destined for a remote subnet it does not directly connect to, **it treats the packet like any other standard IP packet**

        - It looks up the destination IP in its routing table, finds the next-hop router, and forwards it out the appropriate interface

        - It doesn't broadcast it yet because it's not on the final network segment

    - If **the router owns the subnet** and Directed Broadcast is **disabled** (Default)

        - When the packet finally arrives at the router directly connected to that subnet, the router recognizes that the packet is a broadcast intended for its local network

        - Because `no ip directed-broadcast` is configured by default to prevent Smurf attacks, the router drops the packet immediately

- By default, directed broadcast **is disabled**

- Enable directed broadcast:

    ```bash
    interface GigabitEthernet0/0
     ip directed-broadcast 10
    ```