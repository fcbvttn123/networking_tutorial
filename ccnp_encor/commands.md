```bash

# mac address table
SW# show mac address-table
SW# show mac address-table aging-time
SW# clear mac address-table dynamic
SW(config)# mac address-table aging-time <time>
SW(config)# mac address-table static <mac-address> vlan <vlan_id> {interface <interface> | drop}

# IPv4 verification commands
SW# show ip interfaces brief
SW# show ip interface g0/0

# check which route the router selects for a destination
R1# show ip route 203.0.113.96
Routing entry for 203.0.113.0/24
  Known via "ospf 1", distance 110, metric 20, type intra-area
  Last updated from 192.168.1.1 on GigabitEthernet0/0, 00:14:22
  Routing Descriptor Blocks:
  * 192.168.1.1, from 203.0.113.1, via GigabitEthernet0/0
      Route metric is 0, traffic share count is 1

# arp
R1# show arp
R1# show arp <ip-address> detail
R1# clear arp
R1(config)# arp <ip-address> <mac-address> arpa
R1(config)# [no] ip arp proxy disable # proxy arp