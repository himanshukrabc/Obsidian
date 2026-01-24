- **Transmission rates** - Rate at which a link can transfer data - in bits/s
- **Packets** - Data sent from a server is split up into chunks with some header information. These are called packets.
- **Protocol** - Defines how communication happens between end systems and actions taken on transmission and other events.
### Packet Switching
- Transmitted data split into packets.
- Packet switches follow **store and forward** mechanism where you must receive the packet, before transmitting it.
- **output buffer/queue** which stores packets that the router is about to send into that link.
- **IP** - Unique address assigned to each entity on the internet.
  IPV4 - 255:255:255:255
  IPV6 - 2001:db8:85a3:0:0:8a2e:370:7334 _(0-9, a-f)_
- Each router uses a section of the IP address to decide which link to send the packet to. **Forwarding Table** stores the mapping.
### Circuit Switching
- Each link would consist of multiple circuits.
- 