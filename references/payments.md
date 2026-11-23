# Payments and multi-role products

## Non-negotiable payment rules

- Price, currency, discount, and the right to buy are computed/verified
  **server-side**; client values are never the source of truth.
- Payment and order creation are protected by an **idempotency key** (Stripe:
  UUID-class randomness, POST requests, keys retained ~24h ⏳; the IETF
  `Idempotency-Key` header draft expired without becoming an RFC — vendor
  conventions rule here).
- Webhooks are verified by **cryptographic signature** (Stripe:
  `Stripe-Signature`, HMAC-SHA256 over the raw body + timestamp against
  replay; Adyen: HMAC) and **deduplicated by event ID**. Trap: during a
  migration from snapshot to thin events, the two deliveries of one change
  carry different ids, so the dedup key is `event.snapshot_event` when it is
  present and `event.id` otherwise.
- Return 2xx quickly, process asynchronously — heavy work happens off the
  webhook request.
- **A successful user redirect is not payment confirmation.** Truth comes from
  the provider API/webhook.
- Payment and order states are separate: pending, requires_action, paid,
  failed, refunded, disputed.
- A **reconciliation job** periodically compares local records with the
  payment provider — drift happens and must be found by the system, not by an
  angry customer.
- Refunds and manual admin operations require RBAC, a reason, and an audit
  log entry.
- Sandbox and production payment accounts/keys are physically separated.
- Never store card data — hosted/tokenized provider integration (Stripe
  Elements/Checkout, Adyen Drop-in) keeps you in PCI SAQ-A territory.
- A merchant-of-record service (the provider becomes the seller of record for
  tax, fraud and disputes, configurable per market or product) removes tax
  registration and dispute handling from this list; idempotency, webhook
  verification and reconciliation stay yours.
- Agent-initiated purchases: the Agentic Commerce Protocol hands an agent a
  delegated payment token with a monetary cap and an expiry, and the seller
  stays merchant of record. The spec has broken compatibility three times in
  four months, so integrate it behind an adapter and expect the
  payment-handler shape to change.

## Ready-before-first-real-payment checklist

- Staging fully works and mirrors production topology.
- Sandbox payment flow verified end-to-end: to the webhook and the granted
  entitlement.
- Production keys and webhook endpoints created separately.
- Audit log covers refunds, role changes, manual corrections.
- Monitoring + alerts on payment/webhook failures.
- Reconciliation exists, plus a safe way to recover a stuck order.
- Backup and tested migration rollback/roll-forward.
- Terms/Privacy/consent and data minimization in place.
- Support can find a transaction by a safe internal ID without seeing card
  data.

## Two roles, two dashboards (marketplace-shaped products)

| Level | How to design it |
|---|---|
| Identity | One user may hold several roles/relationships with organizations; don't duplicate accounts |
| Authorization | RBAC for general rights + ownership/relationship checks per object; policy-based access for complex rules |
| Routing/UI | Separate apps or route groups — a UX decision, never the security boundary |
| Data access | Every query filters by tenant/owner/relationship server-side; IDOR/BOLA covered by tests |
| Admin impersonation | Only when necessary, with a visible banner, restrictions, and a full audit trail |

## Domain modules worth separating early

Identity & Access · role-specific profiles · Catalog/Offering ·
Orders/Subscriptions (with price snapshots) · Payments (isolated from UI) ·
Notifications (templates, delivery status, retry) · Content/Marketing forms
(leads, consent) · Admin/Audit (privileged actions, immutable trail).

## Scaling without premature microservices

| Stage | Step |
|---|---|
| Start | Stateless API instances, managed DB, CDN, object storage, queue, workers |
| Read growth | Indexes, query optimization, cache, read replicas — only per metrics |
| Job growth | Separate worker pools and queues per load type |
| Media growth | Async processing, CDN, lifecycle policies |
| Team growth | Module ownership, contract tests, CODEOWNERS, independent deployables when needed |
| Service extraction | First define data ownership and the event/API contract; then strangler migration, never big-bang rewrite |
