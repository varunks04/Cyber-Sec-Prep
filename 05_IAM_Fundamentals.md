# Identity and Access Management (IAM) Fundamentals

## 1. Core IAM Concepts

### 1.1 What is IAM?
**Meaning:** The framework of policies and technologies ensuring the right individuals/systems have the right access to the right resources, at the right time, for the right reasons.

### 1.2 Identification vs Authentication vs Authorization (Recap)
| Term | Meaning |
|---|---|
| Identification | Claiming an identity (username) |
| Authentication | Proving the identity (password, MFA) |
| Authorization | Determining what the authenticated identity can access |

### 1.3 Identity Lifecycle
**Meaning:** The full process of managing a user's access from creation to removal.
```
Provisioning → Access Granting → Access Review/Recertification → Modification (role change) → Deprovisioning
```
**Interview Tip:** **Deprovisioning** (removing access when an employee leaves/changes roles) is a frequently tested area — delayed deprovisioning is one of the most common real-world audit findings and insider-risk sources.

**Common Interview Questions:**
- What is the identity lifecycle, and why is timely deprovisioning critical?
- What is access recertification/access review, and why is it done periodically?

---

## 2. Authentication Technologies

### 2.1 MFA / 2FA
**Meaning:** Requiring more than one authentication factor to verify identity.
**Common Interview Questions:**
- What's a phishing-resistant MFA method vs a weaker one? *(FIDO2/hardware keys are phishing-resistant; SMS OTP is weaker — vulnerable to SIM swapping)*

### 2.2 SSO (Single Sign-On)
**Meaning:** Allows a user to authenticate once and gain access to multiple independent systems/applications without re-authenticating.
**Security relevance:** SSO improves user experience and centralizes authentication controls, but a compromised SSO account becomes a **single point of failure** across all connected apps.

### 2.3 SAML vs OAuth vs OpenID Connect (OIDC)
| Protocol | Primary Purpose | Format |
|---|---|---|
| SAML | Authentication (enterprise SSO) | XML |
| OAuth 2.0 | Authorization (delegated access to resources) | JSON/Tokens |
| OIDC | Authentication (built on top of OAuth 2.0) | JSON Web Tokens (JWT) |

**Interview Tip:** The most commonly confused distinction — **OAuth is about authorization** ("can this app access my data?"), while **SAML/OIDC are about authentication** ("who is this user?"). OIDC essentially adds an identity layer on top of OAuth.

**Common Interview Questions:**
- Difference between SAML and OAuth?
- Difference between OAuth and OpenID Connect?
- Give a real-world example of OAuth (e.g., "Login with Google" granting a third-party app limited access).

### 2.4 Kerberos (Recap from AD context)
**Meaning:** Ticket-based authentication protocol used by default in Active Directory, relying on a trusted third party (Key Distribution Center).
**Common Interview Questions:**
- Why is Kerberos considered more secure than NTLM?

---

## 3. Authorization Models

| Model | Full Form | Meaning | Example |
|---|---|---|---|
| RBAC | Role-Based Access Control | Access granted based on assigned role | "Manager" role gets approval rights |
| ABAC | Attribute-Based Access Control | Access granted based on attributes (user, resource, environment) | Access allowed only during business hours from a corporate device |
| DAC | Discretionary Access Control | Resource owner decides who gets access | File owner shares a document with specific users |
| MAC | Mandatory Access Control | System-enforced access based on classification labels (not user discretion) | Military/government classified systems (Top Secret, Secret) |

**Interview Tip:** RBAC is simpler to manage but less flexible; ABAC is more flexible/granular but more complex to configure and audit.

**Common Interview Questions:**
- RBAC vs ABAC — which is easier to manage and why?
- What's the difference between DAC and MAC? *(DAC = owner decides; MAC = system/policy decides, owner has no discretion)*

---

## 4. Privileged Access Management (PAM)

### 4.1 Privileged Accounts
**Meaning:** Accounts with elevated access (admin, root, domain admin, service accounts) — high-value targets for attackers.

### 4.2 PAM Solutions
**Meaning:** Tools/practices to secure, monitor, and control privileged account usage.
**Key practices:**
- **Just-in-Time (JIT) access:** Granting elevated access only for a limited time window when needed, rather than standing privileged access.
- **Credential vaulting:** Storing privileged credentials in a secure vault with rotation, rather than static shared passwords.
- **Session monitoring/recording:** Logging what privileged users actually do during elevated sessions.

**Common Interview Questions:**
- What is Just-in-Time access and why does it reduce risk compared to standing privileges?

### 4.3 Service Accounts
**Meaning:** Non-human accounts used by applications/services to authenticate and interact with systems.
**Security relevance:** Often over-privileged, rarely rotated, and a common lateral-movement/persistence target (e.g., Kerberoasting targets service accounts specifically).

---

## 5. Password & Account Policies

