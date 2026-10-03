# 3. Operating System Security

## 3.1 Windows vs Linux Fundamentals

| Aspect | Windows | Linux |
|---|---|---|
| Kernel type | Monolithic (NT kernel) | Monolithic (with loadable modules) |
| User accounts | SID-based, Administrator | UID-based, root (UID 0) |
| Permissions model | ACLs (granular) | rwx (owner/group/other) + optional ACLs |
| Config storage | Registry | Text config files (`/etc/`) |
| Package mgmt | MSI/EXE installers | apt, yum, dnf, pacman |
| Common logs | Event Viewer (.evtx) | `/var/log/` (syslog, auth.log) |
| Service mgmt | Services.msc, `sc` | systemd (`systemctl`), init.d |

**Common Interview Questions:**
- What's the fundamental difference between how Windows and Linux handle permissions?
- Why do enterprises still run both, and what does that mean for a SOC analyst?

---

## 3.2 Processes, Threads, and Services

| Term | Meaning |
|---|---|
| Process | A running instance of a program with its own memory space |
| Thread | Smallest unit of execution within a process; a process can have multiple threads |
| Service (Windows) / Daemon (Linux) | Background process, typically running without user interaction, often starting at boot |

**Example:** `svchost.exe` on Windows hosts multiple services; on Linux, `sshd` runs as a daemon listening for SSH connections.

```
Parent-Child Process Tree: Normal vs Malicious Execution (SOC Triage View):

Normal Business Workflow:
  [ explorer.exe ] (PID 1420)
         │
         ▼
  [ WINWORD.EXE ] (PID 3840) ──► (User edits .docx file normally)

Malicious Phishing / Macro Execution Tree (Immediate Alert):
  [ explorer.exe ] (PID 1420)
         │
         ▼
  [ WINWORD.EXE ] (PID 3840) ◄── Malicious macro executes on document open
         │
         ▼ (Spawns command prompt: ANOMALY)
  [ cmd.exe ] (PID 5120)
         │
         ▼ (Spawns PowerShell with bypass flags: CRITICAL ALERT)
  [ powershell.exe ] (PID 6240)
         │  -ExecutionPolicy Bypass -NoProfile -EncodedCommand SQBFAFgA...
         │
         ▼ (Spawns living-off-the-land system discovery tools)
  ├── [ whoami.exe ] (PID 7012)
  ├── [ net.exe user ] (PID 7016)
  └── [ curl.exe / certutil.exe ] (PID 7020) ──► Beaconing Outbound C2 (Port 443)
```

**Common Interview Questions:**
- Why would a SOC analyst care about a process tree/parent-child relationship? *(e.g., `winword.exe` spawning `powershell.exe` is highly suspicious — indicates macro-based execution)*

---

## 3.3 Users, Groups & Permissions

### 3.3.1 Linux File Permissions
**Meaning:** Read (r), Write (w), Execute (x) permissions for Owner, Group, and Others.
**Example:** `-rwxr-xr--` → Owner: rwx, Group: r-x, Others: r--
**Numeric (octal) form:** rwx = 4+2+1 = 7. e.g., `chmod 755 file` = rwxr-xr-x.

```
Linux File Permissions & Octal Weight Breakdown:

 Example: -rwxr-xr-- (755)
 ┌───┬─────────────┬─────────────┬─────────────┐
 │ - │  r   w   x  │  r   -   x  │  r   -   -  │
 └───┴─────────────┴─────────────┴─────────────┘
   │    │   │   │     │   │   │     │   │   │
   │    4 + 2 + 1     4 + 0 + 1     4 + 0 + 0
   │    ─────────     ─────────     ─────────
   │        7             5             4
   │      Owner         Group        Others
   ▼
 File Type:
  '-' = Regular File
  'd' = Directory
  'l' = Symbolic Link
```

**Common Interview Questions:**
- What does `chmod 777` mean, and why is it a security risk? *(Full read/write/execute for everyone — no restriction)*
- What is SUID/SGID and why is it a privilege escalation risk? *(A SUID binary runs with the file owner's privileges, not the executing user's — misconfigured SUID root binaries are a classic privesc vector)*

### 3.3.2 Windows Permissions & ACLs
**Meaning:** Windows uses Access Control Lists (ACLs) attached to objects, listing which users/groups have which permissions (Read, Write, Modify, Full Control).
**Interview Tip:** Unlike Linux's simple rwx model, Windows ACLs are more granular (per-user, per-permission-type, allow/deny entries) — this granularity is also why misconfigurations are common.

