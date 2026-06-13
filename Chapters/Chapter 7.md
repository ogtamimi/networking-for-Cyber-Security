# CHAPTER 07: TCP vs. UDP

**Connection models, reliability, the 3-way handshake, and security implications.**

### Why do we need TWO transport protocols?
Why not just one, Because **reliability** and **speed** are opposites. Different applications have different requirements. Sometimes you need perfect, error-free data (TCP). Other times, a delay is worse than losing a tiny piece of data, so you need pure speed (UDP).

### TCP (Transmission Control Protocol) - _The Reliable One_

**Connection-oriented · Guaranteed delivery · Ordered · Error-corrected · Flow controlled**

TCP is built for perfect reliability. Every single byte must arrive in order, with zero errors.
- **Connection-Oriented:** Must complete a formal setup (3-way handshake) BEFORE any data flows, and a clean teardown when finished.
- **Guaranteed Delivery:** Every single packet must be acknowledged. Lost packets are automatically retransmitted.
- **Ordered Delivery:** Packets are numbered with sequence numbers. The receiver reorders them if they arrive out of sequence.
- **Error Detection:** Checksums on every segment ensure corrupted data is detected and retransmitted.
- **Flow Control:** Sliding windows prevent a fast sender from overwhelming a slow receiver.
- **Congestion Control:** TCP automatically slows down when network congestion is detected.


#### The TCP 3-Way Handshake

Think of it like knocking on a door and confirming both sides are ready before speaking.
1. **[SYN]** Client ➔ Server: _"I want to connect. My starting sequence number is X."_
2. **[SYN-ACK]** Server ➔ Client: _"Got it. My sequence is Y. I acknowledge your X."_
3. **[ACK]** Client ➔ Server: _"Perfect. Connection established. Let's go!"_

> **Security Vulnerability: SYN Flood Attack (DoS)** An attacker sends millions of connection requests (`SYN`) with spoofed IPs. The server replies with `SYN-ACK` and waits for the final `ACK` that never comes. The server holds thousands of "half-open" connections until its memory fills up and it crashes, denying service to real users. **Defense:** SYN cookies, rate limiting.

### UDP (User Datagram Protocol) - _The Fast One_

**Connectionless · No delivery guarantee · No ordering · Minimal overhead · Fire and forget**
UDP simply sends the data and moves on. No setup, no confirmation, no delay.
- **No Handshake:** Just send. No setup required. Perfect for single query/response flows.
- **No Guarantees:** Packets may be lost, duplicated, or arrive out of order. If reliability is needed, the application itself must handle it.
- **Minimal Overhead:** An 8-byte header (compared to TCP's 20-60 bytes) makes it incredibly fast for high-throughput, latency-sensitive data.
- **Use Cases:** DNS, Video Streaming, VoIP (Voice calls), Online Gaming, DHCP, SNMP. _(Anything where speed beats perfection)._

> **Security Vulnerability: UDP Amplification DDoS** Because UDP is connectionless, attackers can easily fake (spoof) the source IP.
> 1. **Spoof IP:** Attacker fakes the source IP in a UDP packet to match the _victim's_ IP.
> 2. **Tiny Query:** Attacker sends a tiny (e.g., 50-byte) query to thousands of open servers (like DNS or NTP).
> 3. **Amplification:** The servers reply with massive responses (e.g., 3,000 bytes) - a 60x amplification.
> 4. **Victim Flooded:** Millions of massive responses hit the victim simultaneously, overwhelming their network.

### TCP vs UDP: The Complete Comparison

| Feature            | TCP (Transmission Control Protocol)         | UDP (User Datagram Protocol)    |
| ------------------ | ------------------------------------------- | ------------------------------- |
| **Connection**     | Connection-oriented (3-way handshake)       | Connectionless (no handshake)   |
| **Reliability**    | Guaranteed delivery                         | No guarantee                    |
| **Ordering**       | In-order always                             | May arrive out of order         |
| **Speed**          | Slower (more overhead)                      | Faster (minimal overhead)       |
| **Error Recovery** | Yes - retransmits lost data                 | No - application must handle it |
| **Header Size**    | 20 - 60 bytes                               | 8 bytes only                    |
| **Use Cases**      | Web, Email, File Transfer, SSH, Databases   | DNS, VoIP, Video, Gaming, DHCP  |
| **Security Risks** | SYN Flood, Session Hijacking, Port Scanning | UDP Flood, Amplification DDoS   |

_**Modern Note (QUIC):** HTTP/3 uses a protocol called QUIC. It runs over UDP for speed but adds TCP-like reliability (ordering, error handling) at the application layer!_

