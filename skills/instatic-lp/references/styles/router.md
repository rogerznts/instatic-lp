# Style router

Adapted from taste-skill's router (`design-taste-frontend`, Leonxlnx/taste-skill, MIT) for Instatic pages. What changed in the adaptation:

- **Output is Instatic HTML + `<style>`**, inserted with `site_insert_html`. No React, Tailwind, component libraries or npm design systems. Tailwind-isms from the originals are translated to CSS on framework tokens.
- **The framework decides colors, fonts and scales.** The originals' hex palettes and font picks are replaced by role bindings to framework tokens (`--lp-*`, see SKILL.md step 6). A style decides *treatment*: composition, which scale steps, weight, tracking, casing, radius, borders, depth, motion, density.
- **Behaviour** is CSS first; JS only through `site_write_code_asset`.

## 1. Design read

After the grill, state one line, in the user's language:

> **Leitura:** <page kind> para <audience>, com linguagem <vibe>, puxando para o estilo <style>.

Signals, in order of weight: page kind and goal (grill), audience, vibe words the user used, the reference's visual language, brand assets (framework palette and fonts), quiet constraints (regulated, public sector, accessibility-first, trust-first commerce). Quiet constraints override taste.

Avoid the LLM defaults regardless of style: centered hero over a gradient blob, three equal feature cards, glass on everything, infinite micro-animations, AI-purple glow.

## 2. Dials

| Dial | 1 | 10 |
|---|---|---|
| `VARIANCE` | perfect symmetry | asymmetric, art-directed |
| `MOTION` | static | cinematic |
| `DENSITY` | gallery, airy | cockpit, packed |

Inference:

| Signal | V | M | D |
|---|---|---|---|
| minimalist / clean / calm / editorial | 5–6 | 3–4 | 2–3 |
| premium / luxury / brand / "Apple-like" | 7–8 | 5–7 | 3–4 |
| playful / experimental / agency | 9–10 | 8–10 | 3–4 |
| landing page, default | 7–8 | 5–6 | 3–5 |
| trust-first / regulated / public sector | 3–4 | 2–3 | 4–5 |
| redesign, preserve | match existing | +1 | match |
| redesign, overhaul | +2 | +2 | match |

What the dials mean in Instatic:

| Dial | Low (1–3) | Mid (4–7) | High (8–10) |
|---|---|---|---|
| VARIANCE | symmetric grid, centered or left-aligned stacks | offset overlaps (`margin-top: calc(var(--space-xl) * -1)`), mixed aspect ratios, left headers over centered data | `grid-template-columns: 2fr 1fr 1fr`, masonry-like bento, big empty zones. Above 4, everything collapses to one column under the smallest non-mobile breakpoint. |
| MOTION | `:hover` / `:active` / `:focus-visible` only | + CSS transitions (`cubic-bezier(0.16, 1, 0.3, 1)`, 200–700ms), scroll-driven reveals in CSS | + `position: sticky` stacks, one pinned/scrubbed moment, JS via code asset if CSS can't do it. Get the user's OK before adding a script dependency. |
| DENSITY | section padding `--space-3xl`/`--space-4xl` | `--space-2xl`/`--space-3xl` | `--space-l`/`--space-xl`, 1px rules instead of cards, tabular numbers |

## 3. Styles on offer

| Style | Offer when | Framework needs | Gap fallback |
|---|---|---|---|
| [Minimalist](minimalist.md) | editorial, calm, B2B clarity, content-first, premium without spectacle | a dark neutral, a light neutral, a readable body font | missing pale accents → `color-mix` tints of existing colors |
| [Soft](soft.md) | premium consumer, lifestyle, real estate, health, agencies wanting "expensive"; image-rich offers | a light or very dark substrate, one accent, a display font with presence | no display font → body font heavier + tighter tracking |
| [Brutalist](brutalist.md) | tech, industrial, data, manifesto, events with attitude, portfolios; brands with a strong red/black | high-contrast pair (near-black + off-white) and one loud accent; ideally a mono font | no mono → body font uppercase + wide tracking for micro-type (or propose a mono font token) |
| [Redesign](redesign.md) | the reference is the user's own page (on this site or elsewhere) and they want it better, not different | whatever the existing page uses | — |

