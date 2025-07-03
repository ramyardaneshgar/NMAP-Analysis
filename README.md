# NMAP-Analysis
Analysis of Nmap TCP Connect, TCP SYN, and UDP scan techniques with packet-level inspection, firewall behavior interpretation, and detection surface evaluation across reconnaissance and threat emulation workflows.

By Ramyar Daneshgar 


## 1. TCP Connect Scan (`-sT`)

### Overview

In this scan, I initiated a full TCP three-way handshake using the system’s native TCP stack. This means I delegated session establishment to the operating system itself using `connect()`-style system calls. Because this method does not rely on crafting raw packets, I didn’t need elevated privileges.

The scanner sends a SYN to the target port. If the port is open, the target responds with a SYN-ACK, and the connection completes with an ACK. Once confirmed, Nmap immediately resets the session using RST.

Since this method completes legitimate sessions, it triggers logging at the application level and is easily observed by host-based intrusion detection systems (HIDS).

### Command

```bash
nmap -sT <target_ip>
```

### Behavior

* Open ports: SYN → SYN-ACK → ACK → RST
* Closed ports: SYN → RST-ACK

### Use Case

I use TCP Connect scans when operating without root access, such as on restricted environments or during safe-list validation of exposed services. It’s highly accurate but also highly detectable.

### Lessons Learned

* Effective for confirming active TCP services when stealth is not a concern.
* Logs are created at the application layer, which leaves artifacts for forensic teams.
* Best suited for lab environments, compliance testing, or validation prior to firewall configuration changes.

---

## 2. TCP SYN Scan (`-sS`)

### Overview

This scan allowed me to send TCP SYN packets and interpret responses without completing a full handshake. As soon as I received a SYN-ACK from an open port, I sent a RST, effectively preventing the session from being fully established. This made the scan stealthier and less likely to be logged by services or endpoint detection tools.

This method required raw socket access, which I had through root privileges.

### Command

```bash
sudo nmap -sS <target_ip>
```

### Behavior

* Open ports: SYN → SYN-ACK → RST
* Closed ports: SYN → RST
* Filtered: No response or ICMP unreachable (depending on firewall rules)

### Use Case

This is my preferred method when I need to minimize the footprint on the target while maintaining scan reliability. I use it during red team reconnaissance, external asset mapping, or when operating under strict rules of engagement that limit scan visibility.

### Lessons Learned

* Balances stealth and precision.
* Not logged by most application-layer services.
* Allows me to enumerate services without leaving persistent logs or completing TCP sessions.
* Remains effective even under basic firewall configurations.

---

## 3. UDP Scan (`-sU`)

### Overview

Because UDP is connectionless, I had to change my approach. Instead of relying on handshakes, I sent raw UDP packets to target ports and waited. If a port was closed, I would receive an ICMP Type 3 Code 3 message (port unreachable). If I received no response, I had to treat the port as open|filtered.

Many systems throttle or block ICMP, so the lack of response wasn’t always conclusive. This ambiguity made interpreting results more complex than TCP scans.

### Command

```bash
sudo nmap -sU -F -v <target_ip>
```

* `-F`: Scan top 100 common ports
* `-v`: Enable verbosity for progress and feedback

### Behavior

* Closed ports: UDP → ICMP Type 3 Code 3
* Open or filtered: No response
* Open and active: May respond with service-specific UDP data (DNS, SNMP)

### Use Case

I use UDP scans during exposure assessments when evaluating firewall configurations or identifying unintentional services left open on external interfaces. This includes DNS, NTP, and SNMP audits. Because many services do not reply without crafted payloads, I often follow up with protocol-specific tools.

### Lessons Learned

* Less predictable than TCP scans, especially under ICMP restrictions.
* Valuable for validating the security posture of edge-facing services.
* Most useful when combined with protocol-aware tools or additional reconnaissance layers.
* ICMP blocking heavily degrades reliability, requiring additional verification.

---

## Scan Comparison Summary

| Scan Type           | Privilege Required | Detection Surface | Accuracy | Operational Role                                          |
| ------------------- | ------------------ | ----------------- | -------- | --------------------------------------------------------- |
| TCP Connect (`-sT`) | No                 | High              | High     | Used when unprivileged; logged by most services           |
| TCP SYN (`-sS`)     | Yes                | Medium            | High     | Preferred for stealth; widely used in recon               |
| UDP (`-sU`)         | Yes                | High              | Medium   | Essential for UDP services; requires interpretation logic |

---

## Final Notes

When performing reconnaissance, it’s not enough to know *how* to use Nmap — it’s critical to understand *why* a particular scan behaves the way it does.

By stepping through TCP state behavior, response patterns, and firewall interactions, I was able to gain a deeper understanding of network enumeration from both an offensive and defensive perspective.

This understanding supports:

* Asset discovery during threat emulation
* Detection engineering and alert tuning
* Firewall rule validation and network segmentation testing
* Legal defensibility when documenting engagement methodology

Each scan type serves a distinct purpose in the cybersecurity lifecycle. Whether validating exposure during a pentest or confirming closed ports during blue team hardening, choosing the right method — and understanding its network and logging implications — is foundational to secure operations.
