# Network Security Concepts – Fundamentals

**Author:** Harshit
**Task:** Week 2 – Task 2 | Cybersecurity Internship
**Date:** June 2025

---

## 1. How Data Travels Across Networks

Before understanding network security, it is essential to understand how data actually moves from one device to another.

### 1.1 The OSI Model

The **OSI (Open Systems Interconnection) Model** divides network communication into 7 layers. Each layer has a specific job and passes data to the layer above or below it.

| Layer | Name | Function | Protocol Examples | Attack Surface |
|-------|------|----------|------------------|---------------|
| 7 | Application | User-facing services | HTTP, HTTPS, DNS, FTP, SMTP | XSS, SQLi, phishing |
| 6 | Presentation | Encryption, encoding | SSL/TLS, JPEG, ASCII | SSL stripping |
| 5 | Session | Session management | NetBIOS, RPC | Session hijacking |
| 4 | Transport | End-to-end delivery | TCP, UDP | SYN flood, port scanning |
| 3 | Network | Routing between networks | IP, ICMP, ARP | IP spoofing, MITM |
| 2 | Data Link | Node-to-node transfer | Ethernet, MAC, ARP | ARP poisoning |
| 1 | Physical | Raw bit transmission | Cables, Wi-Fi, fiber | Physical tapping |

**Why this matters for security:** Attacks can occur at any layer. A comprehensive network security strategy must address threats at all levels.

---

### 1.2 Key Protocols Every Security Professional Must Know

#### IP (Internet Protocol)
- Every device on a network has an **IP address** — its unique identifier
- IPv4: `192.168.1.1` (32-bit, ~4.3 billion addresses)
- IPv6: `2001:0db8:85a3::8a2e:0370:7334` (128-bit, virtually unlimited)
- IP is **connectionless and unreliable** — it just routes packets, doesn't guarantee delivery
- **Security concern:** IP addresses can be **spoofed** — attackers can forge the source IP in packets

#### TCP (Transmission Control Protocol)
- **Connection-oriented** — establishes a reliable connection via 3-way handshake before data transfer
- Three-way handshake:
```
Client → Server : SYN       (I want to connect)
Server → Client : SYN-ACK   (OK, I acknowledge)
Client → Server : ACK        (Connection established)
```
- **Security concern:** **SYN Flood attack** — attacker sends thousands of SYN packets without completing the handshake, exhausting server resources

#### UDP (User Datagram Protocol)
- **Connectionless** — sends packets without establishing a connection first
- Faster than TCP but no delivery guarantee
- Used by: DNS, video streaming, online gaming, VoIP
- **Security concern:** UDP is often used in **DDoS amplification attacks** (DNS amplification)

#### DNS (Domain Name System)
- Translates human-readable domain names to IP addresses
- Example: `google.com` → `142.250.195.46`
- Without DNS, users would need to remember IP addresses for every website
- **Security concern:** **DNS poisoning / spoofing** — attacker corrupts DNS cache to redirect users to malicious sites

#### HTTP vs HTTPS
| Feature | HTTP | HTTPS |
|---------|------|-------|
| Encryption | ❌ None | ✅ TLS encryption |
| Data visibility | Plaintext — anyone can read | Encrypted — unreadable to interceptors |
| Port | 80 | 443 |
| Use case | Old/insecure sites | All modern sites |
| Packet sniffer sees | Full request + credentials | Encrypted gibberish |

**Security concern:** Any HTTP traffic can be captured and read in plaintext by anyone on the same network. Always use HTTPS for sensitive data.

---

### 1.3 How a Web Request Actually Works

When you type `https://google.com` and press Enter:

```
1. DNS Lookup
   Your device → DNS server: "What is the IP for google.com?"
   DNS server → Your device: "142.250.195.46"

2. TCP Handshake
   Your device → Google server: SYN
   Google server → Your device: SYN-ACK
   Your device → Google server: ACK

3. TLS Handshake (for HTTPS)
   Negotiate encryption algorithm
   Exchange certificates (verify Google's identity)
   Establish encrypted session

4. HTTP Request sent (now encrypted)
   GET / HTTP/1.1
   Host: google.com

5. Server Response (encrypted)
   HTML, CSS, JS returned

6. Browser renders the page
```

