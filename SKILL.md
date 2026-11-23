---
name: production-ready-architecture
description: "Production-ready architecture review and decision support for web and mobile applications, grounded in verified primary sources (OWASP, IETF RFCs, MDN, web.dev, Stripe, Fowler/DORA). Use whenever the user asks to review or design an architecture, check production readiness, choose between monolith and microservices, design auth (sessions, cookies, JWT, OAuth, token storage, BFF), integrate payments (idempotency, webhooks, reconciliation), plan environments / CI/CD / preview deploys / database migrations, store secrets or mobile tokens, or evaluate a technical spec or contractor proposal, or check performance, accessibility and observability baselines (Core Web Vitals thresholds, WCAG level, current standard versions). Trigger even when the user doesn't say 'architecture' — e.g. 'where do I store the token', 'is this spec production ready', 'check the architecture / the spec', 'how do we set up deploys', 'monolith or microservices', 'we are adding payments'. Not for visual/UI design or copywriting."
---

# Production-Ready Architecture

Help a product designer / product builder make and verify architecture decisions
for web and mobile applications — without pretending to be a DevOps engineer.

The core idea: **architecture is not a list of fashionable technologies.** It is
a set of deliberate boundaries, delivery rules, data protection, and ways to
recover from failure. Red flags matter more than framework choice.

Items marked ⏳ are volatile (draft statuses, CVE lists, standard versions,
plan limits) — re-verify before citing them in a client-facing document.
`references/sources.md` ends with the table of what moves: re-check the row
for a marked fact against its canonical URL before relying on it.

## Workflow

