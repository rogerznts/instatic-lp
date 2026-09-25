# Style: Soft (high-end agency)

Adapted from taste-skill `soft-skill` (`high-end-visual-design`, Leonxlnx/taste-skill, MIT). Pages that feel expensive: machined nested cards, pill CTAs with an icon island, very generous air, heavy-but-fluid motion. Rules here sit on top of the router; colors and fonts come from the framework through the `--lp-*` roles.

## Pick one vibe and one layout (say which in the page spec)

**Vibe** (decides the role binding):

| Vibe | For | Binding |
|---|---|---|
| **Ethereal glass** | tech, AI, SaaS, night events | `--lp-bg` = darkest token (off-black), `--lp-ink` = lightest token, cards = `color-mix(in srgb, var(--lp-ink) 4%, var(--lp-bg))`, hairlines `color-mix(in srgb, var(--lp-ink) 10%, transparent)`, one or two soft radial glows from the accent at ≤ 15%. Needs a dark token in the framework. |
| **Editorial luxury** | lifestyle, real estate, hospitality, fashion, premium services | light warm neutral bg, `--lp-font-display` large (a serif display fits here), accent in small doses; optional grain layer at ~3% |
| **Soft structuralism** | consumer, health, portfolio | white or light gray bg, heavy display type, floating components with very diffused tinted shadows |

**Layout** (the page's signature composition; still follow the router's layout-family variety):

1. **Asymmetric bento** — `grid-template-columns: repeat(12, 1fr)`, cells spanning 8+4, 4+4+4 with row spans; one column under the tablet breakpoint.
2. **Z-axis cascade** — cards overlapping with slight `rotate(-2deg / 3deg)` and negative margins; remove rotation and overlap under the tablet breakpoint (touch conflicts).
3. **Editorial split** — big type on one half, scrolling image pills / staggered cards on the other; stacks full-width on mobile.

## Components

**Double bezel** (all major cards, media and form panels): an outer shell holding an inner core with a concentric smaller radius.

```css
.lp-sol-shell { padding: 6px; border-radius: 2rem; background: color-mix(in srgb, var(--lp-ink) 5%, transparent); border: 1px solid color-mix(in srgb, var(--lp-ink) 6%, transparent); }
.lp-sol-core { border-radius: calc(2rem - 6px); background: var(--lp-surface); padding: var(--space-xl);
  box-shadow: inset 0 1px 1px color-mix(in srgb, var(--lp-bg) 60%, transparent), 0 24px 48px -24px color-mix(in srgb, var(--lp-ink) 14%, transparent); }
```

**Pill CTA with icon island**: the trailing arrow lives in its own circle flush with the right padding, and moves on hover.

```css
.lp-sol-cta { display: inline-flex; align-items: center; gap: var(--space-s); border-radius: 9999px; padding: var(--space-2xs) var(--space-2xs) var(--space-2xs) var(--space-l);
  background: var(--lp-accent); color: var(--lp-accent-ink); font-size: var(--text-s); font-weight: 500;
  transition: transform 500ms cubic-bezier(0.32, 0.72, 0, 1); }
.lp-sol-cta__icon { display: grid; place-items: center; width: 2rem; height: 2rem; border-radius: 9999px; background: color-mix(in srgb, var(--lp-accent-ink) 12%, transparent);
  transition: transform 500ms cubic-bezier(0.32, 0.72, 0, 1); }
.lp-sol-cta:hover .lp-sol-cta__icon { transform: translate(3px, -1px) scale(1.05); }
.lp-sol-cta:active { transform: scale(0.98); }
```

```html
<a class="lp-sol-cta" href="#contato">Agendar visita <span class="lp-sol-cta__icon" aria-hidden="true">↗</span></a>
```

**Eyebrow pill** before major headings (still ≤ 1 per 3 sections): `border-radius: 9999px; padding: var(--space-4xs) var(--space-xs); font-size: var(--text-xs); text-transform: uppercase; letter-spacing: 0.2em; font-weight: 500`.

**Floating island nav** (standalone pages only; with an everywhere template the nav belongs to the template): detached from the top (`margin-top: var(--space-m)`, `width: max-content`, centered), pill radius, sticky, `backdrop-filter: blur(16px)` over a translucent `--lp-bg` mix. Hamburger lines morph into an X (rotate ±45°); the open menu is a full-screen overlay whose links rise in with a staggered delay. Toggle through a code asset (`site_write_code_asset`), or a CSS-only `<details>` / checkbox pattern when possible.

## Space and type
- Section padding `--space-3xl` to `--space-4xl` (double what feels normal).
- Headlines big and tight: `--text-3xl`/`--text-4xl` for short headlines, `letter-spacing: -0.03em`, `line-height: 1–1.05` (1.1 when italic words have descenders).
- Radii large and consistent: containers 2rem family, buttons pill. No square cards.

## Motion (dials M 5–7)
- All transitions custom-eased: `cubic-bezier(0.32, 0.72, 0, 1)` (heavy) or `cubic-bezier(0.16, 1, 0.3, 1)`; never `linear` / `ease-in-out` for UI.
- Entry: heavy fade-up with blur, 800ms feel: keyframes `from { opacity: 0; transform: translateY(var(--space-xl)); filter: blur(8px) }` with the router's scroll-driven pattern, gated by reduced motion.
- Nav links and grid items stagger.

## Don't
- Generic 1px gray borders on flat cards, harsh dark shadows, edge-to-edge sticky nav glued to the top (standalone pages), symmetric 3-column grids without big gaps, instant state changes.
- Thick-stroke generic icon sets; use thin, precise line icons (inline SVG from a line icon set) or none.
- Blur on scrolling content or large areas; grain anywhere but a fixed `pointer-events: none` layer.

## Checklist
- [ ] One vibe and one layout archetype chosen and stated
- [ ] Major cards/media use the double bezel with concentric radii
- [ ] Primary CTA is a pill with the icon island (when it has an arrow)
- [ ] Section padding ≥ `--space-3xl`; collapses to single column with `--space-m` gutters on mobile
- [ ] Every transition custom-eased; entries present and reduced-motion gated; transform/opacity (and filter on entry) only
- [ ] `backdrop-filter` only on fixed/sticky elements
- [ ] Reads as a crafted agency build, not a template with nice fonts
