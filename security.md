# Table of contents

- [Table of contents](#table-of-contents)
- [Port Security](#port-security)
  - [What it is](#what-it-is)
  - [MAC Address Learning Methods](#mac-address-learning-methods)
  - [Violation Modes](#violation-modes)
  - [Configuration Walkthrough (Cisco IOS)](#configuration-walkthrough-cisco-ios)


# Port Security

## What it is

- A Layer 2 traffic-filtering feature on network switches

- It limits the MAC addresses of hosts allowed to access the port

## MAC Address Learning Methods

- Static: Manually configured MAC address(es) saved in the running configuration

- Dynamic: Discovered dynamically and stored in the MAC address table until reboot or aging timeout

- Sticky: Dynamically learned, but automatically added to the running-config. This persists across reboots if you save the configuration (`write memory`)

## Violation Modes

- Shutdown (Default)

    - Places the interface into the `err-disabled` state, shutting down all traffic immediately
    
    - Generates syslog messages and increments the violation counter

- Restrict

    - Drops packets from unknown MAC addresses, increments the violation counter, and generates a syslog/SNMP trap
    
    - Keeping the port active for authorized devices

- Protect: Silently drops unauthorized packets. It does not log messages or increment the violation counter

## Configuration Walkthrough (Cisco IOS)

```bash
# enable port security
Switch(config-if)# switchport port-security

# set maximum allowed MAC addresses
Switch(config-if)# switchport port-security maximum 2

# define MAC learning method
Switch(config-if)# switchport port-security mac-address sticky

# config the violation mode
Switch(config-if)# switchport port-security violation restrict

# verification
Switch# show port-security interface GigabitEthernet1/0/1
Switch# show port-security address

# manually recover an err-disabled port
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
# automatic recovery after a timeout period (e.g., 300 seconds)
Switch(config)# errdisable recovery cause psecure-violation
Switch(config)# errdisable recovery interval 300
```