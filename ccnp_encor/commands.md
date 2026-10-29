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

# ethernet mtu
Router# show interfaces GigabitEthernet0/1
  GigabitEthernet0/1 is up, line protocol is up (connected)
  Hardware is CSR vNIC, address is 5000.0001.0000 (bia 5000.0001.0000)
  Internet address is 192.168.1.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec # ethernet mtu value
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip mtu 9000
# ip mtu
Router# show ip interfaces GigabitEthernet0/1
  GigabitEthernet0/1 is up, line protocol is up
  Internet address is 192.168.1.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes # ip mtu value
  Helper address is not set
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip mtu 9000
# system mtu
SW# show system mtu
  System MTU size is 1500 bytes # Eth MTU for 10Mbps/100Mbps interfaces
  System Jumbo MTU size is 1500 bytes # Eth MTU for 1Gbps/10Gbps interfaces
  System Alternate MTU size is 1500 bytes # A third value that can be assigned to particular interfaces
  Routing MTU size is 1500 bytes # IP MTU for routed port
SW(config)# system mtu 1600
SW(config)# system mtu jumbo 9000

# mss
R1(config)# ip tcp mss <bytes>
R1(config-if)# ip tcp adjust-mss <bytes>
R1# debug ip tcp adjust-mss
R1# debug ip tcp transactions
R1# show tcp