Each step in this flow is a potential attack surface.

---

## 2. Network Security Fundamentals

### 2.1 Network Security Goals (CIA Applied to Networks)

| Goal | Network Context | Control |
|------|----------------|---------|
| **Confidentiality** | Prevent eavesdropping on traffic | TLS encryption, VPN |
| **Integrity** | Prevent modification of data in transit | HMAC, digital signatures, TLS |
| **Availability** | Prevent service disruption | DDoS protection, redundancy, firewalls |

### 2.2 Key Network Security Controls

#### Firewall
A firewall filters network traffic based on rules — allowing or blocking packets based on IP, port, protocol, or state.

**Types:**
- **Packet Filter:** Inspects individual packets (IP, port, protocol) — fast but basic
- **Stateful Firewall:** Tracks connection state — allows only packets that are part of an established connection
- **Next-Gen Firewall (NGFW):** Inspects packet content (deep packet inspection), can detect application-layer attacks

```
Example firewall rule:
ALLOW  TCP  from ANY     to 0.0.0.0  port 443  (HTTPS in)
ALLOW  TCP  from ANY     to 0.0.0.0  port 80   (HTTP in)
DENY   ALL  from ANY     to 0.0.0.0  port 22   (Block SSH from internet)
DENY   ALL  from ANY     to ANY                  (Default deny all else)
```

#### IDS / IPS (Intrusion Detection / Prevention System)
- **IDS:** Monitors network traffic and alerts on suspicious patterns — passive (detect only)
- **IPS:** Actively blocks suspicious traffic in real time — active (detect + block)
- Use signature-based detection (known attack patterns) and anomaly-based detection (unusual behavior)

#### VPN (Virtual Private Network)
- Creates an encrypted tunnel between two endpoints over an untrusted network
- All traffic through the tunnel is encrypted — even on public Wi-Fi
- **Types:** Site-to-site VPN (connects two offices), Remote access VPN (employee to company network)

#### Network Segmentation
- Dividing a network into isolated segments (VLANs) to limit lateral movement
- If one segment is compromised, the attacker cannot easily reach other segments
- Critical systems (payment servers, databases) should be on isolated segments

#### DMZ (Demilitarized Zone)
- A network segment exposed to the internet (web servers, email servers) isolated from the internal network
- Acts as a buffer zone — external users reach the DMZ but cannot directly access internal systems

### 2.3 Wireless Network Security

Wi-Fi introduces additional attack surfaces:

| Protocol | Security Level | Status |
|----------|---------------|--------|
| WEP | ❌ Broken | Do not use |
| WPA | ⚠️ Weak | Deprecated |
| WPA2-Personal | ✅ Acceptable | Widely used |
| WPA2-Enterprise | ✅ Good | Recommended for organizations |
| WPA3 | ✅ Best | Modern standard |

**Common Wi-Fi attacks:**
- **Evil Twin:** Attacker creates a fake Wi-Fi hotspot with the same name as a legitimate network
- **Deauthentication attack:** Attacker forces clients to disconnect from legitimate AP
- **PMKID attack:** Captures handshake data to crack WPA2 passwords offline

---

## 3. Network Security Best Practices

### For Individuals
- Always use HTTPS — check for the padlock before entering credentials
- Use a VPN on public Wi-Fi networks
- Keep router firmware updated
- Use WPA3 or WPA2 for Wi-Fi — never WEP
- Change default router admin credentials
- Disable WPS (Wi-Fi Protected Setup) — it has known vulnerabilities

### For Organizations
- Implement network segmentation and DMZ architecture
- Deploy IDS/IPS on network perimeters and internal segments
- Use zero-trust networking — verify every user and device on every request
- Encrypt all traffic (enforce HTTPS, use TLS 1.3)
- Monitor logs continuously (SIEM)
- Conduct regular network penetration tests
- Maintain an asset inventory — you can't protect what you don't know exists

---

*Document prepared as part of Cybersecurity Internship – Week 2, Task 2*
*References: CompTIA Security+, NIST SP 800-41, Cisco Networking Academy*
