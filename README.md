# NMAP-Analysis
Analysis of Nmap TCP Connect, TCP SYN, and UDP scan techniques with packet-level inspection, firewall behavior interpretation, and detection surface evaluation across reconnaissance and threat emulation workflows.

By Ramyar Daneshgar 

---

## 1. TCP Connect Scan (`-sT`)

### Overview

A TCP Connect scan initiates a full three-way handshake using the system’s native TCP stack. It sends a SYN packet, waits for a SYN-ACK response from the target, and then completes the connection by sending an ACK. Once the connection is confirmed, Nmap immediately resets the session with a RST packet.

This scan does not require root privileges, which makes it useful in restricted environments. Because it establishes real TCP sessions, the activity is visible to application logs and host-based intrusion detection systems.

### Command

```bash
nmap -sT <target_ip>
```

### Behavior

* Open ports respond with SYN-ACK, followed by Nmap completing the handshake and sending RST.
* Closed ports respond with RST-ACK, indicating no service is listening.

### Use Case

When running on systems without elevated privileges, TCP Connect is the most reliable method for identifying open TCP ports. It is effective in confirming service availability and is frequently used in controlled environments such as vulnerability validation, compliance testing, or sandbox labs.

### Lessons Learned

* Produces accurate results but generates significant traffic that can be logged or flagged by security controls.
* Best used when stealth is not required or when root access is unavailable.
* Connection logs on the remote service provide forensic evidence of probing activity.

---

## 2. TCP SYN Scan (`-sS`)

### Overview

The TCP SYN scan, often called a half-open scan, sends a SYN packet and waits for a response. If the target port replies with SYN-ACK, Nmap does not complete the handshake—instead, it sends a RST to tear down the connection before it is established.

This method allows Nmap to identify open ports without initiating a complete TCP session, reducing the likelihood of triggering logging mechanisms at the application layer.

### Command

```bash
sudo nmap -sS <target_ip>
```

### Behavior

* Open ports respond with SYN-ACK, and Nmap replies with RST to avoid completing the session.
* Closed ports respond with RST.
* Filtered ports result in no response or ICMP unreachable messages, depending on firewall configuration.

### Use Case

This is the preferred scan type for users with root privileges. It provides accurate results while minimizing the visibility of scanning activity. It is especially effective during external reconnaissance, stealth assessments, or red team engagements where avoiding detection is important.

### Lessons Learned

* Combines high accuracy with a lower risk of detection compared to full-connect scans.
* Requires raw socket access, making it suitable for privileged environments.
* Can bypass application-level logging in most cases, but may still be detected at the network layer.

---

## 3. UDP Scan (`-sU`)

### Overview

UDP scans operate differently due to the connectionless nature of the protocol. Since UDP does not require handshakes, open ports may not return any response. Nmap interprets silence as a potential open|filtered state and relies on receiving ICMP "port unreachable" messages to identify closed ports.

Because UDP services often do not reply unless a specific payload is sent, and many systems rate-limit or block ICMP, UDP scans are inherently less reliable and more ambiguous.

### Command

```bash
sudo nmap -sU -F -v <target_ip>
```

* `-F` limits the scan to the top 100 most common ports.
* `-v` enables verbose output.

### Behavior

* Closed ports send back ICMP Type 3 Code 3 (port unreachable), confirming no service is listening.
* No response may indicate the port is open, filtered, or the response is being blocked.
* Some open ports (such as DNS or SNMP) may respond with protocol-specific data.

### Use Case

UDP scanning is essential when assessing services that run exclusively over UDP, such as DNS, SNMP, or NTP. Despite its limitations, it can reveal misconfigured or exposed services not identified by TCP scans.

When assessing segmented environments or performing firewall validation, UDP scans confirm whether services are appropriately blocked or if unintended exposure exists.

### Lessons Learned

* Less reliable due to lack of acknowledgment or feedback from open ports.
* Best used with additional tools or payloads to confirm service behavior.
* ICMP filtering and rate limiting can distort scan accuracy.
* Should be complemented with application-layer testing for validation.

---

## Summary of Scan Characteristics

| Scan Type           | Privilege Required | Detection Risk | Accuracy | Preferred Scenario                           |
| ------------------- | ------------------ | -------------- | -------- | -------------------------------------------- |
| TCP Connect (`-sT`) | No                 | High           | High     | Lab testing, no root access                  |
| TCP SYN (`-sS`)     | Yes                | Medium         | High     | Stealth assessments, red teaming             |
| UDP (`-sU`)         | Yes                | Medium–High    | Medium   | Service exposure checks, firewall validation |

---

## Final Notes

Understanding how each scan type operates at the network level is critical when performing reconnaissance or validating defensive controls. This includes:

* Knowing which ports are exposed and responding
* Understanding how session establishment affects log visibility
* Recognizing how firewalls and IDS/IPS platforms influence response behavior
* Interpreting scan results in context, rather than assuming absence of response means absence of service

This level of protocol-aware enumeration ensures precise, targeted scanning and avoids unnecessary noise or misinterpretation. It also reflects a deeper understanding of both adversarial and defensive perspectives, making it a foundational skill in penetration testing, security engineering, and threat detection.
