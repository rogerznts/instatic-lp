---
name: instatic-lp
description: Build, edit and publish landing pages on an Instatic site through its MCP tools (site_read_styles, site_add_page, site_insert_html, site_apply_css, site_render_snapshot, site_publish), always styled with the colors, fonts and type/spacing scales already configured in the site's Instatic Framework panel. Starts from a reference URL when given one: analyzes it, grills the user on what to build, offers design styles (minimalist, soft, brutalist, redesign) adapted from taste-skill, then builds. Use when the user asks to create or change a landing page, LP, page section (hero, pricing, FAQ, footer) or page styling on Instatic, sends a reference URL for a page, asks for a style/redesign, or says "criar uma LP no instatic", "montar a página", "nova página", "usa essa página como referência", "publicar a landing".
---

# Instatic landing pages

You drive the Instatic visual editor over MCP. Edits land in the **draft**; nothing goes live until `site_publish`. If the Instatic tools are missing or a tool says "open the workspace", use the `instatic-setup` skill first.

Tool names below are bare (`site_insert_html`); in the client they are prefixed with the server name (e.g. `mcp__instatic__site_insert_html`).

Flow: **orient → framework inventory → intake (reference URL + grill) → style direction → page spec → build → verify → publish on request.** Steps 1–5 touch nothing on the site; the first write is step 6.

## Tool availability

A token with every capability exposes **53 tools**: context and media (2: `get_context`, `media_upload`), Content collections and documents (15), Site pages, nodes, CSS, tokens, code assets, inspection and publishing (36). The list is filtered by the token's capabilities, so check that the tools the job needs are present before planning:

| Job | Needs |
|---|---|
| Orient + read the framework | `get_context`, `site_list_documents`, `site_read_styles`, `site_list_breakpoints` |
| Build/edit a page | `site_add_page`, `site_insert_html`, `site_read_document`, `site_update_node_props`, `site_replace_node_html`, `site_apply_css` |
| Change the framework (only on explicit request) | `site_set_color_tokens`, `site_set_font_tokens`, `site_set_type_scale`, `site_set_spacing_scale` |
| Behaviour (menu, reveal on scroll) | `site_write_code_asset`, `site_inspect_code_runtime` |
| Images | `media_upload` (else `content_list_media` + placeholders, step 8) |
| Verify | `site_render_snapshot` |
| Go live | `site_publish` |
| CMS entries (blog, cases) | `content_*` — field writes only on the active document: `content_set_active_document` first |

A missing tool means the token lacks its capability, not that the feature doesn't exist: tell the user which capability to add (see [references/tools.md](references/tools.md)); the only built-in fallback is images (step 8). Headless tools (reads, `media_upload`, `site_publish`) work without a browser; every page/CSS/token edit needs the Site editor open as the token owner.

## 1. Orient (headless, cheap)

1. `get_context` — is the Site editor connected? Which templates wrap pages? An **everywhere** template already renders nav/footer on every page: do **not** rebuild them inside the page. With no everywhere template, each page is standalone and must include its own nav and footer.
2. `site_list_documents` — existing pages/templates and their ids. Never invent ids.
3. `site_read_styles` (headless, no `className`, `includeTokens` default) + `site_list_breakpoints` — the configured framework (colors, fonts, type and spacing scales), existing classes and breakpoint widths. Mandatory before writing any CSS.
4. Brand file: if the project has one (see [references/brand-template.md](references/brand-template.md), commonly `instatic-brand.md` at the project root), read it for voice, rules, the framework role mapping and the last style used.

If the editor is not connected, carry on with steps 2–5 (they are reads and conversation) and ask the user to open the Instatic editor (Site workspace, logged in as the token owner) before step 6.

## 2. Framework is the design source of truth

The **Framework** panel in the Instatic editor (Colors, Type, Space) is what `site_read_styles` returns under `/* === Design tokens === */`. Every new page reuses it as-is, so all pages share the same palette, typefaces and rhythm. Styles (step 4) change the *treatment* of these tokens, never the tokens.

Read the token block and build an inventory before writing CSS:

| Framework tab | CSS emitted | Use as |
|---|---|---|
| Colors | `--<slug>`, plus generated steps `--<slug>-<n>`, shades `--<slug>-d-<n>`, tints `--<slug>-l-<n>`; dark-mode overrides in a theme scope | `color`, `background`, `border-color`, gradients, shadows |
| Type — fonts | font tokens with the names the site gave them (e.g. `--font-display` for headings, `--font-body` for body) | `font-family` |
| Type — scale | `--text-<step>` (fluid `clamp()`; default steps `xs … 4xl`, base `m`) | `font-size` |
| Space | `--space-<step>` (fluid `clamp()`; default steps `4xs … 4xl`, base `m`) | `padding`, `margin`, `gap` |

Names above are the defaults; **use the exact names found in the CSS**, never guessed ones. Map roles from what exists: font token whose name says display/heading → `h1–h3`; body → text; color slugs by name/value (dark neutral → text, light neutral → surface, brand/accent → CTA). If a role is ambiguous (e.g. two candidate CTA colors), ask once, then write the mapping into the brand file so the next page doesn't ask again.

