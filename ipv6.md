# Hexadecimal

- Binary / Base 2 (0, 1) / 0b

- Decimal / Base 10 (0 -> 9) / 0d

- Hexadecimal / Base 16 (0->9 A->F) / 0x

    - Each hexadecimal digit (0xD) = 4 bits (0b1101)


# Basics of IPv6

- 128 bits - using `/64`

- IPv6 Addr is written in hex - 32 hex digits (128 / 4 = 32) - 8 groups of 4 digits, delimiter by `:`

    `2001:0DB8:000A:001B:20A1:0020:0080:34BD`

- Shortening IPv6 Addr

    - Leading 0s can be removed: `2001:DB8:A:1B:20A1:20:80:34BD`

    - Consecutive quartets of all 0s can be replaced with `::`

        `2001:0DB8:0000:0000:0000:0000:0080:34BD`

        `2001:0DB8::0080:34BD`

    - Combine both methods: `2001:DB8::80:34BD`

    - Notes: consecutive quartets of 0s can only be abbriviated once

        `2001:0000:0000:0000:20A1:0000:0000:34BD` -> `2001::20A1:0:0:34BD`


# Finding the IPv6 network portion (global unicast addresses)

- Typically, ISP assigns companies a `/48` block and IPv6 subnets use a `/64` prefix length

  - That means an enterprise has **16 bits** to use to make subnets

  - The remaining **64 bits** can be used for hosts
  
  - Example: `2001:0DB8:8B00:0001:0000:0000:0000:0001 /64`

      - `2001:0DB8:8B00`: assigned by ISP (/48)

      - `0001`: used by enterprise to make subnets (/16)

      - `0000:0000:0000:0001`: addresses for hosts

- Finding prefix length (network portion) with multiple of 4s -> `/56`

    - Example: `300D:00F2:0B34:2100:0000:0000:1200:0001 /56`

    - `300D`: 16 bits, `00F2`: 32 bits, `2100`: 48 bits, `2`: 52 bits, `1`: 56 bits

    - Network Portion: `300D:F2:B34:2100:: /56`

- Finding prefix length (network portion) which isn't multiple of 4s -> `/93`

    - Example: `2001:0DB8:8B00:0001:FB89:017B:0020:0011 /93`

    - `2001`: 16 bits, `FB89`: 80 bits, `0`: 84 bits, `1`: 88 bits, `7`: 92 bits

    - `B` -> 0d11 -> 0b1011 -> `1` is the 93rd bit -> change the rest to 0s `1000` -> back to hex `0d8`

    - Network Portion: `2001:DB8:8B00:1:FB89:178::`


# Configure IPv6 Addresses