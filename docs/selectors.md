# Selectors

Twelve rules in the whole stylesheet. This is all of them, and how to verify the
two that are not yet confirmed against a live DOM.

## Spacing, tokens, inline code, blockquote

Every rule is the same scope plus a plain HTML element:

```css
[data-message-author-role="assistant"] .markdown
```

| Hook | Stability | If it breaks |
| --- | --- | --- |
| `[data-message-author-role="assistant"]` | High. Semantic data attribute, confirmed present on the current build. | Everything except width stops applying. One-line fix: find and replace the scope string — `section[data-turn="assistant"]` is the measured alternative. |
| `.markdown` | High. Named for its purpose, not its styling. | Same. |
| `p`, `h2`, `h3`, `ul`, `ol`, `li`, `code`, `blockquote`, `em`, `i`, `strong`, `b`, `hr` | Maximum. Plain HTML. | Cannot break. |
| `:not(pre) > code` | Maximum. Structural. | Cannot break. |
| `blockquote::before/::after`, `blockquote p::before/::after` | Maximum. | Cannot break. |

## Content width — one rule, measured

```css
main { --thread-content-max-width: <value> !important; }
```

Not a guess. The live dump reports:

```
redeclared on 1 element(s): main=74rem [in main]
composer inside main: true
```

That settles it. `<main>` is the only element that declares
`--thread-content-max-width`; nothing below it shadows the value; the composer and
the message columns inherit from that same node. A rule on `<main>` reaches all
three, and `!important` beats ChatGPT's own declaration on the same element.

`<main>` is a semantic landmark, not a utility class, so this is not a
Tailwind-churn hook.

1.4.0 used `*` as insurance against not knowing where the variable was declared.
With the declaration site measured, that insurance is dead weight — it put an
explicit declaration on every node in the thread and bought no coverage that
inheriting from `<main>` does not already give.

If a future build declares the variable deeper, the dump reports it as a second
line and the fix is `main, main *`.

Everything that reads the variable moves together — assistant messages, user
messages, composer — **as long as it reads the variable**. Anything that hard-codes
a literal `max-width` instead is out of reach, and no amount of guessing at class
names will fix it; the check below reports which.

## Console check

Open a conversation that contains a blockquote, press Ctrl+Shift+J, paste this.

**Run it once with DenseGPT disabled** — that is the run that shows what ChatGPT
itself draws. Our `!important` reset masks the original source otherwise. Then run
it again with the style enabled to confirm the fix.

