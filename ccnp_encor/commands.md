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