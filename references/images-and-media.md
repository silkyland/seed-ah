# Images & Media Guide

Every image column found in the census gets a real file — an app full of
broken-image icons demos worse than an empty one.

## What the app expects (read, don't guess)

From the census and code (`file:line` each): storage location (local
`storage/`, S3 bucket, `public/uploads`), expected dimensions/variants
(the resize config, CSS slots, or `<img>` usage), naming conventions, and
whether the DB stores paths, URLs, or IDs into a media table (seed THAT
table properly if so).

## Generation options, in order of preference

1. **AI image generation** — when the running agent has image-generation
   tooling available (built-in tool, MCP server, or a local model), use it
   for hero/cover/content images: prompt with the product's actual domain
   ("magazine cover about urban gardening, minimal, brand color #1A5F4A")
   so images match the product's world. Generate once, commit to the seed
   assets folder — regeneration should not require the tool again.
2. **Deterministic programmatic rendering** — the always-works fallback,
   and the right choice for avatars and bulk thumbnails. Python/Pillow or
   SVG templates: brand-colored backgrounds (pull the color from the
   product's CSS, cite it), initials-avatars for users, title-text covers
   for content. Deterministic from the entity's own data = same seed, same
   images.
3. **Placeholder services (picsum et al.)** — acceptable ONLY for
   throwaway local dev; they leak network dependency into seeds and the
   images change under you. Never for committed demo worlds.

## Hard rules

- **No real people.** No scraped photos, no stock-photo "borrowing", and
  avoid AI-generated photoreal faces too (likeness risk) — illustrations,
  objects, scenery, and initials-avatars cover every case a demo needs.
- **No copyrighted material** — no logos of real brands, no images pulled
  from the web without a license you can cite.
- **Right size, small weight**: match the app's expected dimensions;
  compress (JPEG/WebP quality ~80) — a seed folder of 40MB PNGs annoys
  every clone forever.
- Files live in the seed assets folder (`database/seeds/assets/` or the
  stack's convention), copied into the app's storage by the seeder — so
  the seed is self-contained and the wipe can remove them by the same
  manifest.

## Non-image media

Columns for PDFs/attachments: generate tiny real files (a one-page PDF
via the stack's PDF lib, a small CSV) — a 0-byte fake breaks download
flows during demos.
