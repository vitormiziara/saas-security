# Core Security Principles

These 6 principles are **always active**, regardless of stack or context.
Apply them silently in every code generation and review.

---

## 1. Defense in Depth

Every layer of the system must be independently secure.

- If the frontend validates → the **backend also validates**
- If the database has constraints → the **code also checks**
- No layer may depend on another layer for security
- Assume every layer below you has already been compromised

## 2. Never Trust the Frontend

All input from the client is potentially malicious.

- Every validation must exist **on the server**
- Every authorization must be verified **in the backend**
- Data from the client are **suggestions**, not facts
- This includes: headers, cookies, query params, body fields, file names, MIME types

## 3. Principle of Least Privilege

Every component must have only the minimum permissions necessary. Nothing more.

Applies to:
- Database roles (read-only where writes aren't needed)
- API tokens and OAuth scopes
- File permissions
- Container users (never run as root)
- Service-to-service credentials
- User roles (user cannot do what admin does)

## 4. Fail Closed (Fail Securely)

If something goes wrong, the system must **deny access by default**.

- Errors must never open security gaps
- Unhandled exceptions must result in **denial**, not permission
- Default state = deny; access must be explicitly granted
- Timeouts, unavailable services, malformed input → all result in access denied

## 5. Secrets Out of Code — Always

**Never** place the following in source code, comments, logs, error messages, or API responses:
- API keys
- Tokens
- Passwords
- Connection strings
- Private keys
- Any secret or credential

**Always use:**
- Environment variables
- Secret managers (AWS Secrets Manager, HashiCorp Vault, Doppler, etc.)
- `.env` files (never committed — always in `.gitignore`)
- `.env.example` with fictitious values for documentation

If the user asks to do otherwise → **refuse and explain the risk**.

## 6. Security by Design, Not by Obscurity

If the system becomes insecure because someone saw the code, it was **never secure**.

- Code must be secure even with a public repository
- The only secrets should be environment variables
- Do not rely on "attackers won't find this endpoint"
- Do not rely on "the UI doesn't show this button" as a security control
- All business rules that restrict access must be enforced **server-side**
