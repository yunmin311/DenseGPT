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

| Variable | Options | Default |
| --- | --- | --- |
| Content width | 40rem · 46rem · 54rem · Full | 46rem |
| Inline code | Soft (light fill) · Minimal (no fill) | Soft |

Spacing is set by hand, per element — there is no density slider and no scalar.
Body copy is tight, space *above* a heading is close to stock so sections still
announce themselves, space *below* it is cut by half so the heading sits with its
content. The values and the ratios behind them are in
[`docs/design-principles.md`](docs/design-principles.md).

Nothing is shrunk: no font-size changes, no recolouring, no theme.

**Light and dark are inherited, untouched.** The file has no rule for `html`,
`body` or `:root`, no theme detection at all, and never sets page background, body
text colour or `color-scheme`. The two blocks that need a tint derive it from
`currentColor` or use one low-alpha value that reads correctly over both
backgrounds. Sidebar, composer and buttons are not styled.

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
densegpt.user.css          the whole visual layer, 248 lines
STYLE.md                   response spec
presets/                   per-tool paste blocks
docs/design-principles.md  the density model, and what is never compressed
docs/selectors.md          every DOM hook, its stability, how to repair it
screenshots/
```

## The rule

> Increase information density without reducing semantic clarity.

Density comes from space between things, never from smaller type. Heading,
paragraph, list, inline code, code block, quote and table must stay
distinguishable at a glance at every setting. Anything that fails that is a
regression.

## Status

**1.1.0 — visual rollback after the first live test failed.** Three changes:

- The density scalar is gone. One multiplier compressed within-section and
  between-section gaps equally, which flattens hierarchy exactly when the page gets
  dense enough to need it. Spacing is hand-set per element now, and the Density
  variable is retired.
- All theme detection is gone. `html.dark` tints render over the wrong background
  whenever the detection misses — that is what turned the blockquote into a grey
  bar. Nothing keys on the theme any more.
- `pre code { background: none }` is gone. It stripped the syntax-highlight surface
  off code blocks in dark mode.

Selectors held up in the live test; the failures were in values, not targeting.
`pre div:has(> code)` and the content-width rule are still unconfirmed — if
something looks unstyled, run the console checks in
[`docs/selectors.md`](docs/selectors.md). Content width is the one rule that
depends on a Tailwind class, and it fails silently and alone.

Not planned: browser extension, injected scripts, React UI, config system, custom
fonts, gradients, animation, colour themes.

## References

- [openstyles/stylus](https://github.com/openstyles/stylus) — UserCSS metadata and variable spec
- [catppuccin/userstyles](https://github.com/catppuccin/userstyles) — semantic selector discipline, light/dark structure, maintenance conventions
- [tobimori/awesome-userstyles](https://github.com/tobimori/awesome-userstyles) — ecosystem survey

## License

MIT
