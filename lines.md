# Console Line

`R1(config)# line console 0`

- There is only **one single console line**, so the number is always 0

  - A **single console line** means **only one single console connection** at once


# VTY Lines

`R1(config)# line vty 0 15`

- Virtual TeleType

- Used for SSH and Telnet access

- 16 lines (0 to 15) => up to 16 users can be connected at once

- Use the `transport input` command for connection type

    - `R1(config-line)# transport input telnet`
    - `R1(config-line)# transport input ssh`
    - `R1(config-line)# transport input telnet ssh`
    - `R1(config-line)# transport input all`
    - `R1(config-line)# transport input none`


# Line Auth Method `login <method>`

- Configure the console/telnet line to use **a password** for auth (cannot use this for SSH, **only Console and Telnet**)

    ```bash
    R1(config)# line console 0
    R1(config-line)# password <your_password>
    R1(config-line)# login # auth using configured password
    ```

- Configure the line to use **an account of the device** for auth

    ```bash
    R1(config)# username <your_username> secret <password>
    R1(config)# line console 0
    R1(config-line)# login local
    ```


# Configure ACL for the line `access-class`

```bash
R1(config)# access-list 1 permit host 192.168.2.1
R1(config-line)# access-class 1 in
```