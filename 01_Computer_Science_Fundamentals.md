# 1. Computer Science & Programming Fundamentals

## 1. Data Structures

### 1.1 Array vs Linked List
| Aspect | Array | Linked List |
|---|---|---|
| Memory | Contiguous | Non-contiguous (nodes + pointers) |
| Access | O(1) random access | O(n) traversal needed |
| Insert/Delete | O(n) (shifting required) | O(1) if node reference known |
| Use Case | Fast lookups, fixed-ish size | Frequent insert/delete, unknown size |

### 1.2 Stack vs Queue
| Aspect | Stack | Queue |
|---|---|---|
| Order | LIFO (Last In First Out) | FIFO (First In First Out) |
| Example | Function call stack, undo operations | Task scheduling, print queue |

### 1.3 Hash Table / Hash Map
**Meaning:** Stores key-value pairs using a hash function to map keys to array indices for near O(1) average lookup.
**Security relevance:** Hash tables underpin how passwords are stored (hashed, not the structure itself, but the lookup concept matters for rainbow table discussions).

### 1.4 Trees & Graphs (Awareness Level)
- **Tree:** Hierarchical structure (e.g., file systems, DNS hierarchy, Active Directory OU structure).
- **Graph:** Nodes connected by edges (e.g., network topology diagrams, attack path/lateral movement graphs used in tools like BloodHound).

**Common Interview Questions:**
- How is a tree structure relevant to Active Directory (OUs, domains)?

---

## 2. Algorithms & Complexity

### 2.1 Big-O Notation
**Meaning:** Describes how an algorithm's time/space requirements grow as input size increases.

| Notation | Name | Example |
|---|---|---|
| O(1) | Constant | Hash table lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop through a list |
| O(n log n) | Linearithmic | Efficient sorting (merge sort) |
| O(n²) | Quadratic | Nested loops, bubble sort |

**Interview Tip:** You won't usually be asked to write complex algorithms in a security interview — but understanding *why* a linear log search doesn't scale (vs indexed/hashed lookup in a SIEM) shows practical awareness.

### 2.2 Searching & Sorting (Awareness)
- **Linear search:** O(n), checks every element.
- **Binary search:** O(log n), requires sorted data, repeatedly halves the search space.
- **Sorting (general awareness):** Bubble/Selection (O(n²), simple but slow), Merge/Quick Sort (O(n log n), efficient).


---

## 3. Databases & SQL Basics

### 3.1 Relational Database Concepts
| Term | Meaning |
|---|---|
| Table | Collection of rows/records representing an entity |
| Primary Key | Column(s) uniquely identifying each row in a table |
| Foreign Key | Column referencing a primary key in another table, enforcing referential integrity |
| Index | Data structure (usually B-Tree) speeding up query searches at the cost of write speed |

### 3.2 SQL Joins
| Join Type | Description | Result Set |
|---|---|---|
| **INNER JOIN** | Matches rows in both tables based on join condition | Only matching records from both tables |
| **LEFT JOIN** | Returns all records from left table + matched records from right | All left rows; NULLs for unmatched right rows |
| **RIGHT JOIN** | Returns all records from right table + matched records from left | All right rows; NULLs for unmatched left rows |
| **FULL OUTER JOIN** | Returns all records when there is a match in either left or right | All rows from both; NULLs where conditions don't match |

```sql
-- Example: Identifying users and their associated role permissions
SELECT u.username, r.role_name
FROM users u
LEFT JOIN roles r ON u.role_id = r.id;
```

### 3.3 Database Normalization (1NF to 3NF)
**Purpose:** Eliminate data redundancy, prevent update/delete anomalies, and enforce data integrity.

| Normal Form | Rule Requirement | Example Fix |
|---|---|---|
| **1NF (First)** | Atomic (indivisible) column values; unique rows (Primary Key); no repeating groups | Separate comma-delimited phone numbers into separate rows or a dedicated table |
| **2NF (Second)** | Must be in 1NF + **No Partial Dependency** (all non-key attributes must depend on the *whole* primary key, relevant for composite keys) | Move attributes dependent on only part of a composite key to a separate table |
| **3NF (Third)** | Must be in 2NF + **No Transitive Dependency** (no non-key attribute depends on another non-key attribute: $X \rightarrow Y \rightarrow Z$) | Move `ZipCode → City, State` to a dedicated postal codes table |

