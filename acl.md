# Standard ACL

- Filter traffic based only on **source IP address**. Cannot match destination IP, ports, or protocols

- Applied **closest to the destination**

- **Named** Syntax

    ```bash
    ip access-list standard <NAME>
     permit|deny <source> <wildcard>
    ```

- **Numbered** Syntax

    ```bash
    access-list <1-99> permit|deny <source> <wildcard>
    ```

- Examples

    ```bash
    ip access-list standard BLOCK_LAN1
     deny 10.0.0.0 0.0.0.255
     permit any 

    access-list 10 deny 192.168.1.0 0.0.0.255
    access-list 10 permit any
    ```

- Applying a Standard ACL to an Interface

```bash
interface g0/0
   ip access-group 10 in

interface g0/0
   ip access-group BLOCK_LAN1 out
```


# Extended ACL

- It can filter: src IP, dest IP, protocols (IP, TCP, UDP, ICMP, etc.), port, TCP flags, ICMP types

- You place them closest to the source

- **Named** Syntax

    ```bash
    ip access-list extended <NAME>
     permit tcp 192.168.1.0 0.0.0.255 any eq 443
     deny ip any any 
    ```

- **Numbered** Syntax

    ```bash
    access-list <100-199> permit|deny <protocol> <source> <wildcard> <destination> <wildcard> [operator] [port]
    ```

- Examples

    ```bash
    ## Numbered: Allow web traffic (port 80) from LAN → anywhere
    access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80

    ## Numbered: Block pings (ICMP) to a server
    access-list 101 deny icmp any host 10.1.1.10

    ## Numbered: Permit DNS (UDP port 53) from anyone to one server
    access-list 110 permit udp any host 10.1.1.5 eq 53

    ## Numbered: Block all Telnet (port 23)
    access-list 120 deny tcp any any eq 23

    ## Named: 
    ip access-list extended <NAME>
     permit tcp 192.168.1.0 0.0.0.255 any eq 443
     deny ip any any
    ```