# Environments, git, and delivery to production

## Minimal environment model

| Environment | Purpose | Data | Access |
|---|---|---|---|
| Local | Development on the machine | Seed/fake data | Developer |
| Preview / PR | Verify a specific branch/change | Isolated test data | Team/stakeholders |
| Development (shared) | Integrate unfinished features | Test data | Team |
| Staging | Production-like check of a release candidate | Synthetic or anonymized data | Restricted team |
| Production | Real users | Real data | Least privilege |

Rule: a product that ships real payments, multiple roles, dashboards, admin
operations, webhooks, and email/push from the first release needs **staging
from day one**. Preview environments complement staging, they don't replace it.
Staging matches production in topology and configuration, not in size — only
credentials, data, domains, and limits differ, never the build method.

For a static site or level-A prototype, preview-per-PR alone is a sufficient
verification contour — don't build staging ceremony where there's nothing to
stage.

Database platforms with branching give a preview its own copy of the data,
and on some of them the auth users and the object-storage bucket branch with
it, so "isolated test data" per PR is the default rather than a chore. A
schema change made on a branch through the platform's dashboard is not the
pipeline: the merge diff engines are new (one is labelled alpha), so
migrations still come from versioned files applied by CI, and the branch is
where you verify them, not where you author them.

## Promotion rule

One and the same commit/artifact passes checks and moves through environments.
Never hand-rebuild "roughly the same" code for production.

- Feature branch → Pull Request → automated checks → preview review → merge.
- Main is always deployable.
- Production deploy is automated; manual steps are minimal and documented.
- DB migrations are versioned and applied by the pipeline.
- Every release has a rollback/roll-forward plan.
- Feature flags separate deploy from release for risky features (Hodgson,
  "Feature Toggles"; DORA capabilities).

## CI gates

| Gate | What it checks |
|---|---|
| Install + build | Reproducible build with lockfile; Node pinned by `.nvmrc` to the current LTS line (Node now ships one major a year and every release becomes LTS, so "pick the even number" no longer means anything) |
| Type check | TS/Dart/Kotlin/Swift contracts intact |
| Lint + formatting | Baseline quality and consistency |
| Unit tests | Business rules and transformations |
| Integration tests | DB, repositories, API, auth boundaries |
| E2E smoke | Login, key happy path, payment/creation of the core object |
| Migration check | Migrations apply to an empty and to the previous schema |
| Dependency/secret scan | Known vulnerabilities and committed secrets |
| Browser-API guard | Cross-browser tests plus a Baseline lint rule (e.g. `eslint-plugin-baseline-js`) for web APIs used as progressive enhancement; the type system is not the guard, it lags the browsers and pushes people to `as any` |

## Backward-compatible migrations: expand → migrate → contract

The canonical pattern (Fowler "Parallel Change" / Evolutionary Database Design;
Ambler & Sadalage, *Refactoring Databases*; GitLab and PlanetScale document
production variants):

1. **Expand** — add the new column/table; old code ignores it.
2. **Migrate/backfill** — write to both, backfill history, switch reads.
3. **Contract** — remove the old column **in a separate release**, once no
   running version uses it.

Operational rules:

- Migrations run once, by a controlled job — not concurrently by every
  instance.
- Foreign keys, unique and check constraints protect invariants — the
  application code alone is not enough.
- Primary keys are not a security mechanism; ownership checks are mandatory.
- Separate DB per environment. Never a shared staging/production database.
- Point-in-time recovery / automated backups on; **restore is tested
  periodically** — a backup that has never been restored is a hypothesis.
- Multi-tenant products decide `tenant_id` and the isolation strategy up
  front.

## Infrastructure and access

- Infrastructure and environment config described as code or reproducible
  platform configuration.
- Dev/staging/prod use different accounts/projects and credentials.
- Production access: least privilege, MFA, personal accounts, no shared
  passwords.
- Deploys run under a CI identity, not a developer's personal token.
- Secrets manager, key rotation, access journal. On Vercel every environment
  variable is typed Config or Secret (`vercel env add NAME --visibility
  secret`; Secret values are redacted in build logs), Vercel Connect issues
  short-lived scoped provider tokens at runtime instead of a stored provider
  secret, and deployment protection for production costs nothing on any plan,
  so it is on.
- Production DB unreachable from the public internet — private
  network/proxy/bastion as the platform allows.

## Delivery consensus (why this is the default)

Trunk-based development, PR preview deployments, immutable artifact promotion,
and deploy/release separation via flags are backed by DORA's research
(Accelerate; annual State of DevOps reports; DORA's five software delivery
metrics: deployment frequency, lead time for changes, change failure rate,
failed deployment recovery time, deployment rework rate) and Google SRE
practice — teams
with these capabilities ship faster *and* break less. This is the rare place
where the empirical evidence is strong; treat deviations as burden-of-proof
territory.
