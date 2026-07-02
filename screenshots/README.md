# Screenshots – Evidence Folder

| Filename | What to Capture |
|----------|----------------|
| `wireshark_dns_capture.png` | `dns` filter — DNS query/response packets |
| `wireshark_https_tls.png` | `tcp.port == 443` — TLS handshake + Application Data |
| `wireshark_icmp_ping.png` | `icmp` filter — ping packets after `ping google.com` |
| `wireshark_arp.png` | `arp` filter — ARP broadcasts |

## Steps
1. Wireshark → select Wi-Fi → Start capture
2. Visit websites + run `ping google.com` in CMD
3. Stop after 2-3 min → apply each filter → screenshot
