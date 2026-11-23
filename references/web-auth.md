# Web application architecture: auth, sessions, API

## Server-first, client-by-need

Render and load data on the server by default. Move a component to the browser
only when it needs browser APIs, local interactive state, drag-and-drop,
canvas, geolocation, or realtime UI.

- Server components/SSR read data directly through the server-side data layer —
  no extra HTTP hop to your own API.
- Client components receive minimal serializable props; sensitive fields never
  reach the HTML/flight payload.
- Route handlers/BFF serve browser requests, webhooks, and the public API —
  they are not a mandatory layer for every server-side call.
- Authorization and validation repeat on the server for every mutation/server
  action. Proxy (formerly Middleware) alone is not enough (see CVE section
  below).
- Caching is explicit: public/private, TTL, invalidation, stale-data behavior.

## Browser authentication: preferred order

| Scenario | Preferred pattern | Don't |
|---|---|---|
| Same-origin web app | Opaque session ID in `__Host-` HttpOnly Secure cookie; SameSite=Lax/Strict per UX; server-side session store | Don't put the session token in localStorage |
| SPA with sensitive data | **Backend-for-Frontend**: browser ↔ BFF via HttpOnly cookie; BFF ↔ API with server-side tokens | Don't hand the browser a long-lived refresh token without necessity |
| OAuth/OIDC login | Authorization Code + PKCE; exact redirect URIs; state/nonce; server-side exchange where possible | Don't use implicit flow; don't accept arbitrary redirects |
| Third-party public API | OAuth access token in `Authorization: Bearer`; scopes and short TTL | Don't use the browser session cookie as a universal API credential |
| First factor for a new product | Passkeys (WebAuthn Level 3, W3C Recommendation) through the identity provider or library; password + MFA as the fallback path | Don't ship password-only login; don't implement WebAuthn ceremonies by hand |
| Low-risk prototype | Managed auth and server session | Don't write your own crypto/passwords |

Why BFF ranks first: RFC 10017 "OAuth 2.0 for Browser-Based Applications"
(BCP 212) orders the three architectures "in decreasing order of security" —
BFF (no tokens in the browser at all) → token-mediating backend →
browser-based OAuth client. With a BFF, XSS in the app cannot exfiltrate
tokens; HttpOnly cookies keep even the session out of script reach.
RFC 9700 (BCP 240, Jan 2025) is the published OAuth 2.0 Security BCP:
implicit grant deprecated, Authorization Code + PKCE required, exact redirect
URI matching, refresh token rotation + reuse detection. An update draft
(draft-ietf-oauth-security-topics-update) is in progress ⏳.

## Cookie checklist

- `Secure` — HTTPS only.
- `HttpOnly` — whenever JavaScript doesn't need to read the cookie.
- `SameSite` set explicitly: **Lax** is the practical default; Strict for
  isolated flows (it breaks arrival from external links and OAuth redirects);
  None only together with Secure.
- For host-only session cookies use the `__Host-` prefix: requires Secure,
  `Path=/`, and no `Domain` attribute.
- Short idle timeout + reasonable absolute timeout; renewal must not make the
  session eternal.
- Rotate the session ID after login, password change, MFA, role elevation —
  any privilege change (OWASP Session Management Cheat Sheet; ≥64 bits of
  entropy, server-side state of record).
- Logout invalidates the server-side session/refresh token, not just UI state.
- Don't store PII, permission snapshots, or large JSON in cookies.

## CSRF — the most commonly misunderstood attack

CSRF does **not** require XSS or injecting script into your site. The attacker
triggers a request from their own site; the browser attaches your cookies
automatically. XSS and CSRF are distinct attack classes with distinct defenses.

Layered defense (OWASP CSRF Prevention Cheat Sheet):
SameSite cookies + CSRF tokens + `Origin`/Fetch Metadata (`Sec-Fetch-Site`)
checks + never mutating state via GET.

## JWT: format, not authorization system

JWT (RFC 7519) is a token *format*. It does not make a system secure and it is
not an authorization framework. "JWT cannot be revoked" is too categorical:
short TTL, refresh-token rotation with reuse detection, denylists, token
introspection, and session-version claims are the standard mitigations. Signed
cookies protect integrity but not confidentiality — don't put secrets in them.

