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
## ❌ OSI Model
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
## ❌ P2P applications vs Client Server Applications
- Client Server applications have a dedicated Server to handle all queries of clients. Eg- any webapp.
  P2P applications dont have a dedicated server. There is direct communication with each client. Eg- Bittorrent, Skype etc.
- **P2P applications are self scalable** as the number of hosts increase.
- **ISP favour Client Server** as they expect more data to flow downstream than upstream. P2P violates this pattern.
- **P2P has a security risk**.
- **P2P requires volunteering of bandwidth, storage and computation.**
## Network Layer - HTTP
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
GET \<path\> HTTP/1.1      - GET/POST/PUT/DELETE method, the path to resource, HTTP version
Host: \<host\>             - Hostname/ base URL
Connection: close        - Header indicating to establish non persistent connections
User-agent: Mozilla/5.0  - From which client the call was made
Accept-language: fr      - If there is a specific language version of the object send that else send normal

**HEAD** method - Used for debugging. Server sends back a response but with no object.
### Response Message
HTTP/1.1 200 OK                                - HTTP version StatusCode AccompanyingMessage 
Connection: close                              - Non persistent connection
Date: Tue, 09 Aug 2011 15:44:04 GMT            - Timestamp of response message
Server: Apache/2.2.3 (CentOS)                  - Server detail
Last-Modified: Tue, 09 Aug 2011 15:11:03 GMT   - Indicates when the object being sent was last modified
Content-Length: 6821                           - Length of content
Content-Type: text/html                        - Type of content
## Cookies
- Since HTTP is stateless, it uses cookies to track user sessions and preferences.
- It has 4 parts
	1. a cookie header line in the HTTP response message
	2. a cookie header line in the HTTP request message 
	3. a cookie file kept on the user’s end system and managed by the user’s browser
	4. a back-end database at the server
- When a user clicks on a hyperlink, it sends a request to the server with header requesting a cookie.
- Server responds with a response with **Set-cookie:** uniqueIdForUser
- When the browser sees the header, it appends the cookie file with \<HostName, uniqueId\>
- Now all requests made will have this uniqueId passed. This helps the website track the user. 
- These are persistent so next time you visit, website will know it is you.
## Web Caching
- When request is made, the received object is cached and will be sent back when the same object is requested.
- Can substantially **Reduce the response times** for resources on the internet.
- Can substantially **Reduce the traffic** on the internet
- Usually paired with CDNs.
### Conditional GET
- The copy of object cached might get stale. Conditional GET is used to solve this problem.
- When a cache receives a request for a object, it makes a conditional GET(**If-modified-since** header) call to the server.
- In response, the server returns an object only when the object is modified.
  else it responds with **304: Not Modified** response.
## DNS
- Functions of DNS - 
  Resolves hostname -> Canonical hostname -> IP addresses.
  Mail Server aliasing - @hostname.org -> resolves to IP address.
  Used to distribute traffic between multiple servers of the same host.(Host mapped to multiple IPs)
- It has a DB distributed between multiple servers. Servers communicate between each other using UDP.
- Browser -> DNS Client on user machine -> DNS Servers
- Advantages of centralized DNS 
  Single Point of Failure, Huge Centralized DB, High Traffic and Maintenance.
- Types of DNS Servers -> 
  **Root Server** - Contains addresses of TLD Servers. 13 in total.
  **Top-Level-Domain(TLD) Sever** - Contains addresses of Authoritative servers having address with give TLD. Eg- .org, .com etc
  **Authoritative** - Contains actual translations of host vs IP.
- Browser -> Local DNS Server(Cound be any) <-> Root Server
											  <-> TLD Server
											  <-> Authoritative Server
											  <-> Actual Server
- Sometimes the server might not contain the data of the actual host but it knows the address of another server which might have data.
- There are multiple calls to multiple DNS servers -> **DNS leads to increase in latency**.
- DNS servers can also be used for **caching**.
- **DNS cache records** store the following values *(Name, Value, Type, TTL)*
  **TTL** - time to live
  **Type** - A(Domain:IP), NS(Domain:DNS_IP), CNAME(Domain:Canonical Hostname), MX(MailDomain:IP)
  **Name:Value** - mentioned above
## Transport Layer - TCP/UDP
### UDP
- **Bare bones** - Provides process-to-process(unreliable) data delivery and error checking.
- **Connectionless** - No connection established - no added latency. No connection state - low latency + low storage.
- **Finer application level data control**
- **Small packet overhead**
- *Source Port, Destination Port, Length, CheckSum, Data* 
  Checksum - rounded of, 1's complement of all data points. So sum of checksum and the data shall be 0. If not error.

