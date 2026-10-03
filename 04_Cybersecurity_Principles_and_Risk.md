# 4. Cybersecurity Principles & Risk Management

## 4.1 Core Security Tenets

### 4.1.1 The CIA Triad
The foundational benchmark for evaluating information security across any system, architecture, or policy.

```
                  [ Confidentiality ]
                         /   \
                        /     \
                       /       \
         [ Integrity ] --------- [ Availability ]
```

* **Confidentiality:** Ensuring data and resources are accessible only to authorized entities and shielded from unauthorized observation or disclosure.
  * *Threats:* Eavesdropping, packet sniffing, unauthorized data exfiltration, shoulder surfing.
  * *Primary Controls:* Encryption at rest/in transit, Access Control Lists (ACLs), data classification.
* **Integrity:** Safeguarding the accuracy, completeness, and trustworthiness of data and systems by ensuring unauthorized or accidental modification is prevented and detectable.
  * *Threats:* Man-in-the-Middle (MITM) tampering, unauthorized database updates, unauthorized file alteration.
  * *Primary Controls:* Cryptographic hashing, digital signatures, version control, message authentication codes (MACs).
* **Availability:** Ensuring authorized users have timely, reliable access to data, critical services, and assets when needed.
  * *Threats:* Denial of Service (DoS/DDoS), ransomware locking access, hardware failures, environmental disasters.
  * *Primary Controls:* Hardware redundancy, load balancing, fault tolerance, offline backups, DDoS mitigation scrubbing.

> **Interview Tip:** When analyzing an incident in an interview, always map it to the CIA Triad:
> - *Ransomware* primarily strikes **Availability** (encryption of systems) and often **Confidentiality** (double extortion via data theft).
> - *SQL Injection* can compromise **Confidentiality** (leaking DB records), **Integrity** (altering tables), and **Availability** (`DROP TABLE`).

### 4.1.2 The DAD Triad (The Inversion of CIA)
When an attack occurs, it violates the CIA Triad through its direct opposing consequence:
* **Disclosure** $\leftrightarrow$ Violation of Confidentiality (unauthorized exposure)
* **Alteration** $\leftrightarrow$ Violation of Integrity (unauthorized modification)
* **Destruction / Denial** $\leftrightarrow$ Violation of Availability (disruption or deletion)

### 4.1.3 Extended CIA: The Parkerian Hexad
While CIA is standard, the Parkerian Hexad expands security into six distinct attributes:
1. **Confidentiality:** Authorized viewing.
2. **Integrity:** Correctness and lack of unauthorized alteration.
3. **Availability:** Timely accessibility.
4. **Authenticity:** Proven genuineness of an identity, message, or document.
5. **Utility:** Usefulness of data (e.g., encrypted data without a key retains confidentiality and integrity, but loses utility).
6. **Possession / Control:** Physical or logical ownership (e.g., a sealed stolen backup tape loses possession even if strong encryption maintains confidentiality).

---

## 4.2 AAA Framework & Access Fundamentals

### 4.2.1 Identification vs Authentication vs Authorization
A three-step sequential process that must occur in exact order:

```
[ Step 1: Identification ] ---> [ Step 2: Authentication ] ---> [ Step 3: Authorization ]
  "Who are you claiming to be?"   "Prove you are that person."    "What are you allowed to do?"
  (Username / Employee ID)       (Password / Passkey / MFA)      (Role permissions / ACLs)
```

| Phase | Meaning | Technical Example |
|---|---|---|
| **Identification** | Stating an identity to the system | Providing an account name or email |
| **Authentication** | Validating the authenticity of the claimed identity | Verifying a cryptographic signature, password hash, or TOTP code |
| **Authorization** | Granting explicit permissions based on the verified identity | Kernel checking read/write bits; IAM role granting S3 access |
| **Accounting / Auditing** | Tracking actions and resource consumption for accountability | Writing timestamped logs to SIEM: `User X modified File Y` |

### 4.2.2 Authentication Factors
Authentication mechanisms require proof across one or more independent factor categories:
* **Something you know (Knowledge):** Password, passphrase, PIN.
* **Something you have (Possession):** Hardware security key (FIDO2/WebAuthn), Authenticator App (TOTP), Smart card.
* **Something you are (Inherence):** Biometric traits (fingerprint, facial geometry, retina scan).
* **Somewhere you are (Location):** Geofencing, corporate subnet restriction.
* **Something you do (Behavior):** Keystroke dynamics, mouse movement cadence.

> **Common Misconception:** Entering a password and an email OTP is **Two-Factor Authentication (2FA)** only if they represent different categories (Knowledge + Possession). Entering two passwords is not 2FA; it is multi-step single-factor.

