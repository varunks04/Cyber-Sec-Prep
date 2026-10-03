# 5. Cryptography & Public Key Infrastructure (PKI)

## 5.1 Cryptography Overview

Cryptography protects information by transforming readable plaintext into unintelligible ciphertext, and verifying authenticity and integrity.

```
Plaintext ───[ Encryption Algorithm + Key ]───► Ciphertext
Ciphertext ──[ Decryption Algorithm + Key ]───► Plaintext
```

### 5.1.1 Core Security Objectives of Cryptography
* **Confidentiality:** Only holders of the secret key can decrypt and read the ciphertext.
* **Integrity:** Ensuring the message has not been altered in transit.
* **Authentication:** Verifying the genuine identity of the sender or system.
* **Non-repudiation:** Preventing the creator of a message or signature from denying having generated it.

---

## 5.2 Symmetric vs Asymmetric Cryptography

| Dimension | Symmetric Cryptography (Secret-Key) | Asymmetric Cryptography (Public-Key) |
|---|---|---|
| **Key Architecture** | Single shared secret key for both encryption and decryption | Mathematically linked key pair: **Public Key** (distributed openly) + **Private Key** (kept secret) |
| **Computational Speed** | Extremely fast; hardware-accelerated (AES-NI instructions) | Computationally heavy (~1000× slower than symmetric ciphers) |
| **Key Distribution** | Difficult: Key must be transmitted over a secure out-of-band channel | Simple: Public key is published openly; private key never moves |
| **Key Scalability** | Requires $\frac{n(n-1)}{2}$ keys for $n$ participants to communicate pairwise | Requires $2n$ keys (one public/private pair per user) |
| **Primary Use Cases** | Bulk data encryption (hard drives, database fields, TLS session data) | Key exchange, digital signatures, identity verification |
| **Dominant Algorithms** | **AES** (128/256-bit), ChaCha20, 3DES (deprecated), DES (broken) | **RSA** (2048/4096-bit), **ECC** (ECDSA, Ed25519), Diffie-Hellman |

> **Interview Tip — Hybrid Encryption:** Real-world protocols (HTTPS, SSH, PGP) combine both technologies:
> 1. Use **Asymmetric Cryptography** (ECDHE/RSA) to securely negotiate a shared symmetric key without transmitting it over the wire.
> 2. Use **Symmetric Cryptography** (AES-GCM) with that negotiated session key to encrypt bulk traffic for maximum throughput.

```
Hybrid Encryption Architecture (Used in TLS / HTTPS / PGP):

Client                                                          Server
  │                                                               │
  │ 1. Request Server Public Key (Cert)                           │
  ├──────────────────────────────────────────────────────────────►│
  │◄──────────────────────────────────────────────────────────────┤ (Sends Public Key)
  │                                                               │
  │ 2. Generate random Symmetric Session Key (e.g., AES-256)      │
  │ 3. Encrypt Session Key with Server's Public Key               │
  │                                                               │
  │ 4. Transmit [ Encrypted Session Key ]                         │
  ├──────────────────────────────────────────────────────────────►│
  │                                                               │ 5. Decrypts Session Key
  │                                                               │    using Server Private Key
  │                                                               │
  │◄════════ 6. High-Speed Bulk Data Encrypted with AES-GCM ═════►│
  │          (Both sides use negotiated symmetric session key)    │
```

---

## 5.3 Symmetric Ciphers & Modes of Operation

### 5.3.1 Block Ciphers vs Stream Ciphers
* **Block Ciphers:** Divide plaintext into fixed-size blocks (e.g., AES uses 128-bit blocks) before encryption. If input isn't a multiple of block size, **padding** (PKCS#7) is appended.
* **Stream Ciphers:** Encrypt continuous data byte-by-byte or bit-by-bit by XORing plaintext with a pseudorandom keystream (e.g., ChaCha20). Ideal for low-latency streaming and resource-constrained devices.

### 5.3.2 Block Cipher Modes of Operation
How a block cipher encrypts data spanning multiple sequential blocks:

| Mode | Name | How It Operates | Security Status |
|---|---|---|---|
| **ECB** | Electronic Codebook | Each block encrypted independently with the identical key | **Insecure / Never Use:** Identical plaintext blocks produce identical ciphertext blocks (famous "ECB Penguin" visual pattern leak). |
| **CBC** | Cipher Block Chaining | Each plaintext block is XORed with previous ciphertext block before encryption; Block 1 uses an **Initialization Vector (IV)** | Legacy / Vulnerable to padding oracle attacks if integrity isn't verified separately. |
| **CTR** | Counter Mode | Turns a block cipher into a stream cipher by encrypting an incrementing counter + nonce | Secure if nonce is never reused; enables parallelized encryption/decryption. |
| **GCM** | Galois/Counter Mode | Combines Counter mode with Galois authentication tag (**AEAD**) | **Industry Standard:** Provides both confidentiality and built-in cryptographic integrity verification simultaneously. |

