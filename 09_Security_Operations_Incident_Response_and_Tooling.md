# 9. Security Operations, Incident Response & Tooling

## 9.1 The Security Operations Center (SOC) & Analyst Workflow

A Security Operations Center (SOC) is a centralized organizational unit responsible for continuously monitoring, detecting, analyzing, and responding to cybersecurity incidents.

### 9.1.1 Tiered SOC Structure
```
[ Incoming Telemetry & Alerts: SIEM / EDR / Cloud ]
                         │
                         ▼
┌────────────────────────────────────────────────────────┐
│ Tier 1: Triage Analyst                                 │
│ • Validates alerts; filters false positives            │
│ • Gathers initial context; escalates confirmed threats │
└────────────────────────┬───────────────────────────────┘
                         │ (Escalated Incident)
                         ▼
┌────────────────────────────────────────────────────────┐
│ Tier 2: Incident Responder                             │
│ • Deep-dive host/network forensic investigation        │
│ • Executes containment, eradication, and remediation   │
└────────────────────────┬───────────────────────────────┘
                         │ (Complex / APT Threat)
                         ▼
┌────────────────────────────────────────────────────────┐
│ Tier 3: Threat Hunter & Senior Forensic Analyst        │
│ • Proactive hypothesis-driven threat hunting           │
│ • Reverse engineering malware, tuning SIEM detections  │
└────────────────────────────────────────────────────────┘
```

### 9.1.2 Key SOC Performance Metrics
* **MTTD (Mean Time to Detect):** Average time elapsed between an attacker gaining unauthorized access and security systems/analysts detecting the incident.
* **MTTR (Mean Time to Respond / Remediate):** Average time elapsed from alert generation to full containment and remediation.
* **False Positive Rate:** Percentage of generated alerts that represent benign, normal business activity. High false positive rates lead to **Analyst Alert Fatigue**.

---

## 9.2 Enterprise Defensive Tooling: SIEM, EDR & SOAR

| Technology | Full Form | Primary Function | Data Ingested | Value in Defense |
|---|---|---|---|---|
| **SIEM** | Security Information and Event Management | Centralized log aggregation, normalization, and cross-source correlation rules | Syslog, Windows Event Logs, firewall traffic, DNS queries, cloud audit logs | Correlates disparate events across the enterprise (e.g., failed VPN login followed by immediate server file deletion) |
| **EDR** | Endpoint Detection and Response | Continuous endpoint telemetry, memory inspection, and active host response | Process creation trees, registry edits, network sockets, DLL injections | Deep host visibility; enables instant remote network isolation and process termination |
| **SOAR** | Security Orchestration, Automation, and Response | Automating repetitive workflows and coordinating multi-tool actions | SIEM alerts, threat intelligence feeds | Executes automated playbooks (e.g., auto-querying VirusTotal on alert, auto-blocking an IP on firewall) |

### 9.2.1 EDR vs Traditional Antivirus (AV)
* **Traditional Antivirus:** Relies primarily on **static signature matching** (looking for known MD5/SHA256 hashes or known byte sequences). Completely ineffective against novel malware, zero-days, and fileless LOLBins attacks.
* **EDR (Endpoint Detection and Response):** Relies on **continuous behavioral heuristics** and kernel-level event telemetry. Flags suspicious parent-child process chains (e.g., `winword.exe` $\rightarrow$ `powershell.exe` $\rightarrow$ outbound network connection) regardless of whether the executable file was previously known.

---

## 9.3 Incident Response Lifecycles (NIST SP 800-61 vs SANS)

Both frameworks outline a structured methodology for managing an active compromise:

```
[ NIST SP 800-61 Rev 2 Lifecycle ]
       ┌───────────────────────────────┐
       ▼                               │
[ Preparation ]                 [ Post-Incident Activity ]
       │                        (Lessons Learned)
       ▼                               ▲
[ Detection & Analysis ]               │
       │                               │
       ▼                               │
[ Containment, Eradication & Recovery ]─┘
```

### 9.3.1 Detailed IR Phase Execution (SANS 6-Phase Model)
1. **Preparation:**
   * Establishing response policies, communication call trees, and out-of-band communication channels (Signal/offline phones in case corporate Slack/email is monitored by attackers).
   * Ensuring offline, immutable backups and pre-installing EDR/forensic agent tooling.
2. **Identification (Detection & Analysis):**
   * Validating that an incident is occurring (distinguishing true positives from false alarms).
   * Determining scope: Identifying patient zero, affected assets, and the threat actor's entry vector.
   * Documenting all findings with strict **Chain of Custody** for forensic evidence.
3. **Containment:**
   * *Short-term Containment:* Isolating compromised endpoints from the local network (disconnecting network cable or issuing EDR network isolation) while leaving RAM powered on for memory forensics. Blocking malicious IP addresses on the boundary firewall.
   * *Long-term Containment:* Revoking compromised user credentials, resetting Kerberos `KRBTGT` account passwords (twice), and placing targeted systems in isolated sandbox VLANs.
