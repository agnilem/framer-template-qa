# Framer template QA playbook

How to audit a template before it ships. Built from three real audits: Cohestra (12 pages, CMS heavy), Pillo (3 pages, effects heavy), Elias Vent (portfolio, code components).

---

## The angle changed

Framer removed the marketplace review. Templates publish immediately, no gatekeeper.

That kills the old framing. The Master Guide is written around "what gets you rejected", and ranking findings by rejection risk is now measuring against a thing that does not exist. **Never write "a reviewer would flag this" in an audit.** It is not a reason any more.

What replaced it: **what does a paying buyer hit, and in what order.** A buyer opens the preview on their phone, taps around, buys, remixes, and starts editing. Every finding gets ranked by where it sits on that path.

The Master Guide is still the best inventory of *what to look at*. It is no longer the right authority on *how much anything matters*. Use it as a checklist source, not a severity model.

### Severity tiers

| Tier | Meaning |
|---|---|
| **Costs you a sale** | A visitor or buyer hits it without looking for it. Broken nav, dead CTA, duplicated pricing copy, grammar in a headline. |
| **Worth doing before release** | Real, but the buyer only meets it after purchase, or it degrades quality rather than blocking. Semantics, touch hover, type tiers. |
| **Cosmetic / hygiene** | Only the buyer editing the file sees it. Layer names, unused components, stray frames. |
| **Holding** | Verified clean. Record these so the next pass does not re-litigate them. |

The Holding table matters more than it looks. Three audits in, most re-check time was spent re-proving things that were already fine.

---

## Method: three lenses, in this order

A finding is only real if it survives the lens that can actually see it. Most bad findings come from using the wrong one.

### Lens 1: the canvas, through the agent API

Structure, attributes, components, styles, tokens. This is where you find things the browser cannot show you: unused components, orphan tokens, uploaded-but-unused fonts, stray off-canvas frames, variant wiring.

Dump the whole tree once and analyse locally rather than making dozens of round trips:

```javascript
const tree = await framer.agent.serialize({ id: "<pageId>", depth: 200 }, { pagePath: "/" });
require('fs').writeFileSync('/tmp/page.json', JSON.stringify(tree));
```

Then walk the JSON with a local script. Fast, repeatable, and you can diff it after a fix.

Useful sweeps: attribute-key inventory (tells you what exists at all), `htmlTag` counts, `tag` sequences for headings, `fill` values that are not `var(--token-`, `codeOverride` presence per breakpoint, `visible:false` nodes, generic layer names.

### Lens 2: the served HTML

`curl` the published pages. This is the only place you see what actually ships: how many `<h1>` survive, landmark counts, real link hrefs, duplicated content, meta tags, JSON-LD.

**The single biggest trap:** Framer emits one copy of the markup per breakpoint. Three `<h1>`, three `<nav>`, three `<footer>` is usually **one** authored element rendered three times, not three mistakes. Always divide by the breakpoint count before calling something duplicated. Conversely, content that appears 6 times when there are 3 breakpoints is genuinely doubled.

### Lens 3: the live browser

Anything positional, interactive, or viewport-dependent. Hover, menus, scroll effects, responsive collisions.

**Measure, do not eyeball.** The Pillo phone-position bug was only provable by reading `getBoundingClientRect` at two viewport heights and showing the offset grew with height. A screenshot would have produced an argument; the numbers produced a diagnosis.

```javascript
const r = el.getBoundingClientRect();
({ top: r.top, center: r.top + r.height/2, vpCenter: innerHeight/2,
   delta: r.top + r.height/2 - innerHeight/2 })
```

Run it at 900 and 1400 height. If `delta` changes, the element is anchored to something viewport-dependent.

---

## Traps that produce false findings

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

## What recurs, across every template

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

## The checks

### Costs you a sale

- Every link, clicked. Dead anchors (`href="#2120772874"`), footer links that drop their anchor, CTAs that self-link.
- Mobile menu opens, closes, and closes to the *correct* variant on every page type.
- Pricing copy: plan names consistent everywhere, no duplicated card headlines, no stale prices in hidden rows.
- Grammar in anything a visitor reads before scrolling.
- Responsive collisions and fixed-pixel widths on mobile headers.
- Terminal CTA has a real destination, not a self-link.

### Worth doing before release

- Heading structure per breakpoint, not per document. Level skips, tags that change between breakpoints, quotes and prices tagged as headings.
- Landmarks: `main`, `nav`, `footer`, `section`.
- Alt text on every asset, canvas and CMS. Identical alt on different files is its own smell.
- Hover on touch, especially from code overrides.
- Ticker and heavy effects at the phone breakpoint.
- Forms: 16px inputs, ~55px height, real placeholders, only required fields required, submit states present.
- Type tiers, printed as a table.
- Dark mode token coverage, if a theme toggle ships.
- Retina: source width vs rendered width, target 2x. Hero backgrounds are the usual miss.
- Per-page titles, descriptions, OG images, `noindex` on `/404`, JSON-LD.

### Cosmetic / hygiene

- Layer, variant, component, and token names.
- Unused components, code files, and exports.
- Stray canvas frames.
- Typos in shipped names (`Desctop`, `Butoon`, `Elivated` were all real).
- Components not in folders.

---

## Beyond the site

Four checks added Sep 2026 from Framer's template best-practices list (https://www.framer.com/help/articles/template-best-practices/). They cover what a buyer meets after the remix, which the three lenses above do not.

1. **Buyer guide.** Every template ships a "Getting started" editor-only Design page: pages and shared layout, color and text styles, CMS fields and filters, form Send To, and every code-driven section with what must not be renamed (element IDs, fixed item counts). Build it with the template's own styles, describe only what you inspected, and never add a public route. Lapse was the first (Sep 30 2026). **Open design task:** the current layout (a plain 2-column card grid) is a placeholder until a designed, reusable guide layout exists. Until then, write the content and keep the layout minimal.
2. **Licences.** List every code file's origin and licence from its header, plus uploaded fonts. MIT + Commons Clause (React Bits, Canvas UI) forbids redistribution and travels with every remix; commercial fonts left uploaded but unused travel too. Report as Needs input, never delete or replace unasked.
3. **Originality and fit.** One line on who the template is for and what it does beyond native Framer. If the answer is thin, flag it before the listing is written, not after.
4. **Support and listing facts.** Contact route, refund terms, and requirements (CMS plan, WebGPU fallbacks) are stated in the listing and match the template. Assess only from evidence; never invent terms.

## Reporting

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

## Verifying fixes

Do not trust a write that returned 200. On this project a Resend PATCH returned 200 while silently doing nothing, and a Framer `applyChanges` returned no errors while pruning the node it had just created.

Re-read the thing you changed, from a fresh fetch, and compare it against what you intended. Where possible compare hashes, not substrings: a substring check passed on both the old and new layout and let a broken update through on two of three records.

When an edit is applied but cannot be fully tested (motion, touch taps, anything the harness cannot drive), report it as **"Changed — verification pending"** with the test still needed. Never mark it fixed. Keep the previous value of every edit so your own change can be reverted alone.
