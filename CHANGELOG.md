# Changelog

## [Unreleased]

### Changed

- The skill's description now names performance, accessibility and observability baselines (Core Web Vitals thresholds, WCAG level, current standard versions), so a question about them loads the skill.
- An answer that states a volatile fact now says that the value moves, gives the date it was checked and names the page to re-check it on, also when the page was checked during the session.
- The README gained a "Why it exists" section: the situations the skill is for, before the description of how it works.

### Sources checked on 2026-11-16

The sources below were checked for this release against the page named in the skill.

- **Next.js middleware and proxy advisories** (`references/web-auth.md`, the bibliography and the Volatile sources row): all six CVE ids, months and fixed versions against their GitHub advisory pages. CVE-2026-64642 is fixed in 16.2.11; its advisory lists no affected 15.x line, so the 15.5.21 fix line was removed. No newer advisory of this class was found. The bibliography now lists all six with their GHSA ids and URLs.
- **Next.js remote code execution advisories** (`references/web-auth.md`): the advisory index lists Critical RCEs published after the ones the skill named, in `next/og` ImageResponse and in the Image Optimization API when SVG files are used. Both are now named with their GHSA ids; their fixed versions were not checked.
- **Next.js authentication guide** (`references/web-auth.md`): the two quoted sentences are verbatim on the live page; its URL now stands beside the quote.
- **OWASP ASVS** (Volatile sources row): the "What is the ASVS" page of the 5.0 text states 17 chapters; 5.0.0 is the latest numbered release. The row now carries the page's real URL.
- **Sentry** (Volatile sources row): the pricing page includes logs on the free Developer plan. The row now carries the page's URL.
- **Stripe webhooks and PCI SSC SAQ A** (bibliography): the two short phrases attributed to them are paraphrases, not quotations, and no longer stand in quotation marks.
- **Every URL in the folder**: given an `https://` scheme and checked for status. Two addresses that no longer resolved were replaced: the Feature Toggles article now carries its `.html` address, and the Android entry now points at the Keystore system page.

The values in the other Volatile sources rows were not re-checked for this release; only their URLs were.