4. **Eradication:**
   * Removing the root cause: Terminating malicious processes, deleting persistence mechanisms (Registry Run keys, scheduled tasks, web shells), patching vulnerable public-facing software.
   * Best practice in cloud/virtual environments: Rather than manually "cleaning" an infected OS image, **destroy and redeploy** from verified clean baseline templates.
5. **Recovery:**
   * Restoring systems to clean production operations from trusted backups.
   * Implementing enhanced monitoring and logging on the restored systems to detect any re-infection attempts.
6. **Lessons Learned (Post-Incident Review):**
   * Conducting a post-mortem review within two weeks of the incident.
   * Answering: *What happened? How quickly was it detected? Where were the visibility blind spots? What controls must be added to prevent recurrence?*

---

## 9.4 Threat Intelligence & Adversary Frameworks

### 9.4.1 The Cyber Kill Chain (Lockheed Martin)
Linear model outlining the seven distinct stages of a sophisticated targeted cyber attack:

```
[ Reconnaissance ] ──► [ Weaponization ] ──► [ Delivery ] ──► [ Exploitation ]
                                                                     │
[ Actions on Objectives ] ◄── [ Command & Control ] ◄── [ Installation ]
```

1. **Reconnaissance:** Adversary harvests information (OSINT, LinkedIn, Shodan, port scans).
2. **Weaponization:** Coupling an exploit with a malicious payload (e.g., generating an infected PDF or Office macro).
3. **Delivery:** Transmitting the weaponized package to the victim (phishing email, compromised website, malicious USB).
4. **Exploitation:** Malicious code executes on the victim's machine by triggering an unpatched software vulnerability or user macro execution.
5. **Installation:** Malware installs persistent footholds on the host (services, Registry Run keys, rootkits).
6. **Command and Control (C2):** Compromised host reaches out across the internet to an attacker-controlled server to receive commands.
7. **Actions on Objectives:** Attacker achieves ultimate goal (data exfiltration, ransomware encryption, lateral movement).

> **Defensive Value:** A defender only needs to break **any one link** in the chain to disrupt the entire attack.

### 9.4.2 The MITRE ATT&CK Framework
A globally accessible, community-driven knowledge base of adversary **Tactics, Techniques, and Procedures (TTPs)** based on real-world observations.

* **Tactic (The "Why"):** The adversary's tactical objective (e.g., *Initial Access, Persistence, Privilege Escalation, Defense Evasion, Lateral Movement, Exfiltration*).
* **Technique (The "How"):** The specific technical method used to accomplish the tactical objective (e.g., under *Persistence*, Technique `T1547: Boot or Logon Autostart Execution`).
* **Procedure (The Specific Implementation):** The exact code, software, or script an actor used (e.g., *APT29 using PowerShell to add a specific Registry Run key*).

### 9.4.3 The Pyramid of Pain (David Bianco)
Illustrates how much difficulty and cost a defender inflicts on an adversary when detecting and blocking different types of Indicators of Compromise (IOCs):

```
       ▲  [ TTPs (Tactics, Techniques, Procedures) ] ────► TOUGH / EXTREMELY PAINFUL
      / \ [ Tools (Mimikatz, Cobalt Strike, PsExec) ] ───► CHALLENGING
     /   \ [ Network / Host Artifacts (URI, User-Agent)] ─► ANNOYING
    /     \ [ Domain Names (evil-c2.com) ] ──────────────► SIMPLE
   /       \ [ IP Addresses (198.51.100.24) ] ───────────► EASY
  /         \ [ Hash Values (SHA-256, MD5) ] ────────────► TRIVIAL
 ─────────────
```

* **Hashes (Trivial):** Changing a single byte in a malware file completely alters its SHA-256 hash, bypassing hash-based blocks in seconds.
* **TTPs (Tough):** When defenders detect and block *adversary behaviors* (e.g., detecting pass-the-hash or process hollowing), the attacker cannot simply recompile code; they must spend weeks redesigning their entire offensive tradecraft.

---

## 9.5 Practical Security Tooling & Command Cheat Sheet

### 9.5.1 Nmap (Network Mapper)
Essential tool for network discovery, port scanning, and vulnerability surface auditing:

```bash
# TCP SYN Stealth Scan (Default, root required, half-open scan; does not complete 3-way handshake)
nmap -sS -p 1-1000 192.168.1.10

# Full TCP Connect Scan (Non-root user; completes full SYN -> SYN-ACK -> ACK handshake)
nmap -sT -p 80,443,8080 192.168.1.10

# Service Version Detection + OS Fingerprinting
nmap -sV -O 192.168.1.10

# Comprehensive / Aggressive Scan (Version detection, OS detection, script scanning, traceroute)
nmap -A -T4 192.168.1.0/24

# Scanning UDP Ports (DNS, SNMP, DHCP)
nmap -sU -p 53,161 192.168.1.10

# Vulnerability Script Scan using Nmap Scripting Engine (NSE)
nmap --script vuln 192.168.1.10
```

