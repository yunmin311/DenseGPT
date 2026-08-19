# DenseGPT

**Make ChatGPT denser, calmer and easier to scan — both visually and linguistically.**

Two layers, no extension, no JavaScript:

| Layer | File | Fixes |
| --- | --- | --- |
| Visual density | `densegpt.user.css` | ChatGPT's reading layout — airy paragraph rhythm, oversized heading gaps, loose lists |
| Response density | `STYLE.md` + `presets/` | The model's output — preamble, closing summaries, one-sentence paragraphs, Markdown as decoration |

Use either on its own. They fix the same problem from opposite ends.

## Install — visual layer

1. Install [Stylus](https://github.com/openstyles/stylus) (Chrome / Firefox / Edge).
2. Open [`densegpt.user.css`](densegpt.user.css) → **Raw** → Stylus offers to install it.
3. Reload `chatgpt.com`.

Or: Stylus → **Write new style** → **Import** → paste the file.

### Variables

Stylus → DenseGPT → **Configure**.

One variable, content width:

| Option | Value | |
| --- | --- | --- |
| **Auto** | `clamp(48rem, 72vw, 78rem)` | default — tracks the window |
| Reading | `46rem` | |
| Wide | `54rem` | |
| Ultra Wide | `68rem` | |
| Full | `none` | |

Auto replaces the Widescreen extension: 72% of viewport width, floored at 48rem,
capped at 78rem, re-evaluated live as the window is resized or zoomed. It is
applied through ChatGPT's own `--thread-content-max-width` property, so everything
that reads it — messages and composer — moves together on one centre axis.

Spacing is not configurable. Every value is fixed and verified by eye on real
output — no density slider, no scalar, no calc. They are listed in full in
[`docs/design-principles.md`](docs/design-principles.md).

**One set of tokens.** Two inks and four surfaces, all anchored to `currentColor`:
`--dg-surface-soft` (inline code), `--dg-surface-block` and `--dg-edge`
(blockquote), `--dg-edge-soft` (horizontal rule). A category is separated by
shifting the same ink a few percent toward a desaturated hue — never by giving it
a colour of its own — so a quoted source reads differently from a piece of code
while both still read grey. Surfaces stay at 5–6% alpha, edges at 18–35%, and
there is exactly one literal colour in the file: the hue anchor.

**Light and dark are inherited, untouched.** No theme detection of any kind, and
no rule for `html`, `body` or `:root`. Fenced code blocks are not touched at all —
background, colours, font size and syntax highlighting stay exactly as ChatGPT
ships them. Sidebar, links, tables and buttons are not styled; the composer is
affected only by width.

## Install — response layer

[`STYLE.md`](STYLE.md) is the spec. `presets/` are compiled versions:

- [`presets/chatgpt-custom-instructions.md`](presets/chatgpt-custom-instructions.md) — fits the 1500-char Custom Instructions field
- [`presets/codex.md`](presets/codex.md) — `AGENTS.md` block
- [`presets/claude-code.md`](presets/claude-code.md) — `CLAUDE.md` block

Core rules: conclusion first · never restate context · no preamble or closing
summary · Markdown is structure, not decoration · no consecutive one-sentence
paragraphs · whitespace under ~40% · length follows the question · assumes a CS
background · `"explain in detail"` overrides everything.

## Structure

```
densegpt.user.css          the whole visual layer, 199 lines
STYLE.md                   response spec
presets/                   per-tool paste blocks
docs/design-principles.md  the density model, and what is never compressed
docs/selectors.md          every DOM hook, its stability, how to repair it
screenshots/
```

## The rule

> Increase information density without reducing semantic clarity.

Heading, paragraph, list, inline code, code block and quote must stay
distinguishable at a glance. Anything that fails that is a regression, however
much it tightens the page.

## Status

**1.4.0 — hue-traced tokens, italic weight, hairline rule.** The three flat greys
become two inks and four surfaces: a category is now separated by shifting the
*same* ink 15% toward a desaturated hue, at identical lightness, rather than by a
colour of its own. The blockquote moves onto the cite ink so a quoted source no
longer reads as code. `em`/`i` get `font-weight: 500` (bold-italic excluded, so
`strong` keeps its weight). `hr` becomes a 1px hairline at 18%.

The typed source palette — app, PDF, Markdown, CSS, JSON — is specified in
[`docs/design-principles.md`](docs/design-principles.md) but **not implemented**:
nothing in assistant markdown carries a source type, so there is no selector to
attach it to. `docs/selectors.md` has a console dump that reports what citation
and file chips actually are.

**1.3.0 — one token set, a full blockquote reset, and width on a hook that cannot
be shadowed.** Three literal slate blues are gone, replaced by three
`currentColor` tokens. The blockquote now clears ChatGPT's borders, box-shadow,
background image, generated quote marks and inherited indent before drawing its
own single left rule. Content width moved from three class-matched rules to one
universal-selector override of `--thread-content-max-width`.

Unconfirmed, both answered by the console check in
[`docs/selectors.md`](docs/selectors.md): the true source of the blockquote double
line, and which columns the width rule actually reaches.

**1.2.0 — minimum rollback to the verified baseline, plus Auto width.**

1.0.0 drove every gap from one density scalar; it flattened hierarchy and failed
live. 1.1.0 hand-set the spacing but re-tuned it instead of restoring it, and kept
`currentColor` tints; spacing, inline code and blockquote all read too heavy, and
it failed live too. 1.2.0 is the seven verified values verbatim, nothing else, plus
content width.

Everything else is deleted: the density scalar and all `calc`, all theme
detection, all derived colour, and every rule for `h1`, `h4`–`h6`, `pre`, `table`,
`hr`, `.katex-display`, first/last-child resets, margin-collapse fixes and
Tailwind quote-mark resets.

Unconfirmed: the three content-width rules. If width does nothing, run the console
checks in [`docs/selectors.md`](docs/selectors.md) and report the numbers rather
than adding selectors.

Not planned: browser extension, injected scripts, React UI, config system, custom
fonts, gradients, animation, colour themes.

## References

- [openstyles/stylus](https://github.com/openstyles/stylus) — UserCSS metadata and variable spec
- [catppuccin/userstyles](https://github.com/catppuccin/userstyles) — semantic selector discipline, light/dark structure, maintenance conventions
- [tobimori/awesome-userstyles](https://github.com/tobimori/awesome-userstyles) — ecosystem survey

## License

MIT
