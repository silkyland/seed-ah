# Realism Guide

"Fake but production-like" is a discipline. Ten identical "Test User"
rows fail it; so does a database where everything happened at 14:02 today.

## Identities

- **Locale-correct**: Thai product → Thai names in Thai script (with
  realistic romanized emails), German product → German names. Use the
  stack's Faker with the right locale (`fake('th_TH')`, `Faker\Factory::create('th_TH')`,
  `faker.locale`) — a Somchai in a Thai UI reads real; a "John Smith"
  reads like a template.
- **Emails on a reserved demo domain**: `somchai.p@demo.example` —
  `.example`/`example.com` are IETF-reserved (no real inbox can exist),
  and the domain doubles as the wipe marker.
- **No real people**: never seed names of actual users, colleagues, or
  celebrities.

## Demo accounts (the ones the manifest reports)

One account per role the app defines (read the roles from code/DB, cite),
plus one "rich" account whose data tells a story for demos/screenshots:

| Rule | Why |
|------|-----|
| One account per role (admin, editor, member, …) | every permission level is demoable |
| Passwords: readable, unique to this seed, policy-compliant — e.g. `Demo-Orchid-2026!` style | typeable in a demo, obviously fake, actually logs in |
| Hash through the app's own hasher (cite `file:line`) | bcrypt/argon2 config differences WILL lock you out |
| The "rich" account has history: many entities, mixed statuses, old + new timestamps | screenshots and demos need a lived-in view |
| One deliberately empty account | empty states are part of the product too |

## Distributions — the texture of real data

- **Power law, not uniform**: a few heavy creators, a long tail of
  occasional ones, some zero-activity ghosts. `min(pool) of random picks`
  or a weighted choice gets close enough.
- **Timestamps spread like history**: created_at over the last 6–18
  months, denser recently; activity clusters in business hours of the
  product's timezone; updated_at ≥ created_at (obvious, and yet).
- **Statuses cover the whole enum** — including cancelled/rejected/banned.
  A demo DB where everything succeeded can't demo the product's error
  handling.
- **Numbers in plausible ranges** read from context (prices that match the
  product's market, quantities that make sense) — and totals that ADD UP
  where the schema stores both parts and sums (order total = sum of lines;
  seed the parts, compute the sum).

## Deliberate edge cases (small volume, always present)

- Longest plausible name/title; one name with an apostrophe or hyphen.
- Unicode beyond ASCII in every text field family (Thai, emoji where the
  app allows it).
- Empty optional fields on some rows; maximal fill on others.
- One entity right at a boundary (0 items, max items, price 0).

Tag edge-case rows in the manifest so demo drivers can steer toward or
away from them.

## Framework-native mechanisms (use the repo's own)

| Stack | Mechanism |
|-------|-----------|
| Laravel | model factories + `database/seeders/`, `php artisan db:seed` |
| Symfony | Foundry factories + DoctrineFixturesBundle |
| Django | factory_boy + management command (fixtures for lookups) |
| Rails | `db/seeds.rb` + FactoryBot |
| Node/Prisma | `prisma/seed.ts` + `@faker-js/faker` |
| Plain SQL | last resort — loses model-layer hashing/validation; say so |

Follow an existing seeder/factory in the repo if one exists (`file:line`)
— match its style. **Fixed random seed** (`Faker::seed(20260706)` etc.)
so every run builds the same demo world: re-runnable screenshots,
diffable data, reproducible bugs.

## Idempotency & the wipe

- Seeder is re-runnable without duplicating: upsert on the deterministic
  keys (demo emails, `demo-` slugs) or wipe-then-seed inside a
  transaction.
- The unseed script deletes by the marker (email domain, slug prefix, tag
  column) child-tables-first — and prints what it deleted.
