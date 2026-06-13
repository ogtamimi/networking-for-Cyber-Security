# Chapter 5: IP Addresses & Addressing

## What Is an IP Address?

An IP (Internet Protocol) address is a numerical label assigned to every device connected to a network. Think of it like your home's postal address; it tells the world who you are and where to deliver the data.

It serves two primary functions:
1. **Identification (Who is this device?):** Like your name, it uniquely identifies a device on the network so others know exactly who they're talking to.
2. **Location (Where is this device?):** Like your street address, it tells the network exactly how to route and deliver packets of data.

## IPv4 vs. IPv6
### v4: IPv4 (Internet Protocol Version 4)

The original and still most widely used IP format.
- **Format:** 32-bit address, divided into 4 "octets" (8 bits each) separated by dots. Each number ranges from `0` to `255`.
- **Example:** `192.168.1.100`
- **Capacity:** $2^{32}$ = ~4.3 Billion addresses.

 **The Problem: We ran out!** With over 8 billion people on Earth owning multiple devices, the central pool of IPv4 addresses was exhausted around 2011. Technologies like NAT (covered below) are currently keeping IPv4 alive on life support.

### v6: IPv6 (Internet Protocol Version 6)
The permanent, future-proof solution to IPv4 exhaustion.
- **Format:** 128-bit address, written as 8 groups of 4 hexadecimal digits.
- **Example:** `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **Capacity:** $2^{128}$ = 340 Undecillion addresses (enough to give every atom on Earth's surface an IP address multiple times over).
- **Built-in Security:** IPSec is natively built into IPv6.


> **Security Warning (Dual-Stack):** Many networks currently run both IPv4 and IPv6 simultaneously. If firewalls are only configured for IPv4, attackers can use IPv6 as a blind spot to bypass controls.

## Public vs. Private IP Addresses

| Feature           | Public IP                                             | Private IP                                           |
| ----------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| **Scope**         | Globally unique; routable across the entire internet. | Local scope only; NOT routable on the open internet. |
| **Assignment**    | Assigned by your ISP (Internet Service Provider).     | Assigned by your local router (DHCP).                |
| **Examples**      | `8.8.8.8` (Google), `1.1.1.1` (Cloudflare)            | `192.168.1.5` (Your Laptop), `10.0.0.5`              |
| **Security Risk** | Publicly visible; actively scanned by automated bots. | Hidden from the internet via NAT (obscurity).        |

### Common Private IP Ranges (RFC 1918)
- `10.0.0.0` to `10.255.255.255` (16.7M hosts)
- `172.16.0.0` to `172.31.255.255` (1M hosts)
- `192.168.0.0` to `192.168.255.255` (65K hosts)

## NAT (Network Address Translation)
**How do devices with private IPs talk to the internet?** NAT allows multiple devices on a local network to share **one** single Public IP address.
1. Your laptop (`192.168.1.5`) requests a website.
2. The **Router + NAT** intercepts the request, swaps your private IP for its Public IP (`203.0.113.50`), and logs this in its NAT table.
3. The Internet only ever sees the router's Public IP.
4. When the website responds, the router checks its table and forwards the data back to your specific laptop.

>  **Forensic Challenge:** NAT hides internal devices. If you see an attack originating from `203.0.113.50`, you are only seeing the router. To find the specific compromised laptop, you must cross-reference the router's internal NAT logs.

## Static vs. Dynamic IPs & DHCP
- **Static IP:** Manually configured and never changes. Used by servers, printers, and routers because they need to be reliably found at a fixed location.
- **Dynamic IP (DHCP):** Automatically assigned by a DHCP (Dynamic Host Configuration Protocol) server. Most home/office user devices use this. The IP can change over time.

###  DHCP Attacks
How attackers exploit dynamic addressing:
1. **DHCP Starvation:** Attacker floods the network with fake IP requests, exhausting the available pool of IP addresses. Legitimate devices are denied network access (DoS).
2. **Rogue DHCP & MITM:** With the real pool empty, the attacker sets up a fake DHCP server. They assign IPs to new devices and set _themselves_ as the Default Gateway, allowing them to intercept unencrypted traffic (Man-in-the-Middle).

## Special IP Addresses You Must Know

| IP Address      | Name                     | Description & Security Relevance                                                                                                                       |
| --------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `127.0.0.1`     | **Loopback (Localhost)** | Always points back to the device you are currently on.<br><br>_Malware often uses this to run hidden local services in the background._                |
| `192.168.1.1`   | **Default Gateway**      | Usually your router's IP. It's the exit door for data leaving your LAN.<br><br>_If an attacker controls this, they control all your internet traffic._ |
| `192.168.1.255` | **Broadcast**            | Sends data to ALL devices on the subnet simultaneously.<br><br>_Abused in legacy amplification attacks (e.g., Smurf attacks)._                         |
| `0.0.0.0`       | **Unspecified**          | Used in server configs to mean "Listen on all network interfaces."<br><br>_Can accidentally expose internal services to the public internet._          |

## MAC Address (Layer 2 Identity)

If an IP address is your postal address, the MAC (Media Access Control) address is your physical fingerprint.
- **Format:** `00:1A:2B:3C:4D:5E` (48-bit, Hexadecimal)
- **Structure:** First 3 octets = Manufacturer ID (OUI). Last 3 = Device Serial Number.
- **Scope:** Operates only at Layer 2 (Local Area Network). When data crosses a router to a new network, the MAC address changes, but the IP address remains the same.

> **MAC Spoofing:** While burned into hardware, the OS can easily spoof it. A simple command (`ip link set dev eth0 address XX:XX...`) changes your visible MAC instantly, easily bypassing MAC-based network access controls.

## Subnetting & CIDR Notation
**Subnetting** is the practice of dividing a large network into smaller, isolated networks. Think of an apartment building: one main street address, but many separate apartments inside.

**Why Subnet?**

1. Organization
2. Performance
3. **Security (Blast Radius):** By putting Web Servers in Subnet A and Databases in Subnet B, you can put a firewall between them. If a web server is hacked, the attacker cannot freely pivot to the database.

### CIDR Notation Cheat Sheet
Instead of writing out bulky Subnet Masks (like `255.255.255.0`), we use CIDR (Classless Inter-Domain Routing) notation, which simply states how many bits are locked for the network portion (`/24`).

| CIDR      | Subnet Mask       | Total IPs | Usable IPs | Common Use Case                       |
| --------- | ----------------- | --------- | ---------- | ------------------------------------- |
| **`/8`**  | `255.0.0.0`       | ~16.7M    | ~16.7M     | Huge ISPs / Enterprise                |
| **`/16`** | `255.255.0.0`     | 65,536    | 65,534     | Large Campus Network                  |
| **`/24`** | `255.255.255.0`   | 256       | 254        | **Most Common (Home/Office)**         |
| **`/25`** | `255.255.255.128` | 128       | 126        | Splitting a `/24` in half             |
| **`/26`** | `255.255.255.192` | 64        | 62         | Small department subnet               |
| **`/30`** | `255.255.255.252` | 4         | 2          | Point-to-Point Router Link            |
| **`/32`** | `255.255.255.255` | 1         | 1          | Single specific host (Firewall rules) |

_(Note: The number of "Usable IPs" is always Total IPs minus 2, reserving one for the Network Address and one for the Broadcast Address)._
