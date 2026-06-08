# Phase 1 — Discovery: the old site

_Source we're migrating **from**: `fastforward-web` (Gatsby 4 front end) + `fast-forward-cms` (headless WordPress on Pantheon). Target: this repo — Next.js 16 App Router reading static markdown/YAML._

## Stack (old Gatsby front end)

| Area | Old (Gatsby) | New (Next.js) |
|---|---|---|
| Framework | Gatsby 4.25 + React 18 | Next 16 App Router + React 19 |
| Styling | Tailwind 3 (`tailwind.config.js`) | Tailwind 4 (`@theme {}` in `globals.css`) |
| Data | `gatsby-source-wordpress` → WPGraphQL at build | one-shot export → static `content/` markdown + YAML |
| Images | `gatsby-plugin-image` / `GatsbyImage` | `next/image` with explicit width/height from frontmatter |
| Fonts | `@fontsource` Manrope + JetBrains Mono | `next/font/google` |
| Analytics | `gatsby-plugin-google-gtag` (`G-YPFDS2CY4K`) | `@next/third-parties/google` (SEO next-step) |
| Sitemap/Manifest | gatsby plugins | `app/sitemap.ts` / `manifest.ts` (SEO next-step) |

**Brand palette** (ported verbatim into the Tailwind 4 `@theme` block):
`ff_black #191819`, `ff_gray #7f7d81`, `ff_slateGray #AAAAAA`, `ff_lightGray #EDF1F2`, `ff_red #DD2E2A`, `ff_teal #008CA8`, `ff_tealDarker #004A59`. Site max width `ff_siteWidth 1600px`.

## Content model

- **Pages** (`WpPage`) — default pages + a `landing` layout variant (slug contains "landing").
- **Projects** (`WpPost`, shown as "our work") — sticky-first ordering via a custom `onlySticky` WPGraphQL arg.
- **Services** — custom `service` taxonomy attached to projects.
- **Settings** — site title/description, `postsPerPage`.

**ACF flexible content** on projects (`pageSections.sections[]`), discriminated by `__typename`:
- `MainSection` — tagline, title, `background` (white/gray), rich text (WYSIWYG).
- `ImageBlock` — 1–2 images.
- `ClientQuote` — clientName, tagline, quote.

**Per-project ACF fields:** `additionalPostFields` (label, brandColor, seoDescription), `customImages.cardImage`, `featuredImage`, `services[]`.

## Routing (old → new)

| Old (Gatsby `gatsby-node.js`) | New (Next App Router) |
|---|---|
| `/` homepage archive | `app/page.tsx` |
| `/our-work`, `/page/[num]` pagination | `app/our-work/page.tsx` (pagination **dropped** — 8 projects, page 1 is everything) |
| `/our-work/{slug}` project detail | `app/our-work/[slug]/page.tsx` (`generateStaticParams` + `generateMetadata`) |
| `/{slug}` pages, landing variant by slug | `app/[slug]/page.tsx` (`dynamicParams=false`, dispatch by frontmatter `layout`) |
| `/contact-us`, `/contact-submitted`, `404` | same routes; `not-found.tsx` |

## Components to port

- **Layout:** `SiteHeader` (interactive — mobile menu + focus trap), footer triple (`ContactBlock`, `Footer`, `FooterBlock`).
- **Blocks (7):** `MainSection`, `ImageBlock`, `QuoteBlock`, `ProjectCard`, `FeaturedProjects`, `ServiceBlock`, `TextBlock`.
- **Composites:** `Post` (project detail), `Page`, `LandingPage`, `HomepageBanner` (interactive — rotating-word hero).
- **Helper:** `trapFocus` (focus management for the mobile menu) — ports unchanged.

## Contact flow

Old form POSTs JSON to `https://live-fast-forward-cms.pantheonsite.io/wp-json/ff-website-contacts/v1/submit` (a **custom WP plugin** — see `plugin-report.md`). New site reimplements this as a Next API route → Monday.com (a **next-step**, out of demo scope). Demo ships a placeholder form + `/contact-submitted?success=` confirmation.
