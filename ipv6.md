# Hexadecimal

- Binary / Base 2 (0, 1) / 0b

- Decimal / Base 10 (0 -> 9) / 0d

- Hexadecimal / Base 16 (0->9 A->F) / 0x

    - Each hexadecimal digit (0xD) = 4 bits (0b1101)


# Basics of IPv6

- 128 bits - using `/64`

- IPv6 Addr is written in hex - 32 hex digits (128 / 4 = 32) - 8 groups of 4 digits, delimiter by `:`

    `2001:0DB8:000A:001B:20A1:0020:0080:34BD`

- Shortening IPv6 Addr

    - Leading 0s can be removed: `2001:DB8:A:1B:20A1:20:80:34BD`

    - Consecutive quartets of all 0s can be replaced with `::`

        `2001:0DB8:0000:0000:0000:0000:0080:34BD`

        `2001:0DB8::0080:34BD`

    - Combine both methods: `2001:DB8::80:34BD`

    - Notes: consecutive quartets of 0s can only be abbriviated once

        `2001:0000:0000:0000:20A1:0000:0000:34BD` -> `2001::20A1:0:0:34BD`


# Finding the IPv6 network portion (global unicast addresses)

- Typically, ISP assigns companies a `/48` block and IPv6 subnets use a `/64` prefix length

  - That means an enterprise has **16 bits** to use to make subnets

  - The remaining **64 bits** can be used for hosts
  
  - Example: `2001:0DB8:8B00:0001:0000:0000:0000:0001 /64`

      - `2001:0DB8:8B00`: assigned by ISP (/48)

      - `0001`: used by enterprise to make subnets (/16)

      - `0000:0000:0000:0001`: addresses for hosts

- Finding prefix length (network portion) with multiple of 4s -> `/56`

    - Example: `300D:00F2:0B34:2100:0000:0000:1200:0001 /56`

    - `300D`: 16 bits, `00F2`: 32 bits, `0B34`: 48 bits, `2`: 52 bits, `1`: 56 bits

    - Network Portion: `300D:F2:B34:2100:: /56`

- Finding prefix length (network portion) which isn't multiple of 4s -> `/93`

    - Example: `2001:0DB8:8B00:0001:FB89:017B:0020:0011 /93`

    - `2001`: 16 bits, `FB89`: 80 bits, `0`: 84 bits, `1`: 88 bits, `7`: 92 bits

    - `B` -> 0d11 -> 0b1011 -> `1` is the 93rd bit -> change the rest to 0s `1000` -> back to hex `0d8`

    - Network Portion: `2001:DB8:8B00:1:FB89:178::`


# Configure IPv6 Addresses

```bash
# allow router to perform IPv6 routing
R1(config)# ipv6 unicast routing
# configure interface
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:0:0::1 /64
R1(config-if)# no shutdown
# verification
R1# show ipv6 interface brief
```


# IPv6 Addresses (EUI-64)

## EUI-64 Interface Identifier

- EUI stands for Extended Unique Identifier

- (Modified) EUI-64 is a method of converting a MAC address (48 bits) into **a 64-bit interface identifier**

- This interface identifier can then become the **host portion** of a `/64` IPv6 address

- **The Router/PCs does the conversion automatically**

- Instead of manually assigning the entire IPv6 address to every host

    - a router can **advertise** (Router Advertisement) a `/64` prefix
    
    - and hosts can automatically **generate their own interface ID**, potentially using `EUI-64`

## How to convert the MAC Address

- Divide the MAC address in half

    `1234 5678 90AB` -> `123456` | `7890AB`

- Insert FFFE in the middle

    `1234 56FF FE78 90AB`

- Invert the 7th bit: 0 to 1 or 1 to 0

    The 7th bit is inside the hex `2` which is `0010` in binary

    The 7th bit is `1` which is inverted to `0` -> `0000`

    `1034 56FF FE78 90AB`

## Configuration

```bash
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:: /64 eui-64
```


# Address Types

