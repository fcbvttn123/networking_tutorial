# Ethernet MTU (Interface MTU) - 1500 bytes

- It is the maximum **Payload Size** an Ethernet Frame can carry, **excluding the Ethernet Header and FCS**

    - Ethernet Frame = Ethernet Header (14 bytes) + Ethernet Payload (1500 bytes) + FCS (4 bytes)

    - IP Packet (Ethernet Payload) = IP Header (20 bytes) + IP Payload (1480 bytes)

    - IP Payload = TCP Header (20 bytes) + App Data (1460 bytes)

- The default **Ethernet MTU** is 1500 bytes

- Check MTU of an interface

    ```bash
    Router# show interfaces GigabitEthernet0/1
    GigabitEthernet0/1 is up, line protocol is up (connected)
    Hardware is CSR vNIC, address is 5000.0001.0000 (bia 5000.0001.0000)
    Internet address is 192.168.1.1/24
    MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
    ```

- Change MTU for an interface

    ```bash
    Router(config)# interface GigabitEthernet0/1
    Router(config-if)# ip mtu 9000
    ```


# IP MTU - 1500 bytes

- It is the maximum size of an **IP Packet** (Ethernet Payload) that can be sent over a particular L3 interface (**default 1500 bytes**)

    - If the DF-bit is not set, packets larger than the IP MTU are fragmented

    - If the DF-bit is set, packets larger than the IP MTU are dropped

- IP MTU **only applies to Layer 3 ports**; Layer 2 ports (on a switch) are not L3-aware

- **IPv4 Header Fields** involved in the IP Fragmentation

    - **Identification** Field (16 bits): fragments have the same ID

    - **Flags** Field (3 bits)
    
        - bit 0 is always 0
        
        - bit 1 is the DF bit
        
        - bit 2 is for more-fragments - identify the last fragment of the packet (every fragments, except the last one, have this bit set)

    - **Fragment Offset** Field: identify the position of the fragment of the original packet

- With an Ethernet MTU of 1600 and IP MTU of 1500, the largest IP packet R1 can send without fragmentation is 1500 bytes - **the IP MTU takes precedence**

- The Ethernet MTU and the IP MTU should be the same, if you increase the size of the Ethernet MTU, it will increase the size of the IP MTU

- Check IP MTU of an interface

    ```bash
    Router# show ip interfaces GigabitEthernet0/1
    GigabitEthernet0/1 is up, line protocol is up
    Internet address is 192.168.1.1/24
    Broadcast address is 255.255.255.255
    Address determined by setup command
    MTU is 1500 bytes
    Helper address is not set
    ```

- Change IP MTU for an interface

    ```bash
    Router(config)# interface GigabitEthernet0/1
    Router(config-if)# ip mtu 9000
    ```


# System MTU

- A System MTU is a global configuration setting on a network device that defines the default maximum packet size allowed across all supported interfaces

- The MTU, set on the interface, takes the precedence

- Not all system supports System MTU

- Configuration

```bash
# show system mtu
SW# show system mtu
System MTU size is 1500 bytes # Eth MTU for 10Mbps/100Mbps interfaces
System Jumbo MTU size is 1500 bytes # Eth MTU for 1Gbps/10Gbps interfaces
System Alternate MTU size is 1500 bytes # A third value that can be assigned to particular interfaces
Routing MTU size is 1500 bytes # IP MTU for routed port

# set system mtu
SW(config)# system mtu 1600
SW(config)# system mtu jumbo 9000
```


# MTU Example: GRE Tunnels


# Path MTU Discovery