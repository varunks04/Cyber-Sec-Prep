# 2. Networking Fundamentals for Cybersecurity

## 2.1 Reference Models

### 2.1.1 OSI Model
**Meaning:** A 7-layer conceptual model describing how data moves through a network.

| Layer | Name | Function | Example Protocols/Devices |
|---|---|---|---|
| 7 | Application | User-facing services | HTTP, FTP, DNS |
| 6 | Presentation | Data formatting/encryption | TLS, SSL, JPEG |
| 5 | Session | Manages sessions/connections | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery, reliability | TCP, UDP |
| 3 | Network | Logical addressing, routing | IP, ICMP, Router |
| 2 | Data Link | Physical addressing (MAC) | Ethernet, Switch, ARP |
| 1 | Physical | Raw bit transmission | Cables, NIC, Hubs |

**Common Interview Questions:**
- At which layer does a firewall operate? *(Traditional: Layer 3/4; NGFW/WAF: up to Layer 7)*
- At which layer does a switch vs a router operate?

### 2.1.2 TCP/IP Model
**Meaning:** A simplified 4-layer practical model used on the internet: **Application, Transport, Internet, Network Access**.
**Interview Tip:** Interviewers often ask how OSI layers map to TCP/IP layers — Application+Presentation+Session (OSI) = Application (TCP/IP); Data Link+Physical (OSI) = Network Access (TCP/IP).

---

## 2.2 TCP vs UDP

| Aspect | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable (acknowledgments, retransmission) | Unreliable (no guarantee) |
| Speed | Slower | Faster |
| Use Case | Web (HTTP/HTTPS), Email, File transfer | DNS, VoIP, streaming, DHCP |

**Common Interview Questions:**
- Why does DNS use UDP but sometimes fall back to TCP? *(TCP used for zone transfers or responses >512 bytes)*
- Why is TCP preferred for file transfer and UDP for video calls?

### 2.2.1 TCP Three-Way Handshake
**Meaning:** Process to establish a reliable TCP connection.
```
Client → SYN → Server
Server → SYN-ACK → Client
Client → ACK → Server
```
**Interview Tip:** A **SYN flood** attack abuses this handshake by sending many SYNs without completing the ACK, exhausting server resources (a DoS technique).

### 2.2.2 TCP Flags
| Flag | Meaning |
|---|---|
| SYN | Initiate connection |
| ACK | Acknowledge received data |
| FIN | Gracefully close connection |
| RST | Abruptly reset/reject connection |
| PSH | Push data immediately |
| URG | Urgent data |

**Common Interview Questions:**
- What does an RST flag indicate during a port scan?
- How does Nmap use flags (SYN scan vs full connect scan)?

---

## 2.3 Addressing

### 2.3.1 IP Addresses (IPv4 vs IPv6)
| Aspect | IPv4 | IPv6 |
|---|---|---|
| Length | 32-bit | 128-bit |
| Format | Dotted decimal (192.168.1.1) | Hexadecimal (2001:db8::1) |
| Address space | ~4.3 billion | Practically unlimited |
| NAT dependency | Common due to shortage | Not required |

### 2.3.2 MAC Address
**Meaning:** Unique hardware (Layer 2) address burned into a NIC, format `XX:XX:XX:XX:XX:XX`.
**Security relevance:** MAC spoofing can bypass MAC-based access controls; MAC flooding can overwhelm switch CAM tables.

### 2.3.3 Public vs Private IP
| Type | Range Examples | Use |
|---|---|---|
| Private | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | Internal networks (not routable on internet) |
| Public | Everything else | Internet-routable |

### 2.3.4 Subnetting & CIDR
**Meaning:** Dividing a network into smaller sub-networks; CIDR notation (`/24`) denotes the subnet mask length.
**Example:** `192.168.1.0/24` = 256 addresses, mask `255.255.255.0`.
**Common Interview Questions:**
- How many usable hosts in a /28 subnet? *(2^4 - 2 = 14)*

