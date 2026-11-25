# What it is

- By default a router uses **a single global routing table** that contains all the **directly connected networks and prefixes** that it learned through **static or dynamic routing protocols**

- VRFs are like VLANs for routers, instead of using a single global routing table we can use **multiple virtual routing tables**

- Each interface of the router can be assigned to **a different VRF**

- VRFs are **commonly used for MPLS deployments**, when we use VRFs without MPLS then we call it **VRF lite**


# Configuration

```bash
# create VRFs (2 customers Red and Blue)
ISP(config)#ip vrf Red
ISP(config-vrf)#exit
ISP(config)#ip vrf Blue
ISP(config-vrf)#exit

# add the interfaces of the router to the correct VRF (Blue)
ISP(config)#interface FastEthernet 0/0
ISP(config-if)#ip vrf forwarding Blue
% Interface FastEthernet0/0 IP address 192.168.1.254 removed due to enabling VRF Blue
ISP(config-if)#ip address 192.168.1.254 255.255.255.0
ISP(config)#interface FastEthernet 2/0
ISP(config-if)#ip vrf forwarding Blue
ISP(config-if)#ip address 192.168.3.254 255.255.255.0
# add the interfaces of the router to the correct VRF (Red)
ISP(config)#interface FastEthernet 1/0
ISP(config-if)#ip vrf forwarding Red
ISP(config-if)#ip address 192.168.2.254 255.255.255.0
ISP(config)#interface FastEthernet 3/0
ISP(config-if)#ip vrf forwarding Red
ISP(config-if)#ip address 192.168.4.254 255.255.255.0

# check all the VRFs and their interfaces
ISP#show ip vrf
  Name                             Default RD          Interfaces
  Blue                                                 Fa0/0
                                                       Fa2/0
  Red                                                  Fa1/0
                                                       Fa3/0
# the global routing table of the ISP router: no entries, because all interfaces were added to a VRF
ISP#show ip route connected
# the VRF routing tables of the ISP router: each VRF has its own routing table with the interfaces
ISP#show ip route vrf Blue connected
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.3.0/24 is directly connected, FastEthernet2/0
ISP#show ip route vrf Red connected
C    192.168.4.0/24 is directly connected, FastEthernet3/0
C    192.168.2.0/24 is directly connected, FastEthernet1/0

# if we want to do something on the router like sending a ping then we have to specify which VRF we want to use
ISP#ping vrf Blue 192.168.1.1
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

# if we want to configure a static route we also have to specify the correct VRF
ISP(config)#ip route vrf Blue 1.1.1.1 255.255.255.255 192.168.1.1

# ospf with vrf
ISP(config)#router ospf 1 vrf Blue
ISP(config-router)#network 192.168.1.0 0.0.0.255 area 0
ISP(config-router)#network 192.168.3.0 0.0.0.255 area 0
ISP(config)#router ospf 2 vrf Red
ISP(config-router)#network 192.168.2.0 0.0.0.255 area 0
ISP(config-router)#network 192.168.4.0 0.0.0.255 area 0
# verification
ISP#show ip route vrf Blue ospf
Routing Table: Blue
     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/2] via 192.168.1.1, 00:00:24, FastEthernet0/0
     3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/2] via 192.168.3.3, 00:00:24, FastEthernet2/0