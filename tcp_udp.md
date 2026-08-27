# Packet Fragmentation

## UDP (reassembly happens at the **network** layer)

- Reassembly happens entirely at the IPv4 / IPv6 network layer (Layer 3)

- The receiving operating system uses four specific fields in the **IPv4 packet header** to piece the fragments back together

    - ID (16 bits): A unique ID assigned to the original packet. All fragments from the same original packet share this exact ID

    - Flags (MF - More Fragments - 1 bit): Set to `1` on all fragments except the last one (which is set to `0` to signal the end)

    - Fragment Offset (13 bits): Indicates the position of this fragment's payload relative to the original

    - Protocol / Next Header (8 bits): Identifies the transport protocol (e.g., 17 for UDP) so the receiver knows which protocol handler gets the reassembled payload

## TCP (reassembly happens at the **transport** layer)

- When a TCP connection is established during the 3-way handshake, both sides negotiate the MSS

- MSS is the maximum amount of **TCP Payload** (the data passed to Layer 4) a host can accept in a single packet (typically 1460 bytes for standard Ethernet)

- If an app hands 5,000 bytes to the TCP Socket, TCP splits it into segments based on the agreed MSS

    - Segment 1: Bytes 1 to 1460 (`Seq = 1`)

    - Segment 2: Bytes 1461 to 2920 (`Seq = 1461`)

    - Segment 3: Bytes 2921 to 4380 (`Seq = 2921`)

    - Segment 4: Bytes 4381 to 5000 (`Seq = 4381`)

- Receiver Reassembly

    - The receiver places incoming bytes into its TCP Receive Buffer sorted by `Sequence Number`

    - If Segment 3 arrives before Segment 2, TCP holds Segment 3 in buffer until Segment 2 arrives