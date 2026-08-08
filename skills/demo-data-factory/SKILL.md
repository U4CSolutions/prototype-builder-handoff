---
name: demo-data-factory
description: Patterns for seeding large, realistic, deterministic synthetic datasets into prototypes — labeled stock content with procedural fallback, seeded PRNG catalogs, lifecycle variety, clean teardown. Use when a prototype needs demo content that exercises every feature.
---

# Synthetic demo-data factory

Every feature works against **fake but realistic** data (see `METHODOLOGY.md` principle 1),
clearly labeled as synthetic, removable without touching real user rows.

## Core patterns

1. **Determinism**: seed everything from a seeded PRNG (e.g. a small `mulberry32(seed)`-style
   generator) keyed per record (`seed = recordIndex * <large prime>`). Reseeding reproduces the
   identical dataset — screenshots and automated tests stay stable across runs.
2. **Template catalogs**: per-category templates (name/attribute pools, realistic-range value
   generators) → programmatic expansion to hundreds of records with realistic variance (weighted
   distributions, jitter within a plausible range, category-appropriate trends). Hand-write a
   handful of "hero" records with rich, specific stories; generate the long tail programmatically.
3. **Lifecycle variety**: deterministically assign a spread of states (active, archived,
   completed, overdue, in-progress, etc.) so every status-dependent UI, alert, and analytic has
   real data to render against — not just the happy-path state.
4. **Real-feeling media with a fallback**: if the product needs images, a keyword-matched stock
   photo service with a deterministic seed parameter is a reasonable source; on any fetch failure
   (rate limit, network, missing keyword match), fall back to procedurally generated placeholder
   art rather than leaving a broken image. **Label every synthetic asset visibly** (a caption
   strip reading something like "STOCK PHOTO — demo content" or "ILLUSTRATIVE RENDER") so nobody
   mistakes generated/stock content for real user content. Keyword-match accuracy from a stock
   source is inherently imperfect — that's an acceptable, labeled tradeoff, not a bug to chase.
5. **Synthetic documents**: any generated document (receipts, certificates, reports) should
   carry an equivalent "SYNTHETIC — generated for demo mode" marker.
6. **Tagged, safe teardown**: every seeded row carries an explicit marker (a boolean/tag column,
   a naming convention, whatever fits the schema). A clear/teardown routine deletes *only* marked
   rows and their generated files; foreign-key cascades cover child rows automatically if the
   schema is set up for it. The seed routine should refuse to run (or be a safe no-op) if markers
   already exist, so re-running it doesn't duplicate data.
7. **Simulated integrations are deterministic too**: third-party API responses in demo mode
   should be deterministic mocks (seeded the same way as the rest of the dataset), explicitly
   flagged as demo-sourced in their response shape, with a documented switch to go live (see
   `CONNECTIONS.md` guidance in `HANDOFF-FORMAT.md`). Never require real credentials for a
   feature to be demoable.

## Timing expectations

Set realistic expectations for seed time once external fetches (stock photos, etc.) are involved
— a seed of a few hundred records including remote asset fetches at modest concurrency can
reasonably take tens of seconds. Keep the seed operation's own request/response synchronous and
simple; show a progress/loading state in the UI rather than trying to make the seed itself
instant.