### TCP
- **Reliable Data Transfer** 
- **Full-duplex** - both server and client can send packets simultaneously.
- *Source Port, Destination Port, Sequence Number, Ack Number, SYN, ACK, FIN bits and Recieve Window*
#### Sequence and Acknowledgement Numbers
- Client message -> packets -> have sequence numbers ordering the packets.(Next seq num = seq num + size of packet)
- First sequence number is randomly picked so that this packet is not confused with another one.
- **Cumulative Acknowledgement** Response Sent has Acknowledgement Number -> Ack Number = seq number + 1. 
  If server receives multiple packets in random order, it acknowledges the last one in sequence.
  If a new packet completes a list of packets(due to prior arrival), it acknowledges the highest sequence number one.
#### Reliable Data Transfer
- There are three main events that TCP deals with
  1. *Data Received from application* -> Send packets with sequence numbers(Pick first one at random). Start timer.
  2. *Timer Timed out* -> Resend the least seq number packet which was not acknowldeged. Double the timer now.
  3. *Acknowledgement 'y' Received* -> Update the last acknowledged point to y.
     Duplicate ACK recieved > 3 times -> corrupt data received at server -> Resend(**Fast Retransmit**)
#### Flow Control
- Ensures that the client does not send data if the server's buffer cannot accept it.
- Receive Window indicates the amount of space left in TCP buffer. Dont send data if its larger than that.
#### Congestion Control
- Packet Loss -> TCP assumes Networks congested -> TCP limits the sending rate(Throughput) to a threshold(Congestion window).
- It is slowly ramped back up.
- Packet loss can lead to latency spikes even though servers are fine.
- Retries make outages worse. Retry -> More Traffic -> TCP limits throughput -> Slows everyone down.
#### Head-of-Line(HoL) Blocking
- Since TCP guarantees in-order delivery -> If a packet is missing, next packets will remain in the buffer.
- Application does not know this -> Seems like latency is high.
  Average latency is fine but P99 latency is high.
#### Load Balancers and TCP
- If your backends are stateful, then requests cannot be distributed randomly.
- So **Load Balancers forward TCP connections and not individual HTTP requests**.
#### **Connection Management**
1. **SYN Message** - Client Request with SYN=1, seq=random. 
2. **SYN ACK Message** Server Response with SYN=1, ack = seq + 1. 
3. **ACK Message** - Client sends ack = ack_res + 1. Can also send any request it has to make.
4. **FIN Message** - Client Req with FIN=1
5. **ACK Message** - Acknowledgement sent.
6. **Server FIN Message** - Server releases all state, sends FIN = 1.
## Network Layer - IP
- **best-effort service** 
	- packets **may be dropped**
	- packets **may be delayed**
	- packets **may arrive out of order**
	- packets **may be duplicated**
## Why **P99(Tail) latency** matters more than average
- Users experience the slow requests, not avg ones.
- Assume each service ensures 99% of its requests are fast. If users hit 10 services,
  Probability(all fast) = 0.99^N = 0.904
- Thus 10% of users still experience atleast 1 slow request.
- Tail latency comes from: packet loss, TCP retransmissions, queueing delays and DNS delays
## Why **slow nodes are worse than dead nodes**
- Slow nodes take in packets, responds very slowly and holds resources while doing nothing useful
- thread pool exhaustion, connection pool exhaustion, cascading timeouts
- And because it _eventually_ responds:
	- failure detectors don’t trigger
	- load balancers keep routing traffic to it
## Network failures
- Networks failures are partial.
- Some packets get through, some don’t, some are delayed and some take weird paths
- Because the network components each makes **local decisions** -> failures are localized
## HTTP 1.1 vs HTTP 2
- **HTTP/1.1 suffers from application-level head-of-line blocking**: requests over a connection are effectively serialized, so a slow response blocks others.
- **Browsers mitigate HTTP/1.1 limits by opening multiple TCP connections**, which increases handshake overhead and worsens congestion and tail latency.
- **HTTP/2 multiplexes multiple request/response streams over a single TCP connection**, allowing independent progress of requests.
- **HTTP/2 eliminates HTTP-level HoL blocking but not TCP-level HoL blocking**, since packet loss still stalls all streams on that connection.
- **Fewer TCP connections in HTTP/2 improve congestion control efficiency**, reducing latency and improving stability under load.