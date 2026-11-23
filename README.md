# production-ready-architecture

A Claude skill for reviewing architecture and making architecture decisions for web and mobile applications. It cites a graded list of sources (OWASP, IETF RFCs, MDN, web.dev, Stripe, Fowler and DORA, among others) and marks the facts that can go stale.

## Why it exists

An app built with an AI agent runs and gets its first paying customers. A demo doesn't show whether a stranger can use a paid feature without paying, where the payment key ended up, or whether a bad release can be rolled back. While the app works, a person who has never run a production system has no reason to ask.

The skill is for these situations:

- You build with an agent and have no way to check its architecture decisions. The skill scans the plan for red flags and raises each hit, including when you asked about something else.
- A contractor sent a spec or a proposal, and you have to approve it. You get findings ranked by risk, to take back as questions.
- The agent answers with confidence and names no source. Its training data ends at a cutoff date, so a version number or a draft's status may be old. Here each recommendation names where it comes from (an RFC, an OWASP cheat sheet), and the skill marks the facts that move.
- The advice doesn't fit the size of the product: microservices for a team of three, or prototype shortcuts in a product that takes payments. The risk level sets the required minimum, and leaving the default takes a stated reason.

## What it does

The skill sorts a product into one of three risk levels (prototype, MVP, critical) and scales its advice to that level. It starts from a default: one repository, a modular monolith, one managed relational database, server-first web, a stable API contract for mobile and an automated pipeline. Any deviation from it needs a stated reason and a record.

It checks a plan or a spec against a list of red flags, such as tokens in `localStorage`, webhooks processed without signature verification, or a migration that drops a column in the same release where old code may still run, and reports the findings ranked by risk. A short question ("where do I store the token") gets the pattern, the reason and the source, under a stated assumption when the context is missing.

## Who it's for

Product designers and product builders who make or check architecture decisions for web and mobile apps, or who need a second read of a technical spec or a contractor's proposal.

## Install

Copy the `production-ready-architecture/` folder into your skills directory: `~/.claude/skills/` for every project, or `.claude/skills/` inside one project. Claude loads the skill when a request matches its description, for example "is this spec production ready" or "we are adding payments".

## What's inside

- `SKILL.md`: the workflow, the risk levels, the defaults, the red flags and the map of reference files.
- `references/web-auth.md`: browser auth, cookies, CSRF, JWT, token storage, Next.js server and client boundaries.
- `references/mobile.md`: mobile layers, Keychain and Keystore storage, refresh coordination, offline sync, store billing.
- `references/delivery.md`: environments, CI gates, artifact promotion, expand and contract migrations, infrastructure access.
- `references/payments.md`: idempotency, webhook verification, reconciliation, order and payment state, multi-role products.
- `references/security-privacy.md`: threat modeling questions, authorization, data lifecycle, secrets, CSP, OWASP standard versions.
- `references/reliability-observability.md`: external-service resilience, Core Web Vitals, accessibility, observability signals, failure states.
- `references/checklists.md`: pre-development, pull request and release checklists, an ADR template, a one-page memo.
- `references/sources.md`: the bibliography with authority grades, and the table of volatile sources.

## How the facts stay current

Facts that move (draft statuses, CVE lists, standard versions, plan limits) carry a ⏳ mark. The last section of `references/sources.md` lists each of them with its canonical URL and the exact value the skill states, so a check compares one value with one page. Re-check the row before you cite a marked fact in a client-facing document. `CHANGELOG.md` records which rows were checked and when.

## What it is not

It is not a penetration test, and it is not legal or compliance advice. It does not cover visual or UI design or copywriting.

## License

MIT, see [LICENSE](LICENSE).
