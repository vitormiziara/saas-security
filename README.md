# saas-security — Claude Skill for SaaS Security Auditing

A comprehensive Claude skill that turns your AI assistant into a senior security engineer.
Covers code auditing, vulnerability detection, checklist generation, and security report writing — across any stack.

---

## What This Skill Does

When this skill is active, Claude will:

- **Audit code** for vulnerabilities across 16 security domains (OWASP Top 10 + Race Conditions + File Upload + AI/LLM Security + more)
- **Generate security checklists** for PRs, deploys, and full audits
- **Produce structured security reports** with severity scores and remediation priority
- **Write security tests** alongside functional tests
- **Refuse insecure requests** — if you ask to hardcode a secret or skip validation, Claude will explain the risk and refuse
- **Apply security rules silently** to all code it generates, in any language or framework

---

## Security Domains Covered

| Domain | Checks | Priority |
|---|---|---|
| Access Control (IDOR, SSRF, CORS) | AZ-001 → AZ-006 | 🔴 Critical |
| Authentication (JWT, sessions, MFA) | AU-001 → AU-008 | 🔴 Critical |
| Input Validation (SQL, XSS, injection) | IV-001 → IV-007 | 🔴 Critical |
| Secrets & Configuration | SC-001 → SC-008 | 🔴 Critical |
| Data Security & Privacy | DA-001 → DA-005 | 🟠 High |
| File Upload | FU-001 → FU-006 | 🟠 High |
| Session Security | SS-001 → SS-005 | 🟠 High |
| Dependencies & Supply Chain | DS-001 → DS-003 | 🟠 High |
| API Security | AP-001 → AP-004 | 🟠 High |
| Race Conditions | RC-001 → RC-004 | 🟠 High |
| Cryptography | CR-001 → CR-005 | 🟠 High |
| Security Headers | SH-001 → SH-005 | 🟡 Medium |
| Logging & Monitoring | LM-001 → LM-005 | 🟡 Medium |
| DevSecOps | DO-001 → DO-006 | 🟡 Medium |
| Frontend Security | FE-001 → FE-004 | 🟡 Medium |
| AI/LLM Security | AI-001 → AI-005 | 🟡 Medium |

---

## Installation

### Option 1: Claude.ai Skills (Recommended)

1. Download the `.skill` file from the [Releases](../../releases) page
2. Go to **Claude.ai → Settings → Skills**
3. Upload the `.skill` file
4. The skill is now active in all your conversations

### Option 2: Claude Code (Slash Command)

1. Clone this repository into your project:
   ```bash
   git clone https://github.com/YOUR_USERNAME/saas-security.git .claude/skills/saas-security
   ```
2. In Claude Code, the skill will be available automatically
3. Use `/security-review` or simply describe what you want audited

### Option 3: Manual (Copy to Project)

Copy the `saas-security/` folder to your project and reference it in your Claude setup.

---

## Usage Examples

### Full Code Audit
```
Audit this codebase for security vulnerabilities. 
Focus on authentication, access control, and injection risks.
[paste code or describe the project]
```

### Quick PR Review
```
Security review this PR diff before I merge it.
[paste diff]
```

### Generate Security Checklist
```
Generate a security checklist for deploying this Node.js + PostgreSQL SaaS to production.
```

### Race Condition Check
```
Review this payment flow for race conditions and suggest the correct protection pattern.
[paste code]
```

### Security Report
```
Write a full security audit report for this codebase. Include severity scores and remediation roadmap.
```

---

## Skill Structure

```
saas-security/
├── SKILL.md                          # Main skill instructions + workflow
└── references/
    ├── core-principles.md            # 6 fundamental security principles
    ├── owasp-domains.md              # Full rules for all 16 security domains
    ├── race-conditions.md            # Race condition patterns with code examples
    └── checklists.md                 # Ready-to-use checklists for audit, PR, deploy, testing
```

---

## Non-Negotiable Rules

This skill enforces rules that cannot be overridden, even if you ask nicely:

1. If you ask to hardcode secrets → Claude **refuses and explains the risk**
2. If you ask to "simplify" by removing protections → Claude **refuses and suggests a safe simplification**
3. When in doubt whether something is secure → Claude **assumes it is NOT** and adds the protection
4. Every piece of generated code is mentally reviewed against the **12-question security self-check**
5. Security tests are **always included** when generating functional tests

---

## Audit Report Format

Every full audit produces:

```
[DOMAIN] CHECK-ID — Finding Name
Severity: CRITICAL | HIGH | MEDIUM | LOW | INFO
Evidence: <file, line, or code snippet>
Why it matters: <1-sentence business/security impact>
Fix: <concrete code or configuration change>

---
Security Score: XX/100
Top 5 Findings: [list]
Remediation Priority: [ordered list]
Estimated Effort: [hours/sprints]
```

---

## Inspiration & References

This skill synthesizes:

- **[ESAA-Security](https://github.com/elzobrito/ESAA-Security)** — Event-sourced security auditing architecture with 95 checks across 16 domains
- **OWASP Top 10:2021** — The standard web application security risks
- **OWASP ASVS 5.0** — Application Security Verification Standard
- **CIS Controls v8.1** — Center for Internet Security
- **NIST CSF 2.0 + SP 800-53 Rev. 5** — NIST security frameworks

---

## License

MIT — Use freely, contribute back.

---

## Contributing

Found a missing check? A better pattern? Open a PR.

Areas where contributions are especially welcome:
- New code examples for race condition patterns (different databases/ORMs)
- Stack-specific checklists (Django, Rails, Laravel, NestJS, etc.)
- Additional AI/LLM security checks
- LGPD/GDPR-specific compliance checks
