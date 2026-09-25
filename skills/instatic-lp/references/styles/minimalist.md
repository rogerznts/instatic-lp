# Style: Minimalist (editorial utilitarian)

Adapted from taste-skill `minimalist-skill` (Leonxlnx/taste-skill, MIT). Document-like pages: warm monochrome, strong type contrast, hairlines, flat bento, near-invisible motion. Rules here sit on top of the router; colors and fonts come from the framework through the `--lp-*` roles.

## Role binding

| Role | Pick from the framework | If missing |
|---|---|---|
| `--lp-bg` | white or the lightest warm neutral | white token |
| `--lp-surface` | a light gray/off-white one step from bg | `color-mix(in srgb, var(--lp-ink) 3%, var(--lp-bg))` |
| `--lp-ink` | off-black / charcoal (avoid pure black if a softer dark exists) | darkest token |
| `--lp-muted` | mid gray | `color-mix(in srgb, var(--lp-ink) 55%, var(--lp-bg))` |
| `--lp-line` | very light gray | `color-mix(in srgb, var(--lp-ink) 9%, transparent)` |
| `--lp-accent` | the brand color, used **small** (links, a tag, one detail) | — |
| pale tag tints | — | `color-mix(in srgb, var(--<color>) 12%, var(--lp-bg))` bg + the color itself (or a darker shade) as text |
| display / body fonts | serif display if the framework has one (editorial contrast), sans body | single font: contrast by weight and size |

Primary CTA is **ink on bg** (dark button, light text), not the accent. Large sections never get a saturated background.

## Typography
- Headings: `var(--lp-font-display)`, tight tracking `-0.02em` to `-0.04em`, `line-height: 1.1`; hierarchy by size step + weight, not by color.
- Body: `line-height: 1.6`, `color: var(--lp-ink)`, secondary text `var(--lp-muted)`, paragraphs `max-width: 65ch`.
- Metadata/keys: mono font token if the framework has one; else body font at `--text-xs`, `letter-spacing: 0.05em`, uppercase, sparingly.

## Layout
- Macro-whitespace first: section padding `--space-3xl`/`--space-4xl`; main text column `max-width: 56rem–64rem`.
- Feature grids as asymmetric bento: `grid-template-columns: 2fr 1fr` / spans, cells with `border: 1px solid var(--lp-line)`, radius 8–12px max, padding `--space-l`/`--space-xl`.
- Sections still need depth: one real image, a subtle low-opacity image, a soft radial tint (`radial-gradient(… color-mix(in srgb, var(--lp-accent) 6%, transparent) …)`) or a fine line pattern. No empty flat bands.

## Components

```css
.lp-sol-btn { background: var(--lp-ink); color: var(--lp-bg); border-radius: 6px; padding: var(--space-xs) var(--space-l); font-size: var(--text-s); box-shadow: none; transition: background-color 200ms, transform 200ms cubic-bezier(0.16, 1, 0.3, 1); }
.lp-sol-btn:hover { background: color-mix(in srgb, var(--lp-ink) 82%, var(--lp-bg)); }
.lp-sol-btn:active { transform: scale(0.98); }

.lp-sol-card { border: 1px solid var(--lp-line); border-radius: 12px; padding: var(--space-xl); background: var(--lp-bg); transition: box-shadow 200ms; }
.lp-sol-card:hover { box-shadow: 0 2px 8px color-mix(in srgb, var(--lp-ink) 4%, transparent); }

.lp-sol-tag { border-radius: 9999px; padding: var(--space-4xs) var(--space-xs); font-size: var(--text-xs); letter-spacing: 0.05em; text-transform: uppercase;
  background: color-mix(in srgb, var(--lp-accent) 12%, var(--lp-bg)); color: color-mix(in srgb, var(--lp-accent) 70%, var(--lp-ink)); }

.lp-sol-faq details { border-bottom: 1px solid var(--lp-line); padding-block: var(--space-m); }
.lp-sol-faq summary { list-style: none; display: flex; justify-content: space-between; cursor: pointer; }
.lp-sol-faq summary::after { content: "+"; }
.lp-sol-faq details[open] summary::after { content: "−"; }
```

- FAQ: no boxes, only bottom rules, `+`/`−` toggle (the `−` is a minus sign, not a dash in copy).
- Keyboard shortcuts / codes: `<kbd>` with `--lp-line` border, radius 4px, `--lp-surface` background, mono font if available.
- Software mockups: real screenshot inside a minimal frame (white top bar, three small `--lp-line` dots). Never a fake UI built from divs.

## Imagery
- Desaturated, warm, calm photos; an optional tint overlay to pull them into the palette. No oversaturated stock.
- Illustrations, if any: monochrome line work with one pale accent shape.

## Motion (dials M 3–4)
- Reveal: `translateY` ≈ `--space-s` + opacity over 600ms `cubic-bezier(0.16, 1, 0.3, 1)` (router snippet), stagger 80ms-equivalent.
- Card hover: shadow from none to the ultra-soft tint above. Buttons `scale(0.98)` on `:active`.
- Optional: one very slow ambient radial blob behind the hero (20s+, opacity ≤ 0.04) on a fixed, `pointer-events: none` layer.

## Don't
- Gradients as decoration, neon, glass beyond a subtle sticky-nav blur, heavy shadows (anything above ~5% ink).
- Pill shapes for large containers, cards or primary buttons (pills only for small tags).
- Saturated full-width section backgrounds; accent used as a large fill.
- Emojis anywhere (copy, headings, alt).

## Checklist
- [ ] Primary CTA is ink/bg, accent only in small doses
- [ ] Every card/divider uses `--lp-line` 1px; radius ≤ 12px on containers
- [ ] Section padding at the high end of the space scale; text columns ≤ 65ch
- [ ] At least 2–3 real images on the page; no flat empty bands
- [ ] No gradients/neon/heavy shadows, no emojis
