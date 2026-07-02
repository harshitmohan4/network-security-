# Common Network Attacks – Theory & Prevention

**Author:** Harshit
**Task:** Week 2 – Task 2 | Cybersecurity Internship
**Date:** June 2025

> ⚠️ **Disclaimer:** This document is for educational purposes only. All attack descriptions are theoretical. No attacks were performed on any network, device, or system. Understanding how attacks work is essential for building effective defenses.

---

## 1. Man-in-the-Middle (MITM) Attack

### 1.1 What is MITM?

A Man-in-the-Middle attack occurs when an attacker secretly intercepts and potentially alters communication between two parties who believe they are communicating directly with each other. The attacker positions themselves between the victim and the destination — silently reading, modifying, or injecting data.

```
Normal communication:
[Alice] ←————————————————→ [Server]

MITM attack:
[Alice] ←——→ [Attacker] ←——→ [Server]
              (intercepts & reads/modifies everything)
```

### 1.2 How MITM Occurs

#### Technique 1: ARP Poisoning (most common on local networks)

ARP (Address Resolution Protocol) maps IP addresses to MAC addresses on a local network. It has no authentication — any device can send ARP replies, even unsolicited ones.

```
Normal ARP:
Alice asks: "Who has IP 192.168.1.1?" (the router)
Router replies: "I do — my MAC is AA:BB:CC:DD:EE:FF"

ARP Poisoning:
Alice asks: "Who has IP 192.168.1.1?"
Attacker replies FIRST: "I do — my MAC is 11:22:33:44:55:66"
Alice now sends all traffic to the attacker instead of the router
```

The attacker also poisons the router's ARP cache — telling the router that Alice's IP belongs to the attacker's MAC. Now all traffic flows through the attacker in both directions.

#### Technique 2: DNS Spoofing

Attacker corrupts the DNS cache on a router or DNS server, mapping a legitimate domain to the attacker's IP.

```
Victim types: www.bank.com
Poisoned DNS returns: 192.168.1.100 (attacker's IP)
Victim's browser connects to attacker's fake bank website
```

#### Technique 3: Evil Twin Wi-Fi

```
1. Attacker creates a fake Wi-Fi hotspot named "CoffeeShop_Free_WiFi"
2. Real hotspot also exists nearby
3. Attacker broadcasts stronger signal — victim's device connects to fake AP
4. All victim's traffic now flows through attacker's device
```

#### Technique 4: SSL Stripping

Even when a site uses HTTPS, an attacker can downgrade the connection:
```
1. Victim requests https://bank.com
2. Attacker intercepts and forwards request to bank (over HTTPS)
3. Attacker sends response back to victim over HTTP (not HTTPS)
4. Victim sees http://bank.com — attacker sees all plaintext traffic
```

### 1.3 Real-World Impact

- **Credential theft:** Login credentials captured in plaintext (if HTTP)
- **Session hijacking:** Session cookies stolen → account takeover
- **Data manipulation:** Attacker modifies transaction amounts or recipient details
- **Malware injection:** Attacker injects malicious scripts into HTTP responses

### 1.4 How to Detect MITM

- Unexpected SSL certificate warnings in the browser
- ARP cache showing duplicate MAC addresses for different IPs
- Latency increase (traffic taking an extra hop through attacker)
- Unusual DNS responses (DNS query results not matching expected IPs)

### 1.5 Prevention

| Control | How it Prevents MITM |
|---------|---------------------|
| **HTTPS / TLS** | Encrypts traffic — attacker captures encrypted gibberish |
| **HSTS** | Forces browser to always use HTTPS — prevents SSL stripping |
| **Certificate Pinning** | App rejects certificates not matching expected fingerprint |
| **Dynamic ARP Inspection (DAI)** | Switch validates ARP packets — blocks ARP poisoning |
| **VPN** | Encrypts all traffic end-to-end — MITM attacker sees only encrypted tunnel |
| **DNSSEC** | Cryptographically signs DNS records — prevents DNS spoofing |
| **MFA** | Even if credentials are stolen, attacker can't log in without second factor |