Pick one aesthetic per page. Redesign can pair with one aesthetic ("redesign, overhaul, toward soft"). Never mix two aesthetics.

### Offer format

Offer 2–3 options that fit the design read (plus Redesign when it applies), recommended first, each with what it would look like on **this** framework:

```markdown
**Estilos para esta página**
1. **Soft — Editorial luxury** (recomendado): fundo `--white`, títulos em `--font-display` grandes, cards aninhados com raio grande, CTA pílula em `--gold`, muito respiro, entradas suaves. Dials V7 M5 D3.
2. **Minimalist**: preto/branco do framework, `--gold` só em detalhes, linhas finas, bento plano, quase sem movimento. V5 M3 D3.
3. **Brutalist — Swiss print**: tipografia gigante em caixa alta, grade com linhas visíveis, raio zero, `--red` como único acento. Sem fonte mono no framework: microtextos em `--font-body` caixa alta espaçada. V8 M3 D5.
```

If the reference already has a strong visual language, say which option is closest to it and which one moves away on purpose.

After the choice, read the style file. For a redesign, read `redesign.md` and the paired style.

## 4. Rules for every style

### Framework and consistency
- Colors only through the role binding (`--lp-*`) or framework tokens; one **accent** per page, used identically in every section (nav, hero, footer).
- **Theme lock:** the page is light or dark as a whole. Section tints within the same family are fine; flipping to an inverted section mid-page is not (one deliberate color-block moment per page at most, only if the style or the user asks).
- **Shape lock:** one radius system for the page (all sharp, all soft, or a documented rule like "pill buttons, 16px cards, 8px inputs"), followed everywhere.
- Shadows, when used, are tinted from the palette: `color-mix(in srgb, var(--lp-ink) 8%, transparent)`, never pure black.

### Hero
- Fits the first viewport at every breakpoint: headline ≤ 2 lines on desktop, support ≤ 20 words and ≤ 4 lines, CTA visible without scrolling.
- Max 4 text elements: eyebrow **or** brand strip (or neither), headline, support, CTAs (1 primary + at most 1 secondary). No tagline under the CTAs, no trust strip, price teaser or bullet list inside the hero; those go in the section below.
- Top padding ≤ `--space-3xl` on desktop; more room comes from type or image scale, not padding.
- The hero has a real visual (photo, product, video) unless the style is explicitly typographic (brutalist, editorial manifesto).
- Headline size: `--text-3xl`/`--text-4xl` only for ≤ 5 words; longer headlines step down. A 4-line headline is a size error.

### Layout
- Navigation on one line at desktop, height ≤ 80px.
- A layout family (split text/image, 3-card row, full-width quote, bento…) appears **once** per page; 8 sections → at least 4 families.
- At most 2 consecutive image+text splits (zigzag).
- **Eyebrows** (small uppercase labels over headings): max 1 per 3 sections, hero counts.
- No default "big headline left + small paragraph floating right" section header; stack headline over body (≤ 65ch).
- Bento: exactly as many cells as items (no empty cells); at least 2–3 cells with real visual variation (image, tint, pattern).
- No three identical feature cards in a row; use asymmetric grid, 2-column, list with visuals, or scroll-snap.
- Logo wall sits under the hero, logos only (no category labels), real logos or none.
- Cards only when elevation means hierarchy; otherwise group with space or a rule.
- Every multi-column section declares its mobile collapse; full-height sections use `min-height: 100dvh`, never `100vh`.

