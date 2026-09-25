# Style: Brutalist (industrial / Swiss / telemetry)

Adapted from taste-skill `brutalist-skill` (`industrial-brutalist-ui`, Leonxlnx/taste-skill, MIT). Raw, mechanical pages: rigid grid, visible rules, zero radius, extreme type-scale contrast, one loud accent. Rules here sit on top of the router; colors and fonts come from the framework through the `--lp-*` roles.

## Pick one archetype (never mix)

| Archetype | Character | Binding |
|---|---|---|
| **Swiss industrial print** (light) | newsprint substrate, monolithic heavy sans, visible structural lines, huge numerals bleeding off the grid | `--lp-bg` = off-white/light gray token, `--lp-ink` = near-black token, `--lp-accent` = the red (or loudest) token, **only** accent |
| **Tactical telemetry** (dark) | deactivated CRT, dense mono data, ASCII framing, scanlines | `--lp-bg` = darkest non-pure-black token, `--lp-ink` = light gray/white token, same single accent; a green token may mark **one** status element, never text |

If the framework has no loud accent, use ink-only (monochrome brutalism) and say so in the offer. Never add a hex.

## Fonts

| Role | Ideal | With this framework |
|---|---|---|
| Macro type (headings) | heavy neo-grotesque | the heaviest sans token at its heaviest installed weight (800–900 if installed, else 700); a serif display token only as the rare "textural contrast" element, never for every heading |
| Micro type (labels, nav, data) | monospace | mono token if present; else body token, uppercase, `letter-spacing: 0.08em`, `font-variant-numeric: tabular-nums`. Or propose adding a mono font token (framework change, needs the user's OK) |

- Macro: uppercase, `line-height: 0.85–0.95`, `letter-spacing: -0.03em` to `-0.06em`, largest scale steps (`--text-4xl` and, for one poster moment, `calc(var(--text-4xl) * 2)`).
- Micro: `--text-xs`/`--text-s`, uppercase, `letter-spacing: 0.05–0.1em`, `line-height: 1.2–1.4`. Used for metadata, nav, IDs, labels.
- The hero may be purely typographic (the router's "real visual" rule is satisfied by the type itself).

## Layout
- Strict CSS grid; elements anchor to tracks. Visible compartments: `1px`/`2px solid var(--lp-ink)` borders and full-width `<hr>` rules between units.
- Hairline grid trick: parent `background: var(--lp-ink); gap: 1px`, children `background: var(--lp-bg)`.
- Bimodal density: tightly packed micro-data next to vast empty space framing macro type.
- `border-radius: 0` everywhere (buttons, inputs, images, cards).

```css
.lp-sol-grid { display: grid; grid-template-columns: repeat(12, 1fr); gap: 1px; background: var(--lp-ink); border: 1px solid var(--lp-ink); }
.lp-sol-grid > * { background: var(--lp-bg); padding: var(--space-m); }
.lp-sol-mega { font-family: var(--lp-font-display); font-weight: 900; text-transform: uppercase; font-size: var(--text-4xl); line-height: 0.9; letter-spacing: -0.05em; }
.lp-sol-micro { font-family: var(--lp-font-mono, var(--lp-font-body)); font-size: var(--text-xs); text-transform: uppercase; letter-spacing: 0.08em; }
.lp-sol-cta { border-radius: 0; background: var(--lp-accent); color: var(--lp-accent-ink); padding: var(--space-s) var(--space-l); text-transform: uppercase; letter-spacing: 0.06em; font-weight: 700; border: 2px solid var(--lp-ink); }
.lp-sol-cta:hover { background: var(--lp-ink); color: var(--lp-bg); }
.lp-sol-cta:focus-visible { outline: 3px solid var(--lp-accent); outline-offset: 2px; }
@media (max-width: 767px) { .lp-sol-grid { grid-template-columns: 1fr; } }
```

(`--lp-font-mono` is bound in the page wrapper only when the framework has a mono token.)

## Symbols and devices
- ASCII framing for labels: `[ ENTREGA ]`, `< 01 >`, `>>>`, `///`. Use on labels that name real content, not as filler.
- `®`, `©`, `™`, crosshairs `+` at real grid intersections, bar/warning stripes (`repeating-linear-gradient` with ink and accent) as structural elements.
- Numbers and data in semantic tags: `<data>`, `<dl>`, `<output>`, `<kbd>`, `<samp>`.
- These devices are this style's exception to the router's "no decorative lines/labels" rule: allowed when they organize real content; never fake data (`REV 2.6`, random unit IDs) presented as facts about the business.

## Texture (optional, performance-safe)
- Telemetry scanlines on the page wrapper background: `repeating-linear-gradient(0deg, transparent 0 2px, color-mix(in srgb, var(--lp-ink) 6%, transparent) 2px 4px)`.
- Global noise: fixed `pointer-events: none` pseudo-element layer only.
- Halftone/dither on images: pre-processed images, or `mix-blend-mode: multiply` over a dot pattern.

## Motion (dials M 1–4)
- Mostly static. Hard state changes are on-brand: hover inverts colors instantly or in ≤ 120ms `steps(2)`.
- Optional: one typewriter/counter moment for real data, reduced-motion gated.

## Don't
- Gradients (except stripes/scanlines), soft shadows, translucency/glass, any radius.
- Mixing light and dark substrates; more than one accent.
- Using the green (if any) for more than one element.

## Checklist
- [ ] One archetype; one substrate; one accent (or monochrome)
- [ ] Radius 0 everywhere; compartments visible through rules/1px gaps
- [ ] Macro type uppercase, tight leading, negative tracking; micro type uppercase, tracked, tabular numbers
- [ ] ASCII/industrial devices label real content only; no fake telemetry about the business
- [ ] Grid collapses to one column on mobile with rules intact; no horizontal overflow from giant type
