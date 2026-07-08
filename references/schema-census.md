# Schema Census Guide

Read the database's real shape before inventing a single row. Three
sources, in order — they disagree more often than you'd think, and the
disagreements are findings.

## Census Questions — answer these before inventing data

The census exists to answer this list, not to read files for their own
sake. Every answer cites evidence (`file:line` or pasted introspection
output). A question you cannot answer from evidence becomes a named
**probe** — insert one throwaway row inside a transaction, observe what
the DB does, roll back — or that table waits. Append questions you
discover mid-census; the census is complete when every question is
answered or probed.

1. Which tables does the seed fill, and which are lookup/reference
   tables already owned by a migration or existing seeder (cite the
   owner — never double-seed)?
2. What is the full FK graph and the resulting seed order? Any cycles
   or polymorphic relations, and how does each get handled?
3. For every column to be filled: nullable? default? UNIQUE? CHECK/enum
   values? Validation stricter than the DB?
4. What fires on insert into each table — DB triggers, ORM
   events/observers, queued jobs, webhooks, mail? (See "What fires on
   insert" below.)
5. What already lives in the target — pre-seed row counts per table, and
   do any existing rows match the planned wipe marker? (Must be zero.)
6. How are passwords hashed, and where is the hashing call (`file:line`)?
7. Where does the app store and serve media from (`file:line`), and does
   the DB hold paths, URLs, or IDs into a media table?

## Source 1 — Migrations & ORM models (intent)

Migrations and models carry what raw columns can't: enum meanings,
validation rules, casts, relationships, soft-delete conventions, mutators
that hash or normalize on write. Cite `file:line` per fact. Note
especially:

- Columns the ORM fills automatically (timestamps, UUIDs, slugs) — the
  seeder should go THROUGH the model layer to get them for free.
- Validation stricter than the DB (`file:line`) — seed to the app's rules,
  not just the schema's, or the data works in SQL and breaks in forms.

## Source 2 — Live introspection (reality)

```sql
-- MySQL/MariaDB
SELECT table_name, column_name, data_type, is_nullable, column_key
FROM information_schema.columns WHERE table_schema = DATABASE();
SHOW CREATE TABLE <table>;         -- FKs, CHECKs, charset
-- PostgreSQL: \d+ <table> / information_schema + pg_constraint
-- SQLite: PRAGMA table_info(<table>); PRAGMA foreign_key_list(<table>);
```

Reality wins over intent when they conflict (a migration that never ran, a
hand-edited column) — record the drift as a finding for the user.

## Source 3 — Existing data (texture)

If the dev DB already has rows (even a handful), sample them read-only:
they reveal actual formats (phone patterns, slug styles, JSON shapes)
that no schema documents. NEVER copy the rows — copy the *shape*.

## The dependency graph

From the FK map, compute seed order: parents before children
(users → articles → comments). Watch for:

- **Cycles** (user.created_by → user): seed with NULL then backfill.
- **Polymorphic relations** (commentable_type/commentable_id): the ORM
  models define the legal targets — cite them.
- **Lookup/reference tables** (countries, roles, statuses): check whether
  the app expects them pre-populated (a migration or existing seeder may
  own them — don't double-seed; cite the owner).
- **Pivot tables**: seed AFTER both sides; respect UNIQUE constraints on
  the pair.

## What fires on insert (side effects)

Seeding through the model layer buys hashing, casts, and defaults — and
also triggers everything wired to creation. List what fires per table,
with evidence, BEFORE the seeder is written:

- **ORM events/observers** (Laravel observers/`booted` hooks, Django
  signals, Rails callbacks, Prisma middleware/extensions) — a
  user-registered observer that sends welcome mail turns a 500-user seed
  into 500 emails. Mute them in the seeder with the framework's own
  switch (`Model::withoutEvents`, disconnected signals,
  `skip_callback`) or state explicitly why they are safe to fire.
- **DB triggers** (`SHOW TRIGGERS` / `pg_trigger` / `sqlite_master`) —
  these fire regardless of which layer inserts and cannot be muted from
  the app. Account for the rows they create in BOTH the plan's expected
  counts and the wipe.
- **Queued jobs / webhooks on create** — check the queue driver: a
  `sync` queue runs them during the seed; a real queue silently fills
  with hundreds of jobs pointed at real services.

Every side effect is either muted (cite how) or accepted (say what will
happen). "Probably nothing fires" is not an answer — grep the hooks.

## Census output

One table, feeds the realism plan:

| Table | Rows planned | Depends on | Constraints to respect | Special columns |
|-------|-------------:|------------|------------------------|-----------------|
| users | 12 | — | UNIQUE email; password via Hash::make (app/Models/User.php:31) | avatar (image) |
| articles | 80 | users, categories | status enum: draft/review/published (migration:24) | cover_image |
