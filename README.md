# fastforward.sh — migration demo

This repository is a **live, on-stage migration demo** ("cooking show") that ports
[fastforward.sh](https://fastforward.sh) from a **headless WordPress + Gatsby 4** stack to
**Next.js 16 (App Router)** — replacing runtime WPGraphQL queries with static **MDX / Markdown +
YAML** content, and deploying on **Pantheon's new Next.js platform**.

## What's being migrated

| | Before | After |
| --- | --- | --- |
| Front end | Gatsby 4 | Next.js 16 (App Router) + React 19 |
| Content source | Headless WordPress (WPGraphQL) at runtime | Static Markdown / MDX + YAML in `content/` |
| Styling | — | Tailwind 4 (theme in an `@theme {}` block, no `tailwind.config.ts`) |
| Hosting | — | Pantheon Next.js platform |

Content is exported **once** from WordPress into flat files, so the running site has no live
dependency on WordPress or WPGraphQL.

## How the demo works

Every phase of the migration is **pre-built as its own branch**. On stage, each phase is built
**live** from the previous phase's branch; if a live run stalls or runs long, the presenter reveals
the finished branch (`git switch demo/0N-<name>`). Seed prompts for each phase live in
[demo-runbook.md](demo-runbook.md).

### Branch map (each cumulative)

```
demo/01-discovery            ← docs + plugin report (the show starts here)
demo/03-export               ← export script + content/ + public/content/images/
demo/04-scaffold             ← Next.js scaffold + Tailwind 4 theme + fonts
demo/05-loader-layout        ← lib/ content loader + root layout + header/footer
demo/06-routes-blocks        ← block components + homepage + /our-work + /our-work/[slug]
demo/07-dynamic-contact-404  ← app/[slug] + contact + 404   (also `main`, fully working)
```

## Content export

The WordPress content is exported to Markdown/YAML with an idempotent script
(≈2 pages, 8 projects, 14 services, ~46 media):

```bash
WPGRAPHQL_URL=https://dev-fast-forward-cms.pantheonsite.io/wp/graphql \
  node scripts/export-wp-to-markdown.mjs
```

## Stack & notes

- **Next 16.2.x + React 19 + Tailwind 4.** The theme lives in an `@theme {}` block in
  `app/globals.css` — there is no `tailwind.config.ts`. Custom classes used with a responsive
  prefix (`lg:`) must be defined as `@utility` rules, not `@layer`.
- Fonts via `next/font/google` (Manrope + JetBrains Mono); `engines.node >= 20`.
- If image optimization 500s after `npm ci`, run `npm approve-scripts sharp` once.

## Scope

The demo stops at the **dynamic-pages + placeholder-contact + 404** phase. Intentionally out of
scope (mentioned on stage as next steps, not built here):

- Contact form API route → Monday.com integration.
- SEO (sitemap / robots / manifest / analytics) and the Pantheon launch.

## Reference

- [docs/discovery.md](docs/discovery.md) — discovery findings and content model.
- [docs/plugin-report.md](docs/plugin-report.md) — the old WordPress custom plugin / ACF / taxonomy report.
- [CLAUDE.md](CLAUDE.md) — orientation for AI sessions working in this repo.
