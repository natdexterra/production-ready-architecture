# Curated sources

Cite from this list, most authoritative first. Grades: **A** — published
normative (RFC, W3C REC, official vendor/platform docs) · **B** —
standards-body draft / OWASP cheat sheets · **C** — recognized practitioners
(Fowler, Newman, DORA, published books). SEO blogs, cheat-sheet mirrors, and
Scribd/Medium copies were deliberately excluded — always cite the canonical
origin. ⏳ marks items whose status/version changes; re-verify before formal
citation. The checklist for that is the "Volatile sources" table at the end:
re-check the matching row before relying on a marked fact.

## Auth, sessions, cookies

- [A] RFC 9700 — Best Current Practice for OAuth 2.0 Security (BCP 240, Jan
  2025). https://rfc-editor.org/rfc/rfc9700 · ⏳ update draft in progress
  (draft-ietf-oauth-security-topics-update).
- [A] RFC 10017 — OAuth 2.0 for Browser-Based Applications (BCP 212, Aug
  2026). https://rfc-editor.org/rfc/rfc10017 · §6 ranks BFF > token-mediating backend
  > browser-based OAuth client.
- [A] W3C — Web Authentication Level 3 (Recommendation, Aug 2026), the
  passkeys reference. https://w3.org/TR/webauthn-3.
- [B] OWASP Session Management Cheat Sheet; Authentication Cheat Sheet;
  OAuth2 Cheat Sheet — https://cheatsheetseries.owasp.org.
- [A] MDN — Using HTTP cookies; Set-Cookie (SameSite, `__Host-`).
- [B⏳] RFC 6265bis (cookie prefixes, SameSite formalization) — still a draft.
- [A] OpenID Connect Core 1.0 — https://openid.net/specs.
- [A] RFC 7519 (JWT format), RFC 7636 (PKCE), RFC 8252 (OAuth for Native
  Apps, BCP 212).

## CSRF / XSS

- [B] OWASP CSRF Prevention Cheat Sheet; XSS Prevention Cheat Sheet.
- [A] MDN — CSRF attack docs; `Sec-Fetch-Site` (Fetch Metadata); CSP.
- [B] OWASP Content Security Policy Cheat Sheet (strict policy shapes, delivery
  by header vs `<meta>`, report-only rollout).
- [A] web.dev — strict CSP (nonce for server-rendered pages, hash for static
  or cached pages). https://web.dev/articles/strict-csp.

## Next.js / server-first web

- [A] Next.js docs — App Router, Authentication guide (Proxy is not the
  boundary; checks close to the data source), Data Security, Production
  Checklist, env vars. https://nextjs.org/docs. Monthly security releases on
  https://nextjs.org/blog.
