# 1. Cybersecurity Fundamentals

## 1.1 CIA Triad

### 1.1.1 Confidentiality
**Meaning:** Ensuring information is accessible only to authorized users.

### 1.1.2 Integrity
**Meaning:** Ensuring data is accurate, consistent, and unmodified without authorization.

### 1.1.3 Availability
**Meaning:** Ensuring systems and data are accessible when required by authorized users.

> **Interview Tip:** Interviewers often ask you to map a real attack to the triad — e.g., ransomware primarily hits **Availability** (and sometimes Confidentiality if data is stolen too).

---

## 1.2 AAA and Access Concepts

### 1.2.1 Identification vs Authentication vs Authorization
| Term | Meaning | Example |
|---|---|---|
| Identification | Claiming an identity | Entering a username |
| Authentication | Proving the identity | Entering a password/OTP |
| Authorization | Granting access based on identity | Allowing access to a specific folder |

**Common Interview Questions:**
- Difference between authentication and authorization?
- Give a real-world example combining all three.

### 1.2.2 Accounting / Auditing
**Meaning:** Tracking user activity for accountability (logs, audit trails).

### 1.2.3 AAA
**Meaning:** Authentication, Authorization, Accounting — the standard framework for controlling and tracking access (used in RADIUS/TACACS+).

---

## 1.3 Access Principles

### 1.3.1 Least Privilege
**Meaning:** Users/systems get only the minimum access needed to perform their job.

### 1.3.2 Need-to-Know
**Meaning:** Access to information is restricted to those who need it for their role, even if they have clearance.

### 1.3.3 Defense in Depth
**Meaning:** Using multiple layers of security controls so if one fails, others still protect the system.
**Example:** Firewall + IDS + Antivirus + MFA all protecting the same server.

### 1.3.4 Zero Trust
**Meaning:** "Never trust, always verify" — no user or device is trusted by default, even inside the network perimeter.
**Common Interview Questions:**
- What is Zero Trust and how is it different from traditional perimeter security?
- How is Zero Trust implemented (MFA, micro-segmentation, continuous verification)?

---

## 1.4 Security Controls

| Control Type | Purpose | Example |
|---|---|---|
| Administrative | Policies & procedures | Security awareness training |
| Technical | Technology-enforced | Firewalls, encryption |
| Physical | Physical barriers | Locks, CCTV, badge access |
| Preventive | Stop an incident before it happens | Firewall rules |
| Detective | Identify an incident in progress/after | IDS, SIEM alerts |
| Corrective | Fix damage after incident | Restoring from backup |
| Deterrent | Discourage attackers | Warning banners, visible cameras |
| Compensating | Alternative control when primary isn't feasible | Extra monitoring when patching isn't possible |

**Common Interview Questions:**
- Give an example of each control type.
- Difference between preventive and detective controls?
- What is a compensating control and when is it used?

---

## 1.5 Core Risk Terminology

| Term | Meaning |
|---|---|
| Asset | Anything of value to an organization (data, systems, people) |
| Vulnerability | A weakness that can be exploited |
| Threat | A potential cause of harm (actor or event) |
| Exploit | A method/code used to take advantage of a vulnerability |
| Attack | The actual act of exploiting a vulnerability |
| Risk | Likelihood × Impact of a threat exploiting a vulnerability |
| Exposure | Extent to which an asset is susceptible to loss |
| Security posture | Overall security strength of an organization |
| Attack surface | Total set of points where an attacker could try to enter |
| Threat surface | Similar to attack surface; total exposure to potential threats |

> **Interview Tip:** Threat vs Vulnerability vs Risk is one of the most commonly asked comparison questions.
> - **Threat** = who/what could cause harm
> - **Vulnerability** = the weakness
> - **Risk** = probability that the threat exploits the vulnerability and the resulting impact

**Common Interview Questions:**
- Difference between threat, vulnerability, and risk?
- What is an attack surface, and how do you reduce it?

---

## 1.6 Risk & Policy Management

### 1.6.1 Risk Assessment
**Meaning:** Process of identifying, analyzing, and evaluating risks to assets.

### 1.6.2 Risk Management
**Meaning:** Ongoing process of identifying, assessing, and mitigating/accepting/transferring/avoiding risk.
**Common responses to risk:** Accept, Avoid, Mitigate, Transfer (e.g., insurance).

