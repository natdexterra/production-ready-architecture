# Reliability, performance, observability, accessibility

## External services

- Timeouts are mandatory; "wait forever" is not an option.
- Retry only safe/idempotent operations; exponential backoff + jitter.
- Circuit breaker / temporary disable for a systematically failing
  integration at high risk levels.
- Webhook signatures verified; event IDs deduplicated; processing async.
- The user sees an honest degraded state, not an infinite spinner.

## Performance budgets

| Area | Practice |
|---|---|
| Web | Minimize client JS; lazy-load heavy things; sized+optimized images; measure Core Web Vitals |
| API | No N+1; indexes match real queries; pagination; bounded payloads |
| Mobile | Don't block the main thread; cache deliberately; mind battery/network/background work |
| AI features | Async jobs for long tasks (idle compute may be free now, the user still does not wait); token/cost limits, starting with the AI gateway's own budget caps before building counters; cancellation; caching; human-readable errors |

**Core Web Vitals, 2026 set** (web.dev; 75th percentile, mobile and desktop
separately): **LCP ≤ 2.5 s · INP ≤ 200 ms · CLS ≤ 0.1**. INP replaced FID on
March 12, 2024 — any audit template still pulling FID is outdated. ⏳ The set
evolves; verify at https://web.dev/articles/vitals.

## Accessibility as an architectural requirement

Standard: **WCAG 2.2** (W3C Recommendation, current text dated 12 December
2024; 9 new criteria including focus appearance, target size, accessible
authentication). WCAG 3.0 is a Working Draft, not a target. ARIA APG for
widget patterns.

- Semantic HTML / native controls before custom components.
- Keyboard/focus flow and screen-reader labels are part of the component API.
- Errors are programmatically tied to fields; color is never the only signal.
- Dynamic content announces itself; modals manage focus.
- The design system ships accessibility states — screens don't improvise them.

## Observability

| Signal | What must exist |
|---|---|
| Errors | Stack trace, release/version, environment, route, safe user/session reference |
| Logs | Structured events, severity, timestamp, request/correlation ID; no secrets/PII. Error-tracking vendors now include structured logs on free plans, so this needs no second vendor from the MVP |
| Metrics | Latency, error rate, throughput, queue depth, DB connections, external-service failures |
| Tracing | For distributed/complex flows — request path across services and jobs (W3C Trace Context; OpenTelemetry as the vendor-neutral default) |
| Product analytics | Events describe user outcomes; each has an owner and a schema/version |
| Alerts | Actionable only: who reacts, what threshold, link to a runbook (SRE "four golden signals": latency, traffic, errors, saturation) |

## Failure states are design work

Every critical flow needs, beyond the happy path: loading, empty, partial,
stale, permission denied, offline, timeout, conflict, retrying, failed
permanently, and recovery. If the designer doesn't draw them, the developer
improvises them at 2 a.m. — and the improvised version is a toast nobody logs.

Durable-workflow platforms keep the state of a failed instance for a limited
window (one platform's default is seven days); copy what a post-mortem needs
into your own logs before it expires.
