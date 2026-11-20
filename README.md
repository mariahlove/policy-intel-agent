# policy-intel-agent

A personal policy-intelligence agent. It watches ~60 government sources around the clock, catches new policy drops the moment they're out, detects substantive changes against previous versions, and drafts one-page briefing notes in a fixed house format.

Built with Devin. Building in public — this repo is the build log.

## The brief format

Every brief follows a fixed structure:

- **Headline** — what dropped, one line
- **Source + publish time**, with the original link
- **What changed** — 3 lines max, diffed against the previous version of the policy where there is one
- **Why it matters** — one line on the concrete impact: a funding window, a compliance deadline, an eligibility change
- **Action tag** — FYI / respond / apply, with the deadline date if there is one

Delivery is two-tier: an instant alert when a tier-1 source drops something, plus a 07:30 daily digest of everything else, deduped against a seen-store so nothing is ever briefed twice.

## Architecture

A monorepo in five modules:

- `sources/` — the registry. One YAML file per source: url, feed type (rss / html / api), priority tier, crawl interval, parser hints. Adding a source is a config file, not new code.
- `fetch/` — the fetcher framework. httpx first, Playwright fallback for JavaScript-heavy portals, per-source adapters only where a layout demands one. Official feeds preferred, polite crawl rates, retry with backoff.
- `detect/` — change detection. Boilerplate stripping, content hashing, structural diff against the last snapshot, then an LLM pass that scores whether a change is substantive or cosmetic — the part that keeps it from crying wolf.
- `draft/` — the drafting layer. Fixed templates in the house format above, generated in Chinese, deduped against the seen-store (Postgres) so no item is ever briefed twice.
- `deliver/` — two-tier output: instant alerts for tier-1 sources, the 07:30 digest for the rest, and a Markdown archive of every brief.
- `ops/` — scheduler, queues, and fetcher health monitoring: when a portal changes its layout and a parser breaks, the operator hears about it before the digest does.

## Build plan

- Week 1: source registry + first 15 fetchers live
- Week 2: change detection + dedup
- Week 3: drafting layer + 07:30 digest
- Week 4: instant alerts + hardening

## Status

Phase 1 in progress. The registry currently holds the first core sources (see `sources/registry.yaml`) and grows to ~60.
