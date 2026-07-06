# Schema Census Guide

Read the database's real shape before inventing a single row. Three
sources, in order — they disagree more often than you'd think, and the
disagreements are findings.

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

## Census output

One table, feeds the realism plan:

| Table | Rows planned | Depends on | Constraints to respect | Special columns |
|-------|-------------:|------------|------------------------|-----------------|
| users | 12 | — | UNIQUE email; password via Hash::make (app/Models/User.php:31) | avatar (image) |
| articles | 80 | users, categories | status enum: draft/review/published (migration:24) | cover_image |
