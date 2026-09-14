# SNMP Components

## SNMP Manager

- A management system (like SolarWinds, PRTG, Zabbix, Nagios) that polls devices

## SNMP Agent

- Software running on the network device that holds the information

- All enterprise‑grade networking devices (Cisco, Juniper, Aruba, Fortinet, etc.) come with an SNMP agent built in by default

## MIB

- Management Information Base

- A structured database of what can be monitored

## SNMP Messages

- `GET`: request information

    - **Application Layer** Protocol - **UDP** ports 161 and 162

    - The monitoring system sends SNMP GET requests

    - The device responds with: numerical values, status codes

    - Because these values come from the MIB, the system knows what they mean

- `TRAP/INFORM`: device-initiated alerts

    - A fiber uplink goes down between floors

    - The switch immediately sends an **SNMP TRAP** to the monitoring system

    - Trap: `LinkDown on interface GigabitEthernet1/0/24`

    - The monitoring system sends: email alert, SMS alert, Teams/Slack notification

- `SET`: modify configuration (rarely used for security reasons)


# SNMP Versions

## SNMPv1

- Oldest, simple, weak security

## SNMPv2c

- Weak security (clear-text community strings)

- Configuration

    ```bash
    # Step 1: Configure Read-Only (RO) and Read-Write (RW) Community Strings
    Router(config)# snmp-server community MyRoSecret123 RO 10
    Router(config)# snmp-server community MyRwSecret456 RW 10
    # Step 2: Define an Access Control List (ACL) to restrict NMS management IPs
    Router(config)# access-list 10 permit 192.168.1.50
    Router(config)# access-list 10 permit 192.168.1.51
    Router(config)# access-list 10 deny any log
    # Step 3: Define Switch Location and Contact Info (Optional metadata)
    Router(config)# snmp-server location DataCenter-Rack-04
    Router(config)# snmp-server contact Admin-Team <admin@company.com>
    # Step 4: Enable SNMP Traps/Informs
    Router(config)# snmp-server enable traps
    # Step 5: Specify the SNMP Server (NMS) Receiver IP and Community String
    Router(config)# snmp-server host 192.168.1.50 version 2c MyRoSecret123
    ```

- `snmp-server community MyRoSecret123 RO 10`

    - Community String is used by NMS to query or monitor switch metrics

    - `10`: bind this community string to Standard Access List 10

- `access-list 10 permit 192.168.1.50`

    - Restrict SNMP traffic so only authorized monitoring servers (e.g., 192.168.1.50) can talk to the SNMP agent

- `snmp server {location | contact}`

    - Set system MIB variables (sysLocation and sysContact)

    - This metadata appears inside monitoring tools like PRTG, SolarWinds, or Zabbix to **help engineers identify physical device** location and responsible teams

- `snmp-server enable traps`

    - Instructs the router to send alerts (Traps) to the NMS when critical events occur, rather than waiting for the NMS to poll it

- `snmp-server host 192.168.1.50 version 2c MyRoSecret123`

    - Tells the router where to send those generated Trap messages

## SNMPv3

- Most secure version

- Support: Auth, Encryption, User-based security