> **Interview Tip:** While OLTP (transactional) databases favor 3NF to avoid anomalies, OLAP / SIEM / data warehouses often **denormalize** data (introducing controlled redundancy) to avoid costly multi-table joins during high-volume log querying.

### 3.4 ACID Properties (Transactions)
- **A — Atomicity:** "All or nothing" — all operations in a transaction succeed, or the entire transaction rolls back.
- **C — Consistency:** Database transitions from one valid state to another, preserving all schema constraints and rules.
- **I — Isolation:** Concurrent transactions execute without interfering with one another (isolation levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable).
- **D — Durability:** Once committed, transaction results survive system crashes or power failures (persisted to non-volatile storage / write-ahead logs).

### 3.5 Database Indexes
- **Clustered Index:** Dictates the physical order of data rows on disk (only **one** clustered index per table, typically the Primary Key).
- **Non-Clustered Index:** Separate lookup structure containing sorted key values and pointers to physical data rows (can have multiple per table).

### 3.6 SQL Injection (Security Context)
```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password123';
```
**Security relevance:** Direct concatenation of user input allows attackers to manipulate query syntax (e.g., `' OR '1'='1`). Mitigated using **parameterized queries / prepared statements** and ORMs.

**Common Interview Questions:**
- Difference between 2NF and 3NF?
- What are the ACID properties and why are they critical for financial systems?
- Clustered vs Non-Clustered index — what is the fundamental difference?

---

## 4. Operating System Concepts (CS Theory Layer)

### 4.1 Processes vs Threads
| Aspect | Process | Thread |
|---|---|---|
| Definition | An independent executing program with its own dedicated memory space | Smallest unit of CPU execution inside a process ("lightweight process") |
| Memory | Separate, isolated virtual address space (Code, Data, Heap, Stack) | Shares process Code, Data, and Heap; has its own private Stack and Registers |
| Overhead | High context-switching and creation overhead | Low overhead; rapid switching and inter-thread communication |
| Crash Impact | If one process crashes, other processes remain unaffected | If one thread crashes (e.g., segfault), the entire parent process may terminate |
| Communication | Inter-Process Communication (IPC): pipes, sockets, shared memory, message queues | Direct memory access within shared heap (requires synchronization: mutexes/semaphores) |

### 4.2 CPU Scheduling
- **Preemptive:** OS can interrupt an active process to allocate CPU to a higher-priority task (e.g., Round Robin, Priority Scheduling with preemption).
- **Non-Preemptive:** Process holds the CPU until it voluntarily terminates or enters a wait state (e.g., FCFS).
- **Common Algorithms:**
  - **Round Robin (RR):** Each process gets a fixed time slice (**quantum**). Preemptive, fair, standard in time-sharing systems.
  - **Shortest Job First (SJF):** Optimal average waiting time; risks **starvation** for long processes.
  - **Priority Scheduling:** Process with highest priority runs first. Starvation mitigated by **aging** (gradually increasing waiting process priority).

### 4.3 Deadlocks
**Meaning:** A situation where two or more processes are permanently blocked because each is holding a resource the other needs.

**The 4 Necessary Coffman Conditions (all 4 must hold for deadlock to occur):**
1. **Mutual Exclusion:** Resources cannot be shared simultaneously (only one process uses a resource at a time).
2. **Hold and Wait:** A process holding at least one resource is waiting to acquire additional resources held by others.
3. **No Preemption:** Resources cannot be forcibly confiscated; they can only be released voluntarily by the holding process.
4. **Circular Wait:** A closed chain of processes exists where each process waits for a resource held by the next process in the chain ($P_0 \rightarrow P_1 \rightarrow P_2 \rightarrow P_0$).

**Handling Deadlocks:**
- **Deadlock Prevention:** Invalidate at least one of the 4 Coffman conditions (e.g., impose strict global resource ordering to eliminate circular wait).
- **Deadlock Avoidance:** Dynamically evaluate resource requests using algorithms like **Banker's Algorithm** (ensuring the system stays in a "safe state").
- **Deadlock Detection & Recovery:** Detect via Resource Allocation Graphs; recover by killing processes or preempting resources.

### 4.4 Memory Management, Virtual Memory & Paging
- **Virtual Memory:** Architectural technique allowing execution of processes that may not be completely in physical RAM, giving each process the illusion of a vast, contiguous address space.
- **Paging:**
  - Logical address space is divided into fixed-size chunks called **Pages**.
  - Physical RAM is divided into matching fixed-size chunks called **Frames**.
  - **Page Table:** Hardware/OS structure mapping logical pages to physical frames.
  - **TLB (Translation Lookaside Buffer):** Fast CPU cache storing recent virtual-to-physical address translations.
