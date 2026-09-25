# Reference intake and grill

Turns "here's a URL, make me a page like this" into a decided page spec. Nothing is written to the Instatic site during intake.

## 1. Read the reference

Pick the source by where the reference lives:

| Reference | How to read it |
|---|---|
| Page on **this** Instatic site (`https://<host>/<slug>`, or a page named by the user) | `site_list_documents` → `site_read_document` (follow `pageInfo.nextPart`). Exact structure, classes and copy. Candidate for the **redesign** style. |
| Any other public URL | The client's web fetch tool (WebFetch, `curl`, reader) for text/HTML. If a browser tool is available, also look at it rendered at desktop and mobile widths: visual language can't be read from text alone. |
| Screenshot / image | Read it directly. |
| URL that fails (login wall, bot block, SPA with empty HTML) | Say so and ask for a screenshot or the copy. Don't invent what the page contains. |

## 2. Write the Reference read

Before any question, give the user a compact read of the reference (in their language). This is also your source for recommended answers.

```markdown
**Reference read — <url>**
- Owner: <brand/company>; page kind: <landing / product / event / portfolio / institutional…>
- Offer and goal: <what it sells or asks>; primary CTA: "<label>" → <destination>
- Audience signals: <who it talks to, register, language>
- Sections, in order:
  1. <nav> — <items>
  2. <hero> — headline "<…>", support "<…>", CTA, visual: <photo / product / video / none>
  3. …
- Proof: <testimonials / logos / numbers / cases / press>
- Forms and conversion: <fields, destination if visible>
- Visual language: <light/dark>, palette character, type character (serif/sans, weight, scale contrast), layout families, radius system, density, motion
- What works: <2–4 points worth keeping>
- What's weak: <2–4 points: AI tells, clutter, hidden CTA, walls of text…>
```

Keep copy excerpts short; the read is an inventory, not a transcript.

## 3. Rights

Whose page is it? Ask in the grill if not obvious (first question when unclear).

- **User's own content** (their old site, their page on this Instatic, their doc): copy may be reused; images may be uploaded with `media_upload` (`sourceUrl`) after the user confirms.
- **Third party** (competitor, inspiration): the reference gives **structure, section jobs and level of polish only**. Rewrite all copy from the user's own facts; never reuse its images, logos, testimonials or numbers; never hotlink. Its visual language informs the style choice but its colors and fonts are not copied (the framework decides those anyway).

## 4. Grill

Interview the user until every branch of the page is decided. Rules:

- **One question at a time**, in dependency order (later answers depend on earlier ones). A small batch is fine only for tightly related, independent details (e.g. slug + publish).
- **Every question carries your recommended answer**, derived from the reference read, the framework, the brand file or common sense, with a one-line reason. The user can just say "ok".
- If the client has a structured question tool (e.g. `AskUserQuestion`), use it with 2–4 concrete options, recommended first; otherwise ask in chat.
- **Don't ask what you can find out**: read the reference, the site (`site_list_documents`, `site_read_document`), the framework and the brand file first. Don't ask what the user already said.
- Push back when an answer conflicts with the goal (two primary CTAs, a hero with 6 elements, a 20-row table on a landing page): name the problem, recommend the fix, accept the user's call.
- Stop when the branches below are closed. Don't pad with questions that don't change the page.

### Decision tree (walk in order, skip what's answered)

1. **Relationship to the reference** — own page vs inspiration (rights); same page type or adapting to another offer?
2. **Goal** — the one conversion that matters; the single primary CTA label and destination (WhatsApp, form, checkout, anchor). One label per intent across nav, hero and footer.
3. **Audience and register** — who lands here, from where (ad, organic, email), what they need to believe to click; language and formality.
4. **Offer facts** — what is sold, price/conditions if shown, differentiators, objections to answer. Only facts the user confirms; never invent numbers.
5. **Section plan** — go through the reference's sections: keep / drop / merge / add, each with its job (hook, proof, explain, handle objection, convert). Recommend a sequence; a landing page with 8 sections uses at least 4 different layout families.
6. **Copy** — reuse (own content only), adapt, or write new; who provides facts; tone words to use and avoid.
7. **Proof** — real testimonials, logos, cases, numbers available? If none, the page goes without; no invented social proof.
8. **Assets** — photos/video the user has, generation allowed, or placeholders to replace later.
9. **Forms and integrations** — fields, where submissions go, tracking needs (keep field names stable on redesigns).
10. **Placement** — slug, homepage or not, everywhere template (nav/footer from template) or standalone.
11. **Publishing** — draft only, or publish after verification.

Style (visual direction) is the next step, not a grill branch: the router offers options once the content is decided.

## Page spec

Close intake + style with this summary and wait for a yes before step 6 of the skill.

```markdown
**Page spec — /<slug>** (key `<key>`)
- Reference: <url> (<own | inspiration>)
- Goal: <conversion>; CTA "<label>" → <destination>
- Audience: <…>; language/register: <…>
- Placement: <template | standalone>; publish at the end: <yes/no>
- Design read: <one line from the router>
- Style: <minimalist | soft (+ vibe) | brutalist (+ archetype) | redesign (+ style)>; dials V<n> M<n> D<n>
- Role binding: bg <token>, surface <token>, ink <token>, muted <token/mix>, line <mix>, accent <token>, accent-ink <token>, display font <token>, body font <token>
- Framework gaps: <none | what's missing and the fallback / proposed token>
- Sections:
  1. <name> — <layout family> — <content source> — <asset>
  2. …
- Images: <upload / generate / placeholders: list>
- Form: <fields → destination>
- Open items for the user: <…>
```