### 1.6.3 Security Policies, Standards, Procedures
| Term | Meaning |
|---|---|
| Policy | High-level management directive (what must be done) |
| Standard | Mandatory specific requirements supporting a policy |
| Procedure | Step-by-step instructions to implement a policy/standard |

**Common Interview Questions:**
- Difference between a policy, standard, and procedure?
- Walk through the steps of a basic risk assessment.
- What is risk acceptance vs risk mitigation?

---

## 1.7 Extended CIA — Non-repudiation & AAA Factors

### 1.7.1 Non-repudiation
**Meaning:** Ensures a party cannot deny having performed an action (e.g., sending a message, making a transaction). Often grouped with CIA as part of the broader "CIANA" model.
**Example:** Digital signatures prove a specific user signed a document, and they cannot later deny it.

### 1.7.2 DAD Triad (Opposite of CIA)
**Meaning:** The inverse view used to describe attacks — **Disclosure, Alteration, Destruction** — mapping directly to breaking Confidentiality, Integrity, and Availability respectively.
**Interview Tip:** If asked "what does this attack violate?", think in terms of Disclosure (C), Alteration (I), Destruction (A).

### 1.7.3 Authentication Factors
| Factor Type | Description | Example |
|---|---|---|
| Something you know | Knowledge-based | Password, PIN |
| Something you have | Possession-based | OTP token, smart card |
| Something you are | Inherence-based | Fingerprint, facial recognition |
| Somewhere you are | Location-based | Geofencing, IP restriction |
| Something you do | Behavior-based | Typing pattern, gait analysis |

**Common Interview Questions:**
- Difference between MFA and 2FA? *(2FA is a subset of MFA using exactly two factors; MFA can use two or more.)*

---

## 1.8 Cryptography Fundamentals

### 1.8.1 Symmetric vs Asymmetric Encryption
| Aspect | Symmetric Encryption | Asymmetric Encryption (Public Key) |
|---|---|---|
| Keys | Single shared secret key for encryption & decryption | Key pair: Public key (encrypt) + Private key (decrypt) |
| Speed | Extremely fast (hardware accelerated) | Computationally expensive (~1000x slower) |
| Key Exchange | Difficult — how to safely share key over insecure channel? | Easy — public key can be openly shared |
| Common Algorithms | **AES** (128/256-bit standard), ChaCha20, DES/3DES (deprecated) | **RSA** (2048/4096-bit), **ECC** (ECDSA, Ed25519), Diffie-Hellman |
| Primary Use Case | Bulk data encryption (hard drives, database records, VPN payloads) | Key exchange, digital signatures, identity verification |

> **Interview Tip:** Real-world systems use **Hybrid Encryption** (e.g., in TLS/HTTPS): Asymmetric cryptography (RSA/ECC/ECDH) negotiates a shared session key, then Symmetric cryptography (AES-GCM) encrypts the bulk session data for speed.

**Common Interview Questions:**
- Why don't we use asymmetric encryption for all data transmission if it solves the key distribution problem? *(Too computationally slow for bulk traffic; hybrid encryption is the practical solution)*
- Difference between RSA and ECC? *(ECC provides equivalent security with much smaller key sizes, e.g., 256-bit ECC ≈ 3072-bit RSA, making it faster with less power/bandwidth)*

### 1.8.2 Hashing vs Encryption vs Encoding
| Concept | Nature | Reversible? | Purpose | Examples |
|---|---|---|---|---|
| **Encoding** | Data formatting | Yes (no key needed) | Usability, transport across text systems | Base64, Hex, URL encoding |
| **Hashing** | One-way cryptographic digest | **No** (mathematically irreversible) | Integrity verification, password verification | SHA-256, SHA-3, bcrypt, Argon2 |
| **Encryption** | Secret transformation using a key | **Yes** (with the right key) | Confidentiality | AES-256, RSA |

> **Interview Tip:** Never use simple hashes (MD5, SHA-1, SHA-256) for passwords — always use **salted, slow key-derivation functions** (bcrypt, PBKDF2, Argon2) that resist GPU brute-forcing and rainbow tables.

### 1.8.3 Digital Signatures & Certificates (PKI)
- **Digital Signature Workflow:**
  1. Sender hashes the message: `Hash = H(M)`
  2. Sender encrypts the hash with their **Private Key**: `Signature = Encrypt(Hash, Sender_PrivKey)`
  3. Receiver decrypts the signature with Sender's **Public Key** to retrieve original hash: `Hash = Decrypt(Signature, Sender_PubKey)`
  4. Receiver re-computes `H(M)`. If both hashes match, **Integrity + Authenticity + Non-repudiation** are guaranteed.
