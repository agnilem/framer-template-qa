---
name: framer-template-qa
description: QA a Framer template before it goes on the Framer Marketplace. Inspect the canvas, the served HTML and the live site, fix clear defects in the editor, add a buyer Getting started guide, and report findings ranked by what a buyer would hit. Use when asked to QA, audit, check, or prepare a Framer template for the Marketplace or for sale.
---

# Framer template QA

Audit a Framer template the way a buyer meets it: preview on a phone, remix, then edit. Fix what is clearly broken, leave decisions to the owner, and report with evidence.

The full method lives in `references/playbook.md`. Read it before starting. It holds the severity model, the three lenses, the false-finding traps, the checks, and the report format.

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
5. **Beyond the site.** Licences of code files and fonts, originality and fit, support and listing facts, and the buyer guide (see the playbook section "Beyond the site").
6. **Fix** confirmed defects with clear solutions: spelling, overflow, broken links with an unambiguous destination, CMS bindings, alt text, form labels, style inconsistencies. Re-read each change after writing it and check all three breakpoints.
7. **Buyer guide.** Add a "Getting started" editor-only Design page (never a public route) covering pages and shared layout, styles, CMS fields and filters, form Send To, and every code-driven section with what must not be renamed. Describe only what you inspected. Keep the layout minimal until a designed guide layout exists.
8. **Report** in this order: Fixed and verified · Changed, verification pending · Needs your input · Optional improvements · Verification coverage (passed vs not tested). Every finding carries evidence. Never claim the whole template is ready because a subset passed.
