# DevOps Fundamentals

## 1. Core DevOps Concepts

### 1.1 What is DevOps?
**Meaning:** A set of practices/culture combining Development and Operations to enable faster, more reliable software delivery through automation, collaboration, and continuous feedback.

### 1.2 CI/CD
| Term | Meaning |
|---|---|
| CI (Continuous Integration) | Developers frequently merge code changes into a shared repo; each merge triggers automated builds/tests |
| CD (Continuous Delivery) | Code is automatically prepared for release, deployment is a manual approval away |
| CD (Continuous Deployment) | Every passing change is automatically deployed to production without manual approval |

### 1.3 Git & Version Control
**Meaning:** Distributed version control system tracking source code history and enabling collaborative development.
- **Three-Tree Architecture:**
  - **Working Directory:** Local uncommitted file changes.
  - **Staging Area (Index):** Files marked with `git add` to be included in the next commit.
  - **Repository (Local/Remote):** Permanent snapshots committed via `git commit` and synchronized with `git push`/`git pull`.
- **Key Concepts:**
  - `git merge`: Combines histories of two branches with a merge commit.
  - `git rebase`: Replays commits from one branch on top of another, creating a clean linear history.
  - **Branch Protection Rules:** Requiring code reviews (pull requests), passing CI tests, and signed GPG commits before merging to `main`/`production`.
- **Security Relevance:** Committing hardcoded credentials or private keys to Git is one of the most frequent breach sources. Even if deleted in a later commit, secrets persist permanently in `.git` history unless scrubbed with tools like BFG Repo-Cleaner or `git filter-repo`.

### 1.4 Scripting & Automation (Bash & Python)
- **Bash Scripting:**
  - Ideal for quick OS-level triage, piping data between utilities, and automation scripts.
  - Core toolchain: `grep` (pattern match), `awk` (column manipulation), `sed` (stream editor/substitution), `xargs` (command execution).
- **Python for Security & DevOps:**
  - Ideal for complex automation, interacting with cloud APIs (e.g., `boto3`), parsing JSON/XML logs, and developing custom security tooling.
  - Common libraries: `requests` (HTTP API queries), `os`/`subprocess` (system command execution), `re` (regular expressions), `json` (log parsing).

**Common Interview Questions:**
- Why is a CI/CD pipeline a high-value target for attackers?
- What is the difference between `git merge` and `git rebase`?
- How would you handle a situation where a developer accidentally pushed an AWS secret access key to a GitHub repository?

---

## 2. DevSecOps

### 2.1 What is DevSecOps?
**Meaning:** Integrating security practices directly into the DevOps pipeline ("shifting left") rather than treating security as a final gate before release.

### 2.2 Shift-Left Security
**Meaning:** Finding and fixing security issues as early as possible in the development lifecycle (code/commit stage) rather than after deployment.

### 2.3 Pipeline Security Tooling
| Tool Type | Purpose |
|---|---|
| SAST (Static Application Security Testing) | Scans source code for vulnerabilities without executing it |
| DAST (Dynamic Application Security Testing) | Tests a running application for vulnerabilities (black-box style) |
| SCA (Software Composition Analysis) | Scans third-party/open-source dependencies for known vulnerabilities (CVEs) |
| Secret Scanning | Detects hardcoded credentials/API keys committed to code repositories |
| Container Scanning | Scans container images for vulnerabilities/misconfigurations before deployment |

**Common Interview Questions:**
- Difference between SAST and DAST?
- Why is SCA (dependency scanning) important given how much modern code relies on third-party libraries?
- What's the risk of committing a secret (API key) into a Git repo, even if later deleted? *(Git history retains it; many breaches trace back to exposed secrets in public/private repos)*

---

## 3. Containers & Orchestration

### 3.1 Containers (e.g., Docker)
**Meaning:** Lightweight, portable, isolated environments packaging an application with its dependencies, sharing the host OS kernel (unlike full VMs).

### 3.2 Containers vs Virtual Machines
| Aspect | Container | Virtual Machine |
|---|---|---|
| Isolation level | Process-level (shares host kernel) | Full OS-level (hypervisor) |
| Startup time | Seconds | Minutes |
| Resource overhead | Lightweight | Heavier |
| Security boundary | Weaker (shared kernel = larger attack surface if kernel compromised) | Stronger isolation |

### 3.3 Kubernetes (Orchestration)
**Meaning:** Platform for automating deployment, scaling, and management of containerized applications across clusters.

**Common security concerns:**
- Misconfigured RBAC in Kubernetes clusters
- Exposed Kubernetes dashboards/APIs without authentication
- Overly permissive container privileges (running as root inside containers)
- Unscanned/vulnerable container images pulled from public registries

