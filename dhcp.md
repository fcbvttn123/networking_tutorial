# DORA Process

## Discover (UDP port 68)

- Destination MAC: FFFFFFFF

- Source MAC: 08002B3EAF2A

- Source IP Address: 0.0.0.0

- Dest IP Address: 255.255.255.255

- Client Identifier: 08002B2EAF2A

## Offer (UDP port 67)

- Server IP Address: 172.16.32.12

- Dest MAC Addr: 08002B2EAF2A

- Destination IP: 255.255.255.255 

- Offered IP Address: 192.16.32.51 

- Server Identifier: 00AA00123456 

- Lease Time: 72 hours

## Request

- Dest MAC Address: FF:FF:FF:FF:FF:FF 

- Source IP Address: 0.0.0.0

- Dest IP Address: 255.255.255.255 

- Source MAC Addr: 08002B2EAF2A 

- Server Identifier: 172.16.32.12 

- Request IP Address: 172.16.32.51 

- Client Identifier: 08002B2EAF2A 

- Request paramenters........

## Acknowledgment

- Dest MAC Add: 08002B2EAF2A (client's MAC)

- Source MAC Addr: 00AA00123456 

- Source IP Address: 172.16.32.12 

- Dest IP Address: 255.255.255.255 

- IP Address: 172.16.32.51

- Server Identifier: 172.16.32.12 

- Lease Length: 72 Hours

- Client Identifier: 08002B2EAF2A 

- Other Request Paraments.....




# Security Concerns with DHCP

## DHCP Starvation

- Floods DHCP requests with spoofed MACs to consume all available leases, blocking legitimate clients

## Rogue DHCP Server

- A fake server gives clients incorrect IP settings (gateway/DNS), taking control of their network path

## Man-in-the-Middle

- Done by pushing attacker-controlled gateway/DNS via rogue DHCP to intercept or alter traffic

## DNS Misuse

- Malicious DHCP options can point clients to unauthorized DNS servers for redirection/phishing




# Protection Against DHCP Attacks

- Enable DHCP Snooping on switches and allow DHCP replies only on trusted ports

- Apply Port Security or rate-limiting to reduce spoofed requests and prevent starvation

- Use IP/MAC filtering + monitoring/log analysis to detect and block abnormal lease activity