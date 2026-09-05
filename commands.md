```bash

# routing
Router(config)# show ip route
Router(config)# ip route (network-address) (subnet-mask) (next-hop-ip-address / exit-interface)

# configure loopback interface for the router ID of the OSPF domain
Router(config)# interface loopback 0
Router(config-if)# ip address 1.1.1.1 255.255.255.255
Router(config-if)# exit
# ospf
R1(config)# router ospf <process_id> # enable OSPF
R1(config-router)# router-id <unique_ip_address> # configure router ID for the router
R1(config-router)# network <network_address> <wildcard_mask> area <area_ID> # advertise directly connected routes of the router
R1(config-router)# auto-cost reference-bandwidth 10000 # OSPF cost
R1(config-router)# passive-interface f1/0 # passive interface
R1(config-router)# default-information originate # redistributing the default route
# ospf verification
show ip ospf
show ip ospf neighbor
show ip ospf interface
show ip ospf database

# stp: specify root and secondary bridge
SW(config)# spanning-tree vlan <VLAN_ID> root <primary | secondary>
# stp: configure portfast
SW(config)# interface <interface_name>
SW(config-if)# switchport mode access
SW(config-if)# spanning-tree portfast
# verification commands
show spanning-tree
show spanning-tree summary
show spanning-tree <VLAN_ID>
show spanning-tree active
show spanning-tree detail