```
# Check your ARP cache for suspicious duplicates (Windows):
arp -a

# If two different IPs share the same MAC → possible ARP poisoning
```

---

## 2. DDoS (Distributed Denial of Service) Attack

### 2.1 What is DDoS?

A Denial of Service (DoS) attack aims to make a service unavailable by overwhelming it with traffic or requests, exhausting its resources (bandwidth, CPU, memory, connections). A **Distributed** DoS (DDoS) uses thousands or millions of compromised machines (a **botnet**) to amplify the attack.

```
Normal traffic:
[Users] ——→ [Server] (handles requests normally)

DDoS Attack:
[Botnet: 100,000 compromised devices] ——→ [Server]
                                         (overwhelmed, crashes)
```

### 2.2 Types of DDoS Attacks

#### Volume-Based Attacks (Layer 3/4)
Flood the target with massive amounts of traffic, saturating bandwidth.

| Attack | Method | Scale |
|--------|--------|-------|
| **UDP Flood** | Sends random UDP packets to ports, forcing server to respond with ICMP unreachable | Up to Tbps |
| **ICMP Flood (Ping Flood)** | Overwhelming ping requests | Large volumes |
| **DNS Amplification** | Attacker sends small DNS queries with spoofed source IP → DNS servers send large responses to victim | 70x amplification |
| **NTP Amplification** | Exploits NTP monlist command → huge responses | 556x amplification |

**DNS Amplification example:**
```
Attacker sends: 40-byte DNS query (with victim's IP as source)
DNS server sends: 4,000-byte response → to victim
Amplification: 100x — tiny request, massive response hitting the victim
```

#### Protocol Attacks (Layer 4)
Exploit protocol weaknesses to exhaust server resources.

**SYN Flood:**
```
Attacker sends thousands of SYN packets (spoofed source IPs)
Server responds with SYN-ACK and waits for ACK (never comes)
Server's connection table fills up → legitimate connections refused
```

#### Application Layer Attacks (Layer 7)
Target the application itself with seemingly legitimate requests.

**HTTP Flood:**
```
Botnet sends millions of legitimate-looking HTTP GET/POST requests
Server processes each request (DB queries, computations)
Server CPU/memory exhausted
Much harder to detect — traffic looks real
```

**Slowloris:**
```
Attacker opens many connections to the server
Sends partial HTTP headers very slowly — keeps connections open
Server holds all connections open waiting for completion
Eventually exhausts max connection limit — legitimate users blocked
```

### 2.3 Real-World DDoS Examples

- **GitHub (2018):** 1.35 Tbps attack using Memcached amplification — largest recorded at the time
- **AWS (2020):** 2.3 Tbps attack — largest ever recorded
- **Dyn DNS (2016):** Mirai botnet (IoT devices) took down Twitter, Netflix, Reddit, GitHub

### 2.4 Impact

- Service unavailability — financial losses (Amazon loses ~$220,000/minute of downtime)
- Reputational damage
- Used as a smokescreen while other attacks occur (data theft during chaos)
- Extortion — "pay us or we keep attacking"

### 2.5 Prevention

| Control | Effectiveness |
|---------|--------------|
| **CDN with DDoS protection** (Cloudflare, AWS Shield) | Absorbs volumetric attacks at edge | 
| **Rate limiting** | Limits requests per IP per time window |
| **IP reputation filtering** | Blocks known botnet IPs |
| **Anycast network diffusion** | Spreads attack traffic across many data centers |
| **BGP blackholing** | Drops all traffic to attacked IP at ISP level |
| **Web Application Firewall (WAF)** | Blocks Layer 7 attacks |
| **Network over-provisioning** | Extra bandwidth capacity to absorb spikes |

---

## 3. Packet Sniffing

### 3.1 What is Packet Sniffing?

Packet sniffing (also called network sniffing or eavesdropping) is the practice of capturing and analyzing network packets as they travel across a network. Network interface cards (NICs) can be set to **promiscuous mode**, allowing them to capture all packets on the network — not just those addressed to them.

