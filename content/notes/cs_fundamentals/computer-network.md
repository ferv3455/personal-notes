---
title: Computer Network
type: docs
---

## Overview: Download a Web Page

- **Get the IP address, subnet info, DNS server - DHCP**
  - Broadcast a DHCP Discover packet on the Ethernet (destination MAC `FF:FF:FF:FF:FF:FF`)
  - **DHCP server (often a router)** responds with a DHCP Offer packet, including **an available IP, subnet mask, gateway IP, DNS server IP**
  - Send a DHCP request to use the offered IP
  - DHCP server sends a DHCP Acknowledgment packet, confirming the IP lease
- **Find the MAC address of the gateway - ARP**
  - Check the **ARP cache** maintained in OS. Return if it exists and is valid
  - Broadcast an ARP request packet encapsulated in an Ethernet frame
  - Gateway responds with an ARP reply packet, including its MAC address
- **Find the server IP address - DNS** (DNS outside of the subnet)
  - Check local **DNS cache** maintained in OS. Return if it exists and is valid
  - Send a DNS query packet to the configured DNS server
  - The DNS resolver checks its own cache or does an **iterative query** until it reaches authoritative name servers
  - DNS server returns the IP address
- **Establish a TCP connection** (three-way handshake)
  - Client sends SYN: SYN_SENT
  - Server responds with SYN-ACK: SYN_RCVD
  - Client sends ACK: ESTABLISHED
- **Get the page via an HTTP request**
  - Transmit the HTTP request via TCP (streamed bytes)
- **Close the TCP connection** (four-way handshake)
  - Client sends FIN: FIN_WAIT_1 (client)
  - Server responds with ACK: CLOSE_WAIT (server), FIN_WAIT_2 (client)
  - Server sends FIN: LAST_ACK (server)
  - Client responds with ACK: TIME_WAIT (client), CLOSED (server)

Notes:

- DHCP is built on top of **UDP - IP - Ethernet**.
  - The payload is the DHCP message.
  - The source/destination ports in UDP segments are 68 (client) / 67 (server).
  - The IP addresses may be set to `0.0.0.0` (unknown) / `255.255.255.255` (broadcast).
  - The MAC addresses may be set to `FF:FF:FF:FF:FF:FF` (broadcast).
  - `EtherType` is set to `IPv4` (0x0800).
- ARP is built directly on top of **Ethernet**.
  - The payload is the ARP message.
  - The MAC addresses may be set to `FF:FF:FF:FF:FF:FF` (broadcast).
  - `EtherType` is set to `ARP` (0x0806).
- DNS is built on top of **UDP - IP - Ethernet**.
  - The payload is the **encoded DNS message**: transaction ID, flags, questions, answers.
  - The source/destination ports in UDP segments are 53 (both client and server).
- ICMP is a supporting protocol built on top of **IP - Ethernet**.
  - The payload is the ICMP message.
  - The type/code fields indicate the purpose (e.g., `ping`, `traceroute`).
- **SYN/FIN (TCP) packets consume 1 byte because they need to be acknowledged.** ACK packets don't consume any bytes.


## OSI Model

1. **Application Layer**: User interface and application services.
2. **Presentation Layer**: Data translation and encryption.
3. **Session Layer**: Manages sessions and connections.
4. **Transport Layer**: Reliable data transfer (TCP/UDP).
5. **Network Layer**: Routing and forwarding (IP).
6. **Data Link Layer**: Node-to-node data transfer (MAC).
7. **Physical Layer**: Physical medium and signaling.



## Transport Layer: TCP/UDP

- TCP vs UDP:
  - TCP: point-to-point, full-duplex, connection-oriented, reliable; UDP: connectionless, best-effort
  - TCP is **stream-oriented**, while UDP is message-oriented
  - TCP provides **flow control** and **congestion control**, while UDP does not
  - TCP has higher **overhead** due to connection establishment and packet header
- TCP/IP header size: 40 bytes (20 + 20)
- **TCP flow control**: suggest the receiver window size `rwnd`
  - **Flow control: prevents the sender from overwhelming the receiver**
  - The receiver should fill in the **advertised window size** in the TCP header, which will be used by the sender to control its actual window size (the minimum of `rwnd` and `cwnd`)
- **TCP congestion control**: dynamically adjust the congestion window size `cwnd`
  - **Congestion control: prevents the sender from overwhelming the network**
  - Slow start: increase exponentially
  - Congestion avoidance: increase linearly
  - Fast recovery (three duplicate ACKs): reduce `cwnd` by half
  - Timeout: start all over (slow start)
- **Nagle's Algorithm**: buffer small packets and send them together to reduce overhead - send when the previous packet is acknowledged
- **Packet sticking**: multiple small packets are combined into a single TCP segment to be received together
  - Sender: add length fields to determine the boundary of each packet
  - Receiver: use length fields to extract each packet
- Three-way handshake:
  - Connection initialization: sequence number exchange
  - Confirm that both ends are ready to **send and receive data**
  - Tell whether the SYN packets are retransmitted: if SYN-ACK is not received, then the client will retransmit the SYN packet. **The server needs to know it is not a new connection so that the previously allocated resources can be reused**
  - The final ACK can be used to carry data
- Four-way handshake:
  - **Both ends may have additional data to send.** Therefore, the process is separated into two closures - one for each direction
  - Server-side FIN and ACK may be piggybacked together
  - TIME_WAIT (client): 
    - Ensure **the final ACK is received**. Otherwise, the server will resend the FIN packet, which requires the client to reply
    - Ensure that, if a new connection is established, it will be delivered after **all previous packets have been received**



## HTTP/HTTPS

- HTTP: Hypertext Transfer Protocol
  - Connection will be closed after each request/response in HTTP/1.0. In HTTP/1.1, persistent connections are used by default.
- HTTPS: HTTP over TLS/SSL
  - Safer with encryption and authentication.
  - Higher latency during handshake and data transfer.


## Network Programming

- Server:
  - `socket()` (create a socket descriptor)
  - `bind()` (bind it to the server's socket address)
  - `listen()` (tell the kernel it is used by a server; client by default)
  - `accept()` (wait for a client connection and return a connected descriptor)
  - Read/Write
  - `close()`
- Client:
  - `socket()` (create a socket descriptor)
  - `connect()` (connect to the listening server)
  - Read/Write
  - `close()`
