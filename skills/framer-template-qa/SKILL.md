---
name: framer-template-qa
description: QA a Framer template before it goes on the Framer Marketplace. Inspect the canvas, the served HTML and the live site, check SEO and AEO (search and AI-answer readiness), fix clear defects in the editor, add a buyer Getting started guide, and report findings ranked by what a buyer would hit. Use when asked to QA, audit, check, or prepare a Framer template for the Marketplace or for sale.
---

# Framer template QA

Audit a Framer template the way a buyer meets it: preview on a phone, remix, then edit. Fix what is clearly broken, leave decisions to the owner, and report with evidence.

The full method is in the Playbook section below: the severity model, the three lenses, the false-finding traps, the checks, and the report format. Read it before starting.

## Before you start

- You need the Framer project (edit access) and its live preview URL.
- Canvas checks use the Framer agent CLI (`npx @framer/agent@latest setup`, then `session new <project url>`). Without it, run the served-HTML and live-browser lenses only and say which canvas checks were skipped.
- Reference: https://www.framer.com/help/articles/template-best-practices/ (guidance, not approval criteria; Framer no longer reviews templates).

## Boundaries

- Never publish the project and never edit the Marketplace listing.
- Make small, targeted edits with the project's existing text and color styles. Keep the previous value of every edit so you can revert only your own change.
- Ask before deleting pages, CMS data, assets, fonts or features, changing URLs, CMS schemas, form destinations or integrations, or rewriting copy beyond clear typos.
- Never guess destinations, legal text, licences, pricing, refund terms or support commitments.
- Do not submit forms, send messages or change domains, billing or tracking unless the owner asks.

## Workflow

1. **Inventory.** Pages, breakpoints, layout templates, components, code files, styles, fonts, CMS collections.
2. **Lens 1, canvas.** Dump each page tree once and analyse locally: heading tags per breakpoint, inline styles vs presets, alt text, links, form fields, unused components and fonts, element IDs that code depends on.
3. **Lens 2, served HTML.** Curl every page: titles, descriptions, OG images, H1 count, noindex on 404, real hrefs. Divide repeated markup by the breakpoint count before calling anything duplicated.
4. **Lens 3, live browser.** Every page at 390, 810 and 1280px: horizontal overflow, clipped text, menus, toggles, filters, forms. Measure, don't eyeball.
5. **SEO and AEO.** Page settings, robots.txt, JSON-LD, served HTML, sitemap, canonicals, 404 status, Markdown output, and whether the copy states what the product is, for whom and at what price (see "SEO and AEO" in the Playbook below).
6. **Beyond the site.** Licences of code files and fonts, originality and fit, support and listing facts, and the buyer guide (see "Beyond the site" in the Playbook below).
7. **Fix** confirmed defects with clear solutions: spelling, overflow, broken links with an unambiguous destination, CMS bindings, alt text, form labels, style inconsistencies. Re-read each change after writing it and check all three breakpoints.
8. **Buyer guide.** Add a "Getting started" editor-only Design page (never a public route) covering pages and shared layout, styles, CMS fields and filters, form Send To, SEO and AEO setup, and every code-driven section with what must not be renamed. Describe only what you inspected. Keep the layout minimal until a designed guide layout exists.
9. **Report** in this order: Fixed and verified · Changed, verification pending · Needs your input · Optional improvements · Verification coverage (passed vs not tested). Every finding carries evidence. Never claim the whole template is ready because a subset passed.

## Playbook

How to audit a template before it ships. Built from three real audits: Cohestra (12 pages, CMS heavy), Pillo (3 pages, effects heavy), Elias Vent (portfolio, code components).

---

### The angle changed

Framer removed the marketplace review. Templates publish immediately, no gatekeeper.

That kills the old framing. The Master Guide is written around "what gets you rejected", and ranking findings by rejection risk is now measuring against a thing that does not exist. **Never write "a reviewer would flag this" in an audit.** It is not a reason any more.