- **Digital Certificates (X.509):** Digital document binding a public key to an entity's identity, digitally signed by a trusted **Certificate Authority (CA)**.
- **Revocation checks:** **CRL** (Certificate Revocation List - periodic download) vs **OCSP** (Online Certificate Status Protocol - real-time status query).

**Common Interview Questions:**
- Walk through how a digital signature provides non-repudiation and integrity.
- What happens if a Certificate Authority (CA) root certificate is compromised?

---

## 1.9 Common Cyber Attacks

| Attack | How It Works | Real-World Impact | Primary Defense |
|---|---|---|---|
| **Phishing** | Deceptive communications (email/SMS/voice) impersonating trusted entities | Credential theft, initial access payload delivery | Security awareness training, MFA (FIDO2), DMARC/SPF/DKIM |
| **SQL Injection (SQLi)** | Malicious SQL statements inserted into entry fields to alter database queries | Unauthorized data exfiltration, database tampering, auth bypass | **Parameterized queries / Prepared statements**, input validation |
| **Cross-Site Scripting (XSS)** | Malicious JavaScript injected into trusted web applications and executed in victims' browsers | Session cookie hijacking, defacement, redirection to malware | Context-aware output encoding, Content Security Policy (CSP), `HttpOnly` cookie flags |
| **CSRF (Cross-Site Request Forgery)** | Tricks an authenticated browser into sending unauthorized requests to a vulnerable application | Unauthorized actions performed under victim's active session (e.g., funds transfer, password change) | **Anti-CSRF tokens** (SameSite cookie attributes, custom request headers) |
| **DDoS (Distributed Denial of Service)** | Flooding targeted servers/networks with massive distributed traffic volumes | Service outages, availability failure | Anycast DNS, Cloud scrubbing services (Cloudflare, AWS Shield), rate limiting |
| **Man-in-the-Middle (MITM)** | Attacker secretly relays and alters communications between two parties who believe they are directly communicating | Eavesdropping, sensitive data interception, session hijacking | End-to-end encryption (TLS with certificate pinning), mutual TLS (mTLS), ARP spoofing mitigations |

**Common Interview Questions:**
- Difference between XSS and CSRF? *(XSS steals user session/executes arbitrary JS in victim's browser; CSRF abuses the browser's existing trust/credentials to execute an unintended action without stealing the token directly)*
- Stored vs Reflected vs DOM-based XSS? *(Stored: malicious script persisted in database; Reflected: script reflected off web server in error/search result; DOM: vulnerability exists entirely in client-side JavaScript execution)*

---

## 1.10 OWASP Top 10 (Web Application Security - Quick Reference)

