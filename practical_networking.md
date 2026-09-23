# Contents

- [Contents](#contents)
- [Networking Devices](#networking-devices)
  - [Access Layer](#access-layer)
  - [Distribution Layer](#distribution-layer)
  - [Firewall](#firewall)
  - [Other Devices](#other-devices)
- [Access/Distribution Layer Design](#accessdistribution-layer-design)
  - [Cisco StackWise vs Cisco StackWise Virtual](#cisco-stackwise-vs-cisco-stackwise-virtual)
  - [STP in Stacking Technology](#stp-in-stacking-technology)
- [Firewall Design](#firewall-design)
  - [HA with Direction Connection](#ha-with-direction-connection)
  - [Connect to Distribution Layer](#connect-to-distribution-layer)
  - [Transit VLAN and IP](#transit-vlan-and-ip)
  - [FireCluster Active/Passive Setup](#firecluster-activepassive-setup)
  - [Connect to ISP](#connect-to-isp)
- [Switch Licensing](#switch-licensing)
  - [Licensing Type](#licensing-type)
  - [What Happens When the License Expires?](#what-happens-when-the-license-expires)
  - [Smart Licensing](#smart-licensing)
  - [Deploy Smart Licensing using Direct Cloud Access](#deploy-smart-licensing-using-direct-cloud-access)


# Networking Devices

## Access Layer

- 4x `C9300-48P-E`

    - 48 × 1G access ports

    - 2 dedicated rear-panel stacking ports

    - PoE+

    - StackWise-480 support

    - Dual power supplies

- Uplink Module Options: 2x `C9300-NM-2Y (2 x 25GE/10GE/1GE SFP28)`

    - Port Speeds: The `C9300-NM-2Y` features **two SFP28 slots** supporting 25GE, 10GE, and 1GE

    - Uplink Cable: 4x `25G SFP28 DAC Cable`

- StackWise-480 support

    - Software License: **Network Essentials**

    - Stacking Cable: **STACK-T1-xxCM**

## Distribution Layer

- 2x `C9300X-12Y-A`

    - 12 × SFP28 ports

- Downlink Cable: 4x `Cisco SFP-H25G-CU1M` 25G SFP28 DAC Cable 1m

- StackWise-1T

## Firewall

- WatchGuard Firebox M390

    - Firewall Throughput: Up to 18 Gbps

    - UTM Throughput: Up to 2.4 Gbps (with security services enabled)

    - Interfaces: 8 x Gigabit Ethernet ports standard, with 1 expansion slot for optional modules (such as 1 Gb copper, SFP, or SFP+ fiber slots)

    - Memory & Storage: 8 GB RAM and 64 GB SSD

- The Firebox M390 operates via flexible subscription bundles that turn the appliance into a comprehensive **Unified Threat Management** (UTM) solution

    - **Basic Security Suite** license: Includes Intrusion Prevention Service (IPS), Gateway AntiVirus, WebBlocker, Application Control, spamBlocker, and standard support

    - **Total Security Suite**: Adds advanced protection features like cloud sandboxing (APT Blocker), DNSWatch, and ThreatSync XDR capabilities

## Other Devices

- IP phones

- Security cameras

- Multiple wireless APs


# Access/Distribution Layer Design

## Cisco StackWise vs Cisco StackWise Virtual

- Cisco StackWise uses **dedicated physical cables** to stack up to nine switches into one logical unit

    - Scale: Supports up to 8 to 9 physical switches in a single stack group

    - Connections: Uses proprietary, dedicated stacking cables and modules plugged into the back of the switches

- StackWise Virtual uses standard network interfaces to combine exactly **two high-end chassis** over short or long distances

    - Connections: Uses standard front-panel network ports (10G, 40G, or 100G) via fiber or DAC cables to form a StackWise Virtual Link (SVL)

    - Scale: Strictly limited to a maximum of two member switches or chassis

## STP in Stacking Technology

- No L2 Loop problem

- But still need STP (**Rapid-PVST+, BPDU Guard, Root Guard, Loop Guard**) because someone can still create a loop by

    - plugging two wall jacks together

    - connecting a cheap unmanaged switch

    - creating a loop through a wireless bridge


# Firewall Design

## HA with Direction Connection

- With ~200 users, the goal is **high availability**, not firewall load balancing

- Active/Passive is simpler, easier to troubleshoot, and is the most common WatchGuard deployment

- WatchGuard's own documentation notes that Active/Passive provides HA while one unit remains standby until a failover occurs

- WatchGuard strongly recommends **direct connections** rather than placing switches between the cluster interfaces

## Connect to Distribution Layer

- Firebox 1

    ```bash
    eth0 -> Cluster Link
    eth1 -> Backup Cluster Link

    eth2 -> Dist1
    eth3 -> Dist2
    ```

- Firebox 2

    ```bash
    eth0 -> Cluster Link
    eth1 -> Backup Cluster Link

    eth2 -> Dist1
    eth3 -> Dist2
    ```

- Firebox LACP: `eth2 + eth3` on each firewall become an LACP interface

## Transit VLAN and IP

- The Firebox Cluster has a VIP `172.16.255.1 /29` - 2 firebox share the same VIP

- The Distribution Layer has SVI 999 `172.16.255.2 /29` - SVI, not routed port 

- All ports of the distribution layer and the firebox are access port `VLAN 999`

## FireCluster Active/Passive Setup

- Requirements

    - Total Security Suite (TSS) on the primary unit

    - Standard Support on the secondary unit (if you want the most cost-effective HA deployment)

    - Same Fireware version on both appliances

    - Two dedicated cluster links between the firewalls

- Cluster links

    ```bash
    M390-A eth0 <----> M390-B eth0
    M390-A eth1 <----> M390-B eth1
    ```

- Configuration

    - Open `Policy Manager`

    - Then `FireCluster` → `Setup`

    - Select `Active/Passive`

## Connect to ISP

- **Each firebox has 1 port** connected to the **Access Switch** of the ISP


# Switch Licensing

## Licensing Type

- The `-E` in the model name indicates **Network Essentials licensing**

- The `-A` in the model name indicates **Network Advantage licensing**

## What Happens When the License Expires?

- The Catalyst 9300 has:

    - A perpetual Network license (Essentials or Advantage)

    - A term subscription (Catalyst/DNA subscription)

- The perpetual network license remains tied to the hardware and does not expire

- The subscription is purchased for a term (typically 3, 5, or 7 years)

## Smart Licensing

- When you buy the switches, Cisco expects them to be associated with a Smart Account for license management

- This is how Cisco tracks entitlements and subscriptions

- When a switch fails and Cisco sends a replacement unit via RMA

    - Smart Licensing allows you to easily transfer the software license entitlement from the dead serial number to the new serial number

    - You can do it in your online portal without contacting Cisco licensing support

## Deploy Smart Licensing using Direct Cloud Access

- Enable your switch management VLAN access to HTTPS internet (`smartreceiver.cisco.com`)

- You generate a Token inside your Cisco Smart Software Manager (CSSM) portal

- You paste a couple of CLI commands on the switch to bind it to your token

- The switch automatically checks in periodically over port 443