> **Interview Concept — SYN Scan (`-sS`) vs Connect Scan (`-sT`):**
> * A **SYN Scan** sends a `SYN`. If the port is open, the target replies with `SYN-ACK`. Nmap immediately sends an `RST` to tear down the connection before it completes. This makes it faster and less likely to be logged by legacy applications.
> * A **Connect Scan** uses the operating system's standard `connect()` system call, completing the full 3-way handshake. Slower and reliably recorded in application access logs.

### 9.5.2 Wireshark & Packet Analysis
Wireshark is the standard GUI packet capture and protocol analyzer.

#### Essential Wireshark Display Filters for Threat Triage
| Inspection Target | Wireshark Display Filter Syntax |
|---|---|
| Filter by specific IP | `ip.addr == 192.168.1.50` |
| View outgoing HTTP POST requests | `http.request.method == "POST"` |
| Spot potential TCP SYN floods | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Identify DNS queries and lookups | `dns.flags.response == 0` |
| Detect cleartext credential submissions | `frame contains "password" || frame contains "username"` |
| Spot suspicious TCP RST resets | `tcp.flags.reset == 1` |

### 9.5.3 Tcpdump Command-Line Packet Sniffer
Used on headless Linux servers and firewalls to capture raw traffic:
```bash
# Capture traffic on eth0, don't resolve hostnames/ports (-n), filter for port 80, save to pcap
tcpdump -i eth0 -n "port 80" -w web_traffic.pcap

# Read and inspect an existing pcap file on terminal
tcpdump -r web_traffic.pcap -c 20
```

### 9.5.4 Burp Suite Core Architecture
The industry-standard intercepting HTTP proxy for web application security assessments:
* **Proxy (Intercept):** Sits between browser and server, capturing and allowing manual modification of HTTP/HTTPS requests before they reach the web application.
* **Repeater:** Allows testers to resend individual requests repeatedly with manual payload modifications, observing differential responses.
* **Intruder:** Automates fuzzing and dictionary attacks across targeted parameter fields (e.g., brute-forcing IDs, directories, or parameters).

---

## 9.6 Rapid-Fire Follow-Up Questions (SecOps & Tooling)

- *Why is a TCP SYN scan considered stealthier than a TCP Connect scan?* (A SYN scan terminates the connection with an RST packet immediately after receiving the SYN-ACK, never completing the 3-way handshake, which prevents application-layer connection logging).
- *What is the difference between SIEM and SOAR?* (SIEM aggregates logs and generates correlation alerts; SOAR coordinates automated response actions and executes containment playbooks across security tools).
- *In the Pyramid of Pain, why are hash values considered "trivial" to block while TTPs are "tough"?* (An attacker can change a hash instantly by changing one byte of comments or compiling with different flags; changing TTPs requires the adversary to redesign their entire offensive methodology).
- *During Incident Response containment, why shouldn't an analyst immediately pull the power plug on an infected Windows server?* (Pulling the plug destroys all volatile RAM data, losing running processes, in-memory malware artifacts, network socket connections, and injected code needed for forensic root cause analysis).

---

## Quick-Revision Summary

- **SOC Tiers:** Tier 1 (triage & validation), Tier 2 (incident response), Tier 3 (threat hunting & advanced forensics).
- **Core Metrics:** MTTD (time to detect) and MTTR (time to remediate).
- **SIEM vs EDR:** SIEM aggregates and correlates logs across devices; EDR provides continuous endpoint telemetry and process-level response.
- **Incident Response Phases:** Preparation $\rightarrow$ Identification $\rightarrow$ Containment $\rightarrow$ Eradication $\rightarrow$ Recovery $\rightarrow$ Lessons Learned.
- **Kill Chain (7 Steps):** Recon $\rightarrow$ Weaponize $\rightarrow$ Deliver $\rightarrow$ Exploit $\rightarrow$ Install $\rightarrow$ C2 $\rightarrow$ Actions on Objectives.
- **Pyramid of Pain:** Hashes (trivial), IPs (easy), Domains (simple), Artifacts (annoying), Tools (tough), **TTPs (painful)**.
- **Nmap Flags:** `-sS` (SYN stealth), `-sT` (Connect), `-sV` (version), `-O` (OS), `-p-` (all ports), `-A` (aggressive).
- **Wireshark Filters:** `ip.addr`, `http.request.method`, `tcp.flags.syn==1 && tcp.flags.ack==0`, `dns.flags.response==0`.
