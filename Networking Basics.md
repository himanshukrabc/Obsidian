- **Transmission rates** - Rate at which a link can transfer data - in bits/s
- **Packets** - Data sent from a server is split up into chunks with some header information. These are called packets.
- **Protocol** - Defines how communication happens between end systems and actions taken on transmission and other events.
- **IP Address** - Unique address assigned to each entity on the internet.
  IPV4 - 255:255:255:255
  IPV6 - 2001:db8:85a3:0:0:8a2e:370:7334 _(0-9, a-f)_
- **Socket** - It is the API between application layer and the transport layer. Packets are pushed into and received through sockets. They are identified by *Port Numbers*.
## Packet Switching vs Circuit Switching
### Packet Switching
- Transmitted data split into packets.
- **store and forward mechanism** - Switchers must receive the packet, before transmitting it.
- **output buffer/queue** which stores packets that the router is about to send into that link.
- **Forwarding table** - Map, uses IP(index) to determine which link to forward the packet to.
### Circuit Switching
- Each link consists of multiple circuits.
- To communicate, one circuit is reserved in all the links from source to destination.
- The available throughput is split equally among circuits.
- **FDM(Frequency Domain Multiplexing)** - Circuits have frequency bands(*Bandwidth*) assigned. Send data in the same frequency.
- **TDM(Time Domain Multiplexing)** - Circuits have time intervals assigned where they can send packets.
### Packet Switching vs Circuit Switching
Link Throughput - 1Mbps
- If there are 10 circuits, circuit switching can support 10 concurrent users. Each user gets 100kbps
- In Packet Switching, even if there are 35 users, P(<=10 concurrent)=0.0004. Thus essentially we can support 35 users.
  And for <10users, Each packet will approx get 1Mbps throughput
- However, for >10 concurrent users, packet switching may cause significant delays.
- Connected circuits can be empty despite external demands. Packet switching does not have this problem.
## Latency vs Throughput
Types of Delays which contribute to **Latency** are as follows
- **Processing Delay** - Time to examine the packet header and determine the output link.
- **Queuing Delay** - Time spent in the queue
- **Transmission Delay** - Time for router to push data from top of queue into the outbound link.
- **Propagation Delay** - Time for packet to move from one end to the other.
### Queuing Delay and Packet loss
- Each router maintains a **output queue/buffer** for each of the outbound links where packets will be placed.
- Queuing delay can vary packet to packet.
- If the queue is full when a packet arrives, the packet will be dropped leading to **packet loss**.
#### Traffic Intensity and Queuing Delay
- Traffic Intensity for a link = Inbound Rate / Outbound Rate
- If TI < 1, Queues will be shrinking
  If TI >=1, Queues will be expanding. Such links are prone to packet loss.
- The problem arises due to randomness of packet bursts. When a burst happens, TI -> 1. Server is always busy.
- **Queuing Delay proportional to (TI/1-TI)**. As no of packets is dependent on TI and queuing time is dependent of times when server is free(1-TI)
- So **Queing Delay increases exponentially as TI increases**.
### Throughput
- It is the rate at which **packets are successfully received**.
- Depends on *link capacity, losses and congestion*. = min(sending rate, bottleneck rate)
#### Website is laggy despite high throughput
- Packets sent by the receiver are facing delays in the network
## OSI Model
##### 7. Application Layer
- Defines Request and Responses. Provides network services to applications. Eg-HTTP, HTTPS, FTP, SMTP etc.
##### 6. Presentation Layer
- Translation, Encryption, Encoding and Compression. Eg - TLS/SSL, JSON/XML encoding, UTF-8 vs ASCII
##### 5. Session Layer
- Responsible for connection management. Eg - 
##### 4. Transport Layer
- Handles process to process communication. Eg - TCP and UDP
##### 3. Network Layer
- Handles Host to Host communication. Eg - IP, ICMP
##### 2. Data Link Layer
- Hop to Hop delivery. Eg- Ethernet, Wifi
##### 1. Physical Layer
- Bits on the wire. Eg - cables and fiber.
## P2P applications vs Client Server Applications
- Client Server applications have a dedicated Server to handle all queries of clients. Eg- any webapp.
  P2P applications dont have a dedicated server. There is direct communication with each client. Eg- Bittorrent, Skype etc.
- **P2P applications are self scalable** as the number of hosts increase.
- **ISP favour Client Server** as they expect more data to flow downstream than upstream. P2P violates this pattern.
- **P2P has a security risk**.
- **P2P requires volunteering of bandwidth, storage and computation.**
## HTTP
- Application Layer protocol defines the structure of messages, what to do when messages are sent and received.
- **Uses TCP** as the underlying transport layer protocol
- **Stateless protocol** - Servers do not store the state of connection with any client. All info is there in the req message.
- **Stateless -> Scales better** - 
  No per user state. 
  Request does not have to hit the same server. 
  Server crash != state loss.
### Persistent vs non Persistent connections
#### Non Persistent - 
- Connection established and scraped per request basis.
- For each request there is extra time required to establish the connection leading to delay
#### Persistent - 
- Connection established once. Multiple messages are sent over this connection. **Faster**
- Connection is closed after some time of inactivity decided by the server.
### Request Message
GET <path> HTTP/1.1
Host: <host>
Connection: close
User-agent: Mozilla/5.0