> **Critical Interview Concept — AEAD (Authenticated Encryption with Associated Data):**
> Older architectures encrypted data with CBC and verified integrity using an HMAC separately ("Encrypt-then-MAC"). If implemented poorly, timing discrepancies enabled padding oracle attacks. Modern ciphers use **AES-GCM**, which generates a tamper-evident authentication tag natively.

---

## 5.4 Asymmetric Cryptography & Key Exchange

### 5.4.1 RSA (Rivest–Shamir–Adleman)
* **Mathematical Basis:** The computational difficulty of **factoring large prime numbers**.
* **Key Lengths:** Minimum 2048-bit required today (3072-bit or 4096-bit recommended for high-security applications).
* **Usage:** Digital signatures and legacy key transport.

### 5.4.2 Elliptic Curve Cryptography (ECC)
* **Mathematical Basis:** The discrete logarithm problem over the points of an elliptic curve equation ($y^2 = x^3 + ax + b$).
* **Advantage Over RSA:** Equivalent cryptographic strength with dramatically smaller key sizes:
  * **256-bit ECC $\approx$ 3072-bit RSA**
  * Faster mathematical operations, lower CPU/memory consumption, and reduced network transmission overhead (essential for mobile devices and IoT).
* **Common Standards:** **ECDSA** (signatures), **ECDH** (key exchange), **Curve25519 / Ed25519** (modern high-speed cryptography).

### 5.4.3 Diffie-Hellman & Perfect Forward Secrecy (PFS)
* **Diffie-Hellman (DH):** A mathematical protocol allowing two parties to derive a shared symmetric secret over an untrusted, public communication channel without transmitting the secret itself.

```
Diffie-Hellman Key Exchange (Conceptual Flow):

Alice (Private secret: a)                               Bob (Private secret: b)
  │                                                               │
  ├────── Agreed Public Base Parameters: [ Prime p, Generator g ] ┤
  │                                                               │
  │ Computes Public Value: A = g^a mod p                          │ Computes Public Value: B = g^b mod p
  │                                                               │
  ├────── Sends Public Value A ──────────────────────────────────►│
  │◄───── Sends Public Value B ───────────────────────────────────┤
  │                                                               │
  │ Derives Shared Secret:                                        │ Derives Shared Secret:
  │   K = B^a mod p                                               │   K = A^b mod p
  │     = (g^b)^a mod p = g^(ab) mod p                            │     = (g^a)^b mod p = g^(ab) mod p
  ▼                                                               ▼
[ Both compute identical secret key K without ever sending 'a', 'b', or 'K' over the wire! ]
```

* **Static DH vs Ephemeral Diffie-Hellman (DHE / ECDHE):**
  * In Static DH/RSA, the server uses a persistent private key to encrypt the pre-master secret. If an attacker records encrypted network traffic today and steals the server's private key years later, they can retroactively decrypt all historical traffic.
  * In **Ephemeral Diffie-Hellman (ECDHE)**, a temporary, unique key pair is generated for each individual session and discarded immediately after session key derivation.
* **Perfect Forward Secrecy (PFS):** A cryptographic property ensuring that the compromise of long-term server private keys does not compromise past session keys or expose historical recordings.

---

## 5.5 Cryptographic Hashing & Message Authentication

### 5.5.1 Essential Properties of a Cryptographic Hash Function
A cryptographic hash function maps arbitrary-length input data into a fixed-length string of bytes (digest) and must strictly satisfy five mathematical requirements:
1. **Deterministic:** The identical input always produces the identical output hash.
2. **Fast Computation:** Quick to compute the hash for any given input.
3. **Pre-image Resistance (One-Way):** Given a hash $H$, it is computationally infeasible to find the original message $M$ such that $\text{Hash}(M) = H$.
4. **Second Pre-image Resistance (Weak Collision Resistance):** Given an input $M_1$, it is computationally infeasible to find a different input $M_2$ such that $\text{Hash}(M_1) = \text{Hash}(M_2)$.
5. **Collision Resistance (Strong Collision Resistance):** It is computationally infeasible to find *any* two arbitrary distinct inputs $M_1 \neq M_2$ that produce $\text{Hash}(M_1) = \text{Hash}(M_2)$.
6. **Avalanche Effect:** A change in a single bit of input flips approximately 50% of the output bits unpredictably.