What replaced it: **what does a paying buyer hit, and in what order.** A buyer opens the preview on their phone, taps around, buys, remixes, and starts editing. Every finding gets ranked by where it sits on that path.

The Master Guide is still the best inventory of *what to look at*. It is no longer the right authority on *how much anything matters*. Use it as a checklist source, not a severity model.

#### Severity tiers

| Tier | Meaning |
|---|---|
| **Costs you a sale** | A visitor or buyer hits it without looking for it. Broken nav, dead CTA, duplicated pricing copy, grammar in a headline. |
| **Worth doing before release** | Real, but the buyer only meets it after purchase, or it degrades quality rather than blocking. Semantics, touch hover, type tiers. |
| **Cosmetic / hygiene** | Only the buyer editing the file sees it. Layer names, unused components, stray frames. |
| **Holding** | Verified clean. Record these so the next pass does not re-litigate them. |

The Holding table matters more than it looks. Three audits in, most re-check time was spent re-proving things that were already fine.

---

### Method: three lenses, in this order

A finding is only real if it survives the lens that can actually see it. Most bad findings come from using the wrong one.

#### Lens 1: the canvas, through the agent API

Structure, attributes, components, styles, tokens. This is where you find things the browser cannot show you: unused components, orphan tokens, uploaded-but-unused fonts, stray off-canvas frames, variant wiring.

Dump the whole tree once and analyse locally rather than making dozens of round trips:

```javascript
const tree = await framer.agent.serialize({ id: "<pageId>", depth: 200 }, { pagePath: "/" });
require('fs').writeFileSync('/tmp/page.json', JSON.stringify(tree));
```

Then walk the JSON with a local script. Fast, repeatable, and you can diff it after a fix.

Useful sweeps: attribute-key inventory (tells you what exists at all), `htmlTag` counts, `tag` sequences for headings, `fill` values that are not `var(--token-`, `codeOverride` presence per breakpoint, `visible:false` nodes, generic layer names.

#### Lens 2: the served HTML

`curl` the published pages. This is the only place you see what actually ships: how many `<h1>` survive, landmark counts, real link hrefs, duplicated content, meta tags, JSON-LD.

**The single biggest trap:** Framer emits one copy of the markup per breakpoint. Three `<h1>`, three `<nav>`, three `<footer>` is usually **one** authored element rendered three times, not three mistakes. Always divide by the breakpoint count before calling something duplicated. Conversely, content that appears 6 times when there are 3 breakpoints is genuinely doubled.

#### Lens 3: the live browser

Anything positional, interactive, or viewport-dependent. Hover, menus, scroll effects, responsive collisions.

**Measure, do not eyeball.** The Pillo phone-position bug was only provable by reading `getBoundingClientRect` at two viewport heights and showing the offset grew with height. A screenshot would have produced an argument; the numbers produced a diagnosis.

```javascript
const r = el.getBoundingClientRect();
({ top: r.top, center: r.top + r.height/2, vpCenter: innerHeight/2,
   delta: r.top + r.height/2 - innerHeight/2 })
```

Run it at 900 and 1400 height. If `delta` changes, the element is anchored to something viewport-dependent.

---

### Traps that produce false findings

Every one of these cost real time. Check them before reporting.

| Symptom | Usually is |
|---|---|
| Screenshot comes back black or blank | Capture artifact on WebGL, canvas, or heavy compositing. Verify with computed styles and `img.complete` before claiming the section is broken. |
| Element sits at `opacity: 0.001` forever | A hidden breakpoint replica, not a failed reveal. Check which breakpoint's tree it belongs to; the visible one is a different node. |
| Content appears 3 times in served HTML | Breakpoint replicas. Normal. |
| Heading count looks wrong | Same. Count per breakpoint, not per document. |
| A section renders blank after a scripted jump-scroll | Scroll-triggered reveals may not fire on programmatic `scrollTo`. Scroll incrementally with delays, or check computed opacity directly. |
| Text style "duplicates" with no sizes | Framer preset breakpoint replicas. Never delete them. |

---

### What recurs, across every template