**Common Interview Questions:**
- Why are containers generally considered to have a weaker isolation boundary than VMs?
- What's a common Kubernetes misconfiguration that leads to breaches?
- Why is running a container as root a security concern?

---

## 4. Infrastructure as Code (IaC)

### 4.1 What is IaC?
**Meaning:** Managing and provisioning infrastructure (servers, networks, storage) through machine-readable configuration files instead of manual setup.
**Example tools:** Terraform, AWS CloudFormation, Ansible.

### 4.2 IaC Security Concerns
- Hardcoded secrets/credentials in IaC templates
- Overly permissive IAM policies defined in code (e.g., `"Action": "*", "Resource": "*"`)
- Misconfigurations replicated at scale (a single bad template can misconfigure hundreds of resources instantly)
- Lack of IaC scanning (tools like `tfsec`, `checkov` scan for misconfigurations before deployment)

**Common Interview Questions:**
- Why can a single IaC misconfiguration be more dangerous than a single manual misconfiguration?
- What is IaC scanning, and why should it run before infrastructure is deployed, not after?

---

## 5. Cloud Computing & Deployment Concepts

### 5.1 Cloud Service Models (SPI Model)
| Model | Full Name | Provider Manages | Customer Manages | Examples |
|---|---|---|---|---|
| **IaaS** | Infrastructure as a Service | Physical data center, hardware, hypervisor/virtualization | OS, middleware, runtime, applications, data, IAM, security configs | AWS EC2, Azure VMs, Google Compute Engine |
| **PaaS** | Platform as a Service | Hardware, OS, runtime environment, capacity provisioning | Application code, application configurations, customer data | AWS Elastic Beanstalk, Google App Engine, Heroku |
| **SaaS** | Software as a Service | Entire stack (hardware, OS, application, patching, backups) | User access management, data classification, client configuration | Microsoft 365, Google Workspace, Salesforce |

### 5.2 Cloud Deployment Models
- **Public Cloud:** Multi-tenant infrastructure owned and operated by a third-party cloud provider (AWS, Azure, GCP).
- **Private Cloud:** Cloud infrastructure provisioned for exclusive use by a single organization (on-prem or hosted).
- **Hybrid Cloud:** Connected combination of private infrastructure and public cloud, allowing data/app portability.
- **Multi-Cloud:** Using services from multiple independent public cloud vendors to avoid vendor lock-in and increase resilience.

### 5.3 Shared Responsibility Model
> **Golden Rule:** The Cloud Service Provider (CSP) is responsible for **Security OF the Cloud** (physical facilities, hardware, hypervisors, global infrastructure). The Customer is responsible for **Security IN the Cloud** (customer data, IAM, firewall rules, OS patching for IaaS, and encryption).

| Security Responsibility Layer | IaaS | PaaS | SaaS |
|---|---|---|---|
| Data & Access (IAM) | Customer | Customer | Customer |
| Application & APIs | Customer | Customer | **CSP** |
| OS & Network Configuration | Customer | **CSP** | **CSP** |
| Physical & Virtual Infrastructure | **CSP** | **CSP** | **CSP** |

### 5.4 Deployment Strategies
| Term | Meaning | Security / Operational Benefit |
|---|---|---|
| **Blue-Green Deployment** | Running two identical production environments; traffic switches from old (blue) to new (green) | Near-zero downtime; instant rollback capability if green exhibits flaws |
| **Canary Deployment** | Rolling out an update to a tiny percentage (e.g., 5%) of users before widespread release | Limits blast radius of bugs or vulnerabilities before full production rollout |
| **Rollback** | Instantly reverting to the previous known-good deployment state | Critical incident mitigation when a new release introduces a severe vulnerability |
| **Immutable Infrastructure** | Servers/containers are never modified in-place; changes require destroying and replacing them | Prevents configuration drift and terminates persistent attacker footholds upon redeploy |

**Common Interview Questions:**
- Explain the Shared Responsibility Model — who is responsible for OS security patches in AWS EC2 vs AWS Lambda? *(In EC2 [IaaS], the customer must patch; in Lambda [PaaS/Serverless], AWS patches the underlying runtime)*
- Why is Immutable Infrastructure considered a powerful security control?
- Difference between Blue-Green and Canary deployments?

---

## 6. Monitoring & Logging in DevOps

| Term | Meaning |
|---|---|
| Observability | Ability to understand a system's internal state from its external outputs (logs, metrics, traces) |
| Centralized Logging | Aggregating logs from all services/containers into one platform (feeds into SIEM) |
| Alerting | Automated notifications triggered by defined thresholds/anomalies |

