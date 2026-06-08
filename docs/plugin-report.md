# Phase 2 — Custom plugin report (WordPress CMS)

_Audit of `/dev/fast-forward-cms` (Bedrock-layout headless WordPress on Pantheon). Goal: identify the **server-side behavior** the static Next.js site must reimplement vs. what simply collapses into static content._

## Verdict up front

Only **one** plugin contains real server-side behavior that must be reimplemented: the contact-form handler (`ff-website-contacts`). Everything else is either content (ACF field groups → frontmatter), a taxonomy (→ `services.yml`), or marketplace infrastructure that disappears when we stop serving WordPress at runtime.

---

## 1. `ff-website-contacts`  — CRITICAL, must reimplement

**File:** `web/app/plugins/ff-website-contacts/ff-website-contacts.php`

- **REST route:** `POST /wp-json/ff-website-contacts/v1/submit`, `permission_callback => __return_true` (open endpoint).
- **Payload:** `firstName, lastName, company, website, email, phone1/2/3, comments`.
- **Sanitization:** `sanitize_text_field()` on text fields, `sanitize_email()` on email. Comments sanitized as *text* (not textarea) to dodge a Monday.com GraphQL encoding issue.
- **Integration:** creates an item on **Monday.com board `3979078971`**, group `new_group99744`, mapping company/website/phone/email/comments to Monday columns via a GraphQL mutation.

> ⚠️ **SECURITY — hardcoded secret.** A Monday.com API JWT is committed in the plugin source (~line 91). In the migration this **must** become a `MONDAY_API_TOKEN` env var **and be rotated** (the old token is burned the moment it's in git history). This lands in the Phase-5 next-step, not the demo.

**Migration:** reimplement as `app/api/contact/route.ts` — same payload shape, token from env, Monday call via GraphQL **variables** (not inline JSON) to avoid the PHP's double-escape hazard. _(Next-step — out of demo scope.)_

---

## 2. Theme: `twentytwentytwochild`  — taxonomy + GraphQL extensions (→ static)

**File:** `web/app/themes/twentytwentytwochild/functions.php`

- Registers the custom **`service` taxonomy** with `show_in_graphql` → becomes `content/services.yml`.
- Registers a custom WPGraphQL arg **`onlySticky`** on the posts connection (sticky-first project ordering) → becomes a sort in the static content loader.

**ACF field groups** (`web/app/themes/twentytwentytwochild/acf-json/`) — all become MDX frontmatter:
- `group_5c80353f9a6b6` → `additionalPostFields` (label, brand_color, seo_description).
- `group_646d0bb90dfc9` → `pageSections` flexible content (MainSection / ImageBlock / ClientQuote).
- `group_649319c4a3888` → `customImages` (card_image).

---

## 3. Other plugins — no port needed

| Plugin | Type | Disposition |
|---|---|---|
| `decoupled-kit-acf` | Pantheon example (Related Posts ACF) | Drop — feature unused. |
| `decoupled-preview` | Decoupled preview button | N/A for static site. |
| `bu-assets` | CSS asset shim (near-empty) | Drop. |
| ACF Pro, WPGraphQL, WPGraphQL-for-ACF | content API | Gone — content is exported once. |
| WP Gatsby, WPGraphQL Smart Cache, Pantheon Advanced Page Cache | build/cache infra | N/A at runtime. |
| WP Mail SMTP, WP Webhooks, WP Migrate, Native PHP Sessions, Classic Editor, LH HSTS | WP-side ops | N/A. |

---

## What the new site must own (vs. WordPress)

1. **Contact submission → Monday.com** (the only real logic) — Next API route, secret in env. _(next-step)_
2. **Sticky-first project ordering** — handled in the static content loader.
3. **Everything else** — static content the export script pulls into `content/`.
