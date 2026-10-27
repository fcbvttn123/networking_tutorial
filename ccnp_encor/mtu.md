# What MTU is

- The Maximum Transmission Unit is **the largest unit** a device can send/receive from an interface

- The unit is **usually an IP packet** (including the IP header)

- This **doesn't include the L2 header/trailer**

- 2 separate concepts:

    - **Ethernet MTU**: only the **L3 Header, L4 Header and the Data**

    - **IP MTU**: only the IP Header (we have MSS for L4 Header)

- The default MTU is **1500 bytes**


# Ethernet MTU (Interface MTU)

- The Ethernet MTU specifies the maximum payload size of frames **sent/received by an interface**

- This **includes L2 and L3 interfaces**, since both send Ethernet frames

- MTU is checked at **both ingress (receiving) and egress (sending)**

- If a frame's payload (L3 header + L4 header + Data) is larger than **the interface's MTU**, it will be dropped

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


# IP MTU


# System MTU


# MTU Example: GRE Tunnels


# Path MTU Discovery