| Concept | Meaning |
|---|---|
| Password policy | Rules on password complexity, length, expiration, reuse |
| Account lockout policy | Locks an account after N failed login attempts (mitigates brute force) |
| Password rotation | Periodic forced password change — now often discouraged by modern guidance (e.g., NIST) in favor of length + MFA + breach monitoring |

> **Interview Tip:** Modern best practice (NIST 800-63B) has shifted away from frequent forced password rotation and arbitrary complexity rules, favoring **longer passphrases + MFA + breach-based password checks**. Mentioning this nuance signals current knowledge.

**Common Interview Questions:**
- Why has forced periodic password rotation fallen out of favor in modern guidance?
- What does an account lockout policy protect against, and what's a downside (account lockout DoS)?

---

## 6. Common AD / IAM Attacks (Recap & Extension)

| Attack | Meaning |
|---|---|
| Password Spraying | Trying one common password across many accounts (avoids lockouts) |
| Kerberoasting | Requesting service tickets for accounts with SPNs, then cracking them offline to recover service account passwords |
| Pass-the-Hash | Using a stolen password hash directly to authenticate, without knowing the plaintext password |
| Pass-the-Ticket | Reusing a stolen Kerberos ticket to impersonate a user without credentials |
| Golden Ticket | Forging a Kerberos TGT using a compromised KRBTGT account hash — grants domain-wide persistent access |

**Common Interview Questions:**
- Difference between Pass-the-Hash and Pass-the-Ticket?
- Why does Kerberoasting specifically target service accounts?
- Why is a compromised KRBTGT account considered catastrophic?

---

## 7. Federation & Directory Services

### 7.1 Identity Federation
**Meaning:** Allows identities from one trusted domain/organization to access resources in another, without creating duplicate accounts (e.g., logging into a partner company's app using your own corporate identity).

### 7.2 Directory Services
| Term | Meaning |
|---|---|
| LDAP | Protocol for querying/managing directory information (used by AD) |
| Active Directory | Microsoft's directory service — central store of users, groups, computers, policies |
| Azure AD / Entra ID | Cloud-based identity and access management service (modern successor branding: Microsoft Entra ID) |

**Common Interview Questions:**
- What is identity federation and why is it useful for partner/vendor access?
- Difference between on-prem Active Directory and Azure AD/Entra ID?

---

## 8. IAM in Cloud Environments

| Concept | Meaning |
|---|---|
| Cloud IAM | Service-specific identity/permission management (AWS IAM, Azure IAM, GCP IAM) |
| IAM Policy | JSON/declarative document defining what actions are allowed/denied on which resources |
| Role-based cloud access | Assigning temporary roles instead of long-lived static credentials |
| Principle of Least Privilege (cloud) | Granting only the specific permissions needed, avoiding wildcard (`*`) policies |

**Common Interview Questions:**
- Why is a wildcard IAM policy (`"Action":"*","Resource":"*"`) considered dangerous?
- Why are temporary/role-based cloud credentials preferred over static long-lived access keys?

---

## 9. Access Reviews & Compliance

| Term | Meaning |
|---|---|
| Access Certification/Review | Periodic verification that users still need their current access |
| Segregation of Duties (SoD) | Ensuring no single person has conflicting permissions that could enable fraud (e.g., one person can't both create and approve a payment) |
| Orphaned Accounts | Accounts belonging to users who've left but weren't deprovisioned — a common audit/security finding |

**Common Interview Questions:**
- What is Segregation of Duties and why does it matter in IAM design?
- What risk do orphaned accounts pose, and how would you detect them?

---

## Rapid-Fire Follow-Up Questions

- Why is delayed offboarding/deprovisioning a top audit finding?
- What's the practical difference between authentication and authorization in an IAM access review?
- Why would an organization choose ABAC over RBAC despite the added complexity?
- How does JIT access limit the blast radius of a compromised admin account?
- Why are service accounts often the weakest link in IAM?
- What's the risk of a single compromised SSO identity provider account?
- Why is Segregation of Duties important even in a small/trusted team?
- How would you detect orphaned accounts in an environment?

---

## Quick-Revision Summary

- **IAM core:** Identification → Authentication → Authorization
- **Identity lifecycle:** Provision → Grant → Review → Modify → Deprovision
- **SAML/OIDC = Authentication; OAuth = Authorization**
- **Access models:** RBAC (role), ABAC (attribute), DAC (owner discretion), MAC (system-enforced labels)
- **PAM key practices:** JIT access, credential vaulting, session monitoring
- **AD/IAM attacks:** Password Spraying, Kerberoasting, Pass-the-Hash, Pass-the-Ticket, Golden Ticket
- **Modern password guidance:** Length + MFA > forced frequent rotation
- **Federation:** trust across domains without duplicate accounts; LDAP/AD/Entra ID = directory services
- **Cloud IAM:** avoid wildcard policies, prefer temporary/role-based credentials
- **Compliance:** Access reviews, Segregation of Duties, watch for orphaned accounts