```js
(() => {
  const S = '[data-message-author-role="assistant"] .markdown';
  const R = [], px = v => v && v !== '0px' && v !== 'none' && v !== 'normal';
  const cls = e => (e.getAttribute('class') || '').slice(0, 110);
  const bq = document.querySelector(S + ' blockquote');
  if (!bq) R.push('BLOCKQUOTE: none on screen - ask ChatGPT to quote something first');
  else {
    const c = getComputedStyle(bq);
    R.push('BLOCKQUOTE <' + bq.tagName.toLowerCase() + '> class="' + cls(bq) + '"');
    ['left', 'right', 'top', 'bottom'].forEach(s => px(c['border' + s[0].toUpperCase() + s.slice(1) + 'Width'])
      && R.push('  border-' + s + ': ' + c['border' + s[0].toUpperCase() + s.slice(1) + 'Width'] + ' ' + c['border' + s[0].toUpperCase() + s.slice(1) + 'Color']));
    if (px(c.boxShadow)) R.push('  box-shadow: ' + c.boxShadow);
    R.push('  bg: ' + c.backgroundColor + ' | bg-image: ' + c.backgroundImage);
    R.push('  padding: ' + c.padding + ' | margin: ' + c.margin + ' | text-indent: ' + c.textIndent);
    ['::before', '::after'].forEach(p => { const s = getComputedStyle(bq, p);
      if (s.content !== 'none') R.push('  ' + p + ' content:' + s.content + ' w:' + s.width + ' bg:' + s.backgroundColor + ' border-left:' + s.borderLeftWidth); });
    bq.querySelectorAll('*').forEach(el => { const s = getComputedStyle(el), h = [];
      if (px(s.borderLeftWidth)) h.push('border-left ' + s.borderLeftWidth + ' ' + s.borderLeftColor);
      if (px(s.boxShadow)) h.push('box-shadow ' + s.boxShadow);
      if (px(s.marginLeft)) h.push('margin-left ' + s.marginLeft);
      if (px(s.paddingLeft)) h.push('padding-left ' + s.paddingLeft);
      ['::before', '::after'].forEach(p => { const ps = getComputedStyle(el, p);
        if (ps.content !== 'none') h.push(p + ' ' + ps.content + ' w:' + ps.width + ' bg:' + ps.backgroundColor); });
      if (h.length) R.push('  CHILD <' + el.tagName.toLowerCase() + '> ' + h.join(' | ') + ' class="' + cls(el) + '"'); });
  }
  const gv = e => getComputedStyle(e).getPropertyValue('--thread-content-max-width').trim();
  R.push('WIDTH  root --thread-content-max-width: ' + (gv(document.documentElement) || '(unset)'));
  const re = [...document.querySelectorAll('*')].filter(e => e.parentElement && gv(e) !== gv(e.parentElement));
  R.push('  elements redeclaring it: ' + re.length);
  re.slice(0, 8).forEach(e => R.push('   <' + e.tagName.toLowerCase() + '> ' + gv(e) + ' class="' + cls(e) + '"'));
  const show = (l, e) => R.push('  ' + l + ': ' + (e ? getComputedStyle(e).maxWidth + ' / actual ' + Math.round(e.getBoundingClientRect().width) + 'px class="' + cls(e) + '"' : 'not found'));
  const a = document.querySelector('[data-message-author-role="assistant"]');
  const u = document.querySelector('[data-message-author-role="user"]');
  show('assistant col', a && a.closest('section[data-turn]') && a.closest('section[data-turn]').firstElementChild);
  show('user col', u && u.closest('section[data-turn]') && u.closest('section[data-turn]').firstElementChild);
  show('composer', document.querySelector('main form'));
  R.push('  --user-chat-width: ' + (getComputedStyle(document.documentElement).getPropertyValue('--user-chat-width').trim() || '(unset)'));
  console.log(R.join('\n'));
})()
```

What each line answers:

| Output | Question |
| --- | --- |
| `border-*` on BLOCKQUOTE | is the second line a border on the element itself |
| `box-shadow` on BLOCKQUOTE | is it an inset shadow pretending to be a border |
| `::before` / `::after` with content | is it a pseudo-element bar, or a quote glyph |
| `CHILD <…> border-left` | is it drawn by an inner element — the one case the reset cannot reach |
| `padding-left` / `margin-left` on child | the real source of the horizontal offset |
| `elements redeclaring it` | where `--thread-content-max-width` is actually overridden |
| `assistant col` / `user col` / `composer` | which of the three the width rule actually reaches |
| `--user-chat-width` | whether user bubbles are sized by a separate variable |

## Missing hook: typed source chips

The typed surface palette in [`design-principles.md`](design-principles.md) is
settled, but only two of its seven entries are implemented. Nothing in ChatGPT's
assistant markdown carries a source *type* — there is no `data-source-type`, no
`<cite>`, no per-type class that markdown rendering produces. Block-level
citations map onto `blockquote`, which is why that one is wired; inline citation
chips, app blocks and file chips (PDF, Markdown, CSS, JSON) have no known
selector.

### What a live dump confirmed

A full-thread dump of every `data-*`, `aria-*`, `href`, `download`, `type`, `role`,
`title` and `alt` attribute produced this vocabulary:

| Hook | On | Note |
| --- | --- | --- |
| `data-message-author-role` | inner div of each turn | the scope hook, still alive |
| `data-message-id` | same element | |
| `data-turn` | `<section>` | `"user"` / `"assistant"` — a second scope hook if the first is ever dropped |
| `data-turn-id`, `data-testid="conversation-turn-N"` | `<section>` | |
| `data-turn-id-container`, `data-is-intersecting` | virtualisation wrappers | scroll bookkeeping, not styling hooks |
| `data-message-model-slug` | same element | e.g. `gpt-5-6-thinking` |
| `data-turn-start-message` | same element | |
| `data-start`, `data-end` | **every** rendered markdown node | source offsets from the markdown renderer, on `p`, `strong`, `code`, `li`, `blockquote`… |
| `data-is-last-node`, `data-is-only-node` | last markdown node | |
| `data-testid="copy-turn-action-button"`, `"feedback-turn-action-button"` | action buttons | deliberate non-targets |
| `alt`, `aria-label` on image attachments | `<img>`, its `<button>` | carries the **filename**, e.g. `alt="….png"` |

Turns are `<section>`, **not `<article>`** — a `closest('article')` lookup returns
null on the current build.

`data-start` / `data-end` are the useful discovery lever, not a styling hook: an
ordinary markdown node carries *only* those two. So "every element that has an
attribute other than `data-start`/`data-end`" is an exact, guess-free filter for
everything that is not ordinary markdown — which is what the probes below use.

What two live dumps did **not** contain: any web citation, any non-image file
attachment, any app/connector block. Nothing about their markup is known.

### Confirmed hooks

Three live dumps produced an attribute hook for all three targets.

| Target | Hook | Evidence |
| --- | --- | --- |
| Citation | `[data-testid="webpage-citation-pill"]` | testid inventory, count 1 in a thread with one cited answer |
| Attachment | `[role="group"][aria-label="<filename>"]` on the tile root; the painted surface is the `button` inside it | user-turn dump: `role="group" aria-label="Context_….pdf"`, `aria-label="tsconfig.node.json"` |
| App block | `[data-app-block-preview]` (plus `data-app-block-preview-parking`) | `data-*` name inventory, count 1 each |

The file tile carries its type as the **extension inside `aria-label`**, which is
the `filename` hook — matched with `[aria-label$=".pdf" i]`. It is an attribute
selector on an ARIA attribute, not text matching, and `role="group"` disambiguates
the tile root from the `button` inside it and from the message-action groups,
which also use `role="group"` but never end in a file extension.

All three are implemented: citation, `.pdf` / `.md` / `.markdown` / `.css` /
`.json` tiles, and the app block. Untyped files get no tint.

### The app block, measured

`[data-app-block-preview="true"]` is a real block, not portal scaffolding —
`display: block`, 647 × 148 at the time of the dump, inside the message column. But
it is a **layout wrapper**, not the painted surface:

```html
<div class="group/app-block-preview not-prose mt-4 mb-1 w-full overflow-visible"
     data-app-block-preview="true">
  …
      <div class="relative flex h-full w-full" style="height: 72px;">
        <div class="absolute inset-0 … bg-token-main-surface-primary …">
```

The card is 72px tall inside a 148px wrapper, so tinting the wrapper would paint a
band larger than the card. The paint has to land on the element carrying
`bg-token-main-surface-primary`.

That is a class match, and it is the one in the file — deliberately. It is
ChatGPT's own **semantic surface token**, not a layout utility like `mt-4`, and the
selector is confined to the `[data-app-block-preview]` subtree so it cannot leak.
Nested surfaces are opaque, so the topmost covers those below and tints never
accumulate. If ChatGPT renames its surface tokens this one rule stops tinting and
nothing else is affected.

### Probes

The three targets live in three different places, so one script cannot find them.
Attachments are **not** in the assistant turn: a file the user uploads renders in
the *user* turn. Each probe below prints every element carrying any attribute
other than `data-start`/`data-end` — everything that is not ordinary markdown, and
nothing that has to be guessed at.

**A — Citation.** Needs an assistant turn that actually cites the web: ask
something that forces a search, wait for the answer to render its sources, then:

```js
(() => {
  const a = [...document.querySelectorAll('[data-message-author-role="assistant"]')].pop();
  const turn = a && (a.closest('section[data-turn]') || a);
  if (!turn) return console.log('no assistant turn');
  const rows = [];
  (turn.querySelector('.markdown') || turn).querySelectorAll('*').forEach(e => {
    const at = [...e.attributes].filter(x => x.name !== 'data-start' && x.name !== 'data-end');
    if (at.length) rows.push('<' + e.tagName.toLowerCase() + '> ' +
      at.map(x => x.name + '="' + x.value.slice(0, 90) + '"').join(' ') +
      '  | ' + (e.textContent || '').trim().slice(0, 50));
  });
  console.log('=== A. CITATION (assistant markdown) ===\n' + (rows.join('\n') || 'nothing but plain markdown - this answer cites no sources'));
})()
```

**B — Attachment.** Send one message with a PDF and a `.json` or `.css` attached,
then run this. It reads the **latest user turn**:

```js
(() => {
  const u = [...document.querySelectorAll('[data-message-author-role="user"]')].pop();
  const turn = u && (u.closest('section[data-turn]') || u);
  if (!turn) return console.log('no user turn');
  const rows = [];
  turn.querySelectorAll('*').forEach(e => {
    const at = [...e.attributes].filter(x => x.name !== 'data-start' && x.name !== 'data-end');
    if (at.length) rows.push('<' + e.tagName.toLowerCase() + '> ' +
      at.map(x => x.name + '="' + x.value.slice(0, 90) + '"').join(' ') +
      '  | ' + (e.textContent || '').trim().slice(0, 50));
  });
  console.log('=== B. ATTACHMENT (latest user turn) ===\n' + rows.join('\n'));
})()
```

**C — App / Connector.** Run with an app or connector block on screen. This is an
inventory rather than a dump, so the output stays short: every distinct
`data-testid`, every `role`, and every `data-*` attribute name inside `main`, with
counts. An app block will appear as a testid that is not one of the known
`*-turn-action-button` entries:

```js
(() => {
  const tally = (sel, get) => { const m = {};
    document.querySelectorAll(sel).forEach(e => { const v = get(e); if (v) m[v] = (m[v] || 0) + 1; });
    return Object.entries(m).sort((x, y) => y[1] - x[1]).map(([k, v]) => v + '  ' + k).join('\n'); };
  const names = {};
  document.querySelectorAll('main *').forEach(e => [...e.attributes].forEach(a => {
    if (a.name.startsWith('data-')) names[a.name] = (names[a.name] || 0) + 1; }));
  console.log('=== C1. data-testid ===\n' + tally('[data-testid]', e => e.getAttribute('data-testid')));
  console.log('=== C2. role ===\n' + tally('[role]', e => e.getAttribute('role')));
  console.log('=== C3. data-* names in main ===\n' +
    Object.entries(names).sort((x, y) => y[1] - x[1]).map(([k, v]) => v + '  ' + k).join('\n'));
})()
```

A type is wirable only if the output shows an attribute that *names* the type on a
stable element — `data-*`, `aria-*`, `href`, `download`, or a filename in `alt` /
`download`. A class name, a position, or visible text is not a hook: `nth-child`,
text matching and guessed classes are all banned, so a category with no attribute
stays unimplemented rather than faked.

## No theme hooks

Deliberately none. No `html.dark`, no `[data-theme]`, no `prefers-color-scheme`.
There is no rule for `html`, `body` or `:root`. Every colour in the file is one of
the three tokens, all mixed from `currentColor`. Grep before every release:

```
prefers-color-scheme    →  no matches
color-scheme\s*:        →  no matches
html\.|data-theme       →  no matches
rgba?\(|#[0-9a-f]{3,8}  →  no matches (three color-mix tokens only)
```

## Deliberate non-targets

Not touched: sidebar, top bar, model picker, message action buttons, canvas, links,
tables, bold, and fenced code blocks — background, colours, font size and syntax
highlighting all stay ChatGPT's. The composer is affected only by content width.

## Status

v1.5.0. The scope hook and the width declaration site are both confirmed against a
live DOM. Still unconfirmed: the true source of the blockquote double line, and
the markup of citations, file attachments and app blocks — the dumped thread
contained none of them.