---

## 2.4 Core Resolution & Assignment Protocols

### 2.4.1 ARP (Address Resolution Protocol)
**Meaning:** Resolves an IP address to a MAC address on a local network.
**Security relevance:** **ARP spoofing/poisoning** lets an attacker associate their MAC with another host's IP, enabling Man-in-the-Middle attacks.

### 2.4.2 DNS (Domain Name System)
**Meaning:** Resolves domain names to IP addresses.
**Security relevance:** Vulnerable to DNS spoofing/poisoning, DNS tunneling (data exfiltration), and used heavily in threat intel (malicious domains).

**DNS Records (common):**
| Record | Purpose |
|---|---|
| A | Maps hostname → IPv4 |
| AAAA | Maps hostname → IPv6 |
| MX | Mail server |
| TXT | Arbitrary text (used for SPF/DKIM/DMARC) |
| CNAME | Alias to another hostname |
| NS | Name server for the domain |

### 2.4.3 DHCP (Dynamic Host Configuration Protocol)
**Meaning:** Automatically assigns IP addresses and network config to devices.
**Process (DORA):** Discover → Offer → Request → Acknowledge.
**Security relevance:** Rogue DHCP servers can redirect traffic; DHCP starvation is a DoS technique.

**Common Interview Questions:**
- What is DNS tunneling and why is it a security concern?
- How would you detect a rogue DHCP server?

---

## 2.5 Application-Layer Protocols

| Protocol | Purpose | Port | Transport | Security Relevance |
|---|---|---:|---|---|
| HTTP | Web traffic (unencrypted) | 80 | TCP | Data sent in plaintext |
| HTTPS | Secure web traffic | 443 | TCP | TLS encryption |
| FTP | File transfer (unencrypted) | 21 (control), 20 (data) | TCP | Credentials sent in plaintext |
| SFTP | Secure file transfer (over SSH) | 22 | TCP | Encrypted, preferred over FTP |
| SSH | Secure remote access | 22 | TCP | Encrypted remote administration |
| Telnet | Remote access (unencrypted) | 23 | TCP | Deprecated — plaintext credentials |
| SMTP | Sending email | 25 (also 587 submission) | TCP | Can be abused for spam/BEC |
| POP3 | Retrieve email (downloads & often deletes) | 110 | TCP | Plaintext by default |
| IMAP | Retrieve email (syncs, keeps on server) | 143 | TCP | Plaintext by default |
| ICMP | Diagnostics (ping, traceroute) | N/A (no port) | N/A | Used in ping floods, ICMP tunneling |
| SNMP | Network device monitoring/management | 161/162 | UDP | Default community strings are a common weakness |
| LDAP | Directory service queries (e.g., AD) | 389 (636 for LDAPS) | TCP | Used in AD enumeration attacks |
| RDP | Remote desktop | 3389 | TCP | Frequent brute-force/ransomware entry point |
| SMB | File/printer sharing (Windows) | 445 | TCP | EternalBlue/WannaCry exploited SMBv1 |

> **Interview Tip:** Telnet, FTP, POP3/IMAP (without TLS), and SMBv1 are considered **insecure/deprecated** — never present them as recommended in an interview; always mention their secure alternatives (SSH, SFTP/FTPS, IMAPS/POP3S, SMBv2/3).

**Common Interview Questions:**
- Why is Telnet considered insecure compared to SSH?
- What made SMBv1 dangerous? (EternalBlue/WannaCry context)

---

## 2.6 Perimeter & Traffic-Control Devices

### 2.6.1 NAT vs PAT
| Term | Meaning |
|---|---|
| NAT | Translates private IP ↔ public IP (one-to-one or pooled) |
| PAT | A form of NAT mapping many private IPs to one public IP using different ports (most common form used at home/office) |

### 2.6.2 Firewall
**Meaning:** Filters traffic based on rules (IP, port, protocol, and for NGFW, application/content).
**Types:** Packet-filtering, Stateful, Next-Gen Firewall (NGFW), WAF (application-layer, web-specific).

