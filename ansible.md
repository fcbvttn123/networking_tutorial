# Table of contents

- [Table of contents](#table-of-contents)
- [Lab Setup (EVE-NG)](#lab-setup-eve-ng)
  - [Build the Topology](#build-the-topology)
  - [Bootstrap SSH on Cisco vIOS Switches](#bootstrap-ssh-on-cisco-vios-switches)
  - [Prepare the Ansible Control Node](#prepare-the-ansible-control-node)
- [Folders and Variables](#folders-and-variables)
  - [Folder Structure](#folder-structure)
  - [`group_vars`](#group_vars)
  - [`host_vars`](#host_vars)
  - [`inventory` File Example](#inventory-file-example)
  - [Use the variables in a playbook](#use-the-variables-in-a-playbook)
- [Inventory](#inventory)
  - [File Structure](#file-structure)
  - [Inventory Format (`inventory.yml`)](#inventory-format-inventoryyml)
  - [Group Variables (`group_vars/cisco_ios.yml`)](#group-variables-group_varscisco_iosyml)
  - [How Ansible Uses the Inventory in a Playbook](#how-ansible-uses-the-inventory-in-a-playbook)
- [Playbooks](#playbooks)
  - [A Complete Network Playbook Example](#a-complete-network-playbook-example)
  - [Structure of a Task](#structure-of-a-task)
  - [Key Components Explained](#key-components-explained)
  - [Declarative vs. Imperative Modules](#declarative-vs-imperative-modules)
  - [Running a Playbook](#running-a-playbook)
- [Modules](#modules)
  - [What is a module?](#what-is-a-module)
  - [What's inside the module?](#whats-inside-the-module)
  - [Where do these modules actually come from?](#where-do-these-modules-actually-come-from)


# Lab Setup (EVE-NG)

## Build the Topology

- Open your EVE-NG lab workspace

- Add 2x Cisco vIOS-L2 nodes (using your imported vios_l2 image)

- Add 1x Linux Node (e.g., Ubuntu/Debian image or lightweight Alpine container) to serve as your Ansible Control Node

- Add 1x Network Object set to Management(Cloud0) or an unmanaged internal switch bridge

- Connect Gi0/0 of both Cisco switches and the Linux node's interface (eth0) to the network object

## Bootstrap SSH on Cisco vIOS Switches

Console into each Cisco switch (SW1, SW2) and paste the initial configuration to enable SSH and local user authentication:

```bash
enable
configure terminal
hostname SW1
ip domain-name lab.local

! Create management account (privilege 15 avoids need for enable password handling)
username admin privilege 15 secret cisco123

! Enable management interface (e.g., Gi0/0)
interface GigabitEthernet0/0
 description Management
 ip address 192.168.1.11 255.255.255.0
 no shutdown
exit

! Generate SSH keys
crypto key generate rsa modulus 2048
ip ssh version 2

! Force SSH on VTY lines
line vty 0 15
 login local
 transport input ssh
end
write memory
```

Set SW2's IP address to `192.168.1.12`

## Prepare the Ansible Control Node

- Install Ansible and the Cisco IOS collection

- Log into your Linux control node console, set up static IP 192.168.1.100/24 on eth0, and run:

    ```bash
    # Update packages and install python/pip
    sudo apt update && sudo apt install -y python3-pip git

    # Install Ansible core and Cisco collection
    pip3 install ansible
    sudo apt install ansible
    ansible-galaxy collection install cisco.ios
    ```


# Folders and Variables

## Folder Structure

  ```bash
  ├── inventory.ini             <-- Host/Group level vars (quick/small setups)
  ├── group_vars/
  │   ├── all.yml               <-- Global defaults (NTP servers, DNS, Domain)
  │   └── switches.yml          <-- Group-specific (VLAN lists, syslog servers)
  ├── host_vars/
  │   ├── switch01.yml          <-- Device-specific (IP addresses, hostnames, AS numbers)
  │   └── router01.yml
  └── playbooks/
      └── site.yml              <-- Playbook/Task level vars (temporary/override)
  ```

## `group_vars`

- `group_vars` is `a special directory name` that Ansible recognizes automatically **to load variables associated with inventory groups**

- `group_vars/all.yml`: define global network parameters like NTP servers, DNS servers, AAA settings, or connection credentials

  ```yaml
  ---
  # Connection settings for Ansible
  ansible_connection: ansible.netcommon.network_cli
  ansible_user: admin
  ansible_password: "{{ vault_ansible_password }}" # Injected via Ansible Vault

  # Global Network Infrastructure Parameters
  dns_servers:
    - 1.1.1.1
    - 8.8.8.8

  ntp_servers:
    - 10.0.0.123
    - 10.0.0.124

  domain_name: example.lab
  ```

- `group_vars/<group_name>.yml`: must exactly match the group names defined in your inventory file

- Example: `group_vars/switches.yml`

  ```yaml
  ---
  ansible_connection: ansible.netcommon.network_cli
  ansible_network_os: cisco.ios.ios
  ansible_user: "{{ vault_ansible_user }}"
  ansible_password: "{{ vault_ansible_password }}"
  ```

## `host_vars`

- `host_vars/<hostname>.yml`: define individual device attributes like interface IP addresses, loopbacks, and BGP neighbor IPs

- `host_vars/<hostname>.yml`: must exactly match the host names defined in your inventory file

- The file **does not configure the router by itself**. A playbook must read these variables to apply the configuration

- Example: `host_vars/R1-CORE.yml`

  ```yaml
  ---
  # Device identity and metadata
  hostname: R1-CORE
  site: Toronto
  device_role: core_router
  environment: production

  # Desired configuration data
  loopback_interfaces:
    - name: Loopback0
      ipv4_address: 10.255.0.1
      ipv4_mask: 255.255.255.255

  interfaces:
    - name: GigabitEthernet1
      description: Uplink-to-R2-CORE
      ipv4_address: 10.0.12.1
      ipv4_mask: 255.255.255.252
      enabled: true

    - name: GigabitEthernet2
      description: Uplink-to-DIST-SW1
      ipv4_address: 10.0.21.1
      ipv4_mask: 255.255.255.252
      enabled: true

  # Routing configuration
  ospf:
    process_id: 10
    router_id: 10.255.0.1
    networks:
      - network: 10.0.12.0
        wildcard: 0.0.0.3
        area: 0
      - network: 10.0.21.0
        wildcard: 0.0.0.3
        area: 0
  ```

## `inventory` File Example

  ```yaml
  ---
  all:
    children:
      routers:
        hosts:
          R1-CORE:
            ansible_host: 192.0.2.11
          R2-CORE:
            ansible_host: 192.0.2.12
      switches:
        hosts:
          SW1-ACCESS:
            ansible_host: 192.0.2.21
          SW2-ACCESS:
            ansible_host: 192.0.2.22
  ```

## Use the variables in a playbook

- `playbooks/configure_interfaces.yml`

  ```yaml
  ---
  - name: Configure router interfaces
    hosts: routers
    gather_facts: false

    tasks:
      - name: Configure each interface
        cisco.ios.ios_config:
          parents: "interface {{ item.name }}"
          lines:
            - "description {{ item.description }}"
            - "ip address {{ item.ipv4_address }} {{ item.ipv4_mask }}"
            - "no shutdown"
        loop: "{{ interfaces }}"
  ```

- The target for the task is the line `hosts: routers`

- Run the playbook against `R1-CORE` only: `ansible-playbook -i inventory.yml playbooks/configure_interfaces.yml --limit R1-CORE`


# Inventory

## File Structure

```bash
project/
├── inventory.yml
├── group_vars/
│   ├── all.yml
│   ├── cisco_ios.yml
│   └── arista_eos.yml
└── site.yml
```

## Inventory Format (`inventory.yml`)

```yaml
all:
  children:
    cisco_ios:
      hosts:
        sw-access-01:
          ansible_host: 172.16.1.10
    arista_eos:
      hosts:
        sw-leaf-01:
          ansible_host: 172.16.2.10
```

- The group names `cisco_ios` and `arista_eos` don't automatically tell Ansible **which network OS** to use

- They're just inventory groups unless you add variables that configure the connection/platform

- For example, you will commonly see something like:

    ```yaml
    all:
      children:
        cisco_ios:
          hosts:
            sw-access-01:
              ansible_host: 172.16.1.10
          vars:
            ansible_network_os: cisco.ios.ios
        arista_eos:
          hosts:
            sw-leaf-01:
              ansible_host: 172.16.2.10
          vars:
            ansible_network_os: arista.eos.eos
    ```

## Group Variables (`group_vars/cisco_ios.yml`)

```yaml
ansible_connection: network_cli
ansible_network_os: cisco.ios.ios
ansible_user: netadmin
ansible_become: true
ansible_become_method: enable
```

## How Ansible Uses the Inventory in a Playbook

```yaml
- name: Configure Cisco IOS Switches
  hosts: cisco_ios
  gather_facts: false
  tasks:
    - name: Ensure domain name is configured
      cisco.ios.ios_config:
        lines:
          - ip domain name lab.local
```


# Playbooks

## A Complete Network Playbook Example

```yaml
---
- name: Network Device Baseline Configuration
  hosts: cisco_ios
  gather_facts: false

  vars:
    target_vlan_id: 100
    target_vlan_name: GUEST_WIFI

  tasks:
    - name: Gather running configuration from device
      cisco.ios.ios_facts:
        gather_subset:
          - config

    - name: Save configuration backup to local management machine
      ansible.builtin.copy:
        content: "{{ ansible_facts.net_config }}"
        dest: "./backups/{{ inventory_hostname }}_{{ ansible_date_time.date }}.cfg"

    - name: Ensure VLAN 100 exists
      cisco.ios.ios_vlans:
        config:
          - vlan_id: "{{ target_vlan_id }}"
            name: "{{ target_vlan_name }}"
        state: merged
```

## Structure of a Task

```bash
- name: TASK NAME
  MODULE:
    PARAMETER: VALUE
    PARAMETER: VALUE
```

## Key Components Explained

- Play Header

  - `name`: A readable description of what the play accomplishes

  - `hosts`: The group or specific host from your inventory file to target (e.g., `cisco_ios`, `all`, or `edge_routers`)

  - `gather_facts: false`: Standard Ansible attempts to gather Linux facts (like CPU architecture or disk usage) via Python

    - Network OS devices do not run Python, so you almost always set this to `false` and use network-specific modules (like `cisco.ios.ios_facts`) instead

- Variables (`vars` or `vars_files`)

  - Allow you to avoid hardcoding values inside tasks
  
  - You can define them directly in the playbook, in external variable files, or derive them from `Jinja2` templates

- Tasks (modules)

  - `cisco.ios.ios_facts`: Pulls operational data (hostname, software version, running config)

  - `ansible.builtin.copy`: Runs locally on your control node to save the config output into a local backup file

  - `cisco.ios.ios_vlans`: Configures VLANs using declarative data structures

## Declarative vs. Imperative Modules

- When writing tasks for network automation, you will see two styles of modules

- **Imperative** (Command-based): you give exact CLI commands to execute

  ```yaml
  - name: Run raw CLI commands
    cisco.ios.ios_command:
      commands:
        - show ip interface brief
        - show vlan brief
  ```

- **Declarative** (Resource-based — Recommended): you describe the desired end state, and Ansible determines what CLI commands need to be run to reach that state

  ```yaml
  - name: Configure interface description
    cisco.ios.ios_l2_interfaces:
      config:
        - name: GigabitEthernet0/1
          mode: trunk
          trunk:
            native_vlan: 1
      state: merged
  ```

## Running a Playbook

`ansible-playbook -i inventory.yml site.yml`


# Modules

## What is a module?

- A module is essentially a piece of Ansible functionality that performs a specific operation

- `ansible.builtin.copy:` is a module for copying files, `cisco.ios.ios_config:` is a module for managing Cisco IOS configuration

  ```bash
  ansible.builtin.copy
  │       │       │
  │       │       └── module
  │       └────────── collection
  └────────────────── namespace
  ```

- Other network examples include:

  ```bash
  cisco.ios.ios_command:
  cisco.ios.ios_config:
  arista.eos.eos_command:
  arista.eos.eos_config:
  ```

## What's inside the module?

```bash
- name: Configure VLAN
  cisco.ios.ios_config:
    lines:
      - vlan 100
      - name USERS
```

- Module: `cisco.ios.ios_config:`

- Parameters: passed to the module

    ```bash
    lines:
      - vlan 100
      - name USERS
    ```

- More examples

  ```bash
  # ios commands
  - name: Show interfaces
    cisco.ios.ios_command:
      commands:
        - show ip interface brief
        - show ip interface brief
        - show running-config

  # ios config
  - name: Configure interface
    cisco.ios.ios_config:
      parents:
        - interface GigabitEthernet1/0/1
      lines:
        - description USER-PC
        - switchport mode access
        - switchport access vlan 100

  # copy
  - name: Copy file
    ansible.builtin.copy:
      src: myfile.txt
      dest: /tmp/myfile.txt
  ```

## Where do these modules actually come from?

- Collections are packages containing Ansible content

- For example: `cisco.ios` is a collection

- It contains modules such as:

  ```bash
  cisco.ios
  │
  ├── ios_config
  ├── ios_command
  ├── ios_facts
  ├── ios_interfaces
  ├── ios_l2_interfaces
  ├── ios_l3_interfaces
  └── ...
  ```

- You install collections separately from Ansible itself `ansible-galaxy collection install cisco.ios`