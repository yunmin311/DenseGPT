# Design principles

One acceptance criterion:

> **Increase information density without reducing semantic clarity.**

## The values are the design

There is no density model. Every spacing value is fixed, set per element, and was
verified by eye on real ChatGPT output before it entered this file. Nothing is
derived, multiplied or computed at runtime.

| Element | Value |
| --- | --- |
| `p` | `margin: 0.45em 0` · `line-height: 1.65` |
| `h2` | `margin: 1.15em 0 0.45em` · `font-size: 1.15em` |
| `h3` | `margin: 0.9em 0 0.35em` · `font-size: 1.05em` |
| `ul`, `ol` | `margin-top: 0.35em` · `margin-bottom: 0.45em` |
| `li` | `margin: 0.12em 0` |
| inline code | `background: var(--dg-surface-soft)` · `padding: 0.08em 0.28em` · `border-radius: 4px` · no border, no shadow |
| `blockquote` | left rule `3px solid var(--dg-edge)` · `background: var(--dg-surface-block)` · `padding: 0.55em 0.85em` · `margin: 0.65em 0` · `border-radius: 6px` · inherited text colour |
| `em`, `i` | `font-weight: 500` — italic alone does not separate at this line-height. Bold-italic is excluded so it stays bold. |
| `hr` | `1px solid var(--dg-edge-soft)` · `margin: 1.1em 0` — a hairline, not a chapter break |

Anything not in that table is ChatGPT's own styling, untouched.

## One set of tokens

Two inks. Four surfaces derived from them. Declared once, on the assistant markdown
root:

```css
--dg-ink:           currentColor;
--dg-ink-cite:      color-mix(in srgb, currentColor 65%, #5b7196);

--dg-surface-soft:  color-mix(in srgb, var(--dg-ink) 5%, transparent);
--dg-surface-block: color-mix(in srgb, var(--dg-ink-cite) 7%, transparent);
--dg-edge:          color-mix(in srgb, var(--dg-ink-cite) 35%, transparent);
--dg-edge-soft:     color-mix(in srgb, var(--dg-ink) 18%, transparent);
```

| Token | Used by |
| --- | --- |
| `--dg-surface-soft` | inline code fill |
| `--dg-surface-block` | blockquote fill |
| `--dg-edge` | blockquote left rule |
| `--dg-edge-soft` | horizontal rule |

Nothing else may define a colour. Every token is anchored to `currentColor`, so
lightness comes from the page's own text and both themes adapt with no theme
detection.

### Type is a trace of hue, not a colour

A category is separated by shifting the *same* ink toward a desaturated hue, never
by giving it a colour of its own. `--dg-ink-cite` is 65% currentColor and 35% of a
desaturated blue: first glance grey, second glance faintly blue-grey.

**How much hue survives is a product, not a ratio.** The anchor `#5b7196` has a
blue-minus-red spread of 59 levels; `currentColor` is neutral and contributes
none. What actually reaches the screen is:

```
B − R  ≈  surface alpha  ×  anchor share  ×  59
```

Measured in Chrome, composited over ChatGPT's own light and dark backgrounds —
the numbers come out identical in both, because both ends are anchored to the
page's own text colour:

| | anchor share | alpha | surface B−R | left rule B−R |
| --- | --- | --- | --- | --- |
| until 2.1.1 | 15% | 6% | **0.53** | 3.1 |
| 2.2.0 | 35% | 7% | **1.45** | 7.2 |

Half a level of separation out of 255 is arithmetically a tint and visually plain
grey — which is exactly why the old blockquote read as grey no matter what the
spec said. The **left rule** is where the hue is legible, because 35% of the ink
lands on the page directly instead of through a 7% veil.

The formula is the tuning lever: for a target separation, pick
`alpha × share = target / 59`. Two levels needs 7% × 48%; three needs 7% × 72% or
9% × 56%. Pushing either past its band trades "reads as grey" for visibility, so
the ceiling is a design decision, not a number to optimise.

The constraints that matter, in order:

| Constraint | Value |
| --- | --- |
| surface alpha | 5–7% |
| anchor share | as much as the product allows while staying grey at a glance |
| saturation of the anchor | very low |
| lightness across types | identical (all inherit from `currentColor`) |
| edges and icons | 18–35%, the only things allowed to deepen |

Earlier versions had three unrelated slate blues — `rgba(60,82,115)`,
`rgba(80,100,130)`, `rgba(100,116,139)` — one per element, at three different
lightnesses. That is what made the greys look dirty. There is now exactly one
literal colour in the stylesheet: the hue anchor.

### Typed surfaces — investigated, not shipped

A typed palette for citation pills, file tiles (PDF / Markdown / CSS / JSON) and
app blocks was built and then removed in 1.8.0. The record is in
[`selectors.md`](selectors.md); the short version:

- All three had **stable attribute hooks** — the hooks were never the problem.
- Citation pills and file tiles could be tinted, but only after finding that the
  hook element is a transparent shell with an opaque child that paints the visible
  surface. Two rounds of live probing to land one tint.
- App blocks **cannot** be tinted at all: the card is a cross-document `<iframe>`.

Nothing in that chain was wrong, and none of it was worth the surface area. The
stylesheet is back to the parts that were verified by eye and have stayed stable:
markdown rhythm, one quote/code tint pair, and content width.

## What the file must never do

- **No page-level colour.** Nothing sets background, body text colour or
  `color-scheme`. Light and dark are inherited from ChatGPT exactly as shipped.
- **No theme detection.** No `html.dark`, no `[data-theme]`, no
  `prefers-color-scheme`. A tint that is conditional on a guess about the theme
  renders over the wrong background as soon as the guess is wrong — that is what
  turned the blockquote into a grey bar in 1.0.0.
- **No second visual language.** One semantic level, one token. No rule defines a
  colour of its own.
- **No fenced code block changes.** Background, colours, font size and syntax
  highlighting stay exactly as ChatGPT ships them. `:not(pre) > code` is the guard
  that keeps the inline-code rule off them.
- **No scalar, no calc, no runtime abstraction.**
- **No chrome restyling.** Sidebar, buttons and message controls are not touched.
  The composer is affected only by content width, below.

## Content width

The one rule outside assistant markdown, and the reason the Widescreen extension is
no longer needed.

Four recommended stops plus a free slider. The stops are the design; the slider is
the escape hatch, and it exists because a userstyle that replaces a width extension
has to be able to land on any number its users were already using.

| Option | Posture | Stop | 16px root | Latin measure | CJK measure |
| --- | --- | --- | --- | --- | --- |
| Reading | reading | `44rem` | 704px | ~88 ch | ~44 ch |
| **Balanced** (default) | general | `54rem` | 864px | ~108 ch | ~54 ch |
| Wide | technical | `66rem` | 1056px | ~132 ch | ~66 ch |
| Ultra | large display | `78rem` | 1248px | ~156 ch | ~78 ch |
| Custom | any | 36–90rem | 576–1440px | ~72–180 ch | ~36–90 ch |

Everything is `min(<value>, calc(100vw - 3rem))`. The second term is the responsive
retreat: a narrow window shrinks the column rather than pinning text to the edges,
and 3rem leaves a 1.5rem gutter on each side once the column is centred. The
mechanism, and why a preset and a slider can coexist without conditionals, is in
[`content-width.md`](content-width.md).

The measure columns are the honest caveat. Classic prose comfort is 45–75 Latin
characters and 35–45 CJK; every stop is above that, because ChatGPT output is
mostly code, tables and lists, where width pays for itself and long prose lines are
rare. For continuous Chinese prose, Reading now sits just at the upper edge of
comfort — which is what the 1.9.1 narrowing was for.

It is applied by one rule, `main { --thread-content-max-width: … !important }`,
through ChatGPT's own custom property. A live DOM dump confirmed that `<main>` is
the only element that declares that property and that the composer sits inside it,
so messages and composer inherit the same value from the same node and stay on one
centre axis — see [`selectors.md`](selectors.md) for the measurement.

## History

- **1.0.0** — a single density scalar drove every gap. It compressed within-section
  and between-section spacing by the same proportion, so hierarchy flattened.
  Failed live. Removed.
- **1.1.0** — hand-set spacing, but re-tuned rather than restored, and still
  carrying `currentColor` tints. Spacing, inline code and blockquote all read too
  heavy. Failed live. Removed.
- **1.2.0** — the verified values above, verbatim, plus auto width. Three
  unrelated slate blues, a blockquote that still showed a double left rule, and a
  width override that could be shadowed.
- **1.3.0** — three shared tokens replace every literal colour; the blockquote
  resets ChatGPT's borders, shadows, background image and generated content before
  drawing its own; width moves to one universal-selector override.

## Verification checklist

Both themes, one long answer and one short one:

- [ ] Body — compact, not stuck together
- [ ] H2 / H3 — clearly headings, breathing room above, tight below
- [ ] Lists — tight, each item still scannable
- [ ] Inline code — faint chip, small radius, no border, unchanged size and colour
- [ ] Blockquote — **one** left rule, fill faint, four corners rounded, text
      optically centred with no rightward shift, no second line
- [ ] Inline code and blockquote greys — same family, same lightness; the quote is
      distinguishable from code but still reads grey, not blue
- [ ] Italic — separates from body text without reading as a second bold;
      bold-italic still bold
- [ ] Horizontal rule — a hairline that separates without announcing a chapter
- [ ] Width ON vs OFF — Auto tracks the window as it is resized, message column and
      composer share one centre axis
- [ ] Fenced code blocks — identical to stock, both themes
- [ ] Page background and body text colour — identical to stock, both themes
