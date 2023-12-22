# preparation for final
## TCP & UDP
1. [Sliding Window Protocal](https://www.cs.uni.edu/~diesburg/courses/cs3470_fa14/sessions/s29/s29.pdf)
+ purpose:
  + it guarantees the reliable delivery of data
  + it ensures that data is delivered in order
  + it enforces flow control between the sender and the receiver.

2. three-way handshake/Connection Set up
![](./img/Timeline%20for%20three-way%20handshake%20algorithm.jpg)

3. SYN,ACK,FIN
4. `Nagle’s Algorithm`

   Nagle's algorithm works by combining a number of small outgoing messages and sending them all at once. Specifically, as long as there is a sent packet for which the sender has received no acknowledgment, the sender should keep buffering its output until it has a full packet's worth of output, thus allowing output to be sent all at once.
   ```
   if there is new data to send
      if the window size >= MSS and available data is >= MSS
         send complete MSS segment now
      else
         if there is unconfirmed data still in the pipe
            enqueue data in the buffer until an acknowledge is received
         else
            send data immediately
         end if 
      end if
   end if
   ```
5. TCP delayed ACK

   several ACK responses may be combined together into a single response, reducing protocol overhead.

   Delayed ACKs can give the application the opportunity to update the TCP receive window and also possibly to send an immediate response along with the ACK. 

   problems: If Nagle's algorithm is being used by the sending party, data will be queued by the sender until an ACK is received. If the sender does not send enough data to fill the maximum segment size (for example, if it performs two small writes followed by a blocking read) then the transfer will pause up to the ACK delay timeout.

6. UDP address

![](./img/The%20UDP%20header.png)
+ Can I send a UDP datagram to a broadcast address?
   
   Answer is yes.


   Broadcasting in UDP is achieved by sending the datagram to a special broadcast address. In IPv4,
this address is typically the last address of the subnet. For example, if your subnet is
192.168.1.0/24 , the broadcast address would be 192.168.1.255 . Any UDP datagram sent to
this address will be delivered to all hosts on the subnet.

   But TCP cannot.

   No, you cannot send a TCP (Transmission Control Protocol) frame to a broadcast address. TCP is a
connection-oriented protocol, which means it establishes a direct connection between two
specific endpoints (a client and a server) before any data is exchanged. This connection is
established through a process known as the TCP three-way handshake.

7. sockets

what is socket?
+ a socket is a programming interface and endpoint for network communication. It is a mechanism that allows processes on different devices to communicate over a network. Sockets provide a standard API for network communication, allowing applications to send and receive data over a network.

UDP socket
```
# server
from socket import socket, AF_INET, SOCK_DGRAM
s = socket(AF_INET, SOCK_DGRAM)
s.bind(('127.0.0.1', 11111))
while True:
   data, addr = s.recvfrom(1024)
   print "Connection from", addr
   s.sendto(data.upper(), addr)

# client
from socket import socket, AF_INET, SOCK_DGRAM
s = socket(AF_INET, SOCK_DGRAM)
s.bind(('127.0.0.1', 0)) 
print "using", s.getsocketname()
server = ('127.0.0.1', 11111)
s.sendto("MixedCaseString", server)
data, addr = s.recvfrom(1024)
print "received", data, "from", addr
s.close()
```

TCP socket
```
# server
from socket import socket, AF_INET, SOCK_STREAM
s = socket(AF_INET, SOCK_STREAM)
s.bind(('127.0.0.1', 9999))
s.listen(5) # max queued connections
while True:
   sock, addr = s.accept()
   # use socket sock to communicate 
   # with client process

# client
from socket import socket, AF_INET, SOCK_STREAM
(SERVER, PORT) = ('127.0.0.1', 9999)
s = socket(AF_INET, SOCK_STREAM)
s.connect((SERVER, PORT))
s.send('Hello, world')
data = s.recv(1024)
s.close()
print 'Received', data
```

## DNS
+ Domain name: any name represented in the DNS format
+ DNS label: 
   + each string between two "." (unless the dot is prefixed by “\”) 
   + i.e. foo.bar is 2 labels foo\\.bar is 1 label
+ DNS zone:
   + a set of names that are under the same authority
   + example.com and ftp.example.com, www.example.com 
   + Zone can be deeper than one label, example.us
+ Delegation:
   + Transfer of authority for/to a sub-domain
   + example.org is a delegation from org

![](./img/dividing%20a%20domain%20into%20zones.jpeg)
1. [Record Types](https://github.com/Iris-Song/Computer-Network-Course/blob/main/NOTES/7.%20Application%20Layer.md#record-types-in-dns)
2. recursive vs iterative
![](./img/Iterative%20Name%20Resolution.jpg)
![](./img/Recursive%20Name%20Resolution%20(1).jpg)
+ Performance-wise, which is better?
  + Recursive method puts higher performance demand on each name server
+ Which works better with caching?
  + Recursive method works better with caching
+ How about communication cost?
  + Recursive method can reduce communication cost

## IP Multicast & Unicast & Anycast
### Multicast
protocal: IGMP

clone and send

IP Multicast vs Ethernet Multicast

### Anycast
sequencial packets may be delivered to different anycast nodes

Traffic from differet nodes may follow separate path

## BGP
### Autonomous System
An AS is a group of routers that share similar routing policies and operate within a single administrative domain.

An AS typically belongs to one organization

If an AS connects to the public Internet using an exterior gateway protocol such as BGP, then it must be assigned a unique AS number which is managed by the Internet Assigned Numbers Authority (IANA).

### AS Numbers
AS numbers can be between 1 to 65,535.
+ RIRs manage the AS numbers between 1 and 64,512.
+ The 64,512 - 65,535 numbers are reserved for private use (similar to IP Private addresses).
+ The IANA is enforcing a policy whereby organizations that connect to a single provider use an AS number from the private pool.

### IGP vs EGP
Interior gateway protocol (IGP): 
+ routing protocol operating within an Autonomous System (AS). 
+ e.g. OSPF

Exterior gateway protocol (EGP):
+ A routing protocol operating between different AS. 
+ BGP is an interdomain routing protocol (IDRP) and is an EGP.

### BGP Basics
The Internet is a collection of autonomous systems that are interconnected to allow communication among them.
+ BGP provides the routing between these autonomous systems.

BGP is a path vector protocol.

It is the only routing protocol to use TCP. 
+ OSPF and EIGRP operate directly over IP. IS-IS is at the network layer.
+ RIP uses the User Datagram Protocol (UDP) for its transport layer.

BGP Peers = Neighbors
+ When two routers establish a TCP enabled BGP connection, they are called neighbors or peers.
   + Peer routers exchange multiple connection messages. 
+ Each router running BGP is called a BGP speaker

![](./img/BGP%20Neigbors.jpeg)

IBGP neighbors do not need to be directly connected

EBGP neighbors are usually directly connected. 

EBGP neighbors must have different AS numbers.

## MPLS
LDP: Label Distribution Protocol 
  + A simple non-constrained(doesn't support traffic engineering) protocal

RSVP-TE: Resource Reservation Protocal with Traffic Engineering
+ more complex protocal, with more overhead, but which also includes support for traffic-engineering via network resource reservations.

LSP: Label Switched Path

FEC: Forwarding Equivalence Class 

LSR: Label Switching Router

LER: Label Edge Router

MPLS is most commonly categorized as a layer 2.5 protocol.

It resides between the data-link and the network layers in the TCP-IP model.

MPLS directs data from one network element to another based on short path labels, different from the conventional network addresses.

This thus helps in avoiding complex lookups in the routing table.

MPLS is a virtual circuit packet switching technology.

Here, short labels are attached to network packets which describe how to forward them through the network.

Independent of any routing protocol.

### Packet Switching vs Circuit Switching
+ Circuit Switching:
   + Source first establishes a connection to the destination. 
   + Source sends data over the connection.
   + Source tears down the connection when done.
   + Similar to how telephonic conversation works
+ Packet Switching:
   + Data is divided into packets.
   + Each packet contains its own header. 
   + Destination reconstructs the message

### Virtual Circuit
Combination of packet and circuit switching.

It’s a logical circuit between the source and destination.

Every Virtual Circuit is identified by a VC ID.

The source set-up will establish the path for the VC.

Switch will then map the VC to an outgoing link.

The packet will then have a fixed length label in the header.

### MPLS versus IP routing
IP routing: path to destination determined by destination address alone

MPLS routing: path to destination  can be based on source and dest address

There are two main types of MPLS routing protocols in use today:
- Label Distribution Protocol (LDP): a simple protocol that doesn’t support traffic engineering.
- Resource Reservation Protocol with traffic engineering (RSVP-TE).

### pros & cons of switching vs. routing
1. switching reduce IP routing lookups(routing use 'longest prefix matching', switching use 'exact matching'. Exact matching is cheaper and easier to implement)
2. switching is easy to configure.
3. Switches are often less expensive than routers, making them a cost-effective solution for connecting devices within the same network segment.
4. Routers logically segment networks into subnets, providing a higher level of control and security. 
5. Routing is highly scalable and can accommodate large and diverse networks. It is well-suited for connecting geographically dispersed networks.
6. Routing enables communication between different networks and subnets, allowing for the creation of larger and more complex networks.

### MPLS vs OSPF

+ Purpose and Functionality:
  + MPLS: MPLS is a protocol-agnostic technique for high-performance telecommunications networks that directs data from one network node to the next based on short path labels rather than long network addresses, avoiding complex lookups in a routing table.
  + OSPF: OSPF is a routing protocol used to find the best path for routing data packets between routers in a network. It is specifically designed for IP networks and operates within the Internet Layer of the OSI model.

+ Layer of the OSI Model:
  + MPLS: Operates at the Data Link Layer (Layer 2) and Network Layer (Layer 3) of the OSI model, depending on its implementation.
  + OSPF: Operates at the Network Layer (Layer 3) of the OSI model.

+ Topology Awareness:
  + MPLS: MPLS is not inherently aware of the underlying network topology. It is more concerned with forwarding packets based on labels without necessarily considering the details of the network structure.
  + OSPF: OSPF is a link-state routing protocol that is topology-aware. It builds a detailed map of the network and calculates the best paths based on the cost of links.

+ Routing vs. Label Switching:
  + MPLS: Focuses on label switching, where packets are forwarded based on pre-assigned labels, improving network efficiency and speed.
  + OSPF: Focuses on routing, determining the best paths for IP packets based on metrics such as link bandwidth and cost.

+ Scalability:
  + MPLS: MPLS is often used in large service provider networks for its scalability and ability to efficiently handle large amounts of traffic.
  + OSPF: OSPF is suitable for medium to large enterprise networks, but its scalability may become a challenge in very large networks.

+ Traffic Engineering:
  + MPLS: MPLS allows for explicit traffic engineering, where paths can be explicitly defined and managed to optimize network performance.
  + OSPF: While OSPF can influence routing paths based on metrics, it may not provide the same level of explicit traffic engineering capabilities as MPLS.

## 802.11 vs 802.3
RTS, CTS in wireless.

Collision Detection in 802.11 

+ No. 

Collision Avoidance in 802.11 
+ Yes. CSMA/CA