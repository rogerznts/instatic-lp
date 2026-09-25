# Brand file template

Copy this to the root of your project as `instatic-brand.md` (or tell the agent where it lives) and fill it in. The `instatic-lp` skill reads it before building so every page follows the same identity. Colors, fonts and scales are **not** defined here: they come from the site's Instatic Framework panel (read via `site_read_styles`). This file only says which framework token plays which role and which style the pages use; the agent fills it in after the first page spec is approved.

```markdown
# Brand — <company>

## Site
- Instatic host: https://lp.example.com
- Everywhere template (shared nav/footer)? yes / no
- Default CTA destination: #contato | https://wa.me/... | form

## Framework roles (token names exactly as in site_read_styles)
These become the `--lp-*` bindings on each page wrapper.
- --lp-bg (page background): var(--...)
- --lp-surface (alternate section / card background): var(--...)
- --lp-ink (text): var(--...)
- --lp-muted (secondary text): var(--...) | color-mix(...)
- --lp-line (hairlines): color-mix(in srgb, var(--...) 12%, transparent)
- --lp-accent (CTA / highlights): var(--...)
- --lp-accent-ink (text on accent): var(--...)
- --lp-font-display (headings): var(--font-display)
- --lp-font-body (body): var(--font-body)
- --lp-font-mono (labels/data, only if the framework has one): var(--...)
Type scale: h1 --text-..., h2 --text-..., body --text-m
Section spacing: vertical --space-..., gap --space-...

## Style
- Default style: minimalist | soft (<vibe>) | brutalist (<archetype>)
- Dials: V<n> M<n> D<n>
- Radius system: <sharp | soft 12–16px | pill buttons + 2rem cards | …>
- Pages built: /<slug> (<style>), …

## Existing classes to reuse
- .btn / .btn--primary / .btn--outline
- .section / .section--alt / .container
- ...

## Voice and copy
- Language: pt-BR
- Tone: ...
- Words to avoid: ...

## Rules
- CTA color: ...
- Imagery style: ...
- Legal footer text: ...
```