- [A] Vercel/Next.js security advisories cited in `web-auth.md` (index:
  https://github.com/vercel/next.js/security/advisories). ⏳ check for newer.
  - CVE-2025-29927, GHSA-f82v-jwr5-mffw:
    https://github.com/vercel/next.js/security/advisories/GHSA-f82v-jwr5-mffw;
    post-mortem https://nextjs.org/blog/cve-2025-29927
  - CVE-2026-44573, GHSA-36qx-fr4f-26g5:
    https://github.com/vercel/next.js/security/advisories/GHSA-36qx-fr4f-26g5
  - CVE-2026-44574, GHSA-492v-c6pp-mqqv:
    https://github.com/vercel/next.js/security/advisories/GHSA-492v-c6pp-mqqv
  - CVE-2026-44575, GHSA-267c-6grr-h53f:
    https://github.com/vercel/next.js/security/advisories/GHSA-267c-6grr-h53f
  - CVE-2026-45109, GHSA-26hh-7cqf-hhc6:
    https://github.com/vercel/next.js/security/advisories/GHSA-26hh-7cqf-hhc6
  - CVE-2026-64642, GHSA-6gpp-xcg3-4w24:
    https://github.com/vercel/next.js/security/advisories/GHSA-6gpp-xcg3-4w24
  - Later Critical RCEs named in `web-auth.md`: GHSA-vcvr-r3jv-pc5j:
    https://github.com/vercel/next.js/security/advisories/GHSA-vcvr-r3jv-pc5j;
    GHSA-2464-fhr4-w9xw:
    https://github.com/vercel/next.js/security/advisories/GHSA-2464-fhr4-w9xw

## Mobile

- [A] OWASP MASVS v2.1 + MASTG v2.0 — https://mas.owasp.org. ⏳ version.
- [A] Apple — App Review Guidelines 3.1.1 / 3.1.3 (external purchase links
  per storefront); Apple Newsroom EU terms. https://developer.apple.com.
- [A] Google Play — alternative billing and external offers policy pages.
  https://support.google.com/googleplay/android-developer.
- [A] Apple — Keychain Services (https://developer.apple.com/documentation/security).
- [A] Android — Keystore system; security guidance
  (https://developer.android.com/privacy-and-security/keystore).
- [C] Kleppmann, *Designing Data-Intensive Applications* (sync/consistency);
  Ink & Switch, "Local-first software" (offline-first framing).

## Architecture: monolith → services

- [C] Fowler — "Monolith First"; "Microservice Trade-Offs"; "Microservice
  Prerequisites" (https://martinfowler.com).
- [C] Newman — *Building Microservices*, 2nd ed. (2021); *Monolith to
  Microservices*; BFF pattern (https://samnewman.io/patterns/architectural/bff).
- [C] Brown — modular monoliths (https://simonbrown.je); Skelton & Pais — *Team
  Topologies* (cognitive-load argument against premature splitting).

## Database migrations

- [C] Fowler — "Parallel Change"; "Evolutionary Database Design"
  (https://martinfowler.com/articles/evodb.html).
- [C] Ambler & Sadalage — *Refactoring Databases* (2006), still the standard.
- [A] GitLab — multi-version compatibility / expand-contract docs
  (https://docs.gitlab.com/development/multi_version_compatibility).
- Vendor guides (verify against A/C): PlanetScale "Backward compatible
  database changes"; Prisma Data Guide expand-and-contract.

## Payments

- [A] Stripe — Webhooks (https://docs.stripe.com/webhooks: signature verification,
  raw body, replay tolerance, return 2xx first); Idempotent requests
  (https://docs.stripe.com/api/idempotent_requests, ⏳ ~24h key retention); error
  handling.
- [A] Adyen — Verify HMAC signatures (https://docs.adyen.com).
- [A] PCI SSC — SAQ A scope (redirect/tokenize, don't touch card data).
- Note: IETF `Idempotency-Key` header draft **expired** Apr 2026 without
  becoming an RFC — idempotency remains vendor-convention territory.

## Delivery / CI/CD

- [C] Forsgren, Humble, Kim — *Accelerate*; DORA capabilities & annual
  reports (https://dora.dev: trunk-based development, continuous delivery; the five
  software delivery metrics at https://dora.dev/guides/dora-metrics).
- [A] Node.js — release schedule (one major per year, every release LTS).
  https://nodejs.org/en/blog/announcements.
- [A] Platform docs and changelogs, facts only, never judgement: Vercel
  (deployment protection, environment variable types, Vercel Connect,
  Workflows, Queues, AI Gateway budgets), Cloudflare (Containers, Sandboxes,
  Workers VPC, Workflows retention), Neon (branching of database, auth and
  storage), Supabase (branching default, backups per plan), Sentry (logs
  quota), Stripe (Managed Payments, event destinations), Agentic Commerce
  Protocol (https://agenticcommerce.dev).
- [C] Humble & Farley — *Continuous Delivery*.
- [C] Hodgson — "Feature Toggles" (https://martinfowler.com/articles/feature-toggles.html).
- [A] Vercel — Preview Deployments docs.
- [A] Google SRE Book — https://sre.google (monitoring chapter: four golden signals).

## Observability

- [A] OpenTelemetry docs — https://opentelemetry.io.
- [A] W3C Trace Context — https://w3.org/TR/trace-context.
- [A] Google SRE Book — Monitoring Distributed Systems.

## Threat modeling & standards

- [B] OWASP Threat Modeling Cheat Sheet.
- [C] Shostack — *Threat Modeling: Designing for Security* (four questions).
- [A] Microsoft — STRIDE (https://learn.microsoft.com).
- [A⏳] OWASP ASVS 5.0 (May 2025, 17 chapters); OWASP Top 10:2025
  (https://owasp.org/Top10/2025); OWASP API Security Top 10 (2023 edition; rate
  limiting / BOLA).

## Performance & accessibility

- [A⏳] web.dev — Web Vitals (LCP/INP/CLS thresholds, changelog).
- [A] W3C — WCAG 2.2; ARIA Authoring Practices Guide (APG).
- [A] MDN — Performance APIs; Accessibility.
- [C] Walton — "Modern Web Types" (https://philipwalton.com/articles/modern-web-types):
  why the type system is a poor guard for new browser APIs; guard with
  cross-browser tests and a Baseline lint instead.

## Volatile sources

One row per fact that can silently go stale. "What to check" names the exact
value the skill states, so a refresh compares a value, not a vibe. No dates
here: the date the rows were last checked is in `CHANGELOG.md` at the root of
the skill folder.

| Canonical URL | What to check | Where the skill uses it |
|---|---|---|
| https://rfc-editor.org/rfc/rfc9700 · https://datatracker.ietf.org/doc/draft-ietf-oauth-security-topics-update | Has the update draft become an RFC that obsoletes or amends 9700 | web-auth.md |
| https://rfc-editor.org/rfc/rfc10017 · https://datatracker.ietf.org/doc/rfc10017 | Errata or an obsoleting RFC; §6 ranking unchanged | web-auth.md |
| https://w3.org/TR/webauthn-3 | Still the Recommendation; a Level 4 Recommendation would replace it | web-auth.md, security-privacy.md |
| https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis | Still a draft or published; cookie prefixes and SameSite wording | web-auth.md |
| https://nextjs.org/blog · https://github.com/vercel/next.js/security/advisories | Newer middleware/proxy auth-bypass advisories than CVE-2026-64642, the newest of the six web-auth.md lists; current advice on where auth checks live | web-auth.md |
| https://github.com/OWASP/masvs/releases · https://mas.owasp.org/MASTG | MASVS version (skill states v2.1; the site prints no number, the GitHub release does); MASTG major (skill states v2.0) | mobile.md |
| https://github.com/OWASP/ASVS/blob/master/5.0/en/0x03-What-is-the-ASVS.md | Chapter count (skill states 17) and version | security-privacy.md |
| https://developer.apple.com/app-store/review/guidelines §3.1 · https://support.google.com/googleplay/android-developer (alternative billing) | Which storefronts allow external purchase, the commission or service fee, reporting obligations | mobile.md |
| https://docs.stripe.com/api/idempotent_requests | Idempotency key retention window (skill states ~24h) | payments.md |
| https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header | Revived or still expired | payments.md |
| https://owasp.org/www-project-application-security-verification-standard | ASVS version (skill states 5.0) | security-privacy.md |
| https://owasp.org/Top10 · https://owasp.org/API-Security | Top 10 edition (skill states 2025); API Top 10 edition | security-privacy.md |
| https://web.dev/articles/vitals | The Core Web Vitals set and thresholds (skill states LCP 2.5 s, INP 200 ms, CLS 0.1) | reliability-observability.md |
| https://w3.org/TR/WCAG22 · https://w3.org/TR/WCAG30 | WCAG 2.2 still the Recommendation; WCAG 3.0 status | reliability-observability.md |
| https://dora.dev/guides/dora-metrics · https://dora.dev/research | The metric set (skill states five: deployment frequency, lead time, change failure rate, failed deployment recovery time, deployment rework rate) and the latest report year | delivery.md |
| https://nodejs.org/en/blog/announcements | Release cadence and LTS rule (skill states one major per year, every release LTS) | delivery.md |
| https://vercel.com/docs/deployment-protection · https://vercel.com/docs/environment-variables | Which protection is free on which plan; variable types and the CLI flag | delivery.md |
| https://developers.cloudflare.com/changelog/product/workflows | Failed-instance retention default (skill states seven days) | reliability-observability.md |
| https://sentry.io/pricing/ | Structured logs quota on the free plan (skill states logs are included) | reliability-observability.md |
| https://docs.stripe.com/webhooks/migrate-snapshot-to-thin-events · https://agenticcommerce.dev/docs/changelog | Thin-event dedup field name; ACP spec version and breaking changes | payments.md |
