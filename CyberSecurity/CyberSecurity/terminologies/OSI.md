|Layer|Name|What it does (in plain words)|Common protocols / standards|Hardware devices|
|---|---|---|---|---|
|7|Application|What the user or program actually uses: the “conversation” between apps, such as loading a web page or sending an email|HTTP/HTTPS, DNS, SMTP, FTP, SSH, SNMP, DHCP|Not a box; runs in software on PCs and servers. Firewalls (like OPNsense) and proxies can inspect this layer|
|6|Presentation|The translator: makes data readable at both ends, including formats, compression and encryption|TLS/SSL (encryption), JPEG, PNG, MP4, ASCII, UTF-8|No dedicated device; done by software|
|5|Session|The meeting organiser: opens, keeps alive and closes a conversation between two programs|NetBIOS, RPC, SMB sessions, PPTP|No dedicated device; done by software|
|4|Transport|The delivery manager: splits data into chunks, numbers them, and makes sure they arrive complete and in order, or fast|TCP (reliable), UDP (fast), port numbers (e.g. 443, 502)|Firewalls (filter by port), load balancers|
|3|Network|The postal service: gives every device an address and finds the route between different networks|IP (IPv4, IPv6), ICMP (ping), routing protocols (OSPF, BGP), IPsec|Routers, Layer 3 switches, firewalls; the home Wi-Fi router’s “router” part|
|2|Data Link|The local courier: moves data between devices on the same network using hardware (MAC) addresses|Ethernet (802.3), Wi-Fi (802.11), VLAN tags (802.1Q), ARP, PPP|Switches, bridges, network cards (NICs), wireless access points; the Wi-Fi router’s built-in switch|
|1|Physical|The road itself: the cables, radio waves and signals that carry raw 1s and 0s|Ethernet cabling (Cat5e/Cat6), fibre, Wi-Fi radio, Bluetooth, RS-232/RS-485 serial|Cables, connectors, hubs, repeaters, modems, media converters, antennas|

### OSI Model Representation

![[osi-model-representation.png]]