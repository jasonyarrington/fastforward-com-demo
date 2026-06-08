# Demo runbook — cooking-show migration

Every phase is **pre-built** as a branch. We present the plan, show the finished discovery, then "cook" each phase live. If a live run stalls or runs long, switch to that phase's finished branch and keep going.

## Branch map (each cumulative)

```
demo/01-discovery            ← START HERE. docs/discovery.md + docs/plugin-report.md
demo/03-export               ← export script + generated content/ + public/content/images/
demo/04-scaffold             ← Next.js scaffold (Tailwind 4 theme, fonts, next.config)
demo/05-loader-layout        ← lib/ content loader + root layout + header/footer
demo/06-routes-blocks        ← 7 block components + homepage + /our-work + /our-work/[slug]
demo/07-dynamic-contact-404  ← app/[slug] + contact placeholder + 404   (also `main`)
```

## The cooking move (per phase N)

```bash
git switch -c live demo/0<prev>-…     # start from the previous phase
#  …paste the seed prompt below, let Claude build it live…
git diff demo/0<N>-…                   # (optional) show the live result matches the pre-baked one
# If it stalls or runs long — reveal the finished dish:
git switch demo/0<N>-…                 # or: git reset --hard demo/0<N>-…   (keeps `live` moving)
```

---

## Phase 1–2 — Discovery + plugin report  ·  _already plated_
Open `docs/discovery.md` and `docs/plugin-report.md`. Talk through the content model, the seven components, and the one custom plugin (`ff-website-contacts` → Monday, with the leaked token). No live build.

## Phase 3 — Export script  ·  start `demo/01-discovery` → reveal `demo/03-export`
> Write a one-shot `scripts/export-wp-to-markdown.mjs` that hits the WPGraphQL endpoint and exports pages, projects, services, and settings into `content/` as markdown/YAML, downloading every referenced image into `public/content/images/`. Make it idempotent. Then run it.

Run: `WPGRAPHQL_URL=https://dev-fast-forward-cms.pantheonsite.io/wp/graphql node scripts/export-wp-to-markdown.mjs`
Expect ≈ 2 pages, 8 projects, 14 services, ~46 media, 0 warnings (see `content/_migration-report.md`).

## Phase 4 — Scaffold  ·  start `demo/03-export` → reveal `demo/04-scaffold`
> Scaffold a Next.js 16 App Router app here. Port the Gatsby Tailwind theme into a Tailwind 4 `@theme` block in `app/globals.css` (the `ff_*` palette + `ff_siteWidth`), wire Manrope + JetBrains Mono via `next/font`, and set `engines.node >= 20`.

Gate: `npm install && npm run build`.

## Phase 5 — Content loader + core layout  ·  start `demo/04-scaffold` → reveal `demo/05-loader-layout`
> Build the typed content loader (`lib/content.ts` + `lib/types.ts`) reading `content/`, the root `app/layout.tsx` with fonts and `generateMetadata`, and port `SiteHeader` (client, with focus trap) plus the footer triple.

## Phase 6 — Routes + homepage + blocks  ·  start `demo/05-loader-layout` → reveal `demo/06-routes-blocks`
> Port the seven block components (MainSection, ImageBlock, QuoteBlock, ProjectCard, FeaturedProjects, ServiceBlock, TextBlock) using `next/image`, then build the homepage, the `/our-work` listing, and `/our-work/[slug]` detail with `generateStaticParams` + `generateMetadata`.

Gate: `npm run build` — 13 static + 8 project pages.

## Phase 7 — Dynamic pages + contact + 404  ·  start `demo/06-routes-blocks` → reveal `demo/07-dynamic-contact-404`
> Add `app/[slug]/page.tsx` that dispatches between Page and LandingPage by frontmatter `layout` (`dynamicParams=false`). Wire `/contact-us` (placeholder form), `/contact-submitted` (reads `?success=`), and a `not-found` page.

Gate: `npm run build`; smoke test the URLs incl. a 404 on an unknown slug.

---

## Next steps (mention, don't build)
- **Contact form + Monday route** — `app/api/contact/route.ts`, `MONDAY_API_TOKEN` in env, **rotate the leaked token**.
- **SEO** — OG/Twitter metadata, sitemap/robots/manifest, GA `G-YPFDS2CY4K`.
- **Launch** — README, `.env.local.example`, `/page/:num` → `/` redirect, Pantheon deploy via `scripts/deploy-feature-branch.sh` (preview env per PR; push to `main` is BLOCKed — PR-only).
