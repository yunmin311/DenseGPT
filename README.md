<div align="center">

# DenseGPT

**Make ChatGPT denser, calmer and easier to scan — both visually and linguistically.**

[![UserCSS](https://img.shields.io/badge/UserCSS-Stylus-2f6feb?style=flat-square)](https://github.com/openstyles/stylus)
[![Release](https://img.shields.io/github/v/release/yunmin311/DenseGPT?style=flat-square&color=2f6feb)](https://github.com/yunmin311/DenseGPT/releases)
[![License](https://img.shields.io/github/license/yunmin311/DenseGPT?style=flat-square&color=2f6feb)](LICENSE)
[![JavaScript](https://img.shields.io/badge/JavaScript-none-2f6feb?style=flat-square)](densegpt.user.css)

**English** · [简体中文](README.zh-CN.md)

</div>

---

ChatGPT wastes two kinds of space. The page is airy — oversized heading gaps, loose
lists, a reading column narrower than the window. And the answers are padded —
preamble, closing summaries, one-sentence paragraphs, Markdown used as decoration.

DenseGPT fixes both, from opposite ends.

| Layer | File | Fixes |
| :--- | :--- | :--- |
| **Visual density** | [`densegpt.user.css`](densegpt.user.css) | One UserCSS file. Tightens the reading layout and gives real control over content width. |
| **Response density** | [`STYLE.md`](STYLE.md) + [`presets/`](presets) | Paste-ready instructions that make the model itself answer densely. |

Use either on its own. Together they compound.

## Contents

- [Install](#install)
- [Content width](#content-width)
- [Response density](#response-density)
- [What it changes, exactly](#what-it-changes-exactly)
- [Documentation](#documentation)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Install

1. Install [Stylus](https://github.com/openstyles/stylus) — Chrome, Edge or Firefox.
2. Open **[densegpt.user.css](https://raw.githubusercontent.com/yunmin311/DenseGPT/main/densegpt.user.css)**. Stylus offers to install it.
3. Reload `chatgpt.com`.

> [!IMPORTANT]
> **To update, use Stylus → Manage → Check for updates.**
> Do not re-import. *Write new style → Import* creates a **second** copy of the
> style; two copies both injecting `!important` rules make settings look like they
> do nothing, and a fresh copy resets every saved value.

Nothing beyond Stylus: no injected JavaScript, no network requests, no telemetry.
One CSS file you can read in five minutes.

## Content width

Replaces the Widescreen extension. Stylus → DenseGPT → **Configure**.

| Width preset | Width | For |
| :--- | :--- | :--- |
| Reading | `44rem` / 704px | long prose, narrowest column |
| **Balanced** *(default)* | `54rem` / 864px | everyday reading |
| Wide | `66rem` / 1056px | code blocks and tables |
| Ultra | `78rem` / 1248px | large screens, reference material |
| Custom | slider, 36–90rem | anything in between |

The slider is read **only** when the preset is Custom; the four presets ignore it.

Every value is wrapped in `min(…, calc(100vw - 3rem))`, so a narrow window shrinks
the column instead of pinning text to the edges. It feeds ChatGPT's own
`--thread-content-max-width`, so assistant messages, your messages and the composer
stay on one centre axis — and because it is a `max-width`, a value wider than the
available space is simply inert and can never cause horizontal overflow.

[`docs/width-preview.html`](docs/width-preview.html) is a standalone page — open it
directly, no server — for picking a Custom value without touching ChatGPT.

## Response density

The CSS makes the page denser. This makes the *answers* denser.

[`STYLE.md`](STYLE.md) is the spec; `presets/` are compiled for each tool:

| Preset | Where it goes |
| :--- | :--- |
| [`presets/chatgpt-custom-instructions.md`](presets/chatgpt-custom-instructions.md) | ChatGPT → Settings → Personalization → Custom instructions |
| [`presets/codex.md`](presets/codex.md) | `AGENTS.md` |
| [`presets/claude-code.md`](presets/claude-code.md) | `CLAUDE.md` |

### Paste this into ChatGPT

Settings → **Personalization** → **Custom instructions** → *"What traits should
ChatGPT have?"*. 1395 characters, fits the 1500-character field.

```text
Answer first: conclusion, then only the reasoning that changes what I do next. Never restate my question or the context I pasted.

Density over packaging. Cut filler, not content. No opening pleasantries, no closing summary, no "let me know if" — end on the last useful line.

Markdown is structure, not decoration. Headings only when the answer has 2+ real parts. Lists only for parallel enumerable items. Tables only for 2+ dimensions over 3+ rows. Bold only for the term being defined or the value being decided. Code formatting for identifiers, paths, flags, commands. Never write consecutive one-sentence paragraphs — join them. Keep visual whitespace under ~40% of the answer.

Length follows the question: a one-line question gets a one-line answer. Don't widen scope for completeness, don't append related-but-unasked knowledge, don't produce a report unless I ask for one.

Don't explain things I didn't ask about. Take a position: recommend one option and say why in a clause. State uncertainty in a few words ("probably X, unverified") instead of hedging across a paragraph.

Voice: direct, plain, technical. No metaphors, no marketing verbs, no self-assessment, no enthusiasm padding. Active voice, shorter word.

No images or diagrams unless I ask.

On request these override the default: "explain in detail" = full depth; "write a report" = long form; "just the answer" = one line.
```

**This block is style only.** It tells the model how to write and claims nothing
about who you are, so it is safe to paste as-is whoever you are.

If you also want it to stop explaining things you already know, that is a separate
setting and it is yours to write. Put one honest line in **"What do you do?"** -
your field, and what you don't need re-explained. That does more than any
assumption a shared preset could make on your behalf.

A 582-character short version, and notes on Projects and per-project instructions,
are in [`presets/chatgpt-custom-instructions.md`](presets/chatgpt-custom-instructions.md).

Two rules do most of the work: **no closing summary** and **no consecutive
one-sentence paragraphs**. The style layer is designed to pair with the CSS — tight
paragraph rhythm only reads well when the model stops padding, and dense prose only
scans well when the page gives it a hierarchy.

## What it changes, exactly

Nine rules on assistant markdown, plus two for width. That is the whole stylesheet.

| Element | Value |
| :--- | :--- |
| `p` | `margin: 0.45em 0` · `line-height: 1.65` |
| `h2` | `margin: 1.15em 0 0.45em` · `font-size: 1.15em` |
| `h3` | `margin: 0.9em 0 0.35em` · `font-size: 1.05em` |
| `ul`, `ol` | `margin: 0.35em 0 0.45em` |
| `li` | `margin: 0.12em 0` |
| inline code | 5% `currentColor` fill · `4px` radius · no border, no shadow |
| `blockquote` | one `3px` left rule · faint slate fill · `6px` radius · normal text colour |
| `em`, `i` | `font-weight: 500` — bold-italic still bold |
| `hr` | 1px hairline at 18% |

**Not touched:** body and heading colours, links, tables, bold, fenced code blocks
(background, syntax highlighting and font size stay ChatGPT's), the sidebar, the top
bar and every message control.

**No theme detection.** No `html.dark`, no `prefers-color-scheme`, and no rule for
`html`, `body` or `:root`. Light and dark are inherited from ChatGPT exactly as
shipped. Four colour values exist in the entire file.

> **Increase information density without reducing semantic clarity.**
>
> Heading, paragraph, list, inline code, code block and quote must stay
> distinguishable at a glance. Anything that fails that is a regression, however
> much it tightens the page.

## Documentation

| Document | What is in it |
| :--- | :--- |
| [`docs/design-principles.md`](docs/design-principles.md) | Every value, why it is that value, and what may never be compressed |
| [`docs/content-width.md`](docs/content-width.md) | The width setting end to end: how preset and slider combine, install traps, a console self-test |
| [`docs/selectors.md`](docs/selectors.md) | Every DOM hook, its stability, and how to repair it when ChatGPT changes |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history, including what failed and why |

## Troubleshooting

| Symptom | Cause |
| :--- | :--- |
| Settings do nothing | More than one copy installed. Stylus → Manage → delete the extras. |
| No **Configure** gear | The metadata block failed to parse. Reinstall from the raw URL. |
| Nothing is styled at all | ChatGPT renamed its scope attribute. See [`docs/selectors.md`](docs/selectors.md) — a one-line repair. |
| Width does nothing | ChatGPT added a new declaration site for its width variable. The ancestor-chain probe in [`docs/selectors.md`](docs/selectors.md) finds it. |

[`docs/content-width.md`](docs/content-width.md) carries a console self-test that
reports all of the above from computed styles rather than guesswork.

## Compatibility

Chrome, Edge and Firefox with Stylus. Requires `color-mix()` and `:has()` —
Chrome 111+, Firefox 113+, Safari 16.4+. Scoped to `chatgpt.com` only.

## Contributing

ChatGPT's DOM moves. If a selector breaks, the most useful bug report is the output
of the probe in [`docs/selectors.md`](docs/selectors.md) — computed styles and
bounding rects beat a description of what it looks like.

Spacing values are deliberate and were verified by eye on real output. Changing them
needs a reason beyond preference.

## References

- [openstyles/stylus](https://github.com/openstyles/stylus) — UserCSS metadata and variable spec
- [catppuccin/userstyles](https://github.com/catppuccin/userstyles) — semantic selector discipline, light/dark structure
- [tobimori/awesome-userstyles](https://github.com/tobimori/awesome-userstyles) — ecosystem survey

## License

[MIT](LICENSE) © yunmin311