| # | Vulnerability Category | One-Line Meaning | Key Mitigation |
|---|---|---|---|
| **A01** | **Broken Access Control** | Users can act outside intended permissions (view others' accounts, privesc) | Enforce authorization checks server-side on every request; deny by default |
| **A02** | **Cryptographic Failures** | Data exposed in transit or at rest due to weak or missing encryption | Use strong ciphers (AES-256, TLS 1.3), disable legacy protocols (SSL/TLS 1.0/1.1) |
| **A03** | **Injection** | Hostile data sent to an interpreter (SQL, OS command, LDAP) as part of a command | Parameterized queries, ORMs, input validation & safe APIs |
| **A04** | **Insecure Design** | Architectural flaws where security wasn't designed into the system upfront | Threat modeling, secure design patterns, architecture reviews |
| **A05** | **Security Misconfiguration** | Unpatched flaws, default credentials, open cloud storage, verbose error logs | Automated hardening templates, continuous configuration auditing |
| **A06** | **Vulnerable & Outdated Components** | Using third-party libraries/frameworks containing known CVEs | Software Composition Analysis (SCA), automated dependency patching |
| **A07** | **Identification & Authentication Failures** | Weak password policies, lack of brute-force protection, missing MFA | Enforce MFA, implement rate limiting, modern password policies |
| **A08** | **Software & Data Integrity Failures** | Code/infrastructure updates accepted without verifying digital signatures | Verify cryptographic signatures on updates, secure CI/CD pipelines |
| **A09** | **Security Logging & Monitoring Failures** | Insufficient logging prevents timely detection and incident investigation | Centralized SIEM forwarding, real-time alerting, audit trail tamper protection |
| **A10** | **Server-Side Request Forgery (SSRF)** | Web application coerced into making requests to an unintended resource (internal metadata services) | Restrict outbound requests via allowlists, disable unused URI schemes, IMDSv2 in AWS |

**Common Interview Questions:**
- Why is Broken Access Control currently ranked #1 on OWASP Top 10? *(It is the most prevalent flaw, cannot be fully automated by standard SAST/DAST, and directly leads to data leakage/privilege escalation)*

---

## 1.11 Applying Risk Concepts Practically

### 1.11.1 Risk Formula (Practical View)
```
Risk = Threat × Vulnerability × Impact (often expressed as Likelihood × Impact)
```
**Example:** An internet-facing server (asset) running an unpatched, known-exploitable service (vulnerability) that attackers are actively scanning for (threat) represents **high risk** because likelihood and impact are both high.

### 1.11.2 Qualitative Risk Matrix (commonly asked to sketch)
| Likelihood \ Impact | Low | Medium | High |
|---|---|---|---|
| Low | Low | Low | Medium |
| Medium | Low | Medium | High |
| High | Medium | High | Critical |

### 1.11.3 Attack Surface Reduction — Practical Examples
- Closing unused ports/services
- Disabling unnecessary user accounts
- Patching regularly
- Enforcing MFA
- Network segmentation
- Removing unused software/plugins

**Common Interview Questions:**
- How would you go about reducing an organization's attack surface?
- Give a real example of increasing vs decreasing attack surface.

### 1.11.4 Security Posture Assessment
**Meaning:** Evaluating an organization's overall readiness against threats — combines vulnerability management, patching cadence, monitoring maturity, and policy enforcement.
**How it's assessed in practice:** vulnerability scans, penetration tests, audits, red/blue team exercises, maturity frameworks (e.g., NIST CSF tiers).

---

## 1.12 Rapid-Fire Follow-Up Questions (Section 1)

- Why is Availability sometimes the hardest to guarantee?
- Can a system have Confidentiality without Integrity? Give an example.
- Is encryption alone sufficient for confidentiality? Why or why not?
- Why is "security through obscurity" not considered a valid control?
- Give an example where a compensating control was necessary because a technical fix wasn't immediately possible.
- Why is Zero Trust especially relevant with remote work and cloud adoption?
- What's the difference between a security policy violation and a security incident?
- If an asset has no vulnerabilities, can it still carry risk? *(Yes — new vulnerabilities are discovered over time, and risk also depends on exposure/threat landscape changes.)*
- Why is hybrid encryption the standard approach for TLS rather than pure asymmetric encryption?
- How does an anti-CSRF token prove a request was intentionally generated by a user?
- What is the difference between SQL injection and Cross-Site Scripting (XSS) in terms of where code executes?

---

## Quick-Revision Summary

- **CIA Triad:** Confidentiality, Integrity, Availability
- **AAA:** Authentication, Authorization, Accounting
- **Access order:** Identification → Authentication → Authorization
- **Key principles:** Least Privilege, Need-to-Know, Defense in Depth, Zero Trust
- **Controls:** Administrative / Technical / Physical, each can be Preventive / Detective / Corrective / Deterrent / Compensating
- **Cryptography:** Symmetric (AES - fast, single key) vs Asymmetric (RSA/ECC - key pair, key distribution); Hybrid encryption in TLS
- **Hashing vs Encryption:** Hashing is irreversible one-way (integrity); Encryption is reversible two-way with key (confidentiality)
- **Digital Signature:** Hash encrypted with Sender's Private Key → decrypted with Sender's Public Key (Integrity + Authenticity + Non-repudiation)
- **Common Attacks:** Phishing (credentials/lures), SQLi (DB tampering), XSS (client script injection), CSRF (forged client action), DDoS (availability), MITM (interception)
- **OWASP Top 10 highlights:** #1 Broken Access Control, #2 Cryptographic Failures, #3 Injection, #7 Auth Failures, #10 SSRF
- **Risk formula (concept):** Risk = Threat × Vulnerability × Impact
- **Policy hierarchy:** Policy → Standard → Procedure
- **Extended CIA:** + Non-repudiation (CIANA); DAD = inverse of CIA (Disclosure/Alteration/Destruction)
- **Auth factors:** Know / Have / Are / Location / Behavior
- **Attack surface reduction:** Patch, close ports, disable unused accounts, segment, enforce MFA
