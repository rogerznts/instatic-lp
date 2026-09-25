# Brand file template

Copy this to the root of your project as `instatic-brand.md` (or tell the agent where it lives) and fill it in. The `instatic-lp` skill reads it before building so every page follows the same identity. Anything left blank falls back to the tokens already on the Instatic site.

```markdown
# Brand — <company>

## Site
- Instatic host: https://lp.example.com
- Everywhere template (shared nav/footer)? yes / no
- Default CTA destination: #contato | https://wa.me/... | form

## Tokens (slug → value)
Colors:
- primary: #...        # main CTA background
- primary-contrast: #...
- surface: #...        # page background
- text: #...
- muted: #...
- accent: #...
Fonts:
- --font-heading: <Google Font family>
- --font-body: <Google Font family>
Type scale: base 16px, ratio 1.25 (--text-s … --text-3xl)
Spacing scale: base 8px (--space-xs … --space-3xl)

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
