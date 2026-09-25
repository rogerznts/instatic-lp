---
name: instatic-lp
description: Build, edit and publish landing pages on an Instatic site through its MCP tools (site_add_page, site_insert_html, site_apply_css, site_set_*_tokens, site_render_snapshot, site_publish). Use when the user asks to create or change a landing page, LP, page section (hero, pricing, FAQ, footer), page styling or design tokens on Instatic, or says "criar uma LP no instatic", "montar a página", "publicar a landing".
---

# Instatic landing pages

You drive the Instatic visual editor over MCP. Edits land in the **draft**; nothing goes live until `site_publish`. If the Instatic tools are missing or a tool says "open the workspace", use the `instatic-setup` skill first.

Tool names below are bare (`site_insert_html`); in the client they are prefixed with the server name (e.g. `mcp__instatic__site_insert_html`).

## 0. Brief

Before building, make sure you know: goal of the page (one primary CTA), audience, sections wanted, copy source (provided or to write), slug, and whether it should be published at the end. If the project has a brand file (see [references/brand-template.md](references/brand-template.md) — commonly `instatic-brand.md` at the project root), read it and follow it. Ask only for what is missing; do not interrogate.

## 1. Orient (headless, cheap)

1. `get_context` — is the Site editor connected? Which templates wrap pages? An **everywhere** template already renders nav/footer on every page: do **not** rebuild them inside the page. With no everywhere template, each page is standalone and must include its own nav and footer.
2. `site_list_documents` — existing pages/templates and their ids. Never invent ids.
3. `site_read_styles` + `site_list_breakpoints` — current design tokens, classes and breakpoint widths.

If the editor is not connected, stop and ask the user to open the Instatic editor (Site workspace) logged in as the token owner.

## 2. Design system first

Consistency comes from tokens, not literals.

- If tokens are missing or don't match the brand, create them: `site_set_color_tokens` (→ `var(--<slug>)`), `site_set_font_tokens` (pass `googleFamily` to install a web font), `site_set_type_scale` (→ `--text-*`), `site_set_spacing_scale` (→ `--space-*`). They are create-or-update, safe to re-run.
- In all CSS, reference tokens: `color: var(--primary)`, `font-size: var(--text-l)`, `gap: var(--space-m)`. No raw hex, no raw px for type/spacing, no raw `font-family` when a token exists or should exist.
- Reuse existing classes from `site_read_styles` before inventing new ones. Prefix new page-specific classes (e.g. `lp-hero`, `lp-card`) so they don't collide with site-wide ones.

## 3. Create the page

`site_add_page` → returns `pageId` and `rootNodeId`. Call it **once** (slugs are auto-uniqued; a second call makes a second page). Build into **`rootNodeId`**, not `pageId`. Homepage = slug `index` (set via `site_rename_page`).

To iterate on an existing page instead, `site_duplicate_page` it or edit in place (step 6).

## 4. Build section by section

One `site_insert_html` call per section (nav, hero, social proof, features, pricing, FAQ, CTA, footer → ~5–8 calls), each with `parentId = rootNodeId`. Smaller chunks recover better when one fails.

Each call carries the section's semantic HTML **plus its CSS in a `<style>` block** in the same call:

```html
<style>
  .lp-hero { padding: var(--space-3xl) var(--space-l); background: var(--surface); }
  .lp-hero h1 { font-family: var(--font-heading); font-size: var(--text-3xl); }
  @media (max-width: 767px) { .lp-hero { padding: var(--space-xl) var(--space-m); } }
</style>
<section class="lp-hero">
  <h1>…</h1>
  <p>…</p>
  <a class="btn btn--primary" href="#contato">…</a>
</section>
```

Rules:

- Semantic tags: `<header> <nav> <main> <section> <h1>…<h3> <p> <a> <button> <img> <ul> <article> <footer>`. Exactly one `<h1>` per page. Every `<img>` has `alt`.
- A bare `.foo {}` selector becomes a reusable class; anything else (`.lp-hero h1`, `a:hover`) becomes an ambient rule.
- `<script>` and inline handlers (`onclick`, …) are **stripped**. For behaviour (menu toggle, tabs, accordion) use `site_write_code_asset` with `type:"script"` and `path:"src/scripts/…"`; confirm with `site_inspect_code_runtime`. Prefer CSS-only solutions (`<details>` for FAQ) when possible.
- Responsive from the start: `@media` queries aligned with the widths from `site_list_breakpoints`. Don't invent breakpoints.
- Repeated items (cards, testimonials): insert one, then `site_duplicate_node` and edit, instead of re-inserting.
- Forms: build with real `<form>`, `<label>`, `<input>`; ask the user where submissions should go if unclear.

## 5. Images

- If the connection has `media.write`, `media_upload` with an https `sourceUrl` (or base64 `data`) returns an id and `publicPath` — use `publicPath` in `<img src>`. This makes the page self-contained.
- Otherwise, reuse existing assets (`content_list_media`) or reference external https URLs as placeholders, and tell the user which images to replace via the Instatic UI.
- Never hotlink images you don't have rights to in a page meant for production.

## 6. Editing existing content

`site_read_document` (follow `pageInfo.nextPart` until you have what you need) → every element carries `uid="<nodeId>"`. Then:

- copy/attrs → `site_update_node_props`
- restructure a block → `site_get_node_html` then `site_replace_node_html`
- CSS only → `site_apply_css` with an explicit operation: `merge` (additive, default), `remove-properties`, `replace` (full desired CSS for those selectors), `delete`. Before replace/delete/remove, read the document and copy the selector **exactly**.
- move/rename/delete nodes → `site_move_node` / `site_rename_node` / `site_delete_node`

Shared chrome (nav/footer/theme) usually lives in a template — check templates before editing each page.

## 7. Verify before calling it done

For each breakpoint (`site_render_snapshot` with `breakpointId`, full page or `nodeId` for a section):

- no overflow/visibility warnings, all images `loaded`
- CTA visible above the fold on the smallest breakpoint
- text contrast and hierarchy consistent with tokens

The screenshot may be absent depending on the client; the JSON layout report (geometry, warnings, computed styles) is authoritative. To debug a cascade issue, compare `layout.nodes[].computed` with the source CSS from `site_read_document`. Fix and re-snapshot; don't declare success on a page you haven't inspected.

## 8. Publish only when asked

`site_publish` deploys the **whole site draft** (not only this page) to the live site. Call it **once**, only when the user explicitly asked to publish, after verification. If other people edit the same site, warn that their pending draft changes will go live too. After publishing, give the user the public URL (`https://<host>/<slug>`).

## Reply style

Narrate what changed in a few sentences (page, sections, tokens created, open issues, what still needs a human — e.g. images to upload). Don't paste the raw HTML/CSS back.

See [references/tools.md](references/tools.md) for the full tool catalog and required capabilities.