### 4.2.3 Non-repudiation
**Meaning:** Guaranteeing that a party to a communication or transaction cannot falsely deny the authenticity of their signature or having sent a message.
* **Implementation:** Combines **Integrity** (cryptographic hash) + **Authenticity** (asymmetric private key signature) + **Timestamping**.

---

## 4.3 Core Security Principles

### 4.3.1 Principle of Least Privilege (PoLP)
Users, processes, and service accounts must be granted only the minimum permissions necessary to perform their required tasks, and revoked immediately when no longer needed.
* *Example:* Web applications connect to databases using dedicated accounts with `SELECT`/`INSERT`/`UPDATE` permissions on specific tables, completely blocking `DROP TABLE` or schema alterations.

### 4.3.2 Need-to-Know
Even if a user holds a valid security clearance or administrative title, they are granted access only to the specific sensitive information required for their active mission or assignment.

### 4.3.3 Defense in Depth (Layered Defense)
Deploying multiple coordinated, heterogeneous security controls across physical, technical, and administrative layers so that the failure of any single control does not compromise the environment.

```
[ Outer Perimeter ]  --> Physical Barriers, Guard Desks, CCTV
[ Network Layer ]    --> Firewalls, DDoS Scrubbers, VPN with MFA
[ Host Layer ]       --> OS Hardening, EDR Agents, Patch Management
[ Application Layer]--> WAF, Input Sanitization, Session Tokens
[ Data Layer ]       --> Database Encryption, DLP, Least Privilege ACLs
```

### 4.3.4 Zero Trust Architecture (ZTA)
* **Core Axiom:** *"Never trust, always verify."*
* Eliminates the traditional "castle-and-moat" perimeter model where devices inside the internal corporate network were implicitly trusted.

```
NIST SP 800-207 Zero Trust Architecture (Control Plane vs Data Plane):

                      ┌──────────────────────────────────────┐
                      │        Policy Decision Point (PDP)   │
                      │  ┌───────────────┐ ┌───────────────┐ │
                      │  │ Policy Engine │ │ Policy Admin  │ │
                      │  │ (Evaluation)  │ │ (Token Issue) │ │
                      │  └───────▲───────┘ └───────┬───────┘ │
                      └──────────┼─────────────────┼─────────┘
        Continuous Context       │                 │ Dynamic Policy Update
  (Identity + Device Health      │                 │
   + GeoIP + Behavior Telemetry) │                 │
                                 │ Control Plane   │
  ═══════════════════════════════╪═════════════════╪════════════════════════════
                                 │ Data Plane      ▼
  [ Subject / User Device ] ─────┼─────────► [ Policy Enforcement Point ] ──► [ Enterprise Resource ]
                                             │ (PEP: ZTNA Gateway / Proxy)│    (App / Database)
                                             └────────────────────────────┘
```

* **Three Core Pillars (NIST SP 800-207):**
  1. **Continuous Verification:** Authenticate and authorize dynamically on every request based on all available data points (identity, device health, location, behavior).
  2. **Limit Blast Radius:** Micro-segment networks, enforce least privilege, and containerize workloads.
  3. **Assume Breach:** Design defenses under the expectation that an attacker is already present inside the environment.

---

## 4.4 Security Controls & Defense Classification

Security controls are categorized along two complementary axes: **by implementation mechanism** and **by operational timing/function**.

### 4.4.1 Classification by Mechanism
* **Administrative (Managerial / Procedural):** Policies, procedures, background checks, security awareness training, and disaster recovery planning.
* **Technical (Logical):** Hardware and software safeguards including firewalls, encryption algorithms, IDS/IPS, access control lists, and antivirus.
* **Physical:** Physical barriers preventing direct touch or entry: biometric door locks, mantraps, bollards, CCTV, perimeter fences, and environmental climate controls.

### 4.4.2 Classification by Function
| Control Type | Operational Purpose | Real-World Example |
|---|---|---|
| **Preventive** | Stop an attack or violation before it succeeds | Firewall rule blocking unauthorized ports, input validation |
| **Detective** | Identify and alert on malicious activity during or after occurrence | SIEM alert, Network IDS sensor, file integrity monitoring (FIM) |
| **Corrective** | Remediate damage and restore systems following an incident | Restoring clean files from an immutable backup, re-imaging host |
| **Deterrent** | Discourage potential attackers from attempting an attack | Warning login banners, visible security cameras, legal notices |
| **Compensating** | Alternative measure providing equivalent defense when a primary control is technically infeasible | Segmenting a legacy medical scanner that cannot be patched and monitoring all its traffic |

**Common Interview Questions:**
- *What is a compensating control and when would you recommend one?* (Used when patching or standard controls would break legacy or specialized equipment; e.g., placing vulnerable SCADA controllers behind an isolated jump box with strict network monitoring).
- *Difference between preventive and detective controls?* (Preventive stops the attack at the threshold; detective monitors, alerts, and records evidence without necessarily stopping the flow).

