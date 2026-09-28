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
    ansible-galaxy collection install cisco.ios
    ```


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

- The group names `cisco_ios` and `arista_eos` don't automatically tell Ansible **which network OS** to use. They're just inventory group names

- The group names cisco_ios and arista_eos don't automatically tell Ansible which network OS to use. They're just inventory group names

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