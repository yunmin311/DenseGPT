# Design principles

One acceptance criterion:

> **Increase information density without reducing semantic clarity.**

Everything below is downstream of that sentence. A change that packs more text on
screen but makes it harder to tell a heading from a paragraph is a regression, not
a win.

## The density model

One scalar, `--dg-d`, drives every vertical gap:

| Preset | `--dg-d` |
| --- | --- |
| Balanced | 1.00 |
| Dense (default) | 0.72 |
| Ultra Dense | 0.50 |

Headings do **not** use it directly. They use a softened scalar:

```
--dg-dh = 0.45 + 0.55 × --dg-d      →  1.00 / 0.846 / 0.725
```

That is the whole trick. Body rhythm compresses at full rate, hierarchy markers
compress at roughly half rate, so the *contrast* between a paragraph break and a
section break grows as the page gets denser:

| | Balanced | Dense | Ultra |
| --- | --- | --- | --- |
| paragraph gap | 0.90em | 0.65em | 0.45em |
| H2 top gap | 1.45em | 1.23em | 1.05em |
| ratio (H2 : paragraph) | 1.6× | 1.9× | 2.3× |
| body line-height | 1.72 | 1.64 | 1.57 |

Computed values at each preset:

| Element | Balanced | Dense | Ultra |
| --- | --- | --- | --- |
| `p` margin-block | 0.90em | 0.65em | 0.45em |
| `h2` margin-block | 1.45 / 0.50em | 1.23 / 0.42em | 1.05 / 0.36em |
| `h3` margin-block | 1.15 / 0.40em | 0.97 / 0.34em | 0.83 / 0.29em |
| `ul`, `ol` margin-block | 0.80em | 0.58em | 0.40em |
| `li` margin-block | 0.35em | 0.25em | 0.18em |
| `pre` margin-block | 1.00em | 0.72em | 0.50em |
| code block padding | 1.00rem | 0.76rem | 0.58rem |
| table cell padding | 0.50em | 0.38em | 0.29em |

## What is never compressed

These are the semantic invariants. No preset touches them:

- **Font sizes.** Nothing is shrunk to fit. Density comes from space between
  things, not from smaller type. The only exceptions are inline code (0.9em, a
  typographic correction for monospace x-height) and the untouched code-block size.
- **The distinction between block types.** Heading, paragraph, list, inline code,
  code block, quote and table each keep a visually distinct signature.
- **Colour and weight.** No recolouring of body text, links or quotes.
- **Indentation of nested lists.** Nesting depth stays legible at every preset.
- **ChatGPT's own chrome.** Sidebar, composer, buttons and controls are not styled.

## Per-element decisions

**Inline code** keeps a very light fill and a 4px radius rather than becoming
coloured text. The fill is `color-mix(in srgb, currentColor 6%, transparent)` — it
is derived from the text colour, so it adapts to light and dark with no theme
rules and no hard-coded greys. The `:not(pre) > code` guard is what keeps this off
code blocks; without it, every code block gets a double background.

**Code blocks** lose outer margin and inner padding, keep everything else. Their
font size and syntax colours are untouched — a code block is the one place where
the reader needs the loosest possible scanning, and ChatGPT's own treatment is
already good. The padding rule targets `pre div:has(> code)`, i.e. the element that
structurally holds the code, not a utility class that will be renamed next month.

**Blockquotes** get a single darker left rule (3px), a very light blue-grey fill,
and a radius on all four corners. Text colour stays normal — a quote is a
container, not a de-emphasised aside. Tailwind Typography's injected quotation
marks are removed; the rule and fill already carry the meaning.

**Tables** only lose cell padding. Borders and header weight are ChatGPT's.

**Content width** is the one rule that touches page layout, and the only rule
outside assistant markdown. It overrides `--thread-content-max-width`, which
ChatGPT also uses for the composer — so widening the reading column widens the
composer with it. That is intentional: a 54rem message column above a 46rem
composer looks broken.

## Light and dark

Two mechanisms, in order of preference:

1. **Derive from `currentColor`** wherever possible (inline code fill). Zero theme
   rules, correct in both modes by construction, immune to ChatGPT changing its
   palette.
2. **Explicit tokens** only where a specific hue is required (the blue-grey quote
   tint). Dark values are set for `html.dark` and `html[data-theme="dark"]`, plus a
   `prefers-color-scheme` fallback guarded against an explicitly-chosen light theme.

No hard-coded background or body text colours anywhere. If ChatGPT reskins, this
file still looks correct.

## Explicitly out of scope

Not in V1, and not planned: browser extension, injected JavaScript, React UI, a
config system, sidebar/composer/button restyling, custom fonts, glass or gradient
effects, animation, and full colour themes. Every one of those trades the project's
main property — being a single reviewable CSS file — for polish.

## Verification checklist

Run in one long ChatGPT answer and one short one, in **both** light and dark, at
**each** density preset:

- [ ] English paragraphs — comfortable, not cramped
- [ ] CJK paragraphs — line-height still adequate (CJK needs more than Latin)
- [ ] H1 / H2 / H3 — hierarchy readable at a glance, gaps not equal
- [ ] Two headings in a row — no doubled gap
- [ ] Bulleted and numbered lists — items separable, nesting clear
- [ ] Multi-paragraph list items — no double gap inside `li`
- [ ] Inline code — light fill, readable, *not* recoloured
- [ ] Inline code inside a heading and inside a link — no size or colour break
- [ ] Full code block — padding tight but not clipped, copy button intact,
      horizontal scroll intact, **no inline-code fill leaking in**
- [ ] Blockquote — left rule, faint fill, rounded, normal text colour, no quote marks
- [ ] Table — cells tight, borders intact, wide table still scrolls
- [ ] Links — colour and underline untouched
- [ ] First and last block in a message — no leading or trailing gap
- [ ] Content width — messages and composer aligned
- [ ] Streaming answer — no layout jump while text arrives
