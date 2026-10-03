# 🛡️ Cybersecurity Interview & Career Engineering Curriculum

A comprehensive, production-grade cybersecurity study curriculum and technical interview preparation guide designed for aspiring **Security Operations Center (SOC) Analysts, Junior Information Security Engineers, Cloud/AppSec Associates, and Cyber Risk Analysts**.

Each module builds upon the previous one in strict pedagogical order—establishing computational and networking fundamentals before exploring operating system internals, defensive principles, cryptographic primitives, and enterprise attack scenarios.

---

## 🗺️ Chronological Learning Roadmap

```
                    ┌────────────────────────────────────────────────────────┐
                    │  STAGE 1: COMPUTATIONAL & ARCHITECTURAL FOUNDATIONS     │
                    │  [01] Computer Science Fundamentals                    │
                    │  [02] Computer Networking & Web Protocols              │
                    │  [03] Operating System Security (Windows & Linux)      │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │  STAGE 2: CORE SECURITY PRINCIPLES & CRYPTOGRAPHY      │
                    │  [04] Cybersecurity Principles & Risk Management       │
                    │  [05] Cryptography & Public Key Infrastructure (PKI)   │
                    │  [06] Identity & Access Management (IAM)               │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │  STAGE 3: ATTACK VECTORS & THREAT MANAGEMENT           │
                    │  [07] Web Application & API Security                   │
                    │  [08] Threats, Vulnerability Management & Attacks      │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │  STAGE 4: ENTERPRISE OPERATIONS & MODERN PLATFORMS     │
                    │  [09] Security Operations (SOC), IR & Tooling          │
                    │  [10] DevOps & Cloud Security                          │
                    │  [11] AI & Machine Learning Security                   │
                    └──────────────────────────┬─────────────────────────────┘
                                               │
                                               ▼
                    ┌────────────────────────────────────────────────────────┐
                    │  STAGE 5: RECRUITMENT & INTERVIEW READINESS            │
                    │  [12] Candidate Interview & Assessment Playbook        │
                    │       (Mercer | Mettl SJT, GD Themes, Project Defense) │
                    └────────────────────────────────────────────────────────┘
```

---

## 📚 Curriculum Modules & Table of Contents