```
Windows Access Control Evaluation Architecture:

[ User / Subject ] ──► Holds [ Access Token ]
                               ├── User SID (S-1-5-21-...)
                               ├── Group SIDs (e.g., Domain Admins)
                               ├── Privileges (e.g., SeDebugPrivilege)
                               └── Integrity Level (Low / Medium / High / System)
                                        │
                                        ▼ Access Request (e.g., Write to file)
                    ┌───────────────────────────────────────────────┐
                    │ Security Reference Monitor (SRM)              │
                    └───────────────────────┬───────────────────────┘
                                            │ Evaluates against
                                            ▼
                    ┌───────────────────────────────────────────────┐
                    │ Target Object: Security Descriptor            │
                    ├───────────────────────────────────────────────┤
                    │ • Owner SID                                   │
                    │ • Mandatory Integrity Level (Label)           │
                    │ • Discretionary Access Control List (DACL):   │
                    │     ├── ACE 1: DENY  (User Bob, Write)        │
                    │     ├── ACE 2: ALLOW (Domain Users, Read)     │
                    │     └── ACE 3: ALLOW (Admins, Full Control)   │
                    │ • System Access Control List (SACL - Auditing)│
                    └───────────────────────────────────────────────┘
                    (DACL Rule: Explicit DENY always takes precedence over ALLOW)
```

### 3.3.3 Users & Groups
- **Linux:** `root` (UID 0) has full privileges; regular users use `sudo` for elevated actions.
- **Windows:** Built-in **Administrator** account; **Local Administrators group** grants full local control; **Domain Admins** control the whole AD domain.


---

## 3.4 Windows Registry
**Meaning:** A hierarchical database storing configuration settings for the OS and applications.
**Key hives:** `HKEY_LOCAL_MACHINE (HKLM)`, `HKEY_CURRENT_USER (HKCU)`, `HKEY_USERS`, `HKEY_CLASSES_ROOT`.
**Security relevance:** Malware commonly creates **Run keys** in the registry (`HKLM\...\Run` or `HKCU\...\Run`) for **persistence**, so registry analysis is core to malware investigation.

**Common Interview Questions:**
- Name a common registry persistence technique. *(Run/RunOnce keys, Winlogon Shell/Userinit keys, services)*

---

## 3.5 Logging

### 3.5.1 Windows Event Logs
| Log | Contains |
|---|---|
| Security | Logon/logoff, privilege use, object access, audit events |
| System | OS/driver/service events |
| Application | Application-level events |

**Key Event IDs (frequently asked):**
| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Logoff |
| 4648 | Logon using explicit credentials (runas) |
| 4672 | Special privileges assigned to new logon (admin logon) |
| 4720 | User account created |
| 4732 | Member added to a security-enabled local group |
| 1102 | Audit log cleared (huge red flag — attacker covering tracks) |

**Common Interview Questions:**
- What is Event ID 4625 and why would you monitor it? *(Repeated 4625s = brute force indicator)*
- Why is Event ID 1102 significant in an investigation? *(Indicates possible log tampering / anti-forensics)*

### 3.5.2 Linux Logs
| Log Path | Contains |
|---|---|
| `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/CentOS) | Authentication attempts, sudo usage |
| `/var/log/syslog` or `/var/log/messages` | General system activity |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/cron.log` | Scheduled task execution |

**Common Interview Questions:**
- Where would you check for failed SSH login attempts on Linux? (`auth.log`/`secure`, look for "Failed password")
- How would you check who executed `sudo` commands recently?

---

## 3.6 Authentication Mechanisms (OS-Level)
- **Windows:** NTLM (legacy, weaker) and **Kerberos** (modern, ticket-based, default in AD).
- **Linux:** PAM (Pluggable Authentication Modules) — a flexible framework allowing different authentication methods (password, key-based, MFA) to be plugged in.

**Common Interview Questions:**
- Why is Kerberos preferred over NTLM in Active Directory environments?
- What is PAM in Linux, and why is it useful for enforcing security policy?

---

## 3.7 Privilege Escalation
**Meaning:** Gaining higher-level access than originally granted.

| Type | Meaning |
|---|---|
| Vertical | Low-privilege user gains admin/root-level access |
| Horizontal | Gaining access to another user's data/account at the same privilege level |

**Common techniques (awareness level):**
- Exploiting misconfigured SUID binaries (Linux)
- Weak service permissions allowing binary replacement (Windows)
- Unpatched kernel exploits
- Credential dumping (e.g., Mimikatz) after initial low-priv access
- Scheduled tasks/cron jobs running as higher-privileged accounts

