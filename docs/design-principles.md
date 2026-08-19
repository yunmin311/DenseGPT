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

Anything not in that table is ChatGPT's own styling, untouched.

## One set of tokens

Every colour in the file comes from three tokens, declared once on the assistant
markdown root:

```css
--dg-surface-soft:  color-mix(in srgb, currentColor 4%,   transparent);
--dg-surface-block: color-mix(in srgb, currentColor 5.5%, transparent);
--dg-edge:          color-mix(in srgb, currentColor 35%,  transparent);
```

| Token | Used by | Used by nothing else |
| --- | --- | --- |
| `--dg-surface-soft` | inline code fill | |
| `--dg-surface-block` | blockquote fill | |
| `--dg-edge` | blockquote left rule | |

They are mixed from `currentColor`, so both themes adapt with no theme detection
and no second grey anywhere. The rule is absolute: **no rule may invent its own
colour.** Earlier versions had three unrelated slate blues — `rgba(60,82,115)`,
`rgba(80,100,130)`, `rgba(100,116,139)` — which is what made the greys look dirty
and inconsistent. There are now no literal colour values in the stylesheet at all.

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

It is applied by one rule, `* { --thread-content-max-width: … !important }`,
through ChatGPT's own custom property, so everything that reads that property
moves together. `!important` on the universal selector is what stops an element
that declares the variable on itself from shadowing the override — see
[`selectors.md`](selectors.md) for why that matters and how to confirm which
columns it reaches.

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
- [ ] Inline code and blockquote greys — same family, no colour cast between them
- [ ] Width ON vs OFF — Auto tracks the window as it is resized, message column and
      composer share one centre axis
- [ ] Fenced code blocks — identical to stock, both themes
- [ ] Page background and body text colour — identical to stock, both themes