Rules:

- **Only framework values.** `color`, `background*`, `border-color`, `fill`, `stroke`, shadows → `var(--<color token>)`. `font-family` → `var(<font token>)`. `font-size` → `var(--text-*)`. `padding`/`margin`/`gap` → `var(--space-*)`. No raw hex/rgb/hsl, no named colors, no `font-family` literal, no px/rem for type or spacing. Allowed literals: `0`, `auto`, `100%`, `transparent`, `currentColor`, line-height, letter-spacing, font-weight, border widths, radii, max-widths, durations/easings and layout (`fr`, `%`, `vw`, `ch`).
- Need a lighter/darker/transparent version of a color? Use the generated step/shade/tint variable if it exists, else `color-mix(in srgb, var(--<slug>) 20%, transparent)`. Never a new hex.
- Size between two steps? Pick the nearest step. Don't `calc()` new sizes out of the scale except for simple multiples (`calc(var(--space-m) * -1)`).
- **Never change the framework on your own.** `site_set_color_tokens`, `site_set_font_tokens`, `site_set_type_scale` and `site_set_spacing_scale` rewrite the site-wide framework: every page restyles. Use them only when the user explicitly asks to change the framework, or when the framework is empty (no design-token block) and the user approved creating one. If the design seems to need something the framework lacks (a new color, a third font), stop and ask; propose the token instead of improvising a literal.
- Don't install fonts (`googleFamily`) for a page; use the installed font tokens.
- Reuse existing classes from `site_read_styles` before inventing new ones (they already speak the framework). Framework utility classes (`.text-*`, color/spacing utilities) may be tree-shaken out of the `site_read_styles` output; the variables are always there, so style with the variables.

## 3. Intake: reference URL + grill

- **User sent a reference URL** (or a screenshot / an existing page): follow [references/reference-intake.md](references/reference-intake.md). Read the reference, write the *Reference read*, then grill the user one decision at a time, each question with your recommended answer, until the page is fully decided.
- **No reference**: same grill, shorter. Skip every branch the brand file, the framework or the user's message already answers.

The grill decides goal, audience, offer, sections, copy, assets, slug, template and publishing. Visual direction is decided in step 4.

## 4. Style direction

Follow [references/styles/router.md](references/styles/router.md):

