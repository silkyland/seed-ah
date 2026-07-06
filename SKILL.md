---
name: seed-ah
description: >-
  Seeds a database with fake-but-production-like demo data: reads the real
  DB structure (migrations, ORM models, live introspection), builds a
  dependency-ordered seeding plan using the framework's own seeder
  mechanism, generates realistic locale-correct data (spread timestamps,
  power-law distributions, deliberate edge cases), AI-generates or
  programmatically renders images for image fields, and outputs a seed
  manifest summarizing every table seeded plus a demo-accounts table with
  username, password, and role for each. Hard production gate: verifies
  the target database before writing a single row, and every seeded row is
  wipeable. Use when the user asks to seed demo/test/sample/fake data,
  create fixtures or demo accounts, populate a dev database, or mentions
  seed-ah or /seed-ah.
license: MIT
argument-hint: "[project-root] [volume: S/M/L] [locale]"
---

# Seed-ah!

Empty databases make every screenshot, demo, and dev session lie — and
seeding with "test test test" rows lies differently. This skill seeds data
that *looks like production*: real-shaped names in the right locale,
timestamps that spread like history, distributions with heavy users and
ghosts, images in the image fields — all fake, all wipeable, all
documented in a manifest that tells you exactly who the demo users are.

## The Prime Directive (family rule)

> **The schema is read, never assumed — and production is never touched.**
> Every column seeded exists in evidence (migration/model/introspection,
> cited). The target database is verified and confirmed by the user before
> the first row is written. Every seeded row can be identified and wiped.

## Hard safety rules

1. **Production gate ⛔** — before seeding, resolve the actual connection
   (env/config, `file:line`), print host + database name + environment to
   the user, and require explicit confirmation. Names that smell like
   production (`prod`, a public host, a managed-DB hostname) require the
   user to type the database name back. When in doubt, refuse and suggest
   a local copy.
2. **Wipeable by construction** — seeded rows are identifiable (a
   deterministic marker: known email domain like `@demo.example`, a
   `demo_` slug prefix, or a seed-run tag column where one exists) and an
   **unseed script** ships alongside the seed script.
3. **Demo passwords are demo** — obviously fake, unique to this seed
   (never a real credential, never reused from anywhere), hashed through
   the app's OWN hasher (`file:line` of the hashing call/config). Policy-
   compliant so login actually works.
4. **No real humans** — no real names/emails/photos of actual people.
   Generated faces are avoided too (likeness risk); use initials-avatars
   or illustrations instead.

## Progress checklist

Copy this into your response and check items off:

```
Seed-ah Progress:
- [ ] Step 1: Frame — target env confirmed ⛔, volume, locale, purpose
- [ ] Step 2: Schema census — tables, constraints, FK graph, seed order
- [ ] Step 3: Realism plan — per-table shape, distributions, edge cases
- [ ] Step 4: Seed scripts — framework-native, committed, idempotent
- [ ] Step 5: Images — generated for every image field, correct sizes
- [ ] Step 6: Run + verify — counts, FK integrity, demo logins tested
- [ ] Step 7: Manifest — what was seeded + demo accounts (user/pass/role)
```

## Step 1 — Frame

Volume (S: enough for screenshots / M: realistic dev / L: pagination and
performance testing), locale for the fake data (Thai names for a Thai
product — locale mismatch screams "fake"), and purpose (feeding
love-me-love-my-docs screenshots? a customer demo? dev fixtures?). Then
run the **production gate** from the hard rules — nothing proceeds past
this step without the confirmation.

## Step 2 — Schema census

Read the structure from evidence, per
[references/schema-census.md](references/schema-census.md): migrations and
ORM models first (they carry intent: enums, defaults, validation), live
introspection to confirm (information_schema / SHOW CREATE TABLE / PRAGMA),
FK relationships → a dependency graph that dictates seed order (parents
before children), and every constraint that can reject a row (NOT NULL,
UNIQUE, CHECK, enum values). Note which tables the app writes vs
reference/lookup tables that may already be populated.

## Step 3 — Realism plan

Per table, decide what "production-like" means — rules in
[references/realism-guide.md](references/realism-guide.md): locale-correct
fake identities, timestamps spread over months with plausible rhythms,
power-law distributions (a few heavy users, a long tail, some zeros),
statuses covering the whole enum including the ugly ones (cancelled,
banned, pending), and **deliberate edge cases** (longest plausible name,
unicode, empty optionals) — because demos eventually meet them.

## Step 4 — Write the seed scripts

Use the framework's blessed mechanism — Laravel seeders/factories, Symfony
Foundry, Prisma seed, Django fixtures/factory_boy, plain SQL as last
resort — cited from the project's own conventions (`file:line` of an
existing example if one exists). Scripts are committed (`database/seeds/`
or the stack's home), deterministic where possible (fixed random seed so
re-runs produce the same demo world), and paired with the unseed script.

## Step 5 — Images and media

For every image/file column found in the census, produce real files at
the sizes the app expects — options and rules in
[references/images-and-media.md](references/images-and-media.md): AI
image generation when tooling is available, deterministic programmatic
rendering (brand-colored covers, initials-avatars) as the always-works
fallback. Never scraped/copyrighted images, never real people. Store them
where the app actually serves media from (`file:line` of the storage
config).

## Step 6 — Run and verify

Run the seed. Then verify like it matters:

- Row counts per table match the plan.
- FK integrity: zero orphans (query it, don't assume it).
- **Log in as every demo account** through the app's real auth (HTTP or
  the app's test client) — a demo account that can't log in is a seed
  failure, report it as such.
- Spot-render: one page per major entity shows seeded data without errors.

## Step 7 — The manifest

Write the summary per
[references/manifest-template.md](references/manifest-template.md) to
`SEED_MANIFEST.md` (repo root or docs/): per-table seeded counts and
shape notes, the **demo accounts table — username/email, password, role,
what that account is good for demoing** — image inventory, the exact
re-seed and wipe commands, and the marker that identifies seeded rows.
Warn in the file header: it contains demo credentials by design — keep it
out of public repos unless the database is disposable.

End by reporting the manifest inline: what was seeded (counts), the demo
accounts (user / pass / role), and the one-command re-seed and wipe.