**Legitimate uses:**
- Network troubleshooting by administrators
- Security monitoring and traffic analysis
- Performance monitoring
- Penetration testing (with authorization)

**Malicious uses:**
- Stealing credentials transmitted in plaintext (HTTP, FTP, Telnet)
- Session cookie theft
- Capturing sensitive data (emails, files)
- Reconnaissance for further attacks

### 3.2 How Packet Sniffing Works

#### On a Hub-based Network (old)
```
Hub broadcasts every packet to ALL ports
→ Any device in promiscuous mode sees ALL traffic
→ Trivially easy to sniff
```

#### On a Switch-based Network (modern)
```
Switch learns MAC addresses → sends packets only to the correct port
→ Device normally sees only its own traffic + broadcasts
→ Attacker must use ARP poisoning to redirect traffic through themselves first
```

#### On Wireless Networks
```
Wi-Fi broadcasts signals in all directions
→ Any device in monitor mode can capture all packets in range
→ Unencrypted Wi-Fi (open hotspots) = all traffic visible to anyone
→ WPA2/3 encrypted Wi-Fi = traffic encrypted, but still capturable (harder to decrypt)
```

### 3.3 What Attackers Can See

| Protocol | Port | What's Visible |
|----------|------|---------------|
| HTTP | 80 | Full request + response, credentials, cookies, content |
| FTP | 21 | Username and password in plaintext |
| Telnet | 23 | Everything — all commands, output, credentials |
| SMTP (unencrypted) | 25 | Email content and credentials |
| HTTPS | 443 | Only encrypted data — content not visible |
| DNS | 53 | Domain names being queried (even over HTTPS) |
| SSH | 22 | Encrypted — only connection metadata visible |

### 3.4 Wireshark – The Standard Packet Analyzer

**Wireshark** is the world's most widely used network protocol analyzer. It is open-source, free, and used by both network administrators and security professionals.

**Key features:**
- Captures live traffic on network interfaces
- Deep protocol inspection for 3,000+ protocols
- Powerful display and capture filters
- Can reconstruct TCP streams (reassemble HTTP conversations)
- Saves captures as `.pcap` files for offline analysis

**Common Wireshark filters:**

```
# Show only HTTP traffic
http

# Show only DNS queries
dns

# Show traffic to/from specific IP
ip.addr == 192.168.1.1

# Show only TCP traffic
tcp

# Show traffic on specific port
tcp.port == 443

# Show only packets with TCP SYN flag (new connections)
tcp.flags.syn == 1

# Show only ARP packets (useful for detecting ARP poisoning)
arp
```

### 3.5 Prevention

| Control | How it Prevents Sniffing |
|---------|-------------------------|
| **TLS/HTTPS** | Encrypts traffic — sniffer captures ciphertext only |
| **VPN** | All traffic encrypted in tunnel |
| **Switched networks** | Packets only sent to intended recipient |
| **Wi-Fi encryption (WPA3)** | Wireless traffic encrypted |
| **SFTP / SSH instead of FTP / Telnet** | Encrypted alternatives to plaintext protocols |
| **Network monitoring** | Detect unusual promiscuous mode NICs on the network |
| **802.1X port authentication** | Only authenticated devices can join the network |

---

## Attack Comparison Summary

| Feature | MITM | DDoS | Packet Sniffing |
|---------|------|------|----------------|
| **Goal** | Intercept/alter data | Disrupt service | Eavesdrop on traffic |
| **Target** | Individual users | Server/service availability | Network traffic |
| **Requires network access** | Yes (same network) | No (internet) | Yes (same network) |
| **Passive/Active** | Active | Active | Passive |
| **Detection difficulty** | Moderate | Easy (service goes down) | Hard (no disruption) |
| **Primary defense** | TLS + HSTS + VPN | CDN + rate limiting | Encryption (TLS/VPN) |
| **OWASP relevance** | A02, A07 | A05 (availability) | A02 |

---

*Document prepared as part of Cybersecurity Internship – Week 2, Task 2*
*References: NIST SP 800-41, CompTIA Security+, Wireshark Documentation, Cloudflare DDoS Guide*