1. State the one-line **Design read** and the three dials (variance, motion, density).
2. **Offer the styles** that fit, recommend one, and say what each would look like *with this site's framework* (and which framework gaps it would hit):
   - **Minimalist** — editorial, warm monochrome, hairlines, flat bento. [styles/minimalist.md](references/styles/minimalist.md)
   - **Soft** — high-end agency: nested "double-bezel" cards, pill CTAs, generous air, fluid motion. [styles/soft.md](references/styles/soft.md)
   - **Brutalist** — Swiss print or terminal: rigid grid, visible rules, zero radius, huge type contrast. [styles/brutalist.md](references/styles/brutalist.md)
   - **Redesign** — evolve an existing page (the user's own reference or a page on this site): audit, then targeted upgrades; can combine with one of the above. [styles/redesign.md](references/styles/redesign.md)
3. After the choice, read the chosen style file(s). Their rules apply on top of step 2: a style can never introduce a color, font or size outside the framework.

## 5. Page spec — confirm before building

Summarize everything in the **Page spec** (template in [references/reference-intake.md](references/reference-intake.md#page-spec)): slug, template, goal and single CTA label, audience, design read, style + dials, role binding (which framework token plays each role), section plan (order, layout family, content, assets), copy language/voice, images plan, form destination, publish yes/no. Build only after the user says yes. Save the role binding and the chosen style into the brand file (create it from the template if missing, with the user's OK) so the next page starts from the same decisions.

## 6. Create the page

`site_add_page` → returns `pageId` and `rootNodeId`. Call it **once** (slugs are auto-uniqued; a second call makes a second page). Homepage = slug `index` (set via `site_rename_page`).

To iterate on an existing page instead, `site_duplicate_page` it or edit in place (step 9).

**Page scope.** Pick a short page key (e.g. `sol` for `/solar`). The first insert, under `rootNodeId`, is the page wrapper that binds framework tokens to style roles and sets page defaults. Class names are site-wide: prefix every new class with the key (`lp-sol-hero`) so another page's `.lp-hero` can't restyle this one.

```html
<style>
  .lp-sol {
    --lp-bg: var(--white);            /* role → framework token, from the page spec */
    --lp-surface: var(--gray-100);
    --lp-ink: var(--black);
    --lp-muted: color-mix(in srgb, var(--black) 62%, var(--white));
    --lp-line: color-mix(in srgb, var(--black) 12%, transparent);
    --lp-accent: var(--gold);
    --lp-accent-ink: var(--black);
    --lp-font-display: var(--font-display);
    --lp-font-body: var(--font-body);
    background: var(--lp-bg); color: var(--lp-ink); font-family: var(--lp-font-body);
  }
</style>
<div class="lp-sol"></div>
```

Token names above are examples; bind to the names `site_read_styles` returned. Style files are written against these `--lp-*` roles plus the framework `--text-*` / `--space-*` scales. Every section below goes into the wrapper (`parentId` = the wrapper's node id from the insert result).

## 7. Build section by section

One `site_insert_html` call per section (nav, hero, social proof, features, pricing, FAQ, CTA, footer → ~5–8 calls), each with `parentId` = wrapper id. Smaller chunks recover better when one fails. Build in the order of the page spec, applying the chosen style file.

Each call carries the section's semantic HTML **plus its CSS in a `<style>` block** in the same call:

```html
<style>
  .lp-sol-hero { padding: var(--space-3xl) var(--space-l); background: var(--lp-surface); }
  .lp-sol-hero h1 { font-family: var(--lp-font-display); font-size: var(--text-3xl); }
  .lp-sol-hero p { font-size: var(--text-m); color: var(--lp-muted); max-width: 60ch; }
  @media (max-width: 767px) { .lp-sol-hero { padding: var(--space-xl) var(--space-m); } }
</style>
<section class="lp-sol-hero">
  <h1>…</h1>
  <p>…</p>
  <a class="lp-sol-cta" href="#contato">…</a>
</section>
```

Rules:

- Semantic tags: `<header> <nav> <main> <section> <h1>…<h3> <p> <a> <button> <img> <ul> <article> <footer>`. Exactly one `<h1>` per page. Every `<img>` has `alt`.
- A bare `.foo {}` selector becomes a reusable class; anything else (`.lp-sol-hero h1`, `a:hover`) becomes an ambient rule. `@media` (including `prefers-reduced-motion`), `@supports`, `@container` and `@keyframes` are kept.
- `<script>` and inline handlers (`onclick`, …) are **stripped**. For behaviour (menu toggle, tabs, reveal on scroll) use `site_write_code_asset` with `type:"script"` and `path:"src/scripts/…"`; confirm with `site_inspect_code_runtime`. Prefer CSS-only solutions (`<details>` for FAQ, scroll-driven animations) when possible.
- Responsive from the start: `@media` queries aligned with the widths from `site_list_breakpoints`. Don't invent breakpoints. Every multi-column section declares its mobile collapse in the same call.
- Repeated items (cards, testimonials): insert one, then `site_duplicate_node` and edit, instead of re-inserting.
- Forms: build with real `<form>`, `<label>`, `<input>`; ask the user where submissions should go if unclear.

## 8. Images

- If the connection has `media.write`, `media_upload` with an https `sourceUrl` (or base64 `data`) returns an id and `publicPath` — use `publicPath` in `<img src>`. This makes the page self-contained. If an image-generation tool is available in the client, generate section-specific images and upload them.
- Otherwise, reuse existing assets (`content_list_media`) or reference external https URLs as placeholders, and tell the user which images to replace via the Instatic UI.
- Never hotlink images you don't have rights to in a page meant for production. Images from a third-party reference URL are **never** uploaded or hotlinked; they only inform what kind of image goes where.

## 9. Editing existing content

`site_read_document` (follow `pageInfo.nextPart` until you have what you need) → every element carries `uid="<nodeId>"`. Then:

- copy/attrs → `site_update_node_props`
- restructure a block → `site_get_node_html` then `site_replace_node_html`
- CSS only → `site_apply_css` with an explicit operation: `merge` (additive, default), `remove-properties`, `replace` (full desired CSS for those selectors), `delete`. Before replace/delete/remove, read the document and copy the selector **exactly**.
- move/rename/delete nodes → `site_move_node` / `site_rename_node` / `site_delete_node`

Shared chrome (nav/footer/theme) usually lives in a template — check templates before editing each page.

## 10. Verify before calling it done

For each breakpoint (`site_render_snapshot` with `breakpointId`, full page or `nodeId` for a section):

- no overflow/visibility warnings, all images `loaded`
- CTA visible above the fold on the smallest breakpoint
- text contrast and hierarchy consistent with tokens; `layout.nodes[].computed` colors and font families match framework values (a mismatch means a literal or a stray rule slipped in)
- the **pre-flight** in [references/styles/router.md](references/styles/router.md#pre-flight) plus the chosen style's checklist

The screenshot may be absent depending on the client; the JSON layout report (geometry, warnings, computed styles) is authoritative. To debug a cascade issue, compare `layout.nodes[].computed` with the source CSS from `site_read_document`. Fix and re-snapshot; don't declare success on a page you haven't inspected.

## 11. Publish only when asked

`site_publish` deploys the **whole site draft** (not only this page) to the live site. Call it **once**, only when the user explicitly asked to publish, after verification. If other people edit the same site, warn that their pending draft changes will go live too. After publishing, give the user the public URL (`https://<host>/<slug>`).

## Reply style

Talk to the user in their language. Narrate what changed in a few sentences (page, sections, style applied, open issues, what still needs a human — e.g. images to upload). Don't paste the raw HTML/CSS back.

See [references/tools.md](references/tools.md) for the full tool catalog and required capabilities.
