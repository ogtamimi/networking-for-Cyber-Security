# Chapter 3: The OSI Model

### Why Do We Need the OSI Model?

In the early days of networking, every company made its own proprietary hardware and software. IBM computers could only talk to other IBM computers; DEC computers could only talk to DEC computers. It was chaos.

To solve this, the International Organization for Standardization (ISO) created the **OSI (Open Systems Interconnection) Model** in 1984.

> **The Assembly Line Analogy:** Think of the OSI model like a factory assembly line. Each station (layer) has one specific, independent task. It only communicates with the layer directly above or below it. This separation allows different vendors to build different pieces, and as long as everyone follows the rules, everything works together perfectly.

**Key Benefits:**
- **Universal Framework:** A conceptual guide for how network communication should work.
- **Troubleshooting:** Makes finding problems easier (e.g., "Is this a Layer 3 routing issue or a Layer 7 application issue?").
- **Security:** Every layer has its own distinct attack surface and defense mechanisms.

### The 7 Layers at a Glance

|Layer|Name|PDU (Data Unit)|Core Function|Example Protocols|
|---|---|---|---|---|
|**7**|**Application**|Data|Network services for end-users|HTTP, DNS, SMTP|
|**6**|**Presentation**|Data|Data translation & encryption|TLS/SSL, JPEG|
|**5**|**Session**|Data|Conversation management|NetBIOS, RPC|
|**4**|**Transport**|Segment|End-to-end delivery & ports|TCP, UDP|
|**3**|**Network**|Packet|Logical addressing & routing|IP, ICMP, IPSec|
|**2**|**Data Link**|Frame|Physical addressing (MAC)|Ethernet, Wi-Fi|
|**1**|**Physical**|Bit|Raw transmission of signals|Copper, Fiber|

### Deep Dive: The Layers (Top to Bottom)

**Layer 7: APPLICATION (The Top of the Stack)**