Check these first. Frequency is out of the three audited templates.

| Finding | Seen | Notes |
|---|---|---|
| Uploaded fonts used nowhere | 3/3 | Always PP Neue Montreal and Mono. Pangram Pangram is commercially licensed and travels with the remix. Confirm project-level vs workspace-level before reporting. |
| Multiple `<h1>` in served HTML | 3/3 | Hero authored per breakpoint. Real, but low severity: it is one visible H1. |
| Hover states active on touch breakpoints | 3/3 | Framer variants are usually done right; the leaks are in code components and overrides. Check `codeOverride` per breakpoint and any raw `:hover` CSS inside `.tsx` files for a `@media (hover: hover)` guard. |
| Non-descriptive names | 3/3 | Layers called `Text`, colour styles `01`-`05`, every button variant called `Default`. |
| Text style tier oddities | 3/3 | Inverted tablet/phone sizes, two identical styles, a tier that drops off a cliff. Print the full scale as a table; the oddities are obvious and invisible otherwise. |
| `100vh` sections | 3/3 | Test at 2560 wide and at tall viewports. Often fine. Record as Holding when it passes. |
| Multiple `<nav>` landmarks | 2/3 | Breakpoint replicas plus footer nav. Fix with `aria-label`, not deletion. |
| `<main>` landmark missing | 2/3 | Framer emits `<div id="main">` by default. It can be set properly. |
| `textWrapBalance` off | 2/3 | Count nodes; report the number. Text inside code components needs it set in the component. |
| Stray "Get started" frame on the home canvas | 2/3 | Never publishes, but ships with the remix. |
| Unused components | 2/3 | |
| Duplicated content in served HTML | 2/3 | Ticker tracks and hover-underline label duplicates. Genuine, unlike breakpoint replicas. |

---

### The checks

#### Costs you a sale

- Every link, clicked. Dead anchors (`href="#2120772874"`), footer links that drop their anchor, CTAs that self-link.
- Mobile menu opens, closes, and closes to the *correct* variant on every page type.
- Pricing copy: plan names consistent everywhere, no duplicated card headlines, no stale prices in hidden rows.
- Grammar in anything a visitor reads before scrolling.
- Responsive collisions and fixed-pixel widths on mobile headers.
- Terminal CTA has a real destination, not a self-link.

#### Worth doing before release

- Heading structure per breakpoint, not per document. Level skips, tags that change between breakpoints, quotes and prices tagged as headings.
- Landmarks: `main`, `nav`, `footer`, `section`.
- Alt text on every asset, canvas and CMS. Identical alt on different files is its own smell.
- Hover on touch, especially from code overrides.
- Ticker and heavy effects at the phone breakpoint.
- Forms: 16px inputs, ~55px height, real placeholders, only required fields required, submit states present.
- Type tiers, printed as a table.
- Dark mode token coverage, if a theme toggle ships.
- Retina: source width vs rendered width, target 2x. Hero backgrounds are the usual miss.
- Per-page titles, descriptions, OG images, `noindex` on `/404`, JSON-LD. Full SEO and AEO checks: see SEO and AEO below.

#### Cosmetic / hygiene

- Layer, variant, component, and token names.
- Unused components, code files, and exports.
- Stray canvas frames.
- Typos in shipped names (`Desctop`, `Butoon`, `Elivated` were all real).
- Components not in folders.

---

### SEO and AEO

Written Sep 30 2026 from official docs fetched that day. Search and AI-answer rules change fast: re-check the linked sources before relying on any line here, and never add a numeric rule (title length, word count) that a source does not state.