| Module | File | Core Topics Covered | Primary Target Roles |
|---|---|---|---|
| **01** | [**Computer Science Fundamentals**](01_Computer_Science_Fundamentals.md) | Data Structures, Big-O, DBMS (Joins, 1NF–3NF, ACID, Indexes), OS Concepts (Threads, Scheduling, Deadlocks, Virtual Memory & Paging), OOP (Encapsulation, Abstraction, Inheritance, Polymorphism) | All Engineering & Security Roles |
| **02** | [**Networking Fundamentals**](02_Networking_Fundamentals.md) | OSI vs TCP/IP Models, TCP 3-Way Handshake & Flags, Subnetting ($2^n-2$), ARP, DNS Resolution Hierarchy, HTTP vs HTTPS, SSL/TLS 1.3 Handshake (ECDHE vs RSA), Ports, VPNs | SOC Analyst, Network Security |
| **03** | [**Operating System Security**](03_Operating_System_Security.md) | Windows vs Linux Architecture, File Permissions (`rwx`/octal, ACLs), Windows Registry Hives, Event IDs, Linux Logs (`auth.log`), Sysmon, LSASS Memory Dumping, SUID Privesc, Sudoers Hardening | SOC Analyst, Incident Responder, Sysadmin |
| **04** | [**Cybersecurity Principles & Risk**](04_Cybersecurity_Principles_and_Risk.md) | CIA Triad & DAD Inversion, AAA Framework, Security Controls (Preventive/Detective/Compensating), Quantitative Risk ($\text{ALE} = \text{SLE} \times \text{ARO}$), STRIDE Threat Modeling, GRC (NIST CSF 2.0, ISO 27001, SOC 2) | Security Analyst, GRC Associate |
| **05** | [**Cryptography & PKI**](05_Cryptography_and_PKI.md) | Symmetric (AES-GCM/AEAD) vs Asymmetric (RSA vs ECC), Cryptographic Hashing (SHA-256, Argon2id, bcrypt), Digital Signatures, X.509 PKI Trust Chains, Revocation (CRL, OCSP, OCSP Stapling), PFS | AppSec, Cryptography, Cloud Security |
| **06** | [**Identity & Access Management**](06_Identity_and_Access_Management.md) | Identity Lifecycle, FIDO2/WebAuthn Passkeys, SSO Architecture, SAML vs OAuth 2.0 vs OIDC, Authorization Models (RBAC, ABAC, DAC, MAC), PAM, Kerberos Architecture, AD Attacks (Kerberoasting, Golden Ticket) | IAM Engineer, Enterprise Security |
| **07** | [**Web & API Security**](07_Web_Application_and_API_Security.md) | Same-Origin Policy (SOP), CORS Pre-flight, Cookie Flags (`HttpOnly`, `Secure`, `SameSite`), JWT Security (`alg: none`), OWASP Top 10 with Exploit Payloads & Code Fixes (SQLi, XSS, CSRF, SSRF, IDOR), API Security (BOLA) | AppSec Engineer, PenTester, SOC Analyst |
| **08** | [**Vulnerability Management & Threats**](08_Vulnerability_Management_and_Threats.md) | VAPT Differences (Black/White/Grey Box), Vulnerability Lifecycle, CVSS v3.1 Scoring, CVE/CWE/NVD, Network Attacks (ARP Poisoning, DNS Hijacking, DDoS Vectors), Malware Taxonomy, LOLBins | Vulnerability Analyst, Red/Blue Team |
| **09** | [**SecOps, Incident Response & Tooling**](09_Security_Operations_Incident_Response_and_Tooling.md) | Tiered SOC Structure, SIEM vs EDR vs SOAR, NIST SP 800-61 & SANS 6-Phase IR Lifecycles, Cyber Kill Chain, MITRE ATT&CK, Pyramid of Pain, Nmap Flags, Wireshark Display Filters, tcpdump, Burp Suite | SOC Analyst, Incident Responder |
| **10** | [**DevOps & Cloud Security**](10_DevOps_and_Cloud_Security.md) | Git 3-Tree Architecture, Bash/Python Automation, CI/CD DevSecOps Tooling (SAST, DAST, SCA), Docker vs VM Isolation, Kubernetes RBAC, IaC Security (`tfsec`), Cloud Models (IaaS/PaaS/SaaS), Shared Responsibility | DevSecOps, Cloud Security Associate |
| **11** | [**AI & Machine Learning Security**](11_AI_and_Machine_Learning_Security.md) | ML Fundamentals (Supervised/Unsupervised), Overfitting Mitigations, LLM Architecture (RAG vs Fine-Tuning), AI Attacks (Prompt Injection, Data Poisoning, Adversarial Evasion), OWASP LLM Top 10, UEBA in SOC | AI Security, Next-Gen SecOps |
| **12** | [**Candidate Interview Playbook**](12_Candidate_Interview_Playbook.md) | Mercer \| Mettl Situational Judgment (SJT) Scoring Rules, Group Discussion (GD) 3-Theme Framework, Academic/Internship Defense (C-A-R-S), "Think Aloud" Troubleshooting, Top 15 Technical Interview Q&A | All Candidates Preparing for Placement / Interviews |

---

## 🎯 How to Prepare for Interviews Using This Repository

### 1. The Mercer | Mettl Online Assessment (Round 1)
* **Soft Skills / Situational Judgment (25 Questions | 15 Mins):** Study [Module 12 (Section 12.2)](12_Candidate_Interview_Playbook.md). Review the 6 core evaluator competencies and the decision matrix (prioritizing containment, evidence over assumptions, and transparent escalation over lone-wolf actions).
* **Technical MCQs (25 Questions | 20 Mins):** Review the **Quick-Revision Summaries** and **Comparison Tables** at the end of Modules 01 through 11. Focus heavily on Subnet calculations ($2^n-2$), OSI/TCP-IP layer mapping, Port numbers, Symmetric vs Asymmetric algorithms, and the CIA Triad.

### 2. The Group Discussion Round (Round 2)
* Read [Module 12 (Section 12.3)](12_Candidate_Interview_Playbook.md).
* Memorize the **T-C-C-L-S Framework** (Think $\rightarrow$ Contribute $\rightarrow$ Connect $\rightarrow$ Listen $\rightarrow$ Summarize) and prepare speaking points for the 3 core themes:
  1. *Cybersecurity Basics & Ethics (Privacy vs Monitoring, Password Policies vs Fatigue)*
  2. *AI, Technology, and Work (AI in Learning, Attackers vs Defenders, Human-in-the-Loop)*
  3. *Workplace Judgment and Collaboration (DevSecOps Speed vs Quality, Hybrid Work)*

### 3. The Technical Panel & Project Defense (Round 3)
* Prepare your project defense using the **C-A-R-S Framework** (Context, Action/Individual Contribution, Result, Security Reflection) outlined in [Module 12 (Section 12.4)](12_Candidate_Interview_Playbook.md).
* Master the **Top 15 Technical Questions** in Section 12.5 and the **Rapid-Fire Follow-Up Questions** located at the end of each individual module.

---

## 🛠️ Contribution & Usage Guidelines
* Designed as a modular markdown resource suitable for reading on GitHub, Obsidian, or local Markdown viewers.
* All code examples, payloads, and scripts are presented strictly for defensive education, technical interview preparation, and security engineering research.
