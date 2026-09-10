# Contents

- [Contents](#contents)
- [What it is](#what-it-is)
- [How it works](#how-it-works)
- [RADIUS \& LDAP](#radius--ldap)
- [Common RADIUS Servers](#common-radius-servers)
- [Kerberos (Auth part)](#kerberos-auth-part)

# What it is

- A RADIUS Server is a network service that provides AAA

  - **Authentication** – Verifies who a user or device is

  - **Authorization** – Determines what resources they are allowed to access

  - **Accounting** – Records connection and usage information for auditing and reporting

# How it works

- When a user tries to connect to a network resource: AP (WPA2/WPA3 Enterprise), **VPN**, **Switch Port** (802.1X)

- The network device (called RADIUS client) sends the user's credentials to the **RADIUS server**

- Then, the RADIUS server forwards the authentication request to an **Auth Server** (Active Directory)

# RADIUS & LDAP

- Protocols used between RADIUS Clients and RADIUS Servers is **RADIUS - UDP Port 1812, 1813**

- Protocols used between RADIUS Servers and AD is **LDAP**

# Common RADIUS Servers

- Microsoft Network Policy Server (NPS)

- FreeRADIUS

- Cisco Identity Services Engine (ISE)

- Aruba ClearPass

- Juniper Access Management solutions

# Kerberos (Auth part)

- Kerberos is used mainly for Windows Logon, SSO (RADIUS is for Wi-Fi, VPN, 802.1X, network access)

- Kerberos is most commonly found in Microsoft Active Directory environments

- When a Windows user logs into their domain-joined computer:

    - User enters username and password

    - The workstation contacts a Domain Controller

    - The Domain Controller acts as a Key Distribution Center (KDC)

    - The KDC issues Kerberos tickets

    - The user can then access services (e.g., SMB File Shares) without repeatedly entering credentials