**Common Interview Questions:**
- Why is centralized logging important both for operations and for security investigations?

---

## 7. Configuration Management & Secrets Handling

| Term | Meaning |
|---|---|
| Configuration Management | Automating consistent setup/configuration across servers (e.g., Ansible, Puppet, Chef) |
| Secrets Management | Securely storing/retrieving sensitive credentials (API keys, DB passwords) rather than hardcoding them |
| Vault (e.g., HashiCorp Vault) | Centralized tool for storing, rotating, and controlling access to secrets |
| Environment-based config | Separating config (including secrets) from code via environment variables/config files not committed to source control |

**Common Interview Questions:**
- Why is hardcoding secrets in source code a major security risk even in a private repo?
- What is a secrets manager/vault, and what problem does it solve compared to `.env` files?

---

## 8. Cloud-Native Security Basics

| Term | Meaning |
|---|---|
| Cloud-Native | Applications designed specifically to run in cloud environments, often using containers/microservices |
| Microservices | Breaking an application into small, independently deployable services |
| Service Mesh | Infrastructure layer (e.g., Istio) managing service-to-service communication, often adding encryption/authentication between microservices |
| Zero Trust in DevOps | Applying Zero Trust principles (verify every request) between microservices, not just at the network perimeter |

**Security relevance:** Microservices increase the number of network communication paths (attack surface) compared to a monolithic app, making service-to-service authentication/encryption important.

**Common Interview Questions:**
- Why do microservices architectures increase the internal attack surface compared to a monolith?
- What role does a service mesh play in securing service-to-service communication?

---

## 9. Artifact & Dependency Security

| Term | Meaning |
|---|---|
| Artifact Repository | Stores build outputs/packages (e.g., Nexus, Artifactory, container registries) |
| Dependency Confusion | Attack where a malicious package with the same name as an internal private package is published publicly, tricking build systems into pulling the malicious one |
| SBOM (Software Bill of Materials) | A complete inventory of all components/dependencies in a software product, used for vulnerability tracking |

**Common Interview Questions:**
- What is dependency confusion and how does it exploit public/private package naming?
- What is an SBOM and why has it become increasingly required (e.g., in supply-chain security regulations)?

---

## Rapid-Fire Follow-Up Questions

- Why do DevSecOps practices emphasize "shifting left" instead of testing security only before release?
- What's the practical difference between SAST, DAST, and SCA?
- Why is a compromised CI/CD pipeline considered a supply-chain risk?
- What security risks come from using public, unscanned container base images?
- Why can IaC misconfigurations be more dangerous at scale than traditional manual misconfigurations?
- How does immutable infrastructure limit an attacker's ability to persist after compromise?
- Why is a secrets vault preferred over environment files for sensitive credentials?
- What is dependency confusion, and how can organizations defend against it? (private registry scoping, namespace reservation)
- In the Cloud Shared Responsibility model, if an AWS EC2 instance running customer web services is compromised via an unpatched Apache vulnerability, whose responsibility was it? *(Customer's — customer manages OS and application patches in IaaS)*
- What is the security hazard of deleting a sensitive API key in a new commit without purging git history?

---

## Quick-Revision Summary

- **CI** = integrate & test automatically; **CD** = deliver (manual approval) or deploy (fully automatic)
- **Git workflow:** Working Directory → `git add` (Staging) → `git commit` (Local Repo) → `git push` (Remote); Merge (history combine) vs Rebase (linear replay)
- **Scripting:** Bash (pipes, sed/awk/grep, OS triage) & Python (requests, boto3, json, security automation)
- **DevSecOps = security shifted left into the pipeline**, not just a final gate
- **SAST** (code, static) vs **DAST** (running app, dynamic) vs **SCA** (dependencies/CVEs)
- **Containers:** lightweight, shared kernel, weaker isolation than VMs
- **Kubernetes risks:** misconfigured RBAC, exposed APIs, root containers, unscanned images
- **IaC:** fast, scalable — but misconfigurations replicate everywhere; scan before deploy (tfsec/checkov)
- **Cloud models:** IaaS (hardware/hypervisor by CSP; OS/app by customer), PaaS (app by customer), SaaS (all by CSP)
- **Shared Responsibility:** CSP secures **OF** the cloud; Customer secures **IN** the cloud
- **Deployment strategies:** Blue-Green (full switch), Canary (gradual rollout), Immutable Infrastructure (replaces in-place mutation)
- **Secrets:** never hardcode — use a vault/secrets manager; scan git repositories
- **Microservices:** larger internal attack surface — service mesh helps secure service-to-service traffic
- **Supply chain:** watch for dependency confusion; SBOM = full component inventory for vulnerability tracking