### 2.6.3 IDS vs IPS
| Aspect | IDS | IPS |
|---|---|---|
| Function | Detects and alerts | Detects and actively blocks |
| Placement | Out-of-band (monitors a copy of traffic) | Inline (sits in traffic path) |
| Risk | No latency impact, but no auto-block | Can block legitimate traffic if misconfigured |

**Common Interview Questions:**
- Difference between IDS and IPS?
- Difference between Firewall and IDS? *(Firewall enforces rules; IDS detects and alerts on suspicious patterns)*
- Difference between Firewall and WAF? *(WAF inspects Layer 7 HTTP traffic for web-specific attacks like SQLi/XSS; a traditional firewall filters IP/port)*

### 2.6.4 Proxy vs Reverse Proxy vs VPN
| Term | Meaning |
|---|---|
| Proxy (forward) | Sits between client and internet; hides client identity, filters outbound traffic |
| Reverse Proxy | Sits in front of servers; load balances, hides server identity, terminates TLS |
| VPN | Encrypts traffic between two points over an untrusted network (e.g., remote worker to corporate network) |

**Common Interview Questions:**
- Proxy vs VPN — what's the core difference? *(VPN encrypts all traffic at the network level; a proxy typically only redirects specific application traffic and may not encrypt)*
- What does a reverse proxy protect against? (hides backend architecture, can help mitigate DDoS, centralizes TLS)

### 2.6.5 Load Balancer, Router, Switch, Gateway
| Device | Function |
|---|---|
| Router | Routes traffic between different networks (Layer 3) |
| Switch | Forwards traffic within a network using MAC addresses (Layer 2) |
| Gateway | Entry/exit point between two different networks/protocols |
| Load Balancer | Distributes traffic across multiple servers for performance/availability |

---

## 2.7 Network Segmentation

### 2.7.1 VLAN
**Meaning:** Logically separates a physical network into multiple isolated broadcast domains.
**Security relevance:** Limits lateral movement — compromise in one VLAN doesn't automatically expose others.

### 2.7.2 Network Segmentation (general) & DMZ
**Meaning:** Dividing a network into zones to contain breaches and control traffic flow.
**DMZ:** A buffer network zone (e.g., for public-facing web servers) isolated from the internal trusted network.

**Common Interview Questions:**
- Why is network segmentation important in incident containment?
- What sits in a DMZ, and why not put it directly on the internal network?

---

## 2.8 HTTP Fundamentals (Networking View)

### 2.8.1 Sockets
**Meaning:** Combination of IP address + port number that identifies a specific communication endpoint (e.g., `192.168.1.5:443`).

### 2.8.2 Common Ports Cheat List
| Port | Service |
|---|---|
| 20/21 | FTP |
| 22 | SSH/SFTP |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 67/68 | DHCP |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 161/162 | SNMP |
| 389/636 | LDAP/LDAPS |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |

---

## 2.9 HTTP vs HTTPS & SSL/TLS Handshake

### 2.9.1 HTTP vs HTTPS
| Aspect | HTTP (HyperText Transfer Protocol) | HTTPS (HTTP Secure) |
|---|---|---|
| Protocol / Port | Port 80 (TCP) | Port 443 (TCP) |
| Encryption | Plaintext (unencrypted) | Encrypted via SSL/TLS |
| Security Risk | Vulnerable to eavesdropping, packet sniffing, MITM tampering | Protects confidentiality, data integrity, and server authentication |
| Performance | Slightly lower latency (no crypto overhead) | Modern TLS 1.3 overhead is negligible (<1ms) |

### 2.9.2 SSL/TLS Handshake (Step-by-Step)
**Purpose:** Authenticate the server, negotiate cryptographic algorithms, and establish a shared symmetric session key.

