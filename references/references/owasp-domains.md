# OWASP Security Domains — Full Rules

All 16 domains below must be checked in every full security audit.
For targeted reviews, check only the relevant domain(s).

---

## Domain 1 — A01: Broken Access Control

**Playbook IDs: AZ-001 → AZ-006**

- Access control ALWAYS on the server, never only on the client
- Deny by default — access must be **explicitly granted**
- In EVERY read, edit, and delete operation: verify the authenticated user owns or has permission over that specific resource
- Protect against **IDOR**: never allow access to another user's resource by simply changing an ID
- Protect against **vertical privilege escalation** (user → admin) and **horizontal** (user A → user B)
- **CORS**: restrictive — only authorized domains, never wildcard (`*`) in production with credentials
- Tokens/sessions: invalidate **on the server** at logout, not only on the client
- Protect against **SSRF**: validate and filter all user-provided URLs before any server-side request
- APIs: validate permissions at **each endpoint**, not only on frontend routes

**Audit checks:**
- [ ] Can I access another user's resource by changing the ID in the URL?
- [ ] Can a regular user access admin endpoints?
- [ ] Are tokens invalidated on the server at logout?
- [ ] Are CORS origins restricted to known domains?
- [ ] Are all server-side requests to external URLs validated?

---

## Domain 2 — A02: Security Misconfiguration

**Playbook IDs: SC-001 → SC-008**

- Remove unused functionality, pages, endpoints, and frameworks
- Never expose stack traces, detailed errors, table names, software versions, or debug info in production
- **Required HTTP security headers** on every response:
  - `Content-Security-Policy` (CSP) — restrictive
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY` (or `SAMEORIGIN` if needed)
  - `Strict-Transport-Security` (HSTS) — long `max-age`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` (restrict camera, microphone, geolocation, etc.)
- Disable unnecessary HTTP methods
- Never use default credentials or configurations in any environment
- Database: minimum permissions per service/connection
- If using RLS (Row Level Security): configure on **all tables** without exception
- Disable directory listing
- Development environments must not be publicly accessible

**Audit checks:**
- [ ] Are all 6 security headers present in every response?
- [ ] Are stack traces hidden in production error responses?
- [ ] Are there any endpoints using default credentials?
- [ ] Is RLS enabled on all database tables?
- [ ] Are debug endpoints (e.g., `/debug`, `/__status`) inaccessible in production?

---

## Domain 3 — A03: Software Supply Chain Failures

**Playbook IDs: DS-001 → DS-006**

- Use lockfiles and commit them to the repository (`package-lock.json`, `yarn.lock`, `poetry.lock`, etc.)
- Prefer dependencies with large user bases, active maintenance, and good reputation
- Never import obscure, abandoned, or unverified libraries
- Check dependencies for known vulnerabilities before using (`npm audit`, `pip-audit`, `snyk`, etc.)
- Do not run post-install scripts from packages without reviewing them
- When suggesting a dependency, inform if there are safer or native alternatives

**Audit checks:**
- [ ] Is a lockfile present and committed?
- [ ] Are there known vulnerabilities in dependencies? (`npm audit` / `pip-audit`)
- [ ] Are there any abandoned or low-reputation packages?

---

## Domain 4 — A04: Cryptographic Failures

**Playbook IDs: CR-001 → CR-005**

- **Passwords**: ALWAYS `Argon2id`, `bcrypt`, or `scrypt`. **NEVER** MD5, SHA-1, plain SHA-256, or any hash not designed for passwords
- **Data in transit**: HTTPS/TLS mandatory. Disable TLS 1.0 and 1.1
- **Sensitive data at rest**: encryption with strong algorithm (AES-256-GCM or equivalent)
- Never create custom cryptographic algorithms — use consolidated libraries
- Tokens, session IDs, verification codes: generate with **CSPRNG** (cryptographically secure random generator)
- Token and hash comparison: use **constant-time comparison** to avoid timing attacks
- Never log passwords, tokens, keys, card data, or sensitive personal data
- Encryption keys must be stored in secret managers, never in code

**Audit checks:**
- [ ] Are passwords hashed with Argon2id, bcrypt, or scrypt?
- [ ] Is there any MD5 or SHA-1 usage for password storage?
- [ ] Are sensitive tokens generated with a CSPRNG?
- [ ] Is constant-time comparison used for token validation?
- [ ] Is sensitive data at rest encrypted?

---

## Domain 5 — A05: Injection

