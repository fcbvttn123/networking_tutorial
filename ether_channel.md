# PAgP (Cisco Proprietary)

- Desirable + Desirable

- Desirable + Auto

- Fails: Auto + Auto (neither side initiates negotiation)

# LACP (IEEE 802.3ad / 802.1ax)

- Active + Active

- Active + Passive

- Fails: Passive + Passive (neither side initiates negotiation)

# LACP Configuration

```bash
# optional: reset ports to factory default
SwitchA(config)# default interface range GigabitEthernet 1/0/1 - 2
Interface GigabitEthernet1/0/1 set to default configuration
Interface GigabitEthernet1/0/2 set to default configuration

# group the physical member ports into channel-group 1
SwitchA(config)# interface range GigabitEthernet1/0/1 - 2
SwitchA(config-if-range)# description LACP Uplink to SwitchB
SwitchA(config-if-range)# channel-group 1 mode active
SwitchA(config-if-range)# exit
# apply interface settings
SwitchA(config)# interface Port-channel 1
SwitchA(config-if)# description Trunk Channel to SwitchB
SwitchA(config-if)# switchport trunk encapsulation dot1q
SwitchA(config-if)# switchport mode trunk
SwitchA(config-if)# switchport trunk allowed vlan 10,20,30
SwitchA(config-if)# exit

# verification
SwitchA# show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, subinterface
        f - failed to allocate aggregator
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)       LACP        Gi1/0/1(P)   Gi1/0/2(P)
```