---

## 4.5 Risk Management & Assessment

### 4.5.1 The Risk Equation
```
Risk = Threat × Vulnerability × Impact
```
* **Asset:** Anything of tangible or intangible value requiring protection (customer PII, intellectual property, infrastructure).
* **Vulnerability:** A flaw, weakness, or misconfiguration in an asset that can be taken advantage of.
* **Threat:** Any natural or human agent capable of exploiting a vulnerability to cause harm.
* **Exploit:** The specific mechanism, tool, or code sequence used by a threat actor to leverage a vulnerability.
* **Risk:** The probable loss or impact resulting from a threat exploiting a vulnerability.

### 4.5.2 Quantitative Risk Analysis Formulas
Quantitative analysis assigns financial values to risk:

| Metric | Full Form | Formula | Definition |
|---|---|---|---|
| **AV** | Asset Value | Fixed valuation | Total financial replacement/business value of the asset |
| **EF** | Exposure Factor | Percentage (0.0 to 1.0) | Percentage of asset value lost if a specific threat event occurs |
| **SLE** | Single Loss Expectancy | $\text{SLE} = \text{AV} \times \text{EF}$ | Expected financial loss each time the incident happens |
| **ARO** | Annualized Rate of Occurrence | Estimated occurrences / year | How many times per calendar year the threat is projected to occur |
| **ALE** | Annualized Loss Expectancy | $\text{ALE} = \text{SLE} \times \text{ARO}$ | Expected financial loss per year for this specific threat |

* **Cost-Benefit Analysis for Controls:**
  $$\text{Value of Control} = (\text{ALE}_{\text{before}} - \text{ALE}_{\text{after}}) - \text{Annual Cost of Control}$$
  If the annual cost of the control exceeds the reduction in ALE, the security control is financially unjustified.

### 4.5.3 Qualitative Risk Matrix
For risks where monetary valuation is subjective, organizations use likelihood vs impact matrices:

| Likelihood \ Impact | Low Impact | Medium Impact | High Impact |
|---|---|---|---|
| **High Likelihood** | Medium Risk | High Risk | **Critical Risk** |
| **Medium Likelihood**| Low Risk | Medium Risk | High Risk |
| **Low Likelihood** | Low Risk | Low Risk | Medium Risk |

### 4.5.4 The Four Risk Treatment Strategies
1. **Mitigate (Reduce):** Implement technical or administrative safeguards to reduce likelihood or impact (e.g., deploying MFA).
2. **Transfer (Share):** Shift financial liability to a third party (e.g., purchasing cyber liability insurance, outsourcing to a managed service provider with SLAs).
3. **Accept:** Acknowledge and document the risk without adding controls because remediation costs exceed potential impact.
4. **Avoid:** Eliminate the risk entirely by ceasing the risky activity (e.g., declining to store credit card data directly on premises).

---

## 4.6 Threat Modeling Frameworks

Threat modeling identifies potential security threats early in the system design phase so defenses can be built-in upfront.

### 4.6.1 The STRIDE Model (Microsoft)
Mnemonic mapping each threat category to the corresponding violated security property:

| Threat Category | Violated Security Property | Description | Real-World Mitigation |
|---|---|---|---|
| **S — Spoofing** | Authenticity | Impersonating another user, service, or machine | Strong authentication, MFA, cryptographic certificates |
| **T — Tampering** | Integrity | Unauthorized modification of data in transit or storage | Digital signatures, hashing, write-once storage, HMAC |
| **R — Repudiation** | Non-repudiation | Denying having performed an action | Centralized write-protected audit logs, digital signatures |
| **I — Information Disclosure** | Confidentiality | Exposing sensitive data to unauthorized parties | Encryption (AES-256), strict authorization checks, DLP |
| **D — Denial of Service** | Availability | Crashing, overwhelming, or exhausting resources | Rate limiting, filtering, load balancers, elastic scaling |
| **E — Elevation of Privilege** | Authorization | Gaining unauthorized administrative control | Least privilege, input validation, RBAC, privilege boundaries |

### 4.6.2 DREAD Risk Rating
A qualitative calculation used to prioritize discovered vulnerabilities:
* **D**amage potential: How severe is the damage?
* **R**eproducibility: How easily can the exploit be executed repeatedly?
* **E**xploitability: How much effort/skill is required to launch the attack?
* **A**ffected users: What percentage of user base is impacted?
* **D**iscoverability: How easy is it for an attacker to find the vulnerability?

$$\text{DREAD Score} = \frac{D + R + E + A + D}{5}$$

---

## 4.7 Governance, Risk & Compliance (GRC)