The model: AI answers (Google AI Overviews and AI Mode, ChatGPT search, Perplexity, Claude, Copilot) are built on the same foundations as search. Google says a page only needs to be indexed and eligible for a snippet, and that structured data, llms.txt and "AI rewrites" are not required ([AI features](https://developers.google.com/search/docs/appearance/ai-features), [AI optimization guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide)). So the job is: crawlable, rendered on the server, clearly described, specific content.

#### Canvas (Page Settings and site settings)

- Every page has its own title, description and 1200×630 social image. CMS detail pages fill them from fields (`{{Title}}`, `{{Excerpt}}`, cover image) ([Framer: CMS meta](https://www.framer.com/help/articles/how-can-i-add-meta-titles-and-descriptions-to-each-cms-item/)). Titles descriptive and distinct, descriptions unique summaries ([title links](https://developers.google.com/search/docs/appearance/title-link), [snippets](https://developers.google.com/search/docs/appearance/snippet)).
- "Search engines" stays on for every real page; off only for 404 and utility pages ([Framer: noindex](https://www.framer.com/help/articles/how-do-i-prevent-specific-pages-from-getting-indexed-by-search-engines/)). A leftover noindex on a real page ships to every buyer.
- No custom robots.txt that blocks Googlebot, Bingbot, OAI-SearchBot, PerplexityBot or Claude-SearchBot. Blocking OAI-SearchBot removes a site from ChatGPT search answers, blocking PerplexityBot from Perplexity results, blocking Claude-SearchBot reduces Claude search visibility ([OpenAI bots](https://developers.openai.com/api/docs/bots), [Perplexity crawlers](https://docs.perplexity.ai/docs/resources/perplexity-crawlers), [Anthropic crawlers](https://support.claude.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler)). Training crawlers (GPTBot, ClaudeBot, Google-Extended, Applebot-Extended) are a separate choice and do not affect search inclusion.
- JSON-LD via custom code in the head ([Framer: JSON-LD](https://www.framer.com/help/articles/structured-data-through-json-ld/)): Organization on the site, Article on blog CMS pages with `{{field}}` values, Breadcrumb where it fits. These are in Google's supported list ([search gallery](https://developers.google.com/search/docs/appearance/structured-data/search-gallery)). FAQ rich results stopped showing on May 7 2026 and HowTo is long gone ([updates](https://developers.google.com/search/updates)): never sell FAQ or HowTo markup as a feature.
- Meaningful images are real images with descriptive alt text, not CSS backgrounds; Google does not index CSS images ([image SEO](https://developers.google.com/search/docs/appearance/google-images)).
- Links are real links with descriptive anchor text ([crawlable links](https://developers.google.com/search/docs/crawling-indexing/links-crawlable)). A grid of "Read" or "Learn more" buttons with no descriptive text is a finding.

#### Served HTML

- The words are in the server HTML. Framer pre-renders pages ([Framer: AI agents](https://www.framer.com/help/articles/make-site-readable-by-ai-agents/)), but verify: Lapse once served /terms and /privacy as an empty shell. None of the AI crawler docs say whether they run JavaScript, so treat the served HTML as all they see.
- Per page: one `<title>`, a meta description, og:image, and a self-referencing absolute canonical on the custom domain ([canonicals](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls), [Framer: canonical](https://www.framer.com/help/articles/setting-up-a-custom-canonical-url-in-framer/)).
- `/sitemap.xml` lists every real page and CMS item and nothing noindexed ([sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap), [Framer: sitemap](https://www.framer.com/help/articles/how-can-i-access-the-sitemap-xml-file/)).
- Missing URLs return a real 404 status, not a 200 ([JS SEO basics](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics)).
- Machine-readable copy: `curl -H "Accept: text/markdown"` (or add `?md`) returns a Markdown version of an optimized Framer page ([Framer: AI agents](https://www.framer.com/help/articles/make-site-readable-by-ai-agents/)). Check it reads cleanly; it is what agents fetching the page get.

#### Content (what AI answers quote)

- Each page states plainly, near the top, what the product is, who it is for and what it costs. Answers are assembled from specific, "non-commodity" text, not slogans ([AI optimization guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide)).
- Headings describe their section. FAQ answers are visible text, one question per item.
- Blog posts carry an author, a date and a clear title; About names real (or clearly placeholder) people ([helpful content](https://developers.google.com/search/docs/fundamentals/creating-helpful-content)).
- Placeholder copy is obviously placeholder, so buyers replace it rather than ship it.

#### Buyer guide

The Getting started page gets an SEO and AEO card: fill page titles, descriptions and social images; keep "Search engines" on for real pages; connect the custom domain; add the site to Google Search Console (its "Search generative AI" setting is on by default) and Bing Webmaster Tools; do not block OAI-SearchBot, PerplexityBot or Claude-SearchBot if the site should appear in AI answers.

#### Not needed (do not recommend)

- llms.txt: Google says it ignores it, and OpenAI, Anthropic and Perplexity publish no support for it. Framer lets you upload one ([Framer: llms.txt](https://www.framer.com/help/articles/llms-txt-framer/)); optional, never a finding.
- Rewriting or "chunking" content for AI, pages per query variant (Google calls that scaled content abuse), fixed word counts or page lengths ([AI optimization guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide)).
- Sitemap priority and changefreq (Google ignores them), meta keywords.

#### Severity

Costs you a sale: noindex on a real page, a robots.txt that blocks search or AI search crawlers, content missing from the served HTML. Worth doing before release: titles, descriptions, social images, canonicals, sitemap, 404 status, JSON-LD, alt text, anchor text, the buyer-guide card. Cosmetic: Markdown output tidiness.

### Beyond the site

Four checks added Sep 2026 from Framer's template best-practices list (https://www.framer.com/help/articles/template-best-practices/). They cover what a buyer meets after the remix, which the three lenses above do not.

1. **Buyer guide.** Every template ships a "Getting started" editor-only Design page: pages and shared layout, color and text styles, CMS fields and filters, form Send To, SEO and AEO setup, and every code-driven section with what must not be renamed (element IDs, fixed item counts). Build it with the template's own styles, describe only what you inspected, and never add a public route. Lapse was the first (Sep 30 2026). **Open design task:** the current layout (a plain 2-column card grid) is a placeholder until a designed, reusable guide layout exists. Until then, write the content and keep the layout minimal.
2. **Licences.** List every code file's origin and licence from its header, plus uploaded fonts. MIT + Commons Clause (React Bits, Canvas UI) forbids redistribution and travels with every remix; commercial fonts left uploaded but unused travel too. Report as Needs input, never delete or replace unasked.
3. **Originality and fit.** One line on who the template is for and what it does beyond native Framer. If the answer is thin, flag it before the listing is written, not after.
4. **Support and listing facts.** Contact route, refund terms, and requirements (CMS plan, WebGPU fallbacks) are stated in the listing and match the template. Assess only from evidence; never invent terms.

### Reporting

Structure that worked across all three:

1. **Header block**: project id, live URL, date, method, scope note.
2. **Scoreboard**: counts per severity tier.
3. **Findings**, numbered `N.N` so they can be referenced in the fix pass. Each one: what, evidence, and a **Fix** line where the fix is short.
4. **Holding table**: everything verified clean.
5. **Order of work**: impact per minute, cheapest high-impact first.

Rules:

- Every finding carries evidence: a count, a measurement, a code fragment, or a served-HTML quote. "Looks off" is not a finding.
- Number them. Re-checks and fix passes both need stable references.
- Say when a finding is inference rather than proof, and say what would confirm it.
- Correct your own earlier findings explicitly in a re-check. Two of the three audits had a "corrections to earlier findings" section, and both were the most useful part of the re-check.
- Record what you closed at the owner's call, so it stops coming back.

### Verifying fixes

Do not trust a write that returned 200. On this project a Resend PATCH returned 200 while silently doing nothing, and a Framer `applyChanges` returned no errors while pruning the node it had just created.

Re-read the thing you changed, from a fresh fetch, and compare it against what you intended. Where possible compare hashes, not substrings: a substring check passed on both the old and new layout and let a broken update through on two of three records.

When an edit is applied but cannot be fully tested (motion, touch taps, anything the harness cannot drive), report it as **"Changed — verification pending"** with the test still needed. Never mark it fixed. Keep the previous value of every edit so your own change can be reverted alone.