```
Client                                                  Server
  |                                                       |
  | -------- 1. ClientHello (Cipher suites, ClientRandom)->|
  |                                                       |
  | <------- 2. ServerHello (Chosen cipher, ServerRandom) -|
  | <------- 3. Server Certificate (Public Key, CA signed)-|
  | <------- 4. [Optional] ServerKeyExchange (ECDHE param)-|
  | <------- 5. ServerHelloDone --------------------------|
  |                                                       |
  | [Client verifies certificate against trusted CAs]     |
  |                                                       |
  | -------- 6. ClientKeyExchange (Pre-master secret) --->|
  | -------- 7. ChangeCipherSpec ------------------------>|
  | -------- 8. Client Finished (Encrypted) ------------->|
  |                                                       |
  | <------- 9. ChangeCipherSpec -------------------------|
  | <------- 10. Server Finished (Encrypted) -------------|
  |                                                       |
  |<======= Secure Symmetric Encrypted Data Transfer =====>|
```

**Key Steps Explained:**
1. **ClientHello:** Client sends supported TLS versions, list of supported cipher suites, and a random number (`ClientRandom`).
2. **ServerHello:** Server selects highest compatible TLS version, chosen cipher suite, and its own random number (`ServerRandom`).
3. **Certificate:** Server presents its X.509 digital certificate containing its public key.
4. **Key Exchange & Session Key Derivation:** Client validates the certificate chain against root CAs. Using Diffie-Hellman (ECDHE) or RSA, both sides compute the same **Pre-Master Secret**, deriving the symmetric **Session Key** (`ClientRandom` + `ServerRandom` + `Pre-Master Secret`).
5. **Finished:** Both sides exchange encrypted confirmation messages. All subsequent communication uses symmetric encryption (e.g., AES-256-GCM).

> **Interview Tip:** TLS 1.3 streamlined this handshake from **2 Round Trips (2-RTT)** down to **1 Round Trip (1-RTT)**, and removed legacy insecure ciphers (RC4, CBC mode, RSA key exchange without Forward Secrecy).

