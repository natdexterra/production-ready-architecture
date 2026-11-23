# Checklists and templates

Use these verbatim at the matching gate. Items that don't apply to the
product's risk level may be struck — explicitly, not silently.

## Before development starts

- [ ] Risk level A/B/C determined.
- [ ] Core domains described; a data owner named for every key entity.
- [ ] Default chosen: modular monolith; reasons for any deviation recorded.
- [ ] Context diagram exists: users, web/mobile, backend, DB, third parties.
- [ ] Auth + authorization model, roles, ownership rules defined.
- [ ] Environments defined (dev/preview/staging/prod) and what data each may
      hold.
- [ ] Key failure/offline/empty states identified.
- [ ] Minimal monitoring and product events defined.
- [ ] Migration, backup, and rollback strategy exists.
- [ ] For personal data: purpose, retention, deletion defined.

## Pull Request / preview

- [ ] Build, type check, lint, tests pass.
- [ ] No secrets, test credentials, or production data in the diff.
- [ ] Server-side authorization verified; no UI-only guards.
- [ ] Input validated; output carries no extra fields.
- [ ] New errors have a clear UI and are logged.
- [ ] Loading/empty/error/offline states included.
- [ ] DB migration backward-compatible and tested.
- [ ] Analytics events carry no PII and have a stable schema.
- [ ] Accessibility: keyboard, focus, labels, contrast, dynamic states.
- [ ] Preview checked on a real mobile viewport/device.

## Production release

- [ ] Release candidate is the same verified commit/artifact.
- [ ] Feature flag / rollout plan defined for risky features.
- [ ] DB backup/restore readiness confirmed for dangerous migrations.
- [ ] Rollback or roll-forward steps written down.
- [ ] Dashboards and alerts ready **before** the feature goes live.
- [ ] Login, core flow, permissions, and critical integrations verified.
- [ ] Post-deploy smoke test done.
- [ ] Error rate / latency / business KPI watched through the observation
      window.
- [ ] A named person owns the stop/continue/rollback decision.

## Definition of Done for infrastructure

Infrastructure is done not when "everything works", but when: a change travels
the same path through CI and staging; rights are checked by the server; a
payment can be safely retried and reconciled; a failure is visible in
monitoring; data restores from backup; a release can be disabled or rolled
back; and a load increase doesn't require rewriting the product.

## Architecture Decision Record (mini-template)

Fill for decisions that would be expensive to change.

| Field | What to write |
|---|---|
| Title | The decision in one line: "Use BFF and HttpOnly session cookie for web" |
| Date & status | Proposed / Accepted / Superseded |
| Context | Problem, constraints, risk, scale, team |
| Decision | What exactly and where the boundary runs |
| Alternatives | 2–3 real options and why not |
| Consequences | Pros, cons, new obligations, cost |
| Review trigger | Metrics/conditions that reopen the decision |
| Owner | Who keeps it current |

## One-page memo

Default: modular monolith + managed PostgreSQL + server-first web + stable
mobile API + managed auth + preview deployments + CI + monitoring.

- No browser tokens in localStorage for a sensitive product.
- Web: HttpOnly Secure cookie and server-side session/BFF is the safe default.
- Mobile: short access token, rotated refresh token, Keychain/Keystore.
- Every operation checks authorization on the server.
- One commit travels preview/staging → production.
- Migrations backward-compatible; backup and rollback exist before release.
- No microservices without a proven reason and operational maturity.
- Errors, logs, metrics, and failure-state UX are designed with the happy
  path, not after it.