## Global Unicast Addresses

- They are public addresses: `2000::/3`

- Allocation Structure:

    - `/32` to `/48`: Regional Internet Registries (RIRs) give these large blocks to ISP and enterprise networks

    - `/56` or `/48`: ISPs typically assign these sizes to subscriber sites or homes

    - `/64`: This standard size is used for an individual local subnet or network segment

- No NAT Needed: IPv6 assigns globally unique public addresses **directly to individual devices**

    - With IPv6, your ISP can give your network an IPv6 prefix: `2001:db8:1234:5678::/64`

    - Your devices can then have their own globally routable addresses

        ```bash
        PC1      2001:db8:1234:5678::10
        Laptop   2001:db8:1234:5678::20
        Phone    2001:db8:1234:5678::30
        Server   2001:db8:1234:5678::40
        ```

    - Doesn't that make IPv6 devices exposed to the Internet?

        - Having a globally routable address does not mean the Internet can automatically connect to my computer

        - Your IPv6 router/firewall normally has firewall rules controlling inbound traffic

## Unique Local Addresses

- They are private addresses: `fd00::/8`

- The **first 2 hex digits** are always `FD`

- Components: `FD45:93AC:8A8F:0001:0000:0000:0000:0001`

    - `FD` means it is a private address

    - `45:93AC:8A8F`: 40 bits of the address form a pseudo-random string to guarantee uniqueness

    - `0001`: 16 bits to split the network into subnets

    - `Interface ID`: 64 bits identify the specific device on the network link

- Why do we need ULA if we have GUA?

    - **ULA doesn't exist because IPv6 ran out of addresses like in IPv4**

        - IPv6 has an enormous address space. ULA exists because private/internal addressing is still useful as a network-design concept

    - You could use GUA everywhere: `users - 2001:db8:1234:10::/64`, `servers - 2001:db8:1234:20::/64`

        - But perhaps you also have an internal management network: `fd12:3456:789a:30::/64`

        - You don't necessarily want that management network to be **globally routable**

- Why not just use GUA for everything?

    - You can use GUA for internal devices

    - But consider what happens if your ISP changes your IPv6 prefix

## Link Local Addresses

- It is a unicast IP address, used exclusively for communication between nodes on the same local network segment (link)

- Packets sent using a LLA are non-routable: routers will never forward LLA traffic outside the local network segment

- Mandatory Status: Every active IPv6-enabled interface automatically generates a Link-Local address

    ```bash
    PC1      fe80::1a2b:3c4d:5e6f:7a8b
    PC2      fe80::9abc:def0:1234:5678
    Router   fe80::1111:2222:3333:4444
    ```

- Scope: valid only within the single physical or VLAN broadcast domain

- Address Prefix

    - Always begins with `fe80::/10`
    
    - In practice, because the next 54 bits are filled with zeros, the address almost always starts with `fe80::/64`

- Why we need them: **many IPv6 functions rely on link-local communication**

    - Neighbor Discovery: IPv6 replaced ARP with **Neighbor Discovery Protocol (NDP)**, NDP messages use link-local communication

    - Router Discovery: helps a host learns IPv6 and default gateway using **Router Advertisement** (RA)

    - Routing Protocols: OSPFv3 neighbors often form adjacencies using these link-local addresses

## Multicast Addresses

- IPv6 doesn't use broadcast, we have to use some multicast addresses to send to all devices

- Range: `FF00:: /8`

## Anycast Addresses

- In IPv6, anycast is an address type where **the same IPv6 address** is assigned to multiple devices

- Packets sent to that address are delivered to the nearest device (according to the routing protocol's view of the network)

- How Anycast Works

    - Imagine three DNS servers in different cities: Toronto, New York and London

    - All three servers are configured with the same IPv6 anycast address: `2001:db8:53::53`

    - When a client sends a packet to: `2001:db8:53::53`

    - The routers determine which server is closest according to the routing table

- IPv6 does not have a separate address format that identifies an address as anycast