**Common Interview Questions:**
- Walk through the SSL/TLS handshake step-by-step.
- Why is Ephemeral Diffie-Hellman (ECDHE) preferred over static RSA for key exchange? *(Provides **Perfect Forward Secrecy (PFS)** — if the server's private key is compromised in the future, past recorded sessions cannot be decrypted)*
- What is SNI (Server Name Indication)? *(Allows a client to specify the target hostname during ClientHello so a server hosting multiple SSL certificates on a single IP can serve the correct one)*

---

## 2.10 DNS Resolution — Step by Step
**Meaning:** The full lookup path a DNS query takes to resolve a hostname to an IP.
```
1. Browser cache check
2. OS cache check
3. Query sent to Recursive Resolver (usually ISP or 8.8.8.8/1.1.1.1)
4. Resolver queries a Root Server → returns TLD server address
5. Resolver queries TLD Server (.com, .in, etc.) → returns Authoritative NS
6. Resolver queries Authoritative Name Server → returns the IP (A/AAAA record)
7. Resolver caches and returns result to client
```
**Common Interview Questions:**
- Walk me through what happens when you type a domain into a browser (DNS portion).
- What is a recursive resolver vs an authoritative name server?
- What is DNS caching and why does it matter for security investigations (stale/poisoned entries)?

---

## 2.11 ICMP in Detail
**Meaning:** ICMP doesn't use ports; it's used for diagnostics and error reporting, not general data transfer.

| Type | Purpose |
|---|---|
| Echo Request/Reply (Type 8/0) | `ping` |
| Destination Unreachable (Type 3) | Host/port/network unreachable |
| Time Exceeded (Type 11) | Used by `traceroute` (TTL expiry) |
| Redirect (Type 5) | Suggests a better route (can be abused for MITM) |

**Security relevance:** ICMP can be used for **ping sweeps** (host discovery), **ICMP floods** (DoS), and **ICMP tunneling** (covert data exfiltration channel). Many networks rate-limit or block ICMP externally for this reason.

---

## 2.12 Flow Control & Reliability (TCP Internals)
- **Sequence/Acknowledgment numbers:** Track byte order and confirm receipt.
- **Sliding Window:** Controls how much data can be sent before requiring an ACK, balancing throughput and reliability.
- **Retransmission:** Lost segments (no ACK received in time) are resent.
- **Termination:** Graceful close uses a 4-way FIN/ACK exchange; abrupt close uses RST.

**Common Interview Questions:**
- How does TCP guarantee reliable delivery (sequence numbers, ACKs, retransmission)?
- What's the difference between connection termination via FIN vs RST?

---

## 2.13 VPN Types & Tunneling Protocols
| Type | Use Case |
|---|---|
| Remote Access VPN | Individual user connects to corporate network remotely |
| Site-to-Site VPN | Connects two entire networks/offices securely over the internet |

**Common tunneling/protocols:** IPSec (network-layer, often site-to-site), SSL/TLS VPN (application-friendly, e.g., OpenVPN), WireGuard (modern, lightweight).

**Common Interview Questions:**
- What is IPSec and what are its two main modes (Transport vs Tunnel)?

---

## 2.14 Subnetting Practice (Frequently Tested)
| CIDR | Subnet Mask | Usable Hosts |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 (common for point-to-point links) |

**Interview Tip:** Usable hosts formula = 2^(host bits) − 2 (subtracting network and broadcast addresses).

---

## 2.15 Packet Capture & Traffic Analysis Basics
**Meaning:** Capturing raw network traffic for inspection — the foundation of network-based detection.
**Key ideas:**
- **Promiscuous mode:** NIC captures all traffic on the segment, not just traffic addressed to it.
- **SPAN/mirror port:** Switch port configured to receive a copy of traffic from other ports (used to feed IDS/packet capture tools).
- **pcap file:** Standard format for saved packet captures (used by Wireshark/tcpdump).

**Common Interview Questions:**
- What is a SPAN port and why is it needed to monitor switched traffic?
- What would you look for in a pcap file during an investigation (unusual destination IPs, plaintext credentials, beaconing patterns)?

---

## 2.16 Rapid-Fire Follow-Up Questions (Section 2)

- Why does a SYN flood work, and what mitigations exist (SYN cookies, rate limiting)?
- How would you detect ARP spoofing on a network?
- What's the risk of leaving SNMP with default community strings ("public"/"private")?
- Why is DNS a common channel for data exfiltration (DNS tunneling)?
- What's the difference between NAT and a firewall — can NAT alone be considered a security control?
- Why do organizations still see RDP as a top ransomware entry vector?
- Explain what happens end-to-end when you type a URL into a browser (classic full-stack networking question).

---

## Quick-Revision Summary

- **OSI (top→bottom):** Application, Presentation, Session, Transport, Network, Data Link, Physical
- **TCP vs UDP:** Reliable/connection-oriented vs fast/connectionless
- **Handshake:** SYN → SYN-ACK → ACK
- **DHCP:** Discover → Offer → Request → Acknowledge
- **Key insecure protocols to flag:** Telnet, FTP, SMBv1, unencrypted POP3/IMAP
- **IDS = detect/alert; IPS = detect/block**
- **Firewall = rule-based filtering; WAF = Layer 7 web attack filtering**
- **VLAN/Segmentation/DMZ = limit blast radius and lateral movement**
- **Must-know ports:** 21, 22, 23, 25, 53, 80, 443, 445, 3389
- **DNS resolution order:** Browser cache → OS cache → Recursive resolver → Root → TLD → Authoritative NS
- **ICMP:** No ports; used for ping/traceroute; can be abused for floods/tunneling
- **TCP reliability:** Sequence numbers + ACKs + sliding window + retransmission
- **VPN types:** Remote Access (user↔network) vs Site-to-Site (network↔network)
- **Subnetting shortcut:** Usable hosts = 2^(host bits) − 2
- **Packet capture:** SPAN/mirror port feeds IDS/Wireshark; promiscuous mode captures all segment traffic