- **What it does:** Where users and applications interact with the network. It provides network services directly to end-user applications. (Note: It's _not_ the app itself, but the protocols the app uses).
- **Protocols:** `HTTP`, `HTTPS`, `DNS`, `FTP`, `SMTP`, `SSH`, `DHCP`
- **PDU:** Data
- **Security Relevance:** Because users interact here, human error makes it highly vulnerable. **Most attacks happen here**, including SQL injection, Cross-Site Scripting (XSS), phishing, and malware Command & Control.

---

**Layer 6: PRESENTATION (The Data Translator)**
- **What it does:** Translates data into a format both sides can understand. It handles data encoding, compression, and encryption/decryption.
- **Protocols/Formats:** `TLS/SSL`, `JPEG`, `MP4`, `ASCII`, `Unicode`
- **PDU:** Data
- **Security Relevance:** Weak or outdated encryption can be broken here. Attackers use TLS stripping (downgrading HTTPS to HTTP) and exploit self-signed/expired certificates.

---

**Layer 5: SESSION (The Conversation Manager)**
- **What it does:** Establishes, maintains, and terminates communication sessions between two devices. Think of it like a phone call: it starts, you talk, it manages who speaks when, and it hangs up. It also manages checkpointing for long transfers.
- **Protocols:** `NetBIOS`, `RPC`, `SIP` (VoIP)
- **PDU:** Data
- **Security Relevance:** **Session Hijacking.** If an attacker steals your active session token, they can impersonate you without needing a password.

---

**Layer 4: TRANSPORT (The Delivery Guarantee)**
- **What it does:** Manages end-to-end delivery. It handles segmentation (breaking data into pieces), flow control (preventing overwhelming a device), and manages **Port Numbers** to identify which app gets the data.
- **Protocols:** `TCP` (Reliable, ordered, error-checked) and `UDP` (Fast, connectionless, no guarantee).
- **PDU:** Segment
- **Security Relevance:** Attackers use port scanning (like Nmap) to find open doors. **SYN Flood** attacks abuse the TCP handshake to cause Denial of Service (DoS).

---

**Layer 3: NETWORK (Logical Addressing & Routing)**
- **What it does:** Acts as the network's navigator. It uses logical addressing (IP addresses) to determine the best path (routing) to get data from source to destination across different networks.
- **Devices/Protocols:** Routers, `IPv4/IPv6`, `ICMP` (Ping), `IPSec`, `ARP`
- **PDU:** Packet
- **Security Relevance:** **IP Spoofing** occurs here, where attackers forge a source IP to hide their identity or reflect attacks. BGP hijacking can also redirect internet traffic.

---

**Layer 2: DATA LINK (Local Delivery & MAC Addressing)**
- **What it does:** Takes raw bits from Layer 1 and organizes them into meaningful frames. It handles delivery within a _single local network_ using physical **MAC addresses** burned into hardware. It also provides error detection (CRC checksum).
- **Devices/Protocols:** Switches, Network Interface Cards (NICs), `Ethernet`, `Wi-Fi (802.11)`
- **PDU:** Frame
- **Security Relevance:** **ARP Spoofing** is a classic Layer 2 Man-in-the-Middle attack. Attackers map their MAC address to a legitimate IP to intercept local traffic. MAC flooding is also common.

---

**Layer 1: PHYSICAL (The Actual Wire)**
- **What it does:** The most basic level. It deals with physics—transmitting raw bits (1s and 0s) over a physical medium using voltages, light pulses, or radio frequencies.
- **Media/Devices:** Copper cables, Fiber optics, Radio (Wi-Fi), Hubs, Repeaters.
- **PDU:** Bit
- **Security Relevance:** Physical security is paramount. Attacks include physically cutting cables, wiretapping to intercept signals, or jamming wireless frequencies.



### Memory Trick: How to Remember the Layers

Remember the phrase: **"Please Do Not Throw Sausage Pizza Away"** (Reading bottom to top: Layer 1 → Layer 7)
- **1: P**lease → **PHYSICAL**
- **2: D**o → **DATA LINK**
- **3: N**ot → **NETWORK**
- **4: T**hrow → **TRANSPORT**
- **5: S**ausage → **SESSION**
- **6: P**izza → **PRESENTATION**
- **7: A**way → **APPLICATION**

<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Orbitron&weight=700&size=24&duration=3500&pause=1000&color=00D9FF&center=true&vCenter=true&width=900&lines=Please+Do+Not+Throw+Sausage+Pizza+Away;Physical+→+Data+Link+→+Network+→+Transport;Session+→+Presentation+→+Application" />

</div>

---

**Encapsulation: Data Going DOWN the Stack**

When you send a message, the data travels _down_ your device's OSI stack. Each layer wraps the data with its own header, like placing envelopes inside larger envelopes.

```
[Layer 7] App          ➔ (Data) + HTTP Header
[Layer 4] Transport    ➔ (Segment) + TCP Header (Ports)
[Layer 3] Network      ➔ (Packet) + IP Header (IP Addresses)
[Layer 2] Data Link    ➔ (Frame) + MAC Header 
[Layer 1] Physical     ➔ (Bits) Transmitted as electrical/radio signals 
```

*Note: **Decapsulation** is the exact reverse. The receiving device reads the data up_ the stack, stripping away one header at each layer until the raw application data is revealed.*

---

**Security Summary: Attacks at EVERY Layer**

|Layer|Common Attacks|
|---|---|
|**7. Application**|SQL Injection, XSS, Phishing, Malware C2|
|**6. Presentation**|SSL Stripping, TLS Downgrade, Certificate Spoofing|
|**5. Session**|Session Hijacking, Cookie Theft, MITM|
|**4. Transport**|SYN Flood DoS, Port Scanning, UDP Flood|
|**3. Network**|IP Spoofing, BGP Hijacking, ICMP Smurf Attacks|
|**2. Data Link**|ARP Spoofing, MAC Flooding, MAC Spoofing|
|**1. Physical**|Cable Tapping, Wiretapping, Rogue Access Points, Jamming|

---

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Montserrat&weight=800&size=38&pause=1000&color=00C2FF&center=true&vCenter=true&width=900&lines=OSI+in+Action%3A+Loading+Google.com" />
</p>

Let's put it all together. What happens when you type `google.com` into your browser?
1. **Layer 7 (App):** Your browser builds an HTTP `GET` request.
2. **Layer 6 (Presentation):** TLS encryption is negotiated to secure the connection (HTTPS).
3. **Layer 5 (Session):** A continuous session is established with Google's server.
4. **Layer 4 (Transport):** TCP segments the data and assigns Port 443 (for secure web traffic).
5. **Layer 3 (Network):** IP addresses are added to the packet (Your IP → Google's IP).
6. **Layer 2 (Data Link):** MAC addresses are added to the frame to get the data to your local router.
7. **Layer 1 (Physical):** The data is converted into raw bits and travels over your Wi-Fi or Ethernet cable out to the internet!

