# Changelog

Notable changes, including the ones that failed. A userstyle lives against a DOM
it does not control, so what did not work is as useful as what did.

## 2.5.1

**The width clamp measured the window instead of the column.** Reported symptom:
at Ultra the left sidebar was intermittently squeezed, and a reload cleared it.

The responsive term was `calc(100vw - 3rem)`, and `100vw` is the whole window
*including the sidebar*, so the clamp permitted a column wider than the space
`<main>` actually has. Measured in a mock of the same flex layout: a 1440px window
with a 260px sidebar leaves `main` 1143px, while the Ultra clamp still allowed
1248px — 105px of demand with nowhere to put it. That gap is inert only for as
long as every consumer of `--thread-content-max-width` treats it as a `max-width`;
in the same mock, one consumer using it as a `width` overflowed by exactly that
difference, and an overflowing thread is the pressure that can squeeze a
shrinkable sibling.

Now `calc(100cqi - 3rem)` — the inline size of the nearest query container.
ChatGPT declares one named `main`; the `@w-lg/main:` variant in the turn wrapper's
class list is the evidence for it. Container inline size is definite by
containment, so unlike a percentage the term can never fall into cyclic sizing
inside an auto-sized grid track or a shrink-to-fit box, and with no container
ancestor at all it degrades to the small viewport, which is the old behaviour
rather than a broken one.

Verified: in a headless mock the container-relative clamp never exceeds the
available column at 1280 / 1440 / 1920, in block, flex-item, grid-item and padded
containing blocks, where the viewport-relative clamp did. Not verified: that this
was the only path to the squeeze — the live failure was not reproducible without
the account, so `docs/selectors.md` now carries a capture probe to run while it is
on screen.

## 2.5.0

Blockquote colours restored to the 1.1 values: `border-left: 3px solid
rgba(100, 116, 139, 0.85)` and `background-color: rgba(100, 116, 139, 0.09)`.
The 1.2 pair used briefly in 2.4.0 is lighter and still read as plain grey.

## 2.4.0

Blockquote colour moved from derived tokens to literals. Three versions had tried
to derive it from a shared "cite ink" - 15/6, 35/7, 48/7 percent with the rule at
35 then 25 - each arithmetically correct, each measured in Chrome, and none of them
matching the version that had already been approved by eye. `--dg-ink-cite`,
`--dg-surface-block`, `--dg-edge` and `--dg-ink` were deleted rather than kept
as abstraction for its own sake. Two neutral tokens remain.

## 2.1.1

Content width gained a second selector. `<main>` is the inheritance root but not
the only declaration site: the turn wrapper redeclares
`--thread-content-max-width` on itself through a Tailwind arbitrary property, and
a declaration on the element beats an inherited value, so the column measured 560px
regardless of the preset.

## 2.1.0

**The width setting had never worked.** `@preprocessor default` compiles each
`@var` into a CSS custom property; it does not substitute USO-style placeholders,
which belong to `@preprocessor uso`. Every release up to 2.0.0 used placeholders,
so the declaration compiled to an empty value - and an empty custom property is not
an unset one, so the fallback never fired and `min()` received an empty argument.
Confirmed against Stylus 2.4.10's own `js/worker.js`.

## 2.0.0

Width presets plus a free slider, and two install traps documented: re-importing to
update creates a second copy of the style whose `!important` rules make settings
look dead, and select options that name their own value reset every saved choice
when that value is tuned.

## Earlier

**2.1.0 - the width setting was never wired up.** `@preprocessor default`
compiles each `@var` into a CSS custom property; it does not substitute
`/*[[name]]*/` placeholders, which belong to `@preprocessor uso`. Every release
up to 2.0.0 used placeholders, so the declaration compiled to an empty value and
**content width did nothing at any preset**. The expression now reads Stylus's own
generated `--dg-width-preset` and `--dg-width-custom` directly, and select
options carry stable keys so a label can be reworded without resetting saved
choices. Evidence and the manual end-to-end checklist are in
[`docs/content-width.md`](docs/content-width.md).

**2.0.0 - width presets plus a free slider.** Configure now has two controls:
**Width preset** (Reading / Balanced / Wide / Ultra / Custom, labelled by purpose)
and **Custom width**, a native slider from 36 to 90rem, read only in Custom mode.
Two install bugs are fixed by documentation: re-importing to update created a
*second* copy of the style whose `!important` rules made settings look dead, and
option labels that named their own value reset every saved choice whenever a value
was tuned. Everything is measured in
[`docs/content-width.md`](docs/content-width.md); `content_width` is replaced by
`width_preset` + `width_custom`, so the width choice resets once on upgrade.

**1.8.0 — rolled back to the last stable stylesheet.** Citation pills, file-tile
tints and app blocks are removed. All three had stable attribute hooks, but each
needed live probing to find that the hook element was a transparent shell rather
than the painted surface, and the app card turned out to be a cross-document
`<iframe>` that CSS cannot reach at all. The findings are kept in
[`docs/selectors.md`](docs/selectors.md) as a record; none of it is in the
stylesheet.

What remains is what was verified by eye and has stayed stable: markdown rhythm,
inline code, blockquote, emphasis, horizontal rule, and content width.

**1.5.0 — content width moves onto a measured hook.** A live DOM dump reports
`--thread-content-max-width` declared on exactly one element, `<main>`, with the
composer inside it. The rule shrinks from `* { … }` to `main { … }`: same coverage
of assistant, user and composer columns, without an explicit declaration on every
node in the thread.

Citation, attachment and app-block tints are still **not implemented**. The dumped
thread contained none of those elements, so their markup is unknown, and a guessed
selector is worse than an absent one.

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