0. **Intake: get the context before the advice.** Four facts decide most
   answers, and a confident answer without them is the failure this skill
   exists to prevent: what the product does and for whom; what data it holds
   (money, health, children, other people's personal data); the stage
   (prototype, real users, paying users); the constraints (team size, stack
   already chosen, platform, budget). If the request states them, go on. If
   it does not, ask, in one message, only for the ones the answer turns on
   (two or three questions, not a form). For a quick question ("which
   database", "where do I store the token") do not block: answer under a
   stated assumption and show where the answer flips ("for a prototype X;
   once you take payments Y, because Z").
1. **Determine the risk level** (A/B/C below). Everything else scales from it.
   Don't impose level-C ceremony on a prototype, and don't let a real product
   launch with level-A shortcuts.
2. **Apply the defaults.** Deviations are allowed but must be argued and
   recorded (ADR template in `references/checklists.md`). Close the
   architecture document with a numbered list of invariants that hold
   regardless of stack (one owner per entity, the server checks every right,
   no secret in a client); a change that breaks one is a change to the
   document first, then to the code.
3. **Scan the red flags.** Any hit is worth raising with the user directly,
   even if they asked about something else.
4. **Go deeper by domain** — read the matching reference file (map below).
5. **Ground recommendations in sources.** `references/sources.md` carries the
   curated bibliography (A-grade primary sources first). Cite specifics
   (RFC number, cheat-sheet name), not vibes.

## Risk levels

| Level | Example | Mandatory minimum |
|---|---|---|
| **A — prototype** | Internal demo, no real personal data | Preview URL, secrets out of the repo, basic error handling and logging |
| **B — MVP** | Real users, accounts, limited payments | Dev/preview/prod environments, CI, DB migrations, auth, monitoring, backup, rollback |
| **C — critical** | Money, health, children, sensitive data, B2B SLA | Staging, threat modeling, audit, DR plan, strict access, load and security testing |

A product that takes real payments from day one with multiple user roles,
personal dashboards, admin operations, and webhooks is **level C from the first
release** — staging is mandatory, preview environments complement but do not
replace it.

A level is dated to a trigger, not to a mood. Name the event that moves the
product to the next level (the first paying customer, the first real customer's
data, the first external user's personal data) and write that gate into the
architecture document, so the upgrade is a scheduled change rather than a
surprise found in an audit.

## The architectural default

A good default for a small product: one repository, **modular monolith**, one
managed relational database, server-first web, a clean API contract for mobile,
and an automated pipeline. Split a service out only with proven need:
independent release cadence, separate scaling, compliance boundary, or a
separate team.

Seven principles that survive every stack choice:

1. **Start with a modular monolith.** Separate code by business domain; don't
   create a service per screen. ("Monolith First" — Fowler; reinforced by
   Newman, *Building Microservices* 2nd ed., 2021.)
2. **One source of truth per entity.** Every user, order, subscription has one
   data owner. No "almost identical" parallel copies without a sync strategy.
3. **UI is not a security boundary.** A hidden button or disabled state never
   replaces server-side authorization. The server checks the right to every
   operation. (Next.js middleware bypass CVEs are the canonical proof — see
   `references/web-auth.md`.)
4. **Secrets live only on the trusted side.** API keys, DB credentials, and
   signing secrets never reach the browser bundle, mobile app bundle, git,
   analytics, or screenshots.
5. **Data changes are designed as repeatable.** Idempotency keys, unique
   constraints, and safe retries for payments, webhooks, and flaky networks.
6. **Failure is a normal scenario.** Every external integration gets a timeout,
   bounded retry, a clear error, a log entry, and a fallback or manual
   recovery path.
7. **Observability ships with the MVP.** Minimum: error tracking, structured
   logs with request/correlation IDs, key product events, alerts on critical
   failures.

## Recommended MVP shape

| Layer | Default | Why |
|---|---|---|
| Web UI | Next.js or similar full-stack framework; server-first, client only for interactivity | Smaller public API surface; secrets and data access stay on the server |
| Mobile UI | Flutter / React Native / native with feature-based modules | Separate client, one stable backend contract |
| Backend | Modular monolith + BFF/route handlers for web | Simpler deploys, transactions, debugging |
| Database | Managed PostgreSQL | Transactions, constraints, migrations, mature ecosystem |
| Files | Object storage + signed URLs | Never store binaries in git or on the runtime's local disk |
| Async jobs | Managed queue / cron / durable workflow | Email, imports, AI generation, webhooks out of the request lifecycle; platform-native workflows and queues now ship idempotency keys and consumers with no public URL |
| Auth | Managed identity provider, a proven library, or identity stored in the database itself | Don't reinvent password storage, reset flows, MFA, OAuth; database-resident identity branches with the data, so a preview environment gets its own users |
| Observability | Error tracking + logs + uptime + analytics | Detect and localize problems fast |

## Decisions that must not be made by fashion

| Question | Default | Change the default when… |
|---|---|---|
| Monolith or microservices? | Modular monolith | Independent releases/scaling/ownership are proven AND the team already runs observability + CI/CD well |
| REST or GraphQL? | REST / typed RPC | Many clients with genuinely different data shapes, and the team accepts the complexity/caching/auth cost |
| SQL or NoSQL? | PostgreSQL | A proven access pattern that genuinely doesn't fit the relational model |
| SSR/server-first or SPA? | Server-first hybrid | The product is a complex desktop-like tool and SEO/first-load matter less |
| Build or buy auth? | Buy / managed | Real competence, budget, and unique compliance/identity requirements exist |
| Realtime? | Polling / revalidation first | Low latency + high change frequency needed AND there's a plan for reconnect/ordering/backpressure |
| Kubernetes? | Managed platform / serverless / container service (edge platforms now run full containers and sandboxes and can reach a private network) | A real multi-service operational need and dedicated platform competence |

## Red flags — raise these immediately

- "We'll add staging/backup/logging/auth checks later."
- Production database reachable by developers via a shared password.
- A secret in frontend/mobile code because "nobody will see it anyway."
- Client sends `userId`/`role`/`price` and the server trusts it.
- Every screen talks directly to the DB or a third-party API.
- Microservices for a team of 1–3 without automated deploy and tracing.
- A migration drops/renames a column in the same release where old code may
  still run.
- Errors surface only as a toast and are logged nowhere.
- Webhooks processed without signature verification and deduplication.
- Access/refresh tokens in `localStorage` or plain mobile preferences without
  a risk analysis.
- No way to roll back a release or disable a broken feature.
- No owner and no review date for an architectural decision.

Deliver the red flags as a table with a where-it-is-guarded column pointing at
the invariant, the decision record or the subsystem that closes each one. A flag
with no guard named is still open, whatever the prose around it says.

## Reference map — read by domain

| File | Read when the task involves… |
|---|---|
| `references/web-auth.md` | Browser auth: sessions, cookies, SameSite, JWT, OAuth/OIDC, BFF, CSRF, token storage, Next.js server/client boundaries and middleware CVEs, API/mutation design |
| `references/mobile.md` | Mobile apps: layering, Keychain/Keystore token storage, refresh coordination, offline-first sync, store release realities |
| `references/delivery.md` | Environments, git flow, CI gates, artifact promotion, expand→contract DB migrations, infrastructure access |
| `references/payments.md` | Payment integration: idempotency, webhook verification, reconciliation, order/payment state, RBAC for refunds, multi-role products |
| `references/security-privacy.md` | Threat modeling (5 designer questions), authorization rules, data lifecycle, secrets management, OWASP standards versions |
| `references/reliability-observability.md` | External-service resilience, performance budgets (Core Web Vitals), accessibility as architecture, observability signals, failure-state UX |
| `references/checklists.md` | Pre-development checklist, PR checklist, production release checklist, ADR template, one-page memo |
| `references/sources.md` | The curated bibliography: what to cite, per topic, with authority grades and re-verification notes |

## How to answer

Match the depth to the question: a "where do I store the token" question gets
the pattern plus the reason plus the source, not the whole playbook. A "review
this spec" request walks the workflow above and reports findings ranked by
risk. When the user's plan contradicts a default, say so directly, explain the
cost, and offer the default as the alternative — do not silently comply, and do
not lecture beyond the decision at hand.

Before an answer that states a ⏳ fact goes out, check that it carries three
things: that the value moves, the date it was checked, and the page to re-check
it on (the row's canonical URL in the Volatile sources table). Checking the
page during the session does not replace this: the reader cites the fact
later, in a document that outlives the session.

When a product folder or repository is being created, the default document set
is: a guide (what the product is, settled decisions, the delivery workflow,
conventions), one decision record per decision, a PRD, a living
architecture document (stack with alternatives, subsystems, data, the numbered
invariants, the red-flag table with guards, the dated risk gate), and a design
rules file with its one machine-readable token file. Files the product and its
tools read carry the current contract; decision records carry the history.
