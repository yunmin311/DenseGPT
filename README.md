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
| Density | Balanced · Dense · Ultra Dense | Dense |
| Content width | 40rem · 46rem · 54rem · Full | 46rem |
| Inline code | Soft (light fill) · Minimal (no fill) | Soft |

Density is a single scalar over every vertical gap. Headings compress at roughly
half the rate of body copy, so hierarchy gets *more* legible as the page tightens,
not less — see [`docs/design-principles.md`](docs/design-principles.md).

Nothing is shrunk: no font-size changes, no recolouring, no theme. Sidebar,
composer and buttons are untouched.

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
densegpt.user.css          the whole visual layer, 246 lines
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

V1. The CSS is written against ChatGPT's structural DOM hooks
(`[data-message-author-role="assistant"] .markdown` and plain HTML elements) rather
than utility classes, but it has not yet been re-verified against a live session
after ChatGPT's most recent shell update — run the console checks in
[`docs/selectors.md`](docs/selectors.md) if something looks unstyled. Content width
is the one rule that depends on a Tailwind class, and it fails silently and alone.

Not planned: browser extension, injected scripts, React UI, config system, custom
fonts, gradients, animation, colour themes.

## References

- [openstyles/stylus](https://github.com/openstyles/stylus) — UserCSS metadata and variable spec
- [catppuccin/userstyles](https://github.com/catppuccin/userstyles) — semantic selector discipline, light/dark structure, maintenance conventions
- [tobimori/awesome-userstyles](https://github.com/tobimori/awesome-userstyles) — ecosystem survey

## License

MIT