- **Key Concepts:**
  - **Page Fault:** Hardware interrupt raised when a program accesses a page that is mapped in virtual memory but not currently loaded in physical RAM (OS must fetch it from disk/swap).
  - **Thrashing:** System spending more time swapping pages in/out of secondary storage than executing actual instructions, collapsing performance.

### 4.5 Buffer Overflow (Low-Level Security Link)
Writing data beyond the allocated buffer boundaries overwrites adjacent stack frames (including the function return address), allowing attackers to redirect CPU execution to injected shellcode. Prevented by canary values (StackGuard), ASLR (Address Space Layout Randomization), and DEP/NX (Non-Executable stack).

**Common Interview Questions:**
- What are the four Coffman conditions for a deadlock, and how can breaking one prevent it?
- What is a page fault, and what causes system thrashing?
- Process vs Thread — why is thread context switching much faster than process context switching?

---

## 5. Software Development Basics (Security Context)

| Term | Meaning |
|---|---|
| SDLC | Software Development Lifecycle — Plan, Design, Develop, Test, Deploy, Maintain |
| Secure SDLC (SSDLC) | Integrating security practices (threat modeling, secure coding, security testing) into every SDLC phase |
| Version Control | System for tracking code changes (e.g., Git) |
| CI/CD | Continuous Integration/Continuous Deployment — automated build, test, and release pipeline |

**Common Interview Questions:**
- What is Secure SDLC and why is "shifting left" (adding security early) important?
- Why does committing secrets (API keys/passwords) to Git repositories pose a security risk?

---

## 6. Programming Fundamentals (Awareness for Security Roles)

- **Why scripting matters for a SOC analyst:** Automating log parsing, IOC extraction, and repetitive triage tasks (commonly Python or PowerShell).
- **Common languages in security:** Python (automation, tooling), Bash/PowerShell (system administration & scripting), SQL (querying SIEM/databases).
- **Regular Expressions (Regex):** Pattern matching used extensively in log analysis and SIEM detection rules.

**Common Interview Questions:**
- Why is scripting (especially Python) valuable for a SOC/security analyst role, even if it's not a developer position?
- What is regex, and how might it be used to extract IOCs (IPs, hashes, domains) from logs?

---

## 7. Compilers vs Interpreters

| Term | Meaning |
|---|---|
| Compiler | Translates entire source code into machine code before execution (e.g., C, C++) |
| Interpreter | Executes code line-by-line at runtime (e.g., Python, JavaScript) |

**Security relevance:** Compiled binaries (EXE/ELF) are what's typically analyzed in **malware reverse engineering** (disassembly, decompilation); interpreted scripts (Python/PowerShell/JS) are more human-readable but are heavily used in **fileless malware** and living-off-the-land attacks.

**Common Interview Questions:**
- Why is reverse engineering a compiled binary harder than reading a script?

---

## 8. Client-Server Architecture & APIs

| Term | Meaning |
|---|---|
| Client-Server Model | Clients request services/resources; servers provide them |
| API | Interface allowing applications to communicate (request/response) |
| REST API | Architectural style using HTTP methods (GET/POST/PUT/DELETE) and stateless requests |
| JSON | Lightweight data-interchange format commonly used in APIs |

**Security relevance:** API security is now a top concern (OWASP API Security Top 10) — covered in depth in the Web Security section, but foundational understanding of request/response flow is assumed in interviews.

**Common Interview Questions:**
- Why are APIs a growing attack surface for organizations?

---

## 9. Binary, Hex & Number Systems (Quick Reference)

| System | Base | Example |
|---|---|---|
| Binary | 2 | `1010` = 10 (decimal) |
| Decimal | 10 | Standard counting |
| Hexadecimal | 16 | `0xFF` = 255 (decimal) |

**Security relevance:** Hex is used constantly in security work — hashes, memory addresses, malware byte signatures, and packet captures are all typically represented in hex.

**Common Interview Questions:**
- Why is hexadecimal commonly used to represent hashes and memory data instead of binary?

---

## 10. Object-Oriented Programming (OOP) Concepts

