# Contents

- [Contents](#contents)
- [Directly Connected Network (`C`)](#directly-connected-network-c)
- [Static Route (`S`)](#static-route-s)
- [Default Route (`S*`)](#default-route-s)

# Directly Connected Network (`C`)

```bash
Router(config)# show ip route
C 192.168.2.0/24 is directly connected, Serial0/0/0
```

# Static Route (`S`)

```bash
Router(config)# ip route (network-address) (subnet-mask) (next-hop-ip-address / exit-interface)

Router(config)# show ip route
S 192.168.2.0/24 [1/0] via 192.168.1.2, FastEthernet0/0
```

# Default Route (`S*`)

```bash
Router(config)# ip route 0.0.0.0 0.0.0.0 (next-hop-ip-address / exit-interface)

Gateway of last resort is 192.168.1.1 to network 0.0.0.0
S* 0.0.0.0/0 [1/0] via 192.168.1.1
```