# CLAUDE.md — read me first

This repo is the **target of a live, on-stage migration demo** ("cooking show"). We are porting a Gatsby 4 + headless-WordPress site to **Next.js 16 (App Router)** that reads static markdown/YAML instead of WPGraphQL at runtime.

## How the demo works — IMPORTANT

Every phase is **pre-built** as its own branch. On stage we build a phase **live**, and if a live run stalls or runs long we `git switch` to that phase's finished branch (the "reveal"). So:

- **Build the current phase live** from the previous phase's branch. Use the seed prompt the presenter pastes (they live in `demo-runbook.md`).
- If asked to **reveal / fall back**: `git switch demo/0N-<name>` (or `git reset --hard demo/0N-<name>` to keep a `live` branch moving).
- **Do NOT browse `/dev/fastforward-web-next`.** That's the original migration repo the demo branches were derived from, but it has since drifted far past this migration (blog, Storybook, spam filter…). Reading it will mislead you. The reveal branches are the safety net — use those.

### Branch map (each cumulative)
```
demo/01-discovery            ← docs + plugin report (the show starts here)
demo/03-export               ← export script + content/ + public/content/images/
demo/04-scaffold             ← Next.js scaffold + Tailwind 4 theme + fonts
demo/05-loader-layout        ← lib/ content loader + root layout + header/footer
demo/06-routes-blocks        ← 7 block components + homepage + /our-work + /our-work/[slug]
demo/07-dynamic-contact-404  ← app/[slug] + contact + 404   (also `main`, fully working)
```

## Source material (read-only; these are NOT in this workspace)

If a phase needs to reference the old code, the presenter will add it on demand (`/add-dir`):
- `/dev/fastforward-web` — old **Gatsby** front end (components/routes to port).
- `/dev/fast-forward-cms` — old **headless WordPress** (custom plugin `ff-website-contacts` → Monday; ACF field groups; `service` taxonomy). See `docs/plugin-report.md`.

The discovery findings and the content model are already written up in `docs/discovery.md` and `docs/plugin-report.md` — prefer those over re-reading the source.

## Target stack & gotchas

- Next 16.2.x + React 19 + **Tailwind 4** — the theme lives in an `@theme {}` block in `app/globals.css`; there is **no `tailwind.config.ts`**. A custom class used with a responsive prefix (`lg:`) must be an `@utility` rule, not `@layer`, or the prefix silently no-ops.
- Fonts via `next/font/google` (Manrope + JetBrains Mono); `engines.node >= 20`; `images.qualities: [75, 90]` in `next.config`.
- **Content export:** `WPGRAPHQL_URL=https://dev-fast-forward-cms.pantheonsite.io/wp/graphql node scripts/export-wp-to-markdown.mjs` (idempotent; ≈2 pages, 8 projects, 14 services, ~46 media).
- **sharp / allow-scripts:** if image optimization 500s after `npm ci`, run `npm approve-scripts sharp` once.

## Scope

Demo stops at the **dynamic-pages + placeholder-contact + 404** phase. Deliberately excluded (and there is **no Monday API token in this repo**) — these are next-steps to *mention*, not build:
- Contact API route → Monday.com (`MONDAY_API_TOKEN` in env; **rotate the token that was hardcoded in the old WP plugin**).
- SEO (sitemap/robots/manifest/GA) and Pantheon launch via `scripts/deploy-feature-branch.sh` (deploys are PR-only; direct push to `main` is blocked).
