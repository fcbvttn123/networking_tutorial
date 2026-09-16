# What it is

- Virtual Routing & Forwarding: doesn't work on L2 SWs, only routed interfaces and SVIs (MLS)

- VRF divides one physical router into multiple virtual routers each with their own routing tables (like VLANs for routers)

# How it works

- L3 Interfaces are configured to be in a specific VRF (known as **VRF Instance**)

- VRF is commonly used by ISP to allow one device to carry traffic from multiple customers

    - Each customer's traffic is isolated from the others

    - Customer IP addresses can overlap