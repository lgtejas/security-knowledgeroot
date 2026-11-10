Layer 2 isn’t pure hardware. Ethernet frames are built by **software and hardware working together**: the operating system writes most of the frame, and the network card finishes it and sends it.

**How a frame is built when a computer sends data:**

1. **Application:** a program (a browser, an email client) hands its data to the operating system.
2. **OS network stack (software):**
    - Adds the transport header (TCP or UDP) and the IP header, creating an IP packet.
    - Finds the MAC address of the next device. On the same network, that’s the destination itself; otherwise, it’s the router. It uses ARP for this, asking “who has this IP address?” if it doesn’t already know.
    - Wraps the packet in an Ethernet header: destination MAC, source MAC, and a type field saying what’s inside (for example IPv4).
3. **Network driver (software):** places the frame in memory and tells the network card it’s ready.
4. **Network card, MAC controller (hardware):** reads the frame from memory, adds the preamble that marks the start of a frame, pads very short frames to the minimum size, and calculates a checksum (FCS) so the receiver can detect damage.
5. **Network card, PHY (hardware):** converts the bits into electrical signals on a copper cable, light pulses on fibre, or radio waves for Wi-Fi.

Receiving works in reverse: the hardware decodes the signal, checks the checksum and drops damaged frames or frames meant for other devices, and the OS unwraps the rest and passes the data up to the right program.

**Who creates each part of an Ethernet frame:**

| Field                               | Size          | Created by                                         |
| ----------------------------------- | ------------- | -------------------------------------------------- |
| Preamble + start-of-frame delimiter | 8 bytes       | Network card hardware                              |
| Destination MAC address             | 6 bytes       | Operating system (via ARP)                         |
| Source MAC address                  | 6 bytes       | Operating system / driver (the card’s own address) |
| VLAN tag (optional)                 | 4 bytes       | Operating system or a network switch               |
| Type (EtherType)                    | 2 bytes       | Operating system                                   |
| Payload (usually an IP packet)      | 46–1500 bytes | Operating system network stack                     |
| Frame check sequence (checksum)     | 4 bytes       | Network card hardware                              |


An Ethernet frame is a sequence of bytes, and the diagram shows how it’s laid out. The dashed VLAN tag only appears on networks that use VLANs.
![[ethernet-frame.png]]
**A real example in hexadecimal:** here’s a complete frame for a computer (192.168.1.10) pinging its router (192.168.1.1), as a packet-capture tool like Wireshark shows it. Capture tools usually hide the preamble and SFD, since the card strips them on arrival.

```
0000  00 1a 2b 3c 4d 5e 08 00 27 ab cd ef 08 00 45 00
0010  00 3c 1c 46 00 00 80 01 9b 1f c0 a8 01 0a c0 a8
0020  01 01 08 00 4d 5a 00 01 00 01 61 62 63 64 65 66
0030  67 68 69 6a 6b 6c 6d 6e 6f 70 71 72 73 74 75 76
0040  77 61 62 63 64 65 66 67 68 69 bb cb ff 7c
```

Reading it piece by piece:

|Bytes|Field|Meaning|
|---|---|---|
|`00 1a 2b 3c 4d 5e`|Destination MAC|The router’s network card|
|`08 00 27 ab cd ef`|Source MAC|The sending computer’s network card|
|`08 00`|Type|“The payload is an IPv4 packet”|
|`45 00 … c0 a8 01 01`|IP header (20 bytes)|`c0 a8 01 0a` = 192.168.1.10 (from), `c0 a8 01 01` = 192.168.1.1 (to); `01` before the checksum means the payload is ICMP|
|`08 00 4d 5a 00 01 00 01`|ICMP header|`08` = echo request, i.e. a ping|
|`61 62 63 …`|Ping data|The letters “abcdefghi…”, filler text that ping sends|
|`bb cb ff 7c`|FCS|Checksum the receiver uses to detect corruption|

This frame is 78 bytes, well above the 64-byte minimum. A frame with less data than that gets padded with zeros to reach the minimum



|Field|Size|What it is|Example value|Why it’s needed|
|---|---|---|---|---|
|**Preamble**|7 bytes|A fixed pattern of alternating 1s and 0s (`10101010` repeated)|`55 55 55 55 55 55 55`|Wakes up the receiving card and lets it synchronise its clock with the incoming signal, like a “ready, steady” before the message starts|
|**SFD** (Start Frame Delimiter)|1 byte|One special byte that ends the preamble|`D5` (`10101011`)|Marks exactly where the real frame begins, so the receiver knows the next byte is the destination address|
|**Destination MAC**|6 bytes|The hardware address of the card the frame is going to|`00:1a:2b:3c:4d:5e`, or `ff:ff:ff:ff:ff:ff` to send to everyone (broadcast)|Tells switches where to deliver the frame and tells each card “this one’s for you”, or not|
|**Source MAC**|6 bytes|The hardware address of the card that sent the frame|`08:00:27:ab:cd:ef`|Lets the receiver know who to reply to, and lets switches learn which device sits on which port|
|**VLAN tag** (optional)|4 bytes|A label saying which virtual network (VLAN) the frame belongs to|`81 00 00 14` (VLAN 20)|Lets one cable or switch carry several separate networks; only present on networks that use VLANs|
|**Type** (EtherType)|2 bytes|A code saying what kind of data the payload contains|`08 00` = IPv4, `08 06` = ARP, `86 DD` = IPv6|Tells the receiving computer which part of its software should handle the payload|
|**Payload**|46–1500 bytes|The actual content being carried, usually an IP packet with the real data inside|An IP packet containing a ping, a web page chunk, etc.|It’s the point of the frame; everything else is just packaging. Too little data gets padded to the 46-byte minimum|
|**FCS** (Frame Check Sequence)|4 bytes|A checksum (CRC-32) calculated over the whole frame|`bb cb ff 7c`|The receiver recalculates it; if the values differ, the frame was damaged in transit and is silently discarded|