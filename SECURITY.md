# Security Policy

## About Secuvon

Secuvon is an enterprise AI security platform built by Nexphase Technologies. We scan, protect, and audit LLMs and AI agents for prompt injection, data leakage, jailbreaks, and compliance violations. Security is not just our product, it is foundational to how we build, operate, and deliver our platform.

---

## Reporting a Vulnerability

We take all security reports seriously and appreciate responsible disclosure.

### How to Report

- **Email:** [security@secuvon.com](mailto:security@secuvon.com)
- **Subject line format:** `[SECURITY] Brief description of the issue`
- **Encrypt sensitive details:** PGP key available at [secuvon.com/.well-known/security.txt](https://secuvon.com/.well-known/security.txt)

### What to Include

- A clear description of the vulnerability
- Steps to reproduce the issue
- Affected component(s) (platform, CLI, SDK, agent-guard)
- Impact assessment (what an attacker could achieve)
- Any proof-of-concept code or screenshots
- Your contact information for follow-up

### What NOT to Do

- **Do NOT** open a public GitHub issue for security vulnerabilities
- **Do NOT** access, modify, or delete data belonging to other users
- **Do NOT** perform denial-of-service testing against production systems
- **Do NOT** social-engineer Secuvon employees or users
- **Do NOT** publicly disclose the vulnerability before we have addressed it

### Response Timeline

| Stage | Timeframe |
|-------|-----------|
| Acknowledgement of report | Within 24 hours |
| Initial triage and severity assessment | Within 48 hours |
| Status update with remediation plan | Within 7 days |
| Patch deployed to production | Based on severity (see below) |
| Public disclosure (coordinated) | After patch is deployed + 30 days |

### Severity-Based Response

| Severity | Examples | Patch SLA |
|----------|----------|-----------|
| **Critical** | Remote code execution, authentication bypass, full data breach | 24 hours |
| **High** | Privilege escalation, PII exposure, injection attacks | 72 hours |
| **Medium** | Information disclosure, CSRF, rate limiting bypass | 7 days |
| **Low** | Minor information leakage, best practice violations | 30 days |

---

## Supported Versions

We provide security patches for the latest version of each component:

| Repository | Type | Supported |
|------------|------|-----------|
| `secuvon/platform` | Private, Core platform | Latest release |
| `secuvon/secuvon-cli` | Private, CLI tool | Latest release |
| `secuvon/secuvon-sdk` | Private, Python/Node SDK | Latest release |
| `secuvon/secuvon-benchmark` | Private, Benchmark suite | Latest release |
| `secuvon/agent-guard` | Private, Runtime guard | Latest release |

Older versions are not supported. We strongly recommend always running the latest version.

---

## Security Architecture

### Data Protection

- **API keys at rest:** Encrypted using Fernet symmetric encryption (AES-128-CBC with HMAC-SHA256). Plaintext keys exist in memory only during active scan execution and are zeroed immediately after.
- **API keys in transit:** All API communication uses TLS 1.2+ (HTTPS). API keys are never transmitted in URL parameters.
- **Passwords:** Hashed using bcrypt with automatic salt generation. Plaintext passwords are never stored or logged.
- **Scan results:** Sensitive evidence (LLM responses containing potential PII or attack outputs) is encrypted before database storage using per-scan encryption keys.
- **Database:** PostgreSQL with encrypted connections (SSL mode: require). Connection strings and credentials are managed via environment variables, never hardcoded.

### Authentication & Authorization

- **JWT tokens:** Short-lived access tokens (30 minutes) with refresh token rotation. Tokens are signed using HS256 with a server-side secret key.
- **Session management:** Stateless JWT authentication. No server-side session storage. Tokens are stored in secure, httpOnly cookies with SameSite=Strict.
- **Rate limiting:** Per-user and per-endpoint rate limits enforced via slowapi (backed by Redis). Scan execution is rate-limited to prevent abuse.
- **RBAC:** Role-based access control with five roles (Owner, Admin, Manager, Member, Auditor). Each role has specific permissions for scans, agents, reports, and team management.

### Infrastructure Security

- **CORS:** Strict origin allowlist. Only explicitly configured origins are permitted.
- **CSP:** Content Security Policy headers prevent inline script execution and restrict resource loading.
- **Security headers:** HSTS (Strict-Transport-Security), X-Frame-Options (DENY), X-Content-Type-Options (nosniff), Referrer-Policy (strict-origin-when-cross-origin).
- **Request validation:** All inputs are validated using Pydantic models with strict type checking. SQL injection is prevented by SQLAlchemy's parameterized queries.
- **SSRF prevention:** Endpoint URLs provided by users are validated against an allowlist of permitted schemes and hosts before any outbound HTTP requests.

### Scanning Engine Security

- **Isolation:** Each scan runs in an isolated execution context. Scans for different users never share state, credentials, or results.
- **Credential handling:** User API keys are decrypted from the database only during scan execution, used for the HTTP requests to the target LLM, and immediately zeroed from memory upon scan completion.
- **Network boundaries:** The scanning engine makes outbound requests only to user-specified LLM provider endpoints (OpenAI, Anthropic, Google, or custom). No other outbound connections are made during scanning.
- **Result integrity:** Scan results are cryptographically hashed using SHA-256 with chain verification. Each scan's hash includes the previous scan's hash, creating a tamper-evident audit trail.
- **Audit logging:** All security-relevant actions (login, scan execution, agent creation, report access, data deletion) are logged with user ID, timestamp, IP address, and action details.

### Supply Chain Security

- **Dependency scanning:** Dependabot monitors all dependencies (npm, pip, Docker base images, GitHub Actions) for known vulnerabilities. Security updates are automatically proposed as pull requests.
- **Secret scanning:** GitHub secret scanning with push protection prevents accidental commits of API keys, tokens, or credentials. AI-powered detection catches non-standard secret patterns.
- **Static analysis:** CodeQL runs on every push and pull request, scanning both TypeScript and Python code for security vulnerabilities including XSS, injection, and authentication issues.
- **Code quality:** Automated code quality scanning detects dead code, unreachable logic, and potential bugs before they reach production.
- **Container security:** Docker images use minimal base images (Alpine Linux), run as non-root users, and are rebuilt from pinned dependency versions.

---

## Compliance

Secuvon maps scan results against industry-standard security frameworks:

| Framework | Coverage |
|-----------|----------|
| OWASP Agentic Top 10 | Full mapping across all 10 risk categories |
| OWASP LLM Top 10 | Full mapping across all 10 risk categories |
| MITRE ATLAS | Adversarial ML technique coverage |
| EU AI Act | High-risk AI system requirements |
| NIST AI RMF | Risk management framework controls |
| SOC 2 Type II | Trust service criteria mapping |
| ISO 42001 | AI management system controls |
| HIPAA | Health data protection requirements |
| GDPR | Data protection regulation controls |
| PCI DSS | Payment card industry standards |

---

## Bug Bounty Program

We are currently developing a formal bug bounty program. In the meantime, we will recognize responsible disclosure with:

- Credit in our security acknowledgements (with your permission)
- A letter of appreciation for verified reports
- Priority notification of security updates

---

## Security Contacts

| Contact | Purpose |
|---------|---------|
| [security@secuvon.com](mailto:security@secuvon.com) | Vulnerability reports |
| [privacy@secuvon.com](mailto:privacy@secuvon.com) | Data privacy inquiries |
| [compliance@secuvon.com](mailto:compliance@secuvon.com) | Compliance and audit requests |

---

## Changelog

| Date | Change |
|------|--------|
| 2026-04-18 | Initial comprehensive security policy |

---

*This policy is reviewed quarterly and updated as our security practices evolve. Last reviewed: April 2026.*

*Nexphase Technologies, Building trust in AI, one scan at a time.*
