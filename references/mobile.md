# Mobile application architecture

Mobile is a **public client**: the app can be decompiled, traffic can be
inspected, the device can be lost. No secret embedded in the app bundle counts
as a secret.

## Recommended layers

| Layer | Responsibility |
|---|---|
| Presentation | Screens, UI state, navigation, accessibility, offline/error states |
| Domain | Use cases and business rules, independent of the UI framework |
| Data | Repositories, API client, local DB/cache, DTO ↔ domain mapping |
| Platform | Push, geolocation, camera, secure storage, background tasks, permissions |

## Tokens and secure storage

Standard: OWASP **MASVS v2.1** ⏳, especially MASVS-STORAGE
(no sensitive data in unprotected storage) and MASVS-AUTH (no local credential
checks that bypass the server). Categories: STORAGE, CRYPTO, AUTH, NETWORK,
PLATFORM, CODE, RESILIENCE, PRIVACY. Pair with MASTG v2.0 (https://mas.owasp.org/MASTG)
for concrete tests.

- Access token short-lived; refresh token rotated and stored in **iOS
  Keychain / Android Keystore-backed storage** — never plain
  preferences/DataStore/UserDefaults, files, or logs.
- Never log tokens, authorization headers, password-reset links, PII, or full
  API responses.
- **One refresh coordinator**: parallel 401s must not fire several refresh
  requests. Queue them behind a single refresh.
- On refresh-token reuse, the server blocks the token family/session and
  requires re-login (RFC 9700 rotation + reuse detection).
- Logout clears secure storage, local caches with personal data, and the
  server-side session.

## Offline-first without chaos

| Question | Rule |
|---|---|
| What is the source of truth? | Decide explicitly: server, device, or hybrid |
| What works offline? | A product-defined list; don't promise offline for a critical mutation without a sync strategy |
| How to sync? | Outbox queue + stable local IDs + idempotency keys + retry with backoff |
| How to resolve conflicts? | Per entity: server-wins, client-wins, merge, or manual |
| What does the user see? | Status: saved locally / syncing / failed / synced — with a retry affordance |

Note: authoritative literature on sync is thin; the strongest references are
Kleppmann (*Designing Data-Intensive Applications*) and Ink & Switch's
"Local-first software" essay. Treat vendor blog patterns with caution.

## Release realities

- The backend supports at least the current and several previous app versions.
  A breaking API change needs a compatibility window — you cannot force an
  instant client update.
- Remote config and feature flags enable features safely — not to dodge store
  review rules.
- Deep links / universal links are untrusted input — validate them.
- Push notifications are not a reliable data queue: on open, the app re-fetches
  current state.
- Request permissions in the context of the action, not all at first launch.

## Store billing and external purchases

Subscriptions no longer have to run through the store's billing on every
storefront: Apple allows link-outs to external purchase in the US without an
entitlement and charges a commission on alternative processing in the EU;
Google Play allows alternative billing and external links with a reduced
service fee plus a reporting obligation. The percentages and the storefronts
move by court ruling ⏳ (see `sources.md`). Design questions that do not move:

- Who is merchant of record for each storefront and each market: you, the
  store, or a merchant-of-record provider.
- How an entitlement is granted, verified and revoked when the purchase
  happened on the web and the app only learns about it from your backend.
- What the store must be told (transactions, installs) and by when, so the
  reporting job exists before the first external sale.
- One entitlement model in the backend for all three purchase paths, so the
  app never decides access from a local receipt alone.
