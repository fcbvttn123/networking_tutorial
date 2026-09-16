# Static NAT

- Permanent one-to-one mapping between a private IP and a specific public IP

- Often used for servers that must be reachable from the internet (web, mail, etc.)

- Port numbers are NOT translated (no PAT)

- Public access to internal hosts is allowed (inbound traffic)

- Requires one public IP per inside host

# Dynamic NAT

- Translates private addresses to a pool of public addresses

- Still one-to-one, but mapping occurs only when needed

- Mapping is released when idle or when pool is exhausted

- Still suffers from public IP exhaustion since 1:1 mapping

# PAT (NAT Overload)

- Many private IPs share one public IP

- Uses different TCP/UDP port numbers to differentiate flows

- Solves NAT exhaustion by allowing thousands of users to share one IP

- Translates IP + port (both change)

# Source NAT and Destination NAT

- Source NAT (`SNAT`) changes the source IP address of an outgoing packet

- Destination NAT (`DNAT`) changes the destination IP address and port of an incoming packet

    - Example: a data packet sent to your public IP address and a specific port (e.g., sending an HTTP request to `203.0.113.5:80`)

    - The router or firewall receives the packet on its public interface

    - The router looks at its DNAT rules and replaces the public destination IP and port with the private internal IP and port of your server (`192.168.1.50:80`)

    - When the private server replies, the router reverses the process—rewriting the source of the reply back to the public IP address

# Cisco NAT Order of Operations

- Check ACL / Policy Criteria: `access-list 1 permit <subnet>`

- Routing Decision: Determines the **outgoing interface** based on the destination IP

- NAT Translation: **Occurs only if the packet is actually exiting an interface** designated as `ip nat outside`

# Example Configuration

```bash
# define the Inside Interface (facing your local network)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit
# define the Outside Interface (facing the internet/ISP)
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
# create an Access Control List (ACL) to permit your local subnet
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
# enable PAT overloading the outside interface
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

- `NAT` only triggers when **a packet transitions** from an `ip nat inside` interface to an `ip nat outside` interface

- The packet arrives at interface `G0/0`

- Because this interface is marked ip nat inside, the router identifies it as traffic coming from the local network

- The router inspects the packet's Source IP (`192.168.1.50`). It checks `access-list 1`:

    - Does `192.168.1.50` match `192.168.1.0 /24`? Yes

    - Because it matches, the router flags this packet for translation

- The router looks up the destination IP in its routing table and determines that to reach that destination, the packet must be sent out through G0/1 (which is marked ip nat outside)