### Content
- Per section by default: headline ≤ 8 words, sub ≤ 25 words, one visual or one CTA.
- More than 5 items → a different component (grouped columns, cards with visuals, tabs, `<details>`, scroll-snap), not a longer list with a rule under each row.
- Quotes ≤ 3 lines, attribution = name + role (+ company).
- Only real numbers, testimonials and logos (from the user). No invented proof, no fake-precise stats, no "John Doe / Acme" placeholders.
- One copy register per page. Sentence case headings.
- Copy clichés banned: EN "elevate, seamless, unleash, next-gen, game-changer, delve"; PT "eleve, potencialize, revolucione, descubra o poder, jornada, sem complicação, solução completa, o futuro de, de forma simples e rápida, transforme sua".
- **No em dash (—) or en dash (–) in visible copy.** Use a period, comma, colon or parentheses; ranges use a hyphen.
- Banned decoration: version labels in the hero (unless it's a launch), section numbers as eyebrows (`01 / Serviços`), pills over photos, fake photo credits, decorative status dots, scroll cues ("role para baixo"), locale/weather strips, decorative text strips at the hero bottom (`DESIGN · BUILD · SHIP`), fake product UI built from divs.
- Copy self-audit before shipping: reread every visible string (headings, buttons, alt, footer); rewrite anything unclear, cute-but-wrong or AI-sounding. Boring and clear beats clever and broken.

### CTAs and forms
- One label per intent on the whole page ("Fale conosco" + "Entre em contato" = same intent, pick one).
- CTA label fits one line at desktop (≤ 3 words ideally).
- CTA text vs background ≥ 4.5:1 (3:1 for large text); ghost buttons over photos get a scrim or stroke.
- Labels above inputs, error text below, never placeholder-as-label; inputs, placeholders and focus rings pass contrast on the section background.
- Visible `:focus-visible` on every interactive element.

## 5. Motion in Instatic

- Animate only `transform` and `opacity`. No `window` scroll listeners.
- Everything above MOTION 3 is gated by `prefers-reduced-motion`.
- Every animation has a reason (hierarchy, sequence, feedback, state change). One marquee per page at most.
- `backdrop-filter` only on fixed/sticky elements (nav, overlays); grain/noise only on a fixed `pointer-events: none` layer.

Hover/press:

```css
.lp-sol-cta { transition: transform 200ms cubic-bezier(0.16, 1, 0.3, 1), background-color 200ms; }
.lp-sol-cta:hover { background: color-mix(in srgb, var(--lp-accent) 88%, var(--lp-ink)); }
.lp-sol-cta:active { transform: scale(0.98); }
.lp-sol-cta:focus-visible { outline: 2px solid var(--lp-accent); outline-offset: 3px; }
```

Scroll reveal, CSS only (browsers without scroll-driven animations jump straight to the visible end state, so content is never hidden):

```css
@keyframes lp-sol-rise { from { opacity: 0; transform: translateY(var(--space-l)); } to { opacity: 1; transform: none; } }
@media (prefers-reduced-motion: no-preference) {
  .lp-sol-reveal { animation: lp-sol-rise linear both; animation-timeline: view(); animation-range: entry 0% cover 25%; }
}
```

Stagger inside a group: `animation-range: entry calc(var(--i) * 5%) cover calc(25% + var(--i) * 5%)` with `style="--i:1"` on each item.

JS (only when CSS can't: menu toggle, tabs, class-based reveals): `site_write_code_asset` (`type:"script"`, `path:"src/scripts/<key>-<feature>.js"`), `IntersectionObserver` instead of scroll listeners, then `site_inspect_code_runtime` to confirm it runs on the page.

Dark mode: if framework colors define dark values, the tokens already switch; don't build a theme toggle unless asked, and check both modes in snapshots only when the site actually uses dark mode.

## Pre-flight

Run before declaring the page done (with `site_read_document` + `site_render_snapshot` per breakpoint). Any failed box = not done.

- [ ] Design read, style and dials stated; chosen style's checklist also passed
- [ ] No color, font-family or type/spacing size outside the framework (only `--lp-*` roles, framework vars, `color-mix` of them)
- [ ] One accent, theme lock, shape lock
- [ ] Hero fits the viewport at every breakpoint, ≤ 4 text elements, CTA visible
- [ ] Nav on one line, ≤ 80px
- [ ] Layout families not repeated; ≤ 2 consecutive zigzags; bento cell count exact; ≤ 1 eyebrow per 3 sections
- [ ] CTA: one label per intent, no wrap at desktop, contrast ≥ 4.5:1
- [ ] Forms: labels above, contrast, focus visible
- [ ] Zero em/en dashes in visible copy; copy self-audit done; no banned clichés; no invented proof or numbers
- [ ] None of the banned decorations
- [ ] Images: real or clearly listed placeholders; none taken from a third-party reference
- [ ] Motion: transform/opacity only, reduced-motion gated, each animation justified, content visible without JS
- [ ] Mobile collapse explicit; no horizontal overflow warnings
