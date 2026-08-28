---
name: spale-seo
description: "Modern (2025/2026) SEO methodology for auditing and writing pages \u2014 title tags, meta, headings, structured data, Core Web Vitals, and AI-search citation patterns."
---

# SpaleSEO — Modern Search Optimization Skill

You optimize pages for **classical search (Google/Bing)** *and* **AI answer engines (ChatGPT Search, Perplexity, Google AI Overviews, Claude, Bing Copilot)** at the same time. Most "SEO advice" online is stale — these are the rules that hold in 2025/2026.

---

## Phase 1: Title Tags

**Target length:** 50–60 chars / 500–600px (desktop). Mobile truncates ~430px.

**Format:** `Primary Keyword - Modifier | Brand`

**Rules**
- Front-load the primary keyword in the **first 30 characters**.
- One unique title per URL — never duplicate across templates.
- Use a single separator project-wide (` | ` or ` - `). Skip emoji/special chars Google strips.
- Match search intent over hitting char count. Clarity > length.

**Don't**
- Pad to 60 chars artificially.
- Stuff keywords — Google rewrites titles when they look manipulative.
- Re-use the homepage title across deep pages.

---

## Phase 2: Meta Description

**Target length:** 150–160 chars desktop, ~120 chars mobile. Front-load value in the **first 100 chars**.

**Pattern:** `[focus term] + [concrete value] + [action verb]`. Action verbs that work: *Learn, Compare, Build, Ship, See, Get*.

**Rules**
- Not a ranking factor — it's a CTR factor. Well-written ones can lift CTR ~30–43%.
- Google rewrites 60–70% of them; aim for "rewrite-resistant" intent alignment.
- Unique per URL. Include CTA when appropriate ("Start free", "See pricing").
- Skip on pages where the page text already opens with a stronger SERP-worthy sentence.

---

## Phase 3: Meta Keywords

**Status: dead. Omit entirely.**

- Google has ignored `<meta name="keywords">` since 2009.
- Bing treats it as a spam signal.
- Only Yandex/Baidu retain residual support.
- If you're generating one, you're shipping noise.

---

## Phase 4: Heading Hierarchy (H1–H6)

**Rules**
- **One H1 per page**, matching/expanding the title intent (≤70 chars).
- **H2** for primary sections, **H3** for sub-sections — never skip levels (no H2 → H4).
- Phrase H2/H3 as **questions** when the section answers one — massive AEO/citation win.
- Headings are for *structure*, not styling. Use CSS for visual size.
- Never wrap a logo in H1.

**Why it matters double in 2026:** GPTBot, ClaudeBot, and PerplexityBot rely heavily on heading-driven chunking to decide what to cite. Clean hierarchy = higher AI citation odds.

---

## Phase 5: Body Content & Semantic HTML5

**Required structure**
- Exactly one `<main>` per page (not nested in header/footer/article).
- `<article>` for self-contained content, `<section>` for thematic groupings, `<aside>` for tangential, `<nav>` for nav, `<header>`/`<footer>` per article *and* per page.

**Readability targets**
- Paragraphs: 2–4 sentences (2–4 lines on mobile).
- Average sentence length: <20 words. Flesch reading ease ~60+.
- Use `<ul>`, `<ol>`, and `<table>` — these are disproportionately extracted by AI Overviews and Perplexity.
- Bold key claims with `<strong>` — improves featured-snippet selection.
- **Definitive opening sentence** ("X is Y that does Z.") — that's what LLMs lift verbatim.

---

## Phase 6: Image SEO

**Format priority:** AVIF → WebP → JPEG fallback via `<picture>`.

**Required attributes**
```html
<img
  src="hero.avif"
  alt="Descriptive alt text, 80–140 chars, no 'image of'"
  width="1200" height="630"
  loading="lazy"           <!-- omit for the LCP image -->
  fetchpriority="high"     <!-- only on the LCP image -->
  srcset="hero-800.avif 800w, hero-1600.avif 1600w"
  sizes="(max-width: 800px) 100vw, 1200px"
/>
```

