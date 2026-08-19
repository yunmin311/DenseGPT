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
--dg-ink-cite:      color-mix(in srgb, currentColor 85%, #5b7196);

--dg-surface-soft:  color-mix(in srgb, var(--dg-ink) 5%, transparent);
--dg-surface-block: color-mix(in srgb, var(--dg-ink-cite) 6%, transparent);
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

A category is separated by shifting the *same* ink a few percent, never by giving
it a colour of its own. `--dg-ink-cite` is 85% currentColor and 15% of a
desaturated blue — enough that a quoted source does not read as a piece of code,
not enough to register as blue. First impression stays grey.

The constraints that matter, in order:

| Constraint | Value |
| --- | --- |
| surface alpha | 5–6% — never outside 5–7% |
| hue shift between types | ≤ 15% of a desaturated anchor |
| saturation of the anchor | very low |
| lightness across types | identical (all inherit from `currentColor`) |
| edges and icons | 18–35%, the only things allowed to deepen |

Earlier versions had three unrelated slate blues — `rgba(60,82,115)`,
`rgba(80,100,130)`, `rgba(100,116,139)` — one per element, at three different
lightnesses. That is what made the greys look dirty. There is now exactly one
literal colour in the stylesheet: the hue anchor.

### Typed surfaces

Every ink is `currentColor` 85% plus 15% of a desaturated anchor, so all types
share one lightness and differ only in trace hue.

| Type | Anchor | Surface | Edge | Wired to |
| --- | --- | --- | --- | --- |
| plain inline code | none — neutral | 5% | — | `:not(pre) > code` |
| quote / citation | `#5b7196` grey-blue | 6% block, 7% pill | 35% | `blockquote`, `[data-testid="webpage-citation-pill"]` |
| PDF | `#96706b` warm grey | 6% | 22% | `[aria-label$=".pdf" i]` tile |
| Markdown | `#6b9680` grey-green | 6% | 22% | `[aria-label$=".md" i]` tile |
| CSS | `#806b96` grey-violet | 6% | 22% | `[aria-label$=".css" i]` tile |
| JSON / config | `#96896b` grey-sand | 6% | 22% | `[aria-label$=".json" i]` tile |
| app block | — | — | — | **not wired**, see [`selectors.md`](selectors.md) |

Tokens are declared on `<main>`, not on the markdown root, because file tiles live
in the *user* turn. `currentColor` still resolves on the element that uses the
token, so the tint follows whatever text colour that element has.

Tile tints are painted as a background *image* layer over ChatGPT's own
`background-color`. Replacing the colour would turn an opaque card translucent;
layering keeps the card and adds only the trace.

Untyped files get no tint at all. A category is coded only where the DOM states
its type — guessing it from a class name or from the visible label is how 1.0.0
and 1.2.0 failed.

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

| Option | Value |
| --- | --- |
| Auto (default) | `clamp(48rem, 72vw, 78rem)` |
| Reading | `46rem` |
| Wide | `54rem` |
| Ultra Wide | `68rem` |
| Full | `none` |

Auto tracks the window: 72% of viewport width, never narrower than 48rem, never
wider than 78rem. Resizing or zooming re-evaluates it — nothing has to be
re-selected.

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
