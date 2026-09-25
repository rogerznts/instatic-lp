# Style: Redesign (evolve an existing page)

Adapted from taste-skill `redesign-skill` (`redesign-existing-projects`) and the router's redesign protocol (Leonxlnx/taste-skill, MIT). Not an aesthetic: an audit-then-upgrade protocol for a page the user owns. Pair it with one aesthetic (minimalist, soft, brutalist) when the user wants a new visual language; alone, it improves what's there within its current language.

## Mode (decide in the offer; ask once if unclear)

| Mode | When | Result |
|---|---|---|
| **Preserve** | IA, content and SEO are sound; the page just looks dated or generic | targeted upgrades in the current language; dials: match existing, motion +1 |
| **Overhaul** | visual debt is structural, or the user wants a new look | new visual language (paired style) on the same content and IA; dials +2 variance/motion |
| **New page** | the brand/offer itself is changing | not a redesign: treat as a new page with the reference as inspiration |

## Where the work happens
- **Page on this Instatic site:** `site_duplicate_page` → work on the copy → compare snapshots → on approval, swap slugs with `site_rename_page` (slug changes need explicit approval; see "never silently"). Or edit in place if the user asks.
- **Page elsewhere (the user's old site):** build a new Instatic page from its content (reference intake, own content), then apply the upgrades.

## Scan
Read the current page (`site_read_document` for Instatic pages; the reference read otherwise) and record: the framework tokens it actually uses vs. literals, the section list with jobs, conversion paths (CTAs, forms, anchors, field names), what is signature and must stay, SEO basics (title, description, headings, slugs), and a dial reading of the current page.

## Diagnose (audit checklist)

**Typography** — headings without presence (too small, loose tracking/leading); body lines wider than ~65ch; only 400/700 weights; numbers not tabular in data; all-caps subheads everywhere; orphans (`text-wrap: balance` on headings, `pretty` on paragraphs).

**Color and surfaces** — literals instead of framework tokens; more than one accent; warm and cool grays mixed; pure black/white where softer tokens exist; generic black shadows (tint them); a random inverted section; flat empty bands with no image or texture.

**Layout** — everything centered and symmetric; three equal cards; `100vh` (use `100dvh`); no max-width container (aim ~1200–1440px); forced equal-height cards; same radius on everything (tighter inside, softer outside); no overlap or depth; identical top/bottom padding where the eye wants more below; CTAs not bottom-aligned in card groups; misaligned titles/prices/lists across columns.

**Interaction** — missing hover, `:active` and `:focus-visible`; instant transitions; dead `#` links; no current-page state in nav; anchor jumps without `scroll-behavior: smooth`; animations on `top/left/width/height`.

**Content** — generic names and fake round numbers; clichés (see router); exclamation-heavy or "Oops!" messages; passive voice; Lorem ipsum; Title Case On Every Heading; em/en dashes in copy.

**Components** — generic card (border + shadow + white) everywhere; always one filled + one ghost button; pill "Novo" badges; 3-card testimonial carousel with dots; pricing as three equal towers; 4-column footer link farm.

**Code and omissions** — div soup instead of semantic tags; inline `style=` mixed with classes; missing/meaningless `alt`; arbitrary z-index; missing legal links (privacy, terms), skip link, form validation, meta title/description.

List the findings to the user grouped by severity before changing anything big.

## Fix (priority order, stop when the brief is met)
1. **Typographic treatment** — scale steps, weight (introduce 500/600), tracking, leading, measure. The typeface itself is framework-level: suggesting a font change is a framework change (ask; never `site_set_font_tokens` on your own).
2. **Color cleanup** — replace literals with framework tokens via the role binding, one accent, unified neutrals, tinted shadows.
3. **Hover / active / focus states.**
4. **Layout and spacing** — container, grid, rhythm from the space scale, alignment across columns.
5. **Replace generic components** with the router's alternatives (and the paired style's components).
6. **Missing states** — form errors inline, empty/loading where relevant.
7. **Polish** — type scale fine-tuning, optical alignment, motion at the dial level.

Work through `site_apply_css` (explicit operations, exact selectors) and `site_replace_node_html` per section; keep changes reviewable, snapshot after each group.

## Never silently
Get explicit approval before changing: slugs/URLs, primary nav labels, form field names or order (analytics and autofill), anchor ids used by links or tracking, logo/wordmark, legal/consent copy, the copy voice (visual modernization is not a rewrite). Don't regress existing accessibility wins.

## Checklist
- [ ] Mode stated; paired style (if any) applied fully, not half
- [ ] Audit findings shown before big changes
- [ ] No literals left where a framework token exists
- [ ] Slugs, nav labels, form fields, anchors unchanged (or changed with approval)
- [ ] Before/after snapshots compared at every breakpoint