### 5.5.2 Hash Algorithm Status
* **MD5 (128-bit) & SHA-1 (160-bit):** Cryptographically **broken** due to collision vulnerabilities. Never use for security or signatures (acceptable only for non-adversarial checksums).
* **SHA-2 (SHA-256, SHA-384, SHA-512):** Current industry standard across certificates, blockchain, and integrity checks.
* **SHA-3:** Keccak-based alternative architecture designed as a fallback if SHA-2 ever develops vulnerabilities.

### 5.5.3 Password Hashing vs Data Hashing
> **Interview Question — Why can't you use SHA-256 for storing passwords?**
> SHA-256 is designed to be **fast**. Attackers using modern GPU rigs can calculate billions of SHA-256 hashes per second, making brute-force and dictionary attacks trivial.

**Correct Password Storage Approach:**
* **Salt:** A cryptographically random string generated per-user and concatenated with the password before hashing. Defeats **Rainbow Tables** (precomputed hash lookups) and prevents identical passwords from producing identical hashes.
* **Pepper:** An organization-wide secret key added to passwords and stored outside the database (e.g., in a Hardware Security Module - HSM).
* **Slow, Memory-Hard Key Derivation Functions:** Algorithms purposefully engineered to require configurable CPU time and memory space to compute, crippling GPU/ASIC cracking:
  * **Argon2id:** Winner of the Password Hashing Competition; best-in-class defense against both GPU and side-channel attacks.
  * **bcrypt:** Widely deployed adaptive blowfish-based hashing function.
  * **PBKDF2:** NIST-approved key derivation function using iterative HMAC rounds.

### 5.5.4 HMAC (Hash-Based Message Authentication Code)
Combines a cryptographic hash function with a secret key: $\text{HMAC}(K, M)$.
* Verifies both **Integrity** (message unmodified) and **Authenticity** (only an entity holding the secret key could have produced the code).

---

## 5.6 Digital Signatures & Public Key Infrastructure (PKI)

### 5.6.1 The Digital Signature Workflow
A digital signature provides **Authentication, Integrity, and Non-repudiation**:

```
SENDER SIGNING:
  Message (M) ──────► [ Hash Algorithm ] ───► Hash Digest
                                                  │
                                                  ▼
                        Sender Private Key ──► [ Asymmetric Encrypt ] ──► Digital Signature

RECEIVER VERIFICATION:
  Digital Signature ──► [ Decrypt with Sender Public Key ] ──► Recovered Hash
                                                                    │
  Received Message ───► [ Compute Hash ] ──────────────────► Computed Hash
                                                                    │
                                                     [ Compare: Must Match! ]
```

### 5.6.2 Digital Certificates & The X.509 Standard
A digital certificate binds an entity’s identity (e.g., `google.com`) to their public key, verified and cryptographically signed by a trusted third party.
* **Key Fields in an X.509 Certificate:**
  * **Subject:** Identity of the certificate owner (domain name, organization).
  * **Issuer:** The Certificate Authority (CA) that issued and signed it.
  * **Public Key:** The subject's public key and algorithm (RSA/ECC).
  * **Validity Period:** "Not Before" and "Not After" timestamps.
  * **Subject Alternative Name (SAN):** Modern field listing all domains/subdomains covered.
  * **Signature Algorithm & CA Digital Signature:** Proof of authenticity.

### 5.6.3 PKI Hierarchy & Chain of Trust
1. **Root CA:** The top of the trust pyramid. Its certificate is **self-signed** and pre-installed in the OS/browser **Root Trust Store**. Root CA private keys are kept strictly offline in ultra-secure vaults.
2. **Intermediate CA:** Issued by the Root CA to handle day-to-day certificate signing. If an intermediate key is compromised, it can be revoked without invalidating the Root.
3. **Leaf / End-Entity Certificate:** The certificate installed on web servers, signed by the Intermediate CA.

```
PKI Hierarchical Chain of Trust & Verification Flow:

┌────────────────────────────────────────────────────────┐
│ Root CA Certificate (e.g., DigiCert Global Root CA)    │
│ • Self-signed with Root Private Key                    │ ◄── Pre-installed in Browser / OS
│ • Stored in Trusted Root Certification Authorities     │     Trust Store (Anchor of Trust)
└───────────────────────────┬────────────────────────────┘
                            │ Signs Intermediate Cert using Root Private Key
                            ▼
┌────────────────────────────────────────────────────────┐
│ Intermediate CA Certificate (e.g., DigiCert TLS CA G4) │
│ • Validated using Root CA's pre-trusted Public Key     │
│ • Issues & signs end-entity server certificates        │
└───────────────────────────┬────────────────────────────┘
                            │ Signs Leaf Cert using Intermediate Private Key
                            ▼
┌────────────────────────────────────────────────────────┐
│ Leaf Certificate (e.g., github.com)                    │
│ • Presented by Web Server to Browser in TLS Handshake  │
│ • Validated by Browser decrypting CA signature with    │
│   Intermediate CA's Public Key                         │
└────────────────────────────────────────────────────────┘

Trust Verification Path (Bottom ──► Up):
Browser receives Leaf ──► Validates Intermediate ──► Matches Root in Trust Store = SECURE!
```

### 5.6.4 Certificate Revocation: CRL vs OCSP
When a private key is leaked or an employee leaves, the certificate must be invalidated before its expiration date:
* **CRL (Certificate Revocation List):** A published list of revoked serial numbers downloaded periodically by clients.
  * *Downside:* Large file sizes; clients may accept a revoked certificate between update windows.
* **OCSP (Online Certificate Status Protocol):** Client queries the CA in real-time: *"Is certificate serial #12345 valid?"*
  * *Downside:* Adds latency to connection setup; privacy leak (CA learns which sites the user visits).
* **OCSP Stapling (Best Practice):** The web server periodically queries the CA, caches the timestamped OCSP response, and "staples" it directly to the TLS handshake sent to the client. Faster and protects user privacy.

---

## 5.7 Common Cryptographic Attacks

* **Birthday Attack:** Exploits the mathematics of the birthday paradox to find hash collisions in $2^{n/2}$ operations rather than $2^n$. This is why a 128-bit hash (MD5) only provides 64 bits of collision resistance.
* **Rainbow Table Attack:** Using massive precomputed lookup tables of hashes to reverse passwords. Prevented 100% by using unique **salts**.
* **Downgrade Attack:** Attacker intercepts connection negotiation and forces client and server to fall back to an older, insecure protocol version or cipher (e.g., forcing SSL 3.0 or DES). Prevented by TLS 1.3 and disabling legacy ciphers.
* **Known Plaintext / Chosen Plaintext Attack:** Cryptanalytic attacks where the adversary has access to plaintext and its corresponding ciphertext to deduce the underlying key.

---

## 5.8 Rapid-Fire Follow-Up Questions (Cryptography & PKI)

- *Why does ECC offer better security efficiency than RSA?* (ECC relies on the elliptic curve discrete logarithm problem, which scales exponentially harder with smaller key lengths; 256-bit ECC matches 3072-bit RSA security).
- *What is the difference between encryption, hashing, and encoding?* (Encoding formats data for transport and is keyless and reversible; hashing creates an irreversible fixed-size digest for integrity; encryption reversibly scrambles data using a secret key for confidentiality).
- *What is Perfect Forward Secrecy and which cipher suites provide it?* (PFS ensures future compromise of long-term server private keys cannot decrypt past sessions; provided by ephemeral Diffie-Hellman ciphers like ECDHE).
- *How does OCSP Stapling improve performance and privacy?* (The web server queries the CA directly and attaches the signed OCSP response to the TLS handshake, eliminating client-to-CA round-trip latency and hiding user browsing history from the CA).

---

## Quick-Revision Summary

- **Symmetric:** Fast, single shared key; **AES-256** standard; bulk data encryption.
- **Asymmetric:** Public + Private key pair; key exchange and signatures; **RSA** (large primes) vs **ECC** (elliptic curve, smaller keys).
- **Hybrid Encryption:** Asymmetric negotiates symmetric session key; symmetric encrypts payload.
- **AES-GCM:** Authenticated Encryption with Associated Data (AEAD) — provides confidentiality + integrity simultaneously.
- **PFS (Perfect Forward Secrecy):** Ephemeral keys (ECDHE) prevent retroactive decryption of historical traffic.
- **Hashing:** Irreversible one-way; SHA-256 standard; passwords require slow salted hashes (**Argon2id, bcrypt**).
- **Digital Signature:** Message hash encrypted with **Sender Private Key**; verified with **Sender Public Key**.
- **X.509 PKI:** Root CA $\rightarrow$ Intermediate CA $\rightarrow$ Leaf Certificate.
- **Revocation:** OCSP Stapling is modern standard over legacy CRL lists.
