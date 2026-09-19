# Security Policy

## Supported Versions

Iconic Estates India is deployed as a single running platform rather than
distributed as a versioned library, so security fixes are applied to the
active deployment branch rather than backported across release lines.

| Branch / Deployment          | Supported          |
| ----------------------------- | ------------------ |
| `main` (production)           | :white_check_mark: |
| Active feature branches       | :white_check_mark: |
| Archived / superseded forks   | :x:                |

If you're running a fork or an older snapshot, you're responsible for pulling
in fixes yourself — please don't expect patches to be backported to it.

## Reporting a Vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.**
This platform handles lead/customer personal data (names, mobile numbers,
emails), admin credentials, uploaded documents, and OTP-based authentication —
a public report before a fix ships can be enough for real misuse.

Instead, report privately using one of the following:

- **GitHub Private Vulnerability Reporting** — open the repository's
  **Security** tab → **Report a vulnerability**, if enabled for this repo.
- **Email** — send details to the maintainer's contact email listed on the
  repository's GitHub profile / organization page.

When reporting, please include:

- A description of the vulnerability and its potential impact.
- Steps to reproduce (a minimal example is ideal).
- Which component is affected — the React frontend, the Node/Express API
  (`backend-node/`), the Python analytics service (`backend-python/`), or the
  database schema/migrations.
- Whether the issue requires authentication, and if so, which session type
  (admin JWT or customer-portal OTP session) and role.

### What to expect

- **Acknowledgement** within 3 business days of your report.
- **Initial assessment** (severity and affected components) within 7 days.
- **Status updates** at least every 7 days until the issue is resolved, more
  frequently for high-severity reports.
- **Resolution timeline** depends on severity:
  - **Critical** (e.g. auth bypass, cross-customer data access, OTP bypass,
    SQL injection, remote code execution): fix targeted within 7 days.
  - **High** (e.g. privilege escalation between roles, CORS misconfiguration
    exploited to steal admin sessions, unauthenticated data exposure): fix
    targeted within 14 days.
  - **Medium/Low** (e.g. missing hardening, rate-limit tuning, non-exploitable
    misconfiguration): scheduled into the normal development cycle.

If a report is **accepted**, you'll be credited in the fix's changelog entry
unless you ask to remain anonymous, and notified once the fix is deployed. If
a report is **declined** (not reproducible, out of scope, or judged not to be
a vulnerability), you'll get an explanation and are welcome to provide
additional evidence for reconsideration.

### Scope

In scope:
- Admin authentication and RBAC (`backend-node/middleware/auth.js`,
  `routes/auth.js`) — including whether `admin`/`agent` accounts can reach
  `super_admin`-only actions such as `POST /api/auth/register`
- Customer-portal OTP authentication
  (`backend-node/controllers/customerAuthController.js`,
  `middleware/customerAuth.js`) — OTP brute-forcing, OTP reuse/replay, session
  confusion between the admin JWT and the customer JWT, or one customer
  accessing another customer's leads/inquiries/messages/documents
- The Node REST API (`backend-node/routes/`, `controllers/`) — injection,
  IDOR, mass assignment, broken access control
- The Python FastAPI analytics service (`backend-python/`)
- File upload handling (`backend-node/middleware/upload.js`) and the
  `/uploads` static file route
- CORS configuration in `backend-node/server.js` and the Python service
- Outbound email handling (`backend-node/utils/mailer.js`) — e.g. header
  injection via user-supplied fields
- SQL injection, XSS, CSRF, and insecure direct object references anywhere in
  the stack

Out of scope:
- Findings that require access to `.env` files or database credentials you
  should not already have
- Denial-of-service via raw traffic volume (report application-logic DoS —
  e.g. an endpoint with no pagination limit — separately)
- Social engineering against maintainers or users
- Issues in third-party dependencies — please report those upstream, though a
  link here is appreciated so we can track exposure

### A note on this project's current security posture

A few known gaps are documented in the
[README's Security Notes](README.md#security-notes) rather than hidden:
CORS currently reflects any request origin (`origin: true` in
`backend-node/server.js`), the customer-portal JWT secret falls back to the
admin JWT secret if not set separately, and OTP codes are returned directly in
the API response when SMTP isn't configured (intended for local development
only). These are known, tracked issues — you're welcome to report hardening
suggestions or exploitation paths for them, but please reference the README
section so reports aren't duplicated.

Thank you for helping keep Iconic Estates India and its customers' data safe.