- `width` + `height` always — prevents CLS.
- **Never lazy-load the LCP image.**
- Empty `alt=""` for purely decorative images (don't omit `alt`).
- Descriptive filenames: `red-running-shoe-side.avif`, not `IMG_1029.jpg`.

---

## Phase 7: Internal Linking & Anchor Text

- Descriptive anchors that read naturally — never "click here" / "read more".
- Mix exact-match, partial-match, and branded anchors. Over-optimized exact-match looks spammy.
- Important pages within **3 clicks** of the homepage.
- Internal links: leave dofollow.
- External links: `rel="nofollow"` (untrusted), `rel="sponsored"` (paid/affiliate), `rel="ugc"` (comments/forums). Combine: `rel="sponsored nofollow"`.
- Always set `rel="noopener"` on `target="_blank"` (security, not SEO).
- **Hub-and-spoke**: pillar page (1,500–3,000 words, broad) ↔ cluster pages (1,500–2,500 words, single long-tail intent). Min 3 internal links to every new post.

---

## Phase 8: Core Web Vitals (2025/2026 thresholds)

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| **LCP** | ≤2.5s | ≤4.0s | >4.0s |
| **INP** (replaced FID Mar 2024) | ≤200ms | ≤500ms | >500ms |
| **CLS** | ≤0.1 | ≤0.25 | >0.25 |

Measured at the **75th percentile of real-user CrUX data**. Confirmed ranking signal. Only ~48% of mobile pages currently pass all three.

**LCP levers:** preload LCP image with `fetchpriority="high"`, defer non-critical JS, AVIF/WebP, server TTFB <600ms (CDN/edge), `font-display: swap`.

**INP levers:** break long tasks (>50ms) with `scheduler.yield()` or `setTimeout(0)`, debounce input, defer to `requestIdleCallback`/web workers, audit third-party scripts (chat/analytics/ads are the #1 INP killers).

**CLS levers:** `width`+`height` on every image/iframe (or `aspect-ratio` CSS), `min-height` on ad/embed slots, preload key fonts, `size-adjust`/`ascent-override` for fallback metrics.

---

## Phase 9: Technical SEO

**Canonical**
```html
<link rel="canonical" href="https://example.com/page">
```
- Self-referential on every indexable page. One canonical per page. HTTPS only. Match the sitemap URL.

**Robots meta**
```html
<meta name="robots" content="index,follow,max-image-preview:large">
```
- `max-image-preview:large` recommended for richer SERP/AI snippets.
- Use `noindex,follow` for thin/utility pages (thank-you, internal search, low-value tag pages).

**hreflang** (international)
- Reciprocal pairs. Format: `en-US`, `en-GB` (lowercase lang, uppercase country, hyphen — `en-uk` is wrong).
- One `x-default` per cluster.
- Every alternate URL must return 200 (no 404s, no redirect chains).
- Never canonicalize Spanish→English while declaring hreflang — signals collide.

**Sitemap.xml**
- Only canonical, indexable, 200-status URLs.
- Update `lastmod` accurately.
- Reference from `robots.txt` via `Sitemap: https://...`.
- Split at 50k URLs / 50MB.

**Pagination** — `rel=prev/next` is dead (Google retired it). Modern pattern: each page self-canonicals, has unique `<title>` ("Page 2 of …"), and is internally linked.

**URL structure**
- Lowercase, hyphenated, ASCII only. **Hyphens, never underscores.**
- Slugs <60 chars where possible.
- Pick one trailing-slash convention; 301 the other.
- Mirror breadcrumb depth ≤3 from root: `/category/subcategory/post-slug`.
- Strip tracking params at the canonical (`?utm_*`, `?ref=`, `?sessionid=`).

**Redirects** — 301/308 permanent, 302/307 temporary. **Zero redirect chains** — update internal links to the final URL.

---

## Phase 10: AI-Era SEO (AEO / GEO / LLM SEO)

By Feb 2026, only ~38% of AI-cited URLs ranked in the organic top 10 — AI engines are **decoupling from classical rank**. You must optimize for both.

**Citation patterns that move the needle**
- **Definitive opening sentence:** `"X is Y that does Z."` First 1–2 sentences are what LLMs lift.
- **Question-as-heading + direct answer** in the first 1–3 sentences (the "TL;DR pattern").
- **Quotable atomic claims** with concrete numbers + dates that AI can lift verbatim.
- **Statistics with attribution + dates** ("In 2025, X% of …, per [source]").
- **Lists, tables, comparisons** are disproportionately cited.
- **Entity clarity** — explicitly name and define entities; link to Wikipedia/Wikidata; use `sameAs` in Organization schema.
- **Author + Organization schema** with `sameAs` to Wikipedia/Wikidata to disambiguate.
- **Freshness** — quarterly updates with a visible "Last updated" date. Pages not refreshed quarterly lose citations ~3× faster.

**robots.txt for AI crawlers — strategic split**
```
# Search-visibility crawlers — ALLOW (citation upside)
User-agent: OAI-SearchBot
Allow: /
User-agent: PerplexityBot
Allow: /
User-agent: ChatGPT-User
Allow: /

# Training-only crawlers — pick your stance
User-agent: GPTBot
Disallow: /
User-agent: ClaudeBot
Disallow: /
User-agent: anthropic-ai
Disallow: /
User-agent: CCBot
Disallow: /
User-agent: Google-Extended
Disallow: /
User-agent: Meta-ExternalAgent
Disallow: /
```

**llms.txt** — emerging convention (Anthropic, Stripe, Cloudflare have adopted). No major LLM officially confirms reading it; Google has said AI Overviews ignore it. **Treat as cheap insurance, not core strategy.**

---

## Phase 11: Open Graph & Twitter Cards

**Required minimum**
```html
<meta property="og:title" content="...">
<meta property="og:description" content="...">
<meta property="og:image" content="https://.../og.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:url" content="https://...">
<meta property="og:type" content="article">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:image" content="https://.../og.jpg">
```

- **1200×630 (1.91:1)** is the universal sweet spot — works on Facebook, LinkedIn, Discord, Slack, X.
- <5MB. JPG/PNG/WebP. Center critical text/logo (edges crop on mobile).
- LinkedIn caches aggressively — version the URL (`?v=2`) when changing.
- Discord/Slack require absolute HTTPS URLs.
- Indirect SEO impact (drives social CTR/shares which feed referral signals).

---

## Phase 12: JSON-LD Structured Data

**Format:** JSON-LD only (Google's preferred). Inject in `<head>` or `<body>`. Only mark up content actually visible on the page.

**Still high-ROI types (2026)**
- `Organization` — once on homepage; `name`, `url`, `logo`, `sameAs` (social, Wikipedia, Wikidata), `contactPoint`.
- `WebSite` — homepage, with `SearchAction` for sitelinks search box.
- `BreadcrumbList` — every non-home page.
- `Article` / `BlogPosting` / `NewsArticle` — `headline`, `author` (Person with `sameAs`), `datePublished`, `dateModified`, `image`, `publisher`.
- `Product` + `Offers` + `AggregateRating` + `Review` — drives shopping rich results.
- `Person` — author bios with `sameAs`.
- `VideoObject` — `name`, `thumbnailUrl`, `uploadDate`, `duration`, `contentUrl`.
- `Event` — required for event rich results.

**Skip / deprecated for most sites**
- `FAQPage` — Google retired rich-result eligibility for most sites (kept for gov/health). Markup still helps AI parsing — ship it if your content really has Q&A, but don't expect SERP enhancement.
- `HowTo` — same status.

**Validate:** [Rich Results Test](https://search.google.com/test/rich-results) + Schema Markup Validator. Pages with valid structured data are ~2.3× more likely to surface in AI Overviews.

---

## Phase 13: E-E-A-T & Content Quality Gate

**E-E-A-T = Experience, Expertise, Authoritativeness, Trustworthiness.** Experience (first-hand) is the differentiator vs. AI slop.

**Google's stance:** AI content is allowed; *low-value* content is not. Penalties hit thin/derivative output, not the production method.

**Content checks**
- Author byline with credentials + link to author page with `sameAs` (LinkedIn, ORCID).
- Visible publish + lastModified dates.
- Original media (screenshots, photos, original data) — not stock-only.
- Cited primary sources, not just other blogs.
- "Why are *we* qualified to write this?" answered above the fold.
- First-person experience markers ("I tested", "we benchmarked") for YMYL/review content.

---

## Universal Skip List (do-not-do)

- `<meta name="keywords">` — dead.
- Multiple conflicting canonicals (sitemap vs. tag).
- `noindex` + `Disallow` on the same URL — Google can't see the noindex if it can't crawl.
- Carousels for the LCP element.
- Hidden text "for SEO".
- Auto-translated hreflang without quality gates.
- Lazy-loading the LCP image.
- Generic anchor text ("here", "this link").
- Stuffing schema types regardless of fit.
- Scaled content abuse (mass templated AI pages with one swapped variable).
- Doorway pages (city/state pages that all funnel to the same form).
- Cloaking (different content for Googlebot vs. users).

---

## Audit Run Order (prescriptive)

Fix earlier items before measuring later ones:

1. **Google Search Console** — Coverage, CWV, Manual Actions, Security, Mobile Usability. Fix any manual action *before* anything else.
2. **Crawl** with Screaming Frog or Sitebulb (connect PSI + GSC + GA4 APIs).
3. **Triage in priority order:**
   1. Indexability blockers (robots.txt mistakes, accidental noindex, canonical loops).
   2. Status code health (4xx, 5xx, redirect chains/loops).
   3. Duplicate content (canonicals, trailing-slash, parameter dupes).
   4. Mobile parity + Core Web Vitals failing URLs (CrUX field data, not just Lighthouse lab).
   5. Title + meta description quality (uniqueness, length).
   6. Structured data errors (Rich Results Test).
   7. Internal-link orphans + anchor relevance.
   8. Hreflang reciprocity (if international).
   9. Content quality / E-E-A-T (manual review of top 20% URLs by traffic).
   10. AI-search readiness (definitive openers, entity schema, llms.txt).
4. **Backlinks** — Ahrefs/Semrush; disavow only manual-action-level toxic links.
5. **Re-crawl + monitor.** CrUX 28-day rolling; GSC impressions/CTR by query cluster; AI-engine citations (Otterly, Profound).

---

## Output Format (when auditing)

For each finding:
- **Severity**: CRITICAL / HIGH / MEDIUM / LOW / INFO
- **Phase**: which phase above it falls under
- **Location**: file + line, or URL + element
- **Issue**: one sentence
- **Fix**: the exact code/copy change
- **Impact**: classical search, AI search, or both

When the page is solid, say so confidently and name what's protecting it.

---

## One-Line Distillation

> **Write for humans first; structure ruthlessly so machines (Google + LLMs) can chunk, cite, and rank you.**
