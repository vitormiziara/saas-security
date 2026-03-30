# Security Checklists

Ready-to-use checklists for audits, PRs, deploy, and testing.
Copy and adapt to the project's context.

---

## Checklist 1: Full Security Audit (16 Domains)

Use this for complete codebase reviews.

### 🔴 CRITICAL — Check First

**Access Control (AZ)**
- [ ] AZ-001: Resources protected against IDOR (user cannot access another user's resource by changing ID)
- [ ] AZ-002: No path traversal in file/directory endpoints
- [ ] AZ-003: Vertical privilege escalation blocked (user cannot reach admin routes)
- [ ] AZ-004: Horizontal privilege escalation blocked (user A cannot act as user B)
- [ ] AZ-005: CORS restricted to known domains (no wildcard `*` in production with credentials)
- [ ] AZ-006: Tokens invalidated server-side at logout

**Authentication (AU)**
- [ ] AU-001: Passwords hashed with Argon2id, bcrypt, or scrypt
- [ ] AU-002: Login does not differentiate "email not found" vs "wrong password"
- [ ] AU-003: Rate limiting active on login endpoint
- [ ] AU-004: JWT validated on every request (signature + expiration + audience)
- [ ] AU-005: Access tokens expire in ≤30 minutes
- [ ] AU-006: Refresh token rotation implemented
- [ ] AU-007: Registration flow does not reveal if email is already in use
- [ ] AU-008: Password recovery does not reveal if email is registered

**Input Validation (IV)**
- [ ] IV-001: No SQL injection — all queries parameterized
- [ ] IV-002: No XSS — user input sanitized before rendering
- [ ] IV-003: No `innerHTML` / `dangerouslySetInnerHTML` with user data
- [ ] IV-004: No command injection — no `exec()`/`eval()` with user input
- [ ] IV-005: All text fields have maximum length limits
- [ ] IV-006: Request body size limited
- [ ] IV-007: Pagination has maximum `page_size`

**Secrets & Configuration (SC)**
- [ ] SC-001: No API keys, tokens, or passwords in source code
- [ ] SC-002: No secrets in comments or commit history
- [ ] SC-003: `.env` in `.gitignore`
- [ ] SC-004: `.env.example` with fictitious values documented
- [ ] SC-005: No default credentials in any environment
- [ ] SC-006: No stack traces in production error responses
- [ ] SC-007: All 6 required security headers present
- [ ] SC-008: Debug endpoints disabled in production

### 🟠 HIGH — Check After Critical

**Data Security (DA)**
- [ ] DA-001: Passwords never stored in plain text
- [ ] DA-002: Sensitive data at rest encrypted
- [ ] DA-003: HTTPS/TLS enforced (TLS 1.2+ only)
- [ ] DA-004: Sensitive data never logged
- [ ] DA-005: User data endpoints (view, correct, delete, export) implemented

**File Upload (FU)**
- [ ] FU-001: MIME type validated in header AND magic bytes
- [ ] FU-002: Maximum file size enforced
- [ ] FU-003: Allowed file types defined by allowlist
- [ ] FU-004: Original filename discarded, replaced with UUID
- [ ] FU-005: Files stored outside webroot / in external storage
- [ ] FU-006: Uploaded files cannot be executed by web server

**Session Security (SS)**
- [ ] SS-001: Session tokens generated with CSPRNG
- [ ] SS-002: `HttpOnly`, `Secure`, `SameSite` cookie flags set
- [ ] SS-003: Session ID regenerated after login
- [ ] SS-004: Absolute and idle session timeouts implemented
- [ ] SS-005: Session fixation protection (pre-set session IDs rejected)

**Dependencies (DS)**
- [ ] DS-001: Lockfile committed to repository
- [ ] DS-002: `npm audit` / `pip-audit` shows no HIGH or CRITICAL vulnerabilities
- [ ] DS-003: No abandoned or unverified packages

**API Security (AP)**
- [ ] AP-001: Every endpoint validates auth independently
- [ ] AP-002: Rate limiting per endpoint and per user
- [ ] AP-003: `Content-Type` validated on all write requests
- [ ] AP-004: All list endpoints paginated with max page size

**Race Conditions**
- [ ] RC-001: Financial operations use atomic transactions or SELECT FOR UPDATE
- [ ] RC-002: Single-use coupons protected with UNIQUE CONSTRAINT
- [ ] RC-003: Idempotency keys implemented for payment operations
- [ ] RC-004: Counters/toggles use atomic database operations

**Cryptography (CR)**
- [ ] CR-001: No MD5 or SHA-1 for password hashing
- [ ] CR-002: No custom cryptographic algorithms
- [ ] CR-003: Tokens generated with CSPRNG
- [ ] CR-004: Constant-time comparison for token validation
- [ ] CR-005: Encryption keys stored in secret manager

### 🟡 MEDIUM — Check Last

**Security Headers (SH)**
- [ ] SH-001: `Content-Security-Policy` configured and restrictive
- [ ] SH-002: `X-Content-Type-Options: nosniff` present
- [ ] SH-003: `X-Frame-Options: DENY` present
- [ ] SH-004: `Strict-Transport-Security` with long `max-age`
- [ ] SH-005: `Permissions-Policy` restricts sensitive APIs

**Logging & Monitoring (LM)**
- [ ] LM-001: Login failures logged with IP and timestamp
- [ ] LM-002: Financial operations logged
- [ ] LM-003: Access denied events logged
- [ ] LM-004: No passwords or tokens in logs
- [ ] LM-005: Logs stored in tamper-resistant location

**DevSecOps (DO)**
- [ ] DO-001: HTTPS enforced in production
- [ ] DO-002: Secrets managed via environment variables or secret manager
- [ ] DO-003: Dev/staging/prod environments use separate secrets
- [ ] DO-004: Containers do not run as root
- [ ] DO-005: Automatic backups configured and tested
- [ ] DO-006: Dependencies kept up to date

**Frontend Security (FE)**
- [ ] FE-001: SRI hashes on CDN scripts
- [ ] FE-002: Webhooks validated by signature/origin
- [ ] FE-003: No sensitive data in localStorage or URL parameters
- [ ] FE-004: User-provided URLs validated (protocol + domain)

**AI/LLM Security (AI)** *(if applicable)*
- [ ] AI-001: User input not sent raw to LLM (prompt injection protection)
- [ ] AI-002: No `bypassPermissions` or equivalent in production
- [ ] AI-003: LLM tool access restricted to minimum necessary
- [ ] AI-004: LLM output sanitized before rendering or executing
- [ ] AI-005: LLM interactions with user data logged

---

## Checklist 2: PR Security Review

Quick checklist for reviewing pull requests:

- [ ] Does this PR introduce any new endpoints? → Verify auth + authorization
- [ ] Does this PR handle user input? → Verify validation and sanitization
- [ ] Does this PR write to the database? → Verify parameterized queries and transactions
- [ ] Does this PR involve money, coupons, or limited-use resources? → Verify race condition protection
- [ ] Does this PR add dependencies? → Check with `npm audit` / `pip-audit`
- [ ] Does this PR store files? → Verify upload security
- [ ] Does this PR change authentication or sessions? → Full auth checklist
- [ ] Does this PR expose new data in API responses? → Verify no data leakage
- [ ] Does this PR add logging? → Verify no sensitive data in logs

---

## Checklist 3: Pre-Deploy Security

Before every production deploy:

- [ ] `npm audit` / `pip-audit` — no CRITICAL or HIGH vulnerabilities
- [ ] No secrets in code (`git grep -i "password\|secret\|token\|api_key"`)
- [ ] `.env` not committed (`git log --all -- .env`)
- [ ] All environment variables documented in `.env.example`
- [ ] Security headers tested (https://securityheaders.com)
- [ ] HTTPS certificate valid
- [ ] Error pages do not expose internal information
- [ ] Admin endpoints require authentication
- [ ] Rate limiting active
- [ ] Backups tested (can you restore?)

---

## Checklist 4: Security Testing

Minimum required security tests per endpoint:

### Access Control Tests
```
✓ Unauthenticated request → 401
✓ Token from another user trying to access another's resource → 403
✓ Regular user trying admin action → 403
✓ IDOR attempt (swap ID in request) → 403
```

### Input Validation Tests
```
✓ Required fields empty → validation error
✓ Strings with 10,000+ characters → reject
✓ Wrong types (string where number expected) → reject
✓ Payload with HTML/JS → sanitized or rejected
✓ Classic SQL injection (' OR 1=1 --) → handled by parameterization
```

### Race Condition Tests
```
✓ Same request sent N times in parallel → process only one
✓ Concurrent purchase, like, coupon, refund, withdrawal → one success
```

### Business Rules Tests
```
✓ Action without prerequisite (buy without balance, access without purchase) → reject
✓ Duplicate action (use coupon twice, request refund twice) → reject or idempotent
✓ Out-of-window action (refund past deadline) → reject
```

### Authentication Tests
```
✓ Expired token → 401
✓ Malformed token → 401
✓ Multiple failed logins → rate limiting active
```

---

## Honeypots (Recommended)

Add fake routes that only log access attempts:

```typescript
// Routes that only attackers would try to access
app.all('/admin', honeypotHandler);
app.all('/wp-admin', honeypotHandler);
app.all('/phpmyadmin', honeypotHandler);
app.all('/.env', honeypotHandler);
app.all('/api/v1/internal', honeypotHandler);
app.all('/config', honeypotHandler);

function honeypotHandler(req, res) {
  logger.warn('HONEYPOT_ACCESS', {
    ip: req.ip,
    path: req.path,
    userAgent: req.headers['user-agent'],
    method: req.method,
    timestamp: new Date().toISOString(),
  });
  // Optionally return fake data to waste attacker's time
  res.status(200).json({ status: 'ok', version: '1.0.0' });
}
```