### 4.7.1 Security Governance Hierarchy
1. **Policies:** High-level executive directives approved by management outlining mandatory organizational requirements (*"All corporate workstations must enforce multi-factor authentication"*).
2. **Standards:** Mandatory uniform rules and specifications supporting policies (*"Workstation passwords must be at least 14 characters"*).
3. **Baselines:** Minimum acceptable security configurations (*"CIS Benchmark Level 1 for Windows 11"*).
4. **Guidelines:** Recommended discretionary advice and best practices (*"Users are encouraged to use password managers"*).
5. **Procedures:** Step-by-step instructions on *how* to execute tasks (*"Procedure for enrolling a YubiKey hardware token"*).

### 4.7.2 Major Industry Security Frameworks & Regulations
* **NIST Cybersecurity Framework (CSF 2.0):**
  * Six Core Functions: **Govern $\rightarrow$ Identify $\rightarrow$ Protect $\rightarrow$ Detect $\rightarrow$ Respond $\rightarrow$ Recover**.
  * Flexible, risk-based benchmark used globally across both government and private industry.

```
NIST Cybersecurity Framework (CSF 2.0) Core Functions:

                    ┌─────────────────────────┐
                    │      1. GOVERN (GV)     │
                    │  Oversight & Strategy   │
                    └────────────┬────────────┘
                                 │
           ┌─────────────────────┴─────────────────────┐
           ▼                                           ▼
┌─────────────────────┐                     ┌─────────────────────┐
│   2. IDENTIFY (ID)  │                     │   3. PROTECT (PR)   │
│ Assets, Risks, Gaps │                     │ Safeguards & Access │
└──────────┬──────────┘                     └──────────┬──────────┘
           │                                           │
           └─────────────────────┬─────────────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │     4. DETECT (DE)      │
                    │ Continuous Monitoring   │
                    └────────────┬────────────┘
                                 │
           ┌─────────────────────┴─────────────────────┐
           ▼                                           ▼
┌─────────────────────┐                     ┌─────────────────────┐
│   5. RESPOND (RS)   │                     │   6. RECOVER (RC)   │
│ Containment & Triage│                     │ Restoration & Resil │
└─────────────────────┘                     └─────────────────────┘
```
* **ISO/IEC 27001:** International standard establishing requirements for an **Information Security Management System (ISMS)**; requires formalized risk assessments and continuous audits.
* **SOC 2 (System and Organization Controls):** Standard for SaaS and cloud vendors, auditing service organizations based on **5 Trust Services Criteria**: Security, Availability, Processing Integrity, Confidentiality, and Privacy.
* **Regulatory Compliance Mandates:**
  * **PCI-DSS:** Strict data protection standards for any entity storing, processing, or transmitting credit card information.
  * **HIPAA:** US regulation protecting Protected Health Information (PHI).
  * **GDPR:** European Union regulation enforcing strict data privacy, breach notification (72-hour window), and consumer rights (Right to be Forgotten).

---

## 4.8 Rapid-Fire Follow-Up Questions (Principles & GRC)

- *Why can a system have Confidentiality without Integrity?* (Encrypted data where ciphertext has been corrupted in transit is confidential, but integrity is destroyed).
- *What is the difference between risk acceptance and risk mitigation?* (Mitigation adds controls to reduce the risk; acceptance acknowledges the documented risk without adding controls because cost exceeds impact).
- *In STRIDE, what threat directly attacks Non-repudiation?* (Repudiation — an attacker or rogue employee performs an action and deletes logs to claim they never did it).
- *What is the primary difference between a Security Policy and a Security Procedure?* (Policy defines **what** management mandates; Procedure defines step-by-step **how** to execute it).

---

## Quick-Revision Summary

- **CIA Triad:** Confidentiality (secrecy), Integrity (trustworthiness/unmodified), Availability (accessible).
- **Inversion:** DAD = Disclosure, Alteration, Destruction.
- **AAA:** Identification $\rightarrow$ Authentication $\rightarrow$ Authorization $\rightarrow$ Accounting.
- **Zero Trust:** Continuous verification, assume breach, limit blast radius; no implicit perimeter trust.
- **Control Types:** Administrative / Technical / Physical; Preventive (stop), Detective (alert), Corrective (fix), Compensating (substitute).
- **Risk Math:** $\text{SLE} = \text{AV} \times \text{EF}$; $\text{ALE} = \text{SLE} \times \text{ARO}$.
- **STRIDE Threat Modeling:** Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege.
- **GRC Hierarchy:** Policy (mandatory directive) $\rightarrow$ Standard (mandatory baseline) $\rightarrow$ Guideline (discretionary) $\rightarrow$ Procedure (step-by-step).
- **NIST CSF 2.0:** Govern, Identify, Protect, Detect, Respond, Recover.