### 10.1 Encapsulation
**Meaning:** Bundling data (attributes) and the methods (functions) that operate on that data into a single cohesive unit (a **class**), while restricting direct access to internal object components.
- **Access Specifiers:**
  - `private`: Accessible only within the declaring class.
  - `protected`: Accessible within the declaring class and its subclasses.
  - `public`: Accessible from any code in the program.
- **Security Relevance:** Encapsulation enforces data hiding, preventing arbitrary or malicious state corruption and ensuring state modifications go through validated getter and setter methods.

### 10.2 Abstraction
**Meaning:** Hiding complex implementation details and exposing only the essential features and clean interfaces to the caller.
- **Mechanisms:**
  - **Abstract Classes:** Classes that cannot be instantiated directly and can contain both abstract methods (no body) and concrete methods.
  - **Interfaces:** Pure contracts specifying *what* methods a class must implement, without providing implementation or instance state.

### 10.3 Inheritance
**Meaning:** Mechanism where a new class (derived/subclass) inherits state and behavior from an existing class (base/superclass), enabling code reuse and hierarchical classification.
- **Types:** Single, Multilevel, Hierarchical, Multiple (supported via interfaces in Java/C#, directly in C++/Python).

### 10.4 Polymorphism
**Meaning:** "Many forms" — allowing objects of different classes to be treated as objects of a common superclass, or functions with the same name to behave differently based on context.

| Type | When Resolved | Mechanism | Example |
|---|---|---|---|
| **Compile-Time (Static)** | During compilation | **Method Overloading** (same method name, different parameter types/count) | `calculateRisk(int score)` vs `calculateRisk(int score, float multiplier)` |
| **Runtime (Dynamic)** | During execution | **Method Overriding** (subclass provides custom implementation of base method via `vtable`) | `Firewall.filter()` overriding `NetworkDevice.filter()` |

**Common Interview Questions:**
- Difference between Abstraction and Encapsulation? *(Abstraction hides complexity by showing **what** an object does; Encapsulation hides internal data to protect **how** state is manipulated)*
- Method Overloading vs Method Overriding? *(Overloading is compile-time within the same class; Overriding is runtime across parent-child inheritance)*
- Abstract Class vs Interface — when would you use each?

---

## Rapid-Fire Follow-Up Questions (Core CS)

- Why does a hash table have O(1) average lookup but O(n) worst case?
- What's a real-world security tool that relies on tree structures (e.g., BloodHound for AD attack paths)?
- Why is input validation conceptually tied to both SQL Injection and buffer overflows?
- What does "shift-left" mean in the context of Secure SDLC?
- Why would an attacker prefer a fileless (interpreted/script-based) attack over a compiled malware binary?
- How does breaking the "Circular Wait" condition prevent deadlocks in operating systems?
- What is the difference between a page fault and system thrashing?
- How does encapsulation contribute to writing secure software?

---

## Quick-Revision Summary

- **Array:** fast lookup, slow insert/delete | **Linked List:** flexible insert/delete, slow lookup
- **Stack = LIFO, Queue = FIFO**
- **Hash table:** O(1) average lookup via hash function
- **Big-O:** O(1) < O(log n) < O(n) < O(n log n) < O(n²)
- **SQL Joins:** INNER (both), LEFT (all left + matched right), RIGHT (all right + matched left), FULL OUTER (all both)
- **Normalization:** 1NF (atomic values, PK), 2NF (1NF + no partial dependency), 3NF (2NF + no transitive dependency)
- **ACID Transactions:** Atomicity (all/none), Consistency (rules valid), Isolation (independent), Durability (persists)
- **OS Processes vs Threads:** Process = isolated memory; Thread = shared memory/heap inside process
- **CPU Scheduling:** Round Robin (quantum, fair), SJF (shortest first, starvation risk), Priority (aging fix)
- **Deadlock Coffman Conditions:** Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait (break 1 to prevent)
- **Virtual Memory & Paging:** Virtual pages map to physical frames via Page Table; Page Fault fetches from disk
- **Buffer overflow:** writing beyond allocated memory — stack corruption, mitigated by ASLR, DEP, and Stack Canaries
- **OOP Core Pillars:** Encapsulation (data hiding), Abstraction (interface simplicity), Inheritance (reuse), Polymorphism (overloading/overriding)
- **SSDLC:** security integrated across Plan→Design→Develop→Test→Deploy→Maintain
- **Compiled vs Interpreted:** binaries (reverse engineering) vs scripts (fileless malware/LOLBins)
- **Hex is everywhere:** hashes, memory addresses, byte signatures
