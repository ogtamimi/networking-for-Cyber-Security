# Chapter 6: Ports & Protocols

**What Is a Port?**
> **The Apartment Analogy:** > * **IP Address** = Gets you to the right **building** (the device).
> - **Port Number** = Gets you to the right **apartment** (the specific application/service).

A port is a communication endpoint identified by a number (ranging from `0` to `65,535`). When traffic hits your device, the OS looks at the port number to route the data to the correct application (e.g., your browser, email client, or SSH daemon).

**Key Concepts**
- **Traffic Routing:** Port `:80` goes to the web server, `:22` goes to SSH, etc.
- **Attack Surface:** Every open port is a potential entry point for attackers. **Rule of thumb: If you don't need a port, close it.**
- **Port Scanning:** Attackers (for recon) and defenders (for auditing) use tools like **Nmap** to find open ports and discover vulnerabilities.

---

## The Three Port Ranges

| Range               | Name                    | Description                                                                                                            | Examples                         |
| ------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| **`0 - 1023`**      | **Well-Known Ports**    | Reserved for standard, globally-recognized services. Assigned by IANA. Usually requires root/admin privileges to open. | HTTP (80), HTTPS (443), SSH (22) |
| **`1024 - 49151`**  | **Registered Ports**    | Assigned by IANA for specific applications. No root needed. Less universal but widely recognized.                      | MySQL (3306), RDP (3389)         |
| **`49152 - 65535`** | **Ephemeral (Dynamic)** | Temporary ports auto-assigned by your OS for outgoing connections. Released when the session ends.                     | Random source ports for browsing |

---

## Critical Port Numbers - Memorize Every One

| Port(s)       | Protocol       | Purpose                      | Secure? | Security Notes / Best Practices                                                                 |
| ------------- | -------------- | ---------------------------- | ------- | ----------------------------------------------------------------------------------------------- |
| **20, 21**    | **FTP**        | File Transfer                | NO      | Cleartext creds — always use SFTP. Open FTP is a huge red flag (especially Anonymous FTP).      |
| **22**        | **SSH / SFTP** | Secure Shell / File Transfer | YES     | Encrypted remote access. Brute-forced 24/7. **Disable password auth; use SSH keys & Fail2Ban.** |
| **23**        | **Telnet**     | Remote Access                | NO      | **NEVER use.** Fully plaintext commands and passwords. Pre-1970s design.                        |
| **25**        | **SMTP**       | Email Send                   | Opt     | Abused for spam/phishing. Add SPF, DKIM, and DMARC to verify senders.                           |
| **53**        | **DNS**        | Name Resolution              | Opt     | Target for DNS poisoning and data exfiltration (DNS tunnelling).                                |
| **67, 68**    | **DHCP**       | IP Assignment                | NO      | Vulnerable to Rogue DHCP / MITM attacks.                                                        |
| **80**        | **HTTP**       | Web Traffic                  | NO      | All data sniffable (plaintext). Always use HTTPS instead.                                       |
| **110 / 995** | **POP3**       | Email Retrieve               | ⚠️ / ✅  | **995 = TLS version**. Downloads and deletes from server.                                       |
| **143 / 993** | **IMAP**       | Email Sync                   | ⚠️ / ✅  | **993 = TLS version**. Syncs across devices. Modern standard for receiving.                     |
| **443**       | **HTTPS**      | Secure Web                   | YES     | TLS encrypted end-to-end. Always use.                                                           |
| **3306**      | **MySQL**      | Database                     | Usly    | Exposed DBs are critical vulnerabilities. Keep internal.                                        |
| **3389**      | **RDP**        | Windows Remote               | YES     | Heavily brute-forced. Hide behind a VPN and change default port.                                |

---

## Deep Dives into Core Protocols

### 1. DNS (Domain Name System) - Port 53

_The "Phone Book" of the Internet: Converts names (google.com) into IP addresses (142.250.80.46)._
**How your browser finds a website (in milliseconds):**
1. **Browser Cache:** Do I know this IP already?
2. **OS DNS Cache:** Does the operating system know? (Checks hosts file).
3. **DNS Resolver:** Asks ISP or public resolver (e.g., `8.8.8.8`).
4. **Root Servers:** Resolver asks the 13 global root servers ("Who handles `.com`?").
5. **TLD Nameserver:** Root points to the Top-Level Domain server ("Who handles `google.com`?").
6. **Authoritative NS:** TLD points to Google's official DNS server.
7. **IP Returned:** Server returns the exact IP address.
8. **Connect!** Browser connects to that IP on Port 443.

**Common DNS Attacks:**
- **DNS Poisoning (Spoofing):** Fake IP records are injected. You type `bank.com` but are routed to an attacker's fake site.
- **DNS Tunnelling:** Attackers hide malware Command & Control (C2) traffic inside legitimate-looking DNS queries to bypass firewalls.

### 2. HTTP vs HTTPS - Web Traffic

**HTTP (Port 80) - Plaintext**
- Sends standard requests (e.g., `GET /index.html`).
- **Danger:** Completely readable via tools like Wireshark or Burp Suite. Passwords, cookies, and session tokens are exposed.
- No server identity verification.

**HTTPS (Port 443) - Encrypted**
- HTTP wrapped in a **TLS (Transport Layer Security)** tunnel.
- **Secure:** Data is encrypted end-to-end (looks like gibberish to interceptors).
- Server identity is cryptographically verified via a digital certificate (the browser padlock).
- _Defense:_ Use HSTS (HTTP Strict Transport Security) to prevent SSL-stripping downgrade attacks.

### 3. Remote Access & File Transfer

**✅ SSH (Port 22) vs ❌ Telnet (Port 23)**
- **Telnet** sends _everything_ (including admin passwords) in cleartext. If an attacker is on the network, the system is compromised.
- **SSH** encrypts all commands and supports cryptographic key-pair authentication.

**✅ SFTP (Port 22) vs ❌ FTP (Ports 20/21)**
- **FTP** is an archaic protocol that transmits files and credentials in cleartext. "Anonymous FTP" allows login without credentials and is a massive red flag.
- **SFTP** (SSH File Transfer Protocol) runs through a secure SSH tunnel. **Always use this.**


### 4. Email Protocols

_Email is the #1 attack vector in the world (phishing)._
- **SMTP (Port 25 / 587 / 465):** **S**ending emails. Moves mail from client to server, and server to server. Lacks built-in security, requiring SPF/DKIM/DMARC to stop spoofing.
- **IMAP (Port 143 / 993 TLS):** **R**eceiving emails. Syncs emails across all devices and keeps folders on the server. Modern standard.
- **POP3 (Port 110 / 995 TLS):** **R**eceiving emails. Downloads emails to a single device and deletes them from the server. Older and less common today.