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
| inline code | `background: rgba(0,0,0,0.045)` · `padding: 0.08em 0.28em` · `border-radius: 4px` · no border, no shadow |
| `blockquote` | left rule `3px solid rgba(60,82,115,0.72)` · `background: rgba(80,100,130,0.065)` · `padding: 0.55em 0.85em` · `margin: 0.65em 0` · `border-radius: 6px` · inherited text colour |

Seven rules. Anything not in that table is ChatGPT's own styling, untouched.

## What the file must never do

- **No page-level colour.** Nothing sets background, body text colour or
  `color-scheme`. Light and dark are inherited from ChatGPT exactly as shipped.
- **No theme detection.** No `html.dark`, no `[data-theme]`, no
  `prefers-color-scheme`. A tint that is conditional on a guess about the theme
  renders over the wrong background as soon as the guess is wrong — that is what
  turned the blockquote into a grey bar in 1.0.0.
- **No derived colour.** No `color-mix`, no `currentColor` arithmetic. The two
  tints are literal `rgba()` values.
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
wider than 78rem. It is applied through ChatGPT's own
`--thread-content-max-width` custom property so that the message column and the
composer move together — see [`selectors.md`](selectors.md) for the three rules
and how to confirm what they hit.

## History

- **1.0.0** — a single density scalar drove every gap. It compressed within-section
  and between-section spacing by the same proportion, so hierarchy flattened.
  Failed live. Removed.
- **1.1.0** — hand-set spacing, but re-tuned rather than restored, and still
  carrying `currentColor` tints. Spacing, inline code and blockquote all read too
  heavy. Failed live. Removed.
- **1.2.0** — the seven verified values above, verbatim, plus auto width. No other
  rules.

## Verification checklist

Both themes, one long answer and one short one:

- [ ] Body — compact, not stuck together
- [ ] H2 / H3 — clearly headings, breathing room above, tight below
- [ ] Lists — tight, each item still scannable
- [ ] Inline code — faint chip, small radius, no border, unchanged size and colour
- [ ] Blockquote — left rule reads first, fill faint, four corners rounded, not a bar
- [ ] Width ON vs OFF — Auto tracks the window, message column and composer aligned
- [ ] Fenced code blocks — identical to stock, both themes
- [ ] Page background and body text colour — identical to stock, both themes
