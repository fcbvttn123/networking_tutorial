# Contents

- [Contents](#contents)
- [Classification](#classification)
  - [IGP](#igp)
  - [EGP](#egp)
- [Distance Vector vs Link State](#distance-vector-vs-link-state)
- [Administrative Distance \& Metric (`[120/3]`)](#administrative-distance--metric-1203)
  - [Admin Distance](#admin-distance)
  - [Metric (`hop` for RIP, `cost` for OSPF)](#metric-hop-for-rip-cost-for-ospf)

# Classification

## IGP

- Distance Vector: EIGRP

- Link State: OSPF

## EGP

- Path Vector

- Examples: EGP, BGPv4

# Distance Vector vs Link State

- Distance vector sees the network only from the perspective of direct neighbors

- Link state has a full, complete map of the entire network topology

# Administrative Distance & Metric (`[120/3]`)

## Admin Distance

- A lower AD is preferred over a higher AD (when the same destination is learned from multiple protocols)

- Examples: Connected (AD 0), Static Route (AD 1), OSPF (AD 110), EIGRP (AD 90)

## Metric (`hop` for RIP, `cost` for OSPF)

- A metric is used within the same routing protocol to determine the best path

- A lower metric is usually preferred

- Example

    - `10.10.10.0/24 via R2 cost 10` and `10.10.10.0/24 via R3 cost 20`

    - Both routes come from OSPF (same AD = 110)

    - OSPF compares the metric (cost) and chooses the route with cost 10