**Playbook IDs: IV-001 → IV-007**

### SQL Injection
- ALWAYS use parameterized queries or prepared statements
- **NEVER** concatenate user input into SQL
- Check ORM usage — even ORMs can have raw query escape hatches

### XSS (Cross-Site Scripting)
- Sanitize ALL user input before rendering
- Use context-appropriate output encoding (HTML, JS, URL, CSS)
- Restrictive CSP as an additional layer
- **NEVER** use raw HTML mechanisms with user data: `innerHTML`, `dangerouslySetInnerHTML`, `v-html`, `{!! !!}`, etc.

### Command Injection
- Never execute OS commands with user input
- If unavoidable: use a strict allowlist

### NoSQL Injection
- Validate and type-check queries in NoSQL databases
- Never use user input directly in query operators (`$where`, `$regex`, etc.)

### Template Injection
- Never insert user input directly into server-side templates

**Audit checks:**
- [ ] Are all SQL queries parameterized? Any string concatenation with user input?
- [ ] Are there any uses of `innerHTML`, `dangerouslySetInnerHTML`, or `v-html` with user data?
- [ ] Is there any `exec()`, `eval()`, or `shell_exec()` using user input?
- [ ] Are NoSQL queries validated against injection operators?

---

## Domain 6 — A06: Insecure Design

**Playbook IDs: (covered by authorization + business logic checks)**

All business rules must be explicitly defined and enforced **in the backend**:

- "Only the owner can edit/delete their resource"
- "Only users who paid can access the content"
- "Only authenticated users can perform action X"
- "Balance/stock cannot go negative"
- "Coupon/code can only be used once per user"
- "Refund can only be requested within the deadline and only once"

Every business rule involving money, permissions, or content access **must have explicit server-side validation** — never rely only on the frontend hiding a button.

For each feature, think about abuse scenarios: *"What happens if a malicious user tries to exploit this?"*

**Audit checks:**
- [ ] Are all ownership rules enforced server-side?
- [ ] Can a user consume paid content without having paid?
- [ ] Can a coupon be used more than once?
- [ ] Can a refund be requested multiple times?
- [ ] Can balance or stock go negative through concurrent requests?

---

## Domain 7 — A07: Identification and Authentication Failures

**Playbook IDs: AU-001 → AU-008**

- Prefer mature, maintained authentication providers (Supabase Auth, Auth0, Clerk, Firebase Auth, NextAuth, Devise, etc.) over implementing from scratch
- If implementing custom authentication:
  - Password hash with Argon2id/bcrypt/scrypt
  - Minimum 8 characters
  - Check against leaked password lists when feasible
- **Rate limiting on login**: progressive lockout after failed attempts
- **Generic error messages**: NEVER differentiate "email not found" from "incorrect password" — always "invalid credentials"
- **Session/JWT tokens**:
  - Access tokens with short expiration (15–30 min)
  - Refresh tokens with rotation
  - Validate signature, expiration, audience, and issuer on **every request**
  - Invalidate on the server at logout
- Protect against session fixation and session hijacking
- **MFA** (multi-factor authentication) where feasible

**Audit checks:**
- [ ] Does the login error differentiate "email not found" from "wrong password"? (Should not)
- [ ] Is there rate limiting on the login endpoint?
- [ ] Are JWT tokens validated on every request (signature + expiration + audience)?
- [ ] Are tokens invalidated on the server at logout?
- [ ] Are access tokens short-lived (≤30 min)?

---

## Domain 8 — A08: Software and Data Integrity Failures

**Playbook IDs: (covered across supply chain + CI/CD checks)**

- Never deserialize data from untrusted sources without strict validation
- Verify integrity of dependencies and updates (checksums, signatures)
- Use Subresource Integrity (SRI) for scripts loaded from CDNs
- Protect CI/CD pipelines against unauthorized modifications
- Do not blindly trust webhooks — validate signature/origin

**Audit checks:**
- [ ] Are CDN scripts using SRI hashes?
- [ ] Are webhooks validated for signature/origin?
- [ ] Is deserialized data from external sources strictly validated?

---

## Domain 9 — A09: Security Logging and Alerting Failures

**Playbook IDs: LM-001 → LM-005**

**Log ALL relevant security events:**
- Login, logout, authentication failures
- Access denied
- Create/modify/delete sensitive resources
- Permission changes
- Financial operations

**Log format** — structured (JSON) with:
- `timestamp` — ISO 8601
- `ip` — client IP
- `userId` — authenticated user ID
- `action` — what was attempted
- `resource` — what was targeted
- `result` — success | failure | denied
- `userAgent` — client user agent

**NEVER log:**
- Passwords
- Tokens
- Card data
- Sensitive personal data (SSN, health data, etc.)

**Audit checks:**
- [ ] Are login failures logged with IP and user agent?
- [ ] Are financial operations logged?
- [ ] Are there any logs that include passwords or tokens?
- [ ] Are logs stored in a tamper-resistant location?

---

## Domain 10 — A10: Mishandling of Exceptional Conditions

**Playbook IDs: (covered across all domains)**

- Catch and handle ALL exceptions
- Never expose internal details (stack traces, queries, table names, file paths) in error responses
- Return generic, safe error messages to the client
- Ensure any unhandled exception results in access denial (fail closed)
- Test behavior with malformed input, unavailable services, timeouts, and unexpected payloads

**Audit checks:**
- [ ] Do error responses expose stack traces or internal paths?
- [ ] Are there unhandled promise rejections or uncaught exceptions?
- [ ] Do 500 errors reveal database or file system details?

---

## Domain 11 — File Upload Security

**Playbook IDs: FU-001 → FU-006**

- Validate MIME type **in the header AND in the file's magic bytes**
- Limit maximum file size
- Limit allowed types via **allowlist** (not blocklist)
- Rename with generated name (UUID), **never use the original filename**
- Store outside the webroot / in external storage (S3, GCS, etc.)
- **Never execute or interpret files from the user**
- Scan for malware if the use case requires it

**Audit checks:**
- [ ] Is the MIME type validated beyond the file extension?
- [ ] Are magic bytes checked for uploaded files?
- [ ] Is the original filename discarded and replaced with a UUID?
- [ ] Are uploads stored outside the webroot?
- [ ] Can an uploaded file be executed by the web server?

---

## Domain 12 — Session Security

**Playbook IDs: SS-001 → SS-006**

- Session tokens must be generated with CSPRNG
- Session tokens must be invalidated server-side on logout
- Use `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax`) cookie flags
- Regenerate session ID after privilege level change (e.g., login, sudo)
- Implement absolute and idle session timeouts
- Protect against session fixation (do not accept pre-set session IDs)

---

## Domain 13 — API Security

**Playbook IDs: AP-001 → AP-007**

- Every API endpoint must validate authentication and authorization independently
- Rate limiting per endpoint, per user, and globally
- Validate `Content-Type` on all POST/PUT/PATCH requests
- Paginate all list endpoints with a maximum `page_size` (never allow requesting 999999 records)
- Use versioned APIs (`/api/v1/`) and deprecate old versions with notice
- Document which endpoints are public vs. authenticated vs. admin-only

---

## Domain 14 — AI/LLM Security

**Playbook IDs: AI-001 → AI-005**

- Protect against **prompt injection**: never send raw user input directly to an LLM without delimiters, escaping, or injection filters
- Restrict LLM tool access to the minimum necessary (principle of least privilege for agents)
- Never enable `bypassPermissions`, `--approval-mode yolo`, or equivalent flags in production
- Sanitize and validate LLM output before rendering or executing it
- Log all LLM interactions that involve user data

---

## Domain 15 — User Enumeration Protection

**Playbook IDs: AU-007, AU-008**

- **Login**: "Invalid credentials" (never separate "email not found" vs "wrong password")
- **Registration**: "If this email is not registered, you will receive a link"
- **Password recovery**: "If this email is registered, we will send instructions"
- **Consistent response time** (timing-safe) on all these routes
- **Aggressive rate limiting** on user lookup routes

---

## Domain 16 — Data Protection and Privacy

**Playbook IDs: DA-001 → DA-005**

- Collect **only the data strictly necessary** for the functionality (minimization)
- Use data only for the purpose disclosed to the user
- Implement endpoints for the user to:
  - View their own data
  - Correct their data
  - Request deletion/anonymization
  - Revoke consent
  - Export their data (portability)
- Sensitive data (health, religion, sexual orientation, biometrics, financial, documents) has **reinforced protection**
- Clearly inform which data is collected and how it is used (Privacy Policy)
- Inform about cookie usage and obtain consent when applicable
- Do not share data with third parties without consent or legal basis
- In case of a security incident with data: notify affected users and relevant authorities
- Logs and backups also contain personal data — include them in retention and deletion policies