**Common Interview Questions:**
- Difference between vertical and horizontal privilege escalation?
- Name one Windows and one Linux privilege escalation technique.

---

## 3.8 Scheduled Tasks & Persistence
| OS | Mechanism |
|---|---|
| Windows | Scheduled Tasks (`schtasks`), Startup folder, Registry Run keys, Services |
| Linux | Cron jobs (`crontab -l`), `/etc/cron.d/`, systemd timers, `.bashrc`/`.profile` modifications |

**Interview Tip:** Persistence mechanisms are a major theme in both malware analysis and MITRE ATT&CK (Persistence tactic) — always mention this connection if asked.

---

## 3.9 File Systems, System Calls, Kernel vs User Space
| Term | Meaning |
|---|---|
| File System | Structure for storing/organizing data (NTFS - Windows, ext4 - Linux) |
| System Call | Interface for a program to request a service from the OS kernel (e.g., open a file, create a process) |
| Kernel space | Privileged mode where the OS core runs, full hardware access |
| User space | Restricted mode where applications run, cannot directly access hardware |

**Common Interview Questions:**
- Why does separating kernel space and user space matter for security? *(A vulnerability in user space is contained; a kernel-space compromise can take over the entire system)*

---

## 3.10 Important Linux Commands for Security Analysts

| Command | Use in Security Investigations |
|---|---|
| `ls` | List files/directories (check `-la` for hidden files & permissions) |
| `cd` / `pwd` | Navigate/confirm current location during investigation |
| `cat` | View file contents (e.g., quick log review) |
| `grep` | Search for patterns in logs (e.g., `grep "Failed password" auth.log`) |
| `find` | Locate files (e.g., recently modified files: `find / -mtime -1`) |
| `ps` | List running processes (check for suspicious process names) |
| `top` | Real-time resource usage (spot cryptomining/high CPU malware) |
| `netstat` | View network connections/listening ports (legacy, being replaced by `ss`) |
| `ss` | Modern replacement for `netstat`, view active sockets/connections |
| `ip` | View/configure network interfaces (`ip a`, `ip route`) |
| `curl` / `wget` | Test connectivity, download files (also used by attackers to pull payloads) |
| `chmod` | Change file permissions (audit misconfigurations) |
| `chown` | Change file ownership |
| `sudo` | Run commands with elevated privileges (audit `sudoers` misuse) |
| `whoami` | Confirm current user context |
| `id` | Show UID/GID and group memberships |
| `history` | Review previously executed shell commands (attacker command history) |
| `journalctl` | Query systemd logs |
| `systemctl` | Manage/inspect services (`systemctl list-units --type=service`) |
| `tail` | View end of a file, often with `-f` to follow logs live |

**Common Interview Questions:**
- How would you check for suspicious network connections on a Linux box? (`ss -tulnp` or `netstat -tulnp`)
- How would you search a log file for a specific IP address? (`grep "<IP>" /var/log/auth.log`)
- How do you check what commands a compromised user recently ran? (`history`, though attackers may clear it — check `.bash_history` timestamps and shell config)

---

## 3.12 Environment Variables
**Meaning:** Key-value pairs used by the OS/applications to configure behavior (e.g., `PATH`, `HOME`, `TEMP`).
**Security relevance:** Attackers abuse `PATH` manipulation to trick a system into running a malicious binary instead of the legitimate one (a form of privilege escalation/execution hijacking). Malware also often stores C2 config or staging paths in environment variables.

**Common Interview Questions:**
- What is the `PATH` variable and how can it be abused? *(Placing a malicious executable earlier in PATH with the same name as a trusted binary)*

---

## 3.13 LSASS, SAM & Credential Storage (Windows)
| Component | Purpose |
|---|---|
| SAM (Security Account Manager) | Local database storing hashed local user passwords |
| LSASS (Local Security Authority Subsystem Service) | Process that handles authentication and stores credentials/tickets in memory |
| NTDS.dit | Active Directory's database file storing domain account hashes (on Domain Controllers) |

**Security relevance:** **Credential dumping** tools (e.g., Mimikatz) target LSASS memory to extract plaintext passwords, hashes, or Kerberos tickets. This is why LSASS access is heavily monitored (Sysmon Event ID 10 - ProcessAccess) and often protected (Credential Guard).

