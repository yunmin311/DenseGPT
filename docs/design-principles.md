# Design principles

One acceptance criterion:

> **Increase information density without reducing semantic clarity.**

Everything below is downstream of that sentence. A change that packs more text on
screen but makes it harder to tell a heading from a paragraph is a regression, not
a win.

## No density scalar

V1.0 drove every vertical gap from a single scalar. It failed on real screens: one
multiplier compresses the gap *inside* a section and the gap *between* sections by
the same proportion, so hierarchy flattens exactly when the page gets dense enough
to need it. Removed in 1.1.0.

Spacing is now set by hand, per element. Hierarchy is a set of deliberate ratios,
not one number multiplied across the page:

| Element | DenseGPT | ChatGPT stock | Note |
| --- | --- | --- | --- |
| body `line-height` | 1.7 | ~1.75 | compact but never sticky; CJK needs the room |
| `p` margin | 0.8em | ~1.25em | the unit everything else is judged against |
| `h1` margin | 1.8em / 0.5em | ~2.2em / 1em | |
| `h2` margin | 1.7em / 0.45em | ~2em / 1em | above ≈ stock, below cut by half |
| `h3` margin | 1.4em / 0.4em | ~1.6em / 0.6em | |
| `h4`–`h6` margin | 1.15em / 0.35em | | |
| heading + heading | 0.7em | | stacked headings must not double up |
| `ul`, `ol` margin | 0.75em | ~1.25em | |
| `li` margin | 0.35em | ~0.5em | ≈ ⅓ of the paragraph gap |
| nested list margin | 0.3em | | |
| `p` + list | 0.4em | | the list usually finishes the sentence |
| `pre` margin | 1em | ~1.7em | |
| code block padding | 0.85rem 0.95rem | 1rem | |
| `blockquote` padding | 0.55em 0.9em | | |
| `th`, `td` padding | 0.45em 0.7em | | |
| `hr` margin | 1.6em | | |

The two ratios that carry the hierarchy:

- **Section break : paragraph break = 1.7em : 0.8em ≈ 2.1×.** A new H2 is
  unmistakably a new section, at a glance, without scrolling back.
- **Above a heading : below it ≈ 3.8×.** The heading sits with the content it
  introduces instead of floating midway between two blocks. This only works
  because of the `heading + block { margin-top: 0 }` rule — otherwise the next
  block's own top margin wins the margin collapse and the small bottom value is
  never seen.

## What is never compressed

Semantic invariants:

- **Font sizes.** Nothing is shrunk to fit. Density comes from space between
  things, never from smaller type. The single exception is inline code at 0.9em, a
  typographic correction for monospace x-height.
- **The distinction between block types.** Heading, paragraph, list, inline code,
  code block, quote and table each keep a visually distinct signature.
- **Colour and weight.** No recolouring of body text, links, headings or quotes.
- **Indentation of nested lists.**
- **ChatGPT's own chrome.** Sidebar, composer, buttons and controls are not styled.

## Light and dark

**DenseGPT contains no rule for `html`, `body` or `:root`, and never sets page
background, body text colour or `color-scheme`.** Both themes are inherited from
ChatGPT, untouched. There is also no theme detection — no `html.dark`, no
`prefers-color-scheme`. That was the 1.0.0 dark-mode bug: theme-conditional tints
mean that whenever the detection misses, a light-theme fill renders over a dark
page, which is what turned the blockquote into a grey bar across the column.

Two mechanisms replace it, neither of which needs to know the theme:

1. **Derive from `currentColor`.** Inline code fills with
   `color-mix(in srgb, currentColor 5%, transparent)` — computed from the text
   colour of the element it sits in, so it is correct in both themes by
   construction and survives any ChatGPT palette change.
2. **One low-alpha value that works over either background.** The blockquote rule
   and fill are `rgba(100, 116, 139, …)` at 0.85 and 0.09. Over white it reads as a
   slightly darker blue-grey; over ChatGPT's dark surface it reads as a slightly
   lighter one. Same declaration, no branch.

## Per-element decisions

**Inline code** keeps a very light fill, a 4px radius, and no border. It is never
recoloured, so it does not become the focal point of a line. The `:not(pre) > code`
guard is what keeps this off code blocks; without it every code block gets a second
background.

**Code blocks** lose outer margin and inner padding, and nothing else. Font size,
syntax colours and the highlight background stay exactly as ChatGPT ships them —
1.0.0 reset `background` on `pre code`, which strips the highlight surface and
shows through to the page in dark mode. The padding rule targets
`pre div:has(> code)`, the element that structurally holds the code, not a utility
class that will be renamed next month.

**Blockquotes** get a single slightly deeper left rule (3px), a very faint fill,
and a 6px radius on all four corners. Text colour stays normal — a quote is a
container, not a de-emphasised aside. Tailwind Typography's injected quotation
marks are removed; the rule and fill already carry the meaning.

**Tables** only lose cell padding. Borders and header weight are ChatGPT's.

**Content width** is the one rule that touches page layout, and the only rule
outside assistant markdown. It overrides `--thread-content-max-width`, which
ChatGPT also uses for the composer — so widening the reading column widens the
composer with it. That is intentional: a 54rem message column above a 46rem
composer looks broken.

## Explicitly out of scope

Not in V1, not planned: browser extension, injected JavaScript, React UI, config
system, sidebar/composer/button restyling, custom fonts, glass or gradient effects,
animation, colour themes. Each trades the project's main property — being a single
reviewable CSS file — for polish.

## Verification checklist

Run in one long ChatGPT answer and one short one, in **both** light and dark:

- [ ] English paragraphs — compact, not stuck together
- [ ] CJK paragraphs — line-height still adequate
- [ ] H1 / H2 / H3 — clear breathing room *above*, tight *below*
- [ ] A heading immediately followed by a list or code block — still tight below
- [ ] Two headings in a row — no doubled gap
- [ ] Bulleted and numbered lists — tighter than stock, each item still scannable
- [ ] Multi-paragraph list items — no double gap inside `li`
- [ ] Inline code — barely-there fill, small radius, no border, not a focal point
- [ ] Inline code inside a heading and inside a link — no size or colour break
- [ ] Full code block — padding tight but not clipped, **highlight background and
      syntax colours unchanged**, copy button and horizontal scroll intact
- [ ] Blockquote — left rule reads first, fill is faint, four corners rounded,
      normal text colour, **not a grey bar**
- [ ] Table — cells tight, borders intact, wide table still scrolls
- [ ] Links — colour and underline untouched
- [ ] First and last block in a message — no leading or trailing gap
- [ ] Page background and body text colour — identical to stock, both themes
- [ ] Content width — messages and composer aligned
- [ ] Streaming answer — no layout jump while text arrives