## Token storage in the browser

Long-lived bearer/refresh tokens in `localStorage`/`sessionStorage` are
readable by any script that runs in the page — one XSS and they're gone. For
anything sensitive prefer BFF + HttpOnly cookie, or at minimum short-lived
tokens held in memory.

## Next.js specifics (App Router)

Server Components, Server Actions, and Route Handlers keep secrets
server-side, but every Server Action needs its own auth check, and any env var
prefixed `NEXT_PUBLIC_` ships to the user.

**Proxy (formerly Middleware) is not a security boundary.** Next.js's own auth
guide (https://nextjs.org/docs/app/guides/authentication) now says it: "While
Proxy can be useful for initial checks, it should not be your only line of
defense in protecting your data. The majority of security checks should be
performed as close as possible to your data source". The routing edge has
failed six times in this class:

- **CVE-2025-29927** (CVSS 9.1, Mar 2025) — `x-middleware-subrequest` header
  bypassed middleware auth entirely. Vercel's post-mortem: perform
  authorization checks at the data layer.
- **CVE-2026-44573** (CVSS 7.5, May 2026) — Pages Router + i18n: locale-less
  `/_next/data/…/*.json` requests skip middleware, leaking SSR JSON of
  protected pages. Fixed in 15.5.16 / 16.2.5.
- **CVE-2026-44574** (May 2026) — dynamic route parameter injection routes
  around the middleware matcher.
- **CVE-2026-44575** (CVSS 7.5, May 2026) — App Router: crafted `.rsc` /
  segment-prefetch URLs resolve to protected pages without matching the
  middleware rule. Fixed in 15.5.16 / 16.2.5.
- **CVE-2026-45109** (May 2026) — incomplete fix of the previous one: the
  patch did not apply to `middleware.ts` under Turbopack. A fix verified with
  one bundler is not a fix for the other.
- **CVE-2026-64642** (Jul 2026) — App Router built with Turbopack and a
  single `i18n.locales` entry bypasses proxy auth. Fixed in 16.2.11.

The same window brought two SSRF issues (rewrites with an attacker-controlled
destination host; Server Actions on custom servers) and two Critical
unauthenticated RCEs (image optimization via AVIF, Windows-hosted servers),
fixed in 16.3.3 / 15.5.24. More Critical RCEs followed in later releases, among
them `next/og` ImageResponse (GHSA-vcvr-r3jv-pc5j) and the Image Optimization
API when SVG files are used (GHSA-2464-fhr4-w9xw). Self-hosted image
optimization is attack surface: either use the platform's optimizer or treat
the monthly Next.js security release as your patch cadence.

⏳ Check for newer advisories (https://nextjs.org/blog, GitHub advisories; the
project publishes a security release monthly) whenever recommending a Next.js
auth design. The stable lesson: authorize next to data access / in the use
case, not (only) at the routing edge.

## API and mutations

- GET/HEAD never change state.
- POST creates, PUT replaces, PATCH partially updates, DELETE deletes —
  contract consistency matters more than purity.
- Every mutation validates input and authorization on the server.
- Errors carry stable machine-readable codes, a request ID, and a safe user
  message.
- Pagination: cursor-based for large/changing lists; server-enforced limits.
- Idempotency keys for payments, order creation, webhooks, repeatable commands.
- Version the API only on real incompatibility; prefer additive changes.
- Rate-limit login, reset, search, AI/expensive endpoints, public forms.

## Frontend structure

Structure by feature/domain, not only by technical type. The subscription
screen keeps its UI, schema, queries, mutations, tests, and analytics
contracts together.

| Good boundary | Bad boundary |
|---|---|
| `features/billing`, `features/trips` | `components/`, `hooks/`, `utils/` with hundreds of unrelated files |
| `shared/ui` — only genuinely reusable primitives | A universal component with dozens of boolean props |
| Data-access layer returns domain models | Components know the DB structure |
| `server-only` module for secrets and privileged ops | One `utils.ts` imported by both server and browser |