**Common Interview Questions:**
- What is LSASS and why is it a high-value target for attackers?
- What is credential dumping, and how might a SOC detect an attempt to access LSASS?

---

## 3.14 UAC (User Account Control)
**Meaning:** Windows feature that prompts for consent/credentials before allowing actions requiring elevated (administrator) privileges.
**Security relevance:** UAC bypass techniques are a well-known post-exploitation technique to silently gain admin rights without triggering a visible prompt.

**Common Interview Questions:**
- What is a "UAC bypass" in the context of an attack chain?

---

## 3.15 Sysmon (System Monitor)
**Meaning:** A free Microsoft Sysinternals tool that logs detailed system activity (process creation, network connections, file changes, registry changes) far beyond default Windows Event Logs — a staple in SOC/EDR-style detection.

**Key Sysmon Event IDs:**
| Event ID | Meaning |
|---|---|
| 1 | Process creation (includes full command line & parent process) |
| 3 | Network connection |
| 7 | Image/DLL loaded |
| 10 | Process accessed (e.g., something reading LSASS memory) |
| 11 | File created |
| 13 | Registry value set |

**Common Interview Questions:**
- Why is Sysmon commonly deployed alongside default Windows logging?
- Which Sysmon Event ID would show you the full command line of a spawned process? (Event ID 1)

---

## 3.16 Sudoers & PAM in Depth (Linux)
- **`/etc/sudoers`:** Controls which users/groups can run which commands as root (edited safely via `visudo`). Misconfigured entries (e.g., allowing `NOPASSWD: ALL`) are a common privilege escalation and audit finding.
- **PAM stack:** Modular config files under `/etc/pam.d/` control authentication behavior (password complexity, account lockout, MFA integration) without changing application code.

**Common Interview Questions:**
- What risk does `NOPASSWD: ALL` in sudoers introduce?
- Why use `visudo` instead of editing `/etc/sudoers` directly? *(Syntax validation prevents locking yourself out)*

---

## 3.17 Windows Security Baseline Concepts
| Concept | Meaning |
|---|---|
| Group Policy Object (GPO) | Centrally enforced configuration/security settings across domain-joined machines |
| Windows Defender / Windows Security | Built-in antivirus/EPP |
| BitLocker | Full-disk encryption for Windows |
| AppLocker / WDAC | Application allow-listing to block unauthorized executables |

**Common Interview Questions:**
- How does a GPO help enforce security policy at scale?
- What's the value of application allow-listing (AppLocker) compared to traditional antivirus?

---

## 3.18 Rapid-Fire Follow-Up Questions (Section 3)

- Why would a SOC analyst be suspicious of `powershell.exe` spawned from `outlook.exe`?
- What's the danger of a world-writable file or directory?
- Why check both Registry Run keys and scheduled tasks when investigating persistence?
- What does it mean if `auth.log` shows many "Failed password" entries from different usernames in seconds? (Password spraying/brute force)
- Why is clearing of Linux `.bash_history` or Windows Event ID 1102 considered a red flag rather than routine cleanup?
- What's the risk of leaving default/legacy accounts enabled on an OS image?

---

## Quick-Revision Summary

- **Windows perms:** ACL-based, granular allow/deny; **Linux perms:** rwx for owner/group/other, octal notation
- **Persistence hotspots:** Registry Run keys, Scheduled Tasks/cron, Services, Startup folder
- **Key Windows Event IDs:** 4624 (success logon), 4625 (failed logon), 4672 (admin logon), 1102 (log cleared)
- **Linux auth logs:** `/var/log/auth.log` or `/var/log/secure`
- **Auth mechanisms:** Kerberos (modern AD) > NTLM (legacy); PAM (Linux pluggable auth)
- **Privesc types:** Vertical (higher privilege) vs Horizontal (same level, different user)
- **Kernel vs user space:** Kernel compromise = full system takeover; user space = contained
- **Go-to commands:** `ss -tulnp`, `ps aux`, `grep`, `find -mtime`, `journalctl`, `history`
- **Credential targets:** SAM (local hashes), LSASS (in-memory creds), NTDS.dit (domain hashes on DC)
- **UAC:** Prompts before privilege elevation; bypasses are a known post-exploitation technique
- **Sysmon Event 1** = process creation w/ command line; **Event 10** = process access (LSASS dumping indicator)
- **Sudoers:** Edit only via `visudo`; watch for risky `NOPASSWD: ALL` entries
- **Windows hardening tools:** GPO (policy), BitLocker (disk encryption), AppLocker/WDAC (allow-listing)
