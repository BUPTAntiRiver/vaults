# Chapter 4 The Network Layer
## 4.1 Introduction
### 4.1.1 Forwarding and Routing
The role of network layer is to **move packets from a sending host to a receiving host**. To do so it has two important functions:
- *Forwarding*: The router moves the received packet from its input link to appropriate output link, which all router-local actions. Processing one **single packet**.
- *Routing*: The router determines the route or path taken by packets as they flow from a sender to a receiver. Refers to network-wide process. Determine where **packets** go.

Every router has a **forwarding table**. A forwarding table can tell which interface should the packet go to by examining the value of a field in its header.
Here comes the question, where does the forwarding table comes from? The answer is by a **centralized** or **decentralized routing algorithm**. In either case, a
router receives routing protocol messages, which are used to configure its forwarding table.

There is actually a third important network-layer function called **connection setup** which is used in specific scenario, and it acts like the handshake in TCP protocol. We will discuss this in Section 4.2.

### 4.1.2 Network Service Models
About the Internet, it uses a *best-effort service*, which is like a euphemism for *no service at all*.

## 4.2 Virtual Circuit and Datagram Networks
The network-layer connection service and connectionless service are similar to those in transport-layer. (with handshake or without)
Computer networks that provide only a **connection service** at the network layer are called **virtual-circuit** (VC) networks; computer networks that provide only a **connectionless service** at the network layer are called **datagram networks**.
### 4.2.1 Virtual-Circuit Networks
A VC is consisted of:
1. a path between the source and destination hosts,
2. VC numbers, one number for each link along the path,
3. entries in the forwarding table in each router along the path.
### 4.2.2 Datagram Networks
In a **datagram network**, each time an end system wants to send a packet, it stamps the packet with the address of the destination end system and then pops the packet into the network.
Routers forward datagram with their destination address, which is a 32bit zero one string. Routers have forwarding tables that uses **prefix match** to determine which interface should the datagram go to.
When there are multiple matches, the router uses the **longest prefix matching rule**.

### 4.2.3 Origins of VC and Datagram Networks

VC has its roots in the telephony world. VC is more complex than datagram networks. But the Internet as a datagram network, on the other hand, grew out of the need to connect the computers together. **Sophisticated** higher level layer with **simple** network-layer. 
## 4.3 What is Inside a Router?
Four components:
- *Input ports.* Common packets are forwarded to the appropriate output port. Control packets are forwarded to the routing processor.
- *Switching fabric.* Connects the router's input ports to its output ports.
- *Output ports.* When a link is bidirectional, an output port to the link will be paired with the input port for that link on the same line card.
- *Routing processor.* Executes routing protocols and maintains routing information and forwarding tables.
### 4.3.1 Input Ports
Although the forwarding table is computed by the routing processor, a shadow copy of the forwarding table is typically stored at each input port locally and updated, as needed, by the routing processor.
To reduce the time needed to look up the forward table, many presenting structure of forwarding table had been developed, the most powerful one is **Content addressable memories(CAM)**, which returns the content of the forwarding table entry for that address in essentially constant time. Another method is to keep recently accessed entries in a cache.
### 4.3.2 Switching Fabric
Switching can be down in a number of ways:
- *Via memory.* Switching done with control of the CPU(routing processor).  Note that if the memory bandwidth is such that B packets per second can be written into, or read from, memory, then the overall forwarding throughput(the total rate at which packets are transferred from input ports to output ports) must be less than B/2.
- *Via a bus.* The bus transfer only one packet at a time. The switching bandwidth is limited to the bus speed.
- *Via an interconnection network.* Can overcome the bandwidth limitation of a single bus. It is a network consists of $2n$ buses for $n$ input ports and output ports.
### 4.3.3 Output Ports
Store the packets and transmit them over the outgoing link.
### 4.3.4 Queuing
When the queues grow large, the router's buffer space will use up and cause **packet loss**.
Define **switching fabric speed** as the rate at which the switching fabric can move the packets from input ports to output ports. If switching fabric speed is at least $n$ times as fast as the input line speed then no queuing can occur at input ports. But what will happen in output ports?
The packets from different input ports may go to the same output port, so if all input ports forward 1 packet to 1 output port then there would be $n$ packets at that port. However the port could only send away 1 packet at a time. So the memory of the output port is used up and then it would drop the packet.
## 4.4 The Internet Protocol(IP): Forwarding and Addressing in the Internet
### 4.4.1 Datagram Format
Key fields:
- *Version number.* 4 bits specify the IP protocol version.
- *Header length.* An IPv4 datagram can contain a variable number of options, these 4 bits are needed to determine where in the IP datagram the data actually begins.
- *Type of service.* Used to distinguish different kinds of datagrams. For example, real-time datagrams and non-real-time ones.
- *Datagram length.* Total length of the IP datagram.
- *Identifier, flags, fragmentation offset.* Information about IP fragmentation.
**IP Data Fragmentation**
*ID.* Which datagram does this fragment belongs to.
*Offset.* Meaning the data should be inserted beginning at byte *offset*.
*Flag.* If 1 means there is more, if 0 means this is the last fragment.
### 4.4.2 IPv4 Addressing
Let's first think about a question, how hosts are connected into the network? A host typically has a single link into the network.
The boundary between the host and the physical link is call an **interface**.
IP requires each host and router interface to have its own IP address.
Each IP address is 32 bits long. These addresses are typically written in so-called **dotted-decimal notation**.For example, 192.0.0.1.
#### Subnet
The interconnecting router interface and host interfaces forms a subnet. IP addressing assigns an address to this subnet, be like: 233.1.1.0/24, where the /24 notation is known as a **subnet mask**, indicates that the left most 24 bits define the IP address.
#### How does a host get IP address?
- Hard-coded by system admin in a file.
- **DHCP**: dynamically get address from a server.
#### NAT: Network Address Translation
All datagrams leaving local network have same single source NAT IP address(**outgoing** address), different source port numbers.
Datagrams with source or destination in this network have the same single source NAT IP address(**incoming** address) for source, destination.
Maintain the relationship in a **NAT translation table**.
# Chapter 6 Wireless and Mobile Networks
#### Switch
Self-learning: when received data from a host, remember its sender, and check whether it has the receiver, if it has, just send it, if it doesn't have, broadcast.
#### Two working modes
##### Infrastructure mode
Traditional networks that the services are provided by a **base station**.
##### Ad hoc networks 自组网
Has **no base station**, the hosts themselves must provide for services like routing, DNS-like name translation, address assignment and more.
#### Avoiding collisions
A 先传输一段较短的 RTS 来建立通道，如果确认收到 RTS 则广播相应的 CTS 表明确立通道，防止其他主机传输数据，然后 A 再传输全部的数据，BS 确认收到后发送 ACK
#### Cellular network 蜂窝网
#### Mobility
