# Security, data, and privacy

## Threat modeling for a designer — five questions

Before any feature with accounts, payments, files, geolocation, or roles:

1. What data or actions here are valuable to an attacker?
2. Who can call the endpoint directly, bypassing the UI?
3. Can someone change a foreign ID in the URL/body and reach another user's
   object? (IDOR/BOLA — test it, don't assume.)
4. What happens if the request is replayed 2–10 times?
5. What ends up in logs, analytics, crash reports, email, push?

For deeper structure: OWASP Threat Modeling Cheat Sheet; Shostack's
four-question frame (what are we building / what can go wrong / what do we do
about it / did we do a good job); STRIDE as the threat taxonomy.

## Authorization

- Authentication answers "who is this"; authorization — "what may they do".
  Check both.
- Check access **next to the data access / use case**, not only in middleware
  or UI (middleware-bypass CVEs in `web-auth.md` are the cautionary tale).
- Never trust `role`/`ownerId` arriving from the client.
- Deny by default; minimum scopes/roles.
- Admin/support actions get an audit log: who, when, what, on which object.

## Data lifecycle

| Stage | Rule |
|---|---|
| Collection | Only what the specific purpose needs; explain why |
| Storage | Classify data; encryption at rest via the managed provider; secrets separate |
| Access | Least privilege for people, services, and DB users |
| Transfer | HTTPS/TLS; signed URLs with short TTL for files |
| Backup | Automatic backups + periodically tested restore |
| Deletion | Defined retention and real deletion/anonymization — including caches and third-party services |

## Secrets

- Secrets live in a secrets manager or platform env vars — not in a `.env`
  committed to git.
- `NEXT_PUBLIC_`-prefixed vars (and analogues) are public; the user will see
  them.
- Different credentials per environment (dev/staging/prod).
- Keys carry minimal permissions, an owner, a rotation date, and a revocation
  procedure.
- After a leak: revoke/rotate **first**, investigate second. Deleting from git
  history is not remediation on its own.

## Content Security Policy

Every web product sends a `Content-Security-Policy` response header; a spec or
a build without one is a finding.

| Page | Policy |
|---|---|
| HTML rendered on the server per request | Nonce-based: `script-src 'nonce-{random}' 'strict-dynamic'; object-src 'none'; base-uri 'none'`, a new nonce for every response |
| Static or cached HTML, a statically served SPA | Hash-based: the same policy with the `'sha256-…'` of each inline script in place of the nonce |

Red flags: a host allow-list (`script-src 'self' cdn.example.com`) presented
as XSS protection, allow-lists are bypassable in most configurations;
`'unsafe-inline'` with no nonce or hash beside it; a middleware that stamps
the nonce onto every script tag, an injected script gets it too; the policy in
a `<meta>` tag when framing protection or violation reports are needed,
`frame-ancestors` and reporting work only from the header. Roll out with
`Content-Security-Policy-Report-Only`, then enforce. CSP is the second layer:
output encoding and sanitizing stay the first.

## Standards to check against (versions move ⏳, see `sources.md`)

- **OWASP ASVS 5.0** (May 2025) — the baseline verification standard for web
  apps; 17 chapters, V1 Encoding and Sanitization to V17 WebRTC.
- **OWASP Top 10:2025** — the current awareness list. Two entries map onto
  this skill's principles: A09 is now "Security Logging and Alerting
  Failures" (logging without alerting counts as a failure), and A10
  "Mishandling of Exceptional Conditions" is "failure is a normal scenario"
  written as a risk; A03 became Software Supply Chain Failures.
- **OWASP API Security Top 10** (2023 edition) — BOLA, rate limiting.
- **OWASP MASVS v2.1** — mobile equivalent (see `mobile.md`).
- **WebAuthn Level 3** (W3C Recommendation) — the passkeys reference for the
  first factor in `web-auth.md`.
- **OWASP Cheat Sheet Series** — the working reference for concrete topics
  (Session Management, CSRF, Authentication, Threat Modeling); always cite the
  canonical https://cheatsheetseries.owasp.org, not mirrors.
