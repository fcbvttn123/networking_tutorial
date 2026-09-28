# Lab Setup (EVE-NG)

## Build the Topology

- Open your EVE-NG lab workspace

- Add 2x Cisco vIOS-L2 nodes (using your imported vios_l2 image)

- Add 1x Linux Node (e.g., Ubuntu/Debian image or lightweight Alpine container) to serve as your Ansible Control Node

- Add 1x Network Object set to Management(Cloud0) or an unmanaged internal switch bridge

- Connect Gi0/0 of both Cisco switches and the Linux node's interface (eth0) to the network object