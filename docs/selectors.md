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
| `data-testid="copy-turn-action-button"` etc. | action buttons | deliberate non-targets |
| `alt`, `aria-label` on image attachments | `<img>`, its `<button>` | carries the **filename**, e.g. `alt="…​.png"` |

Turns are `<section>`, **not `<article>`** — a `closest('article')` lookup returns
null on the current build.

What the dump did **not** contain: any web citation, any non-image file
attachment, any app/connector block. The thread had none, so nothing about their
markup is known.

### Still needed

Produce one assistant turn that actually contains the three targets — ask
something that makes ChatGPT cite a web source, attach a PDF and a `.json` or
`.css` file, use an app/connector block if you have one — then paste this. It
scopes to a single turn so the output is not truncated:

```js
(() => {
  const a = [...document.querySelectorAll('[data-message-author-role="assistant"]')].pop();
  const turn = (a && (a.closest('section[data-turn]') || a.closest('[data-turn-id]'))) || a;
  if (!turn) return console.log('no assistant turn found');
  const keep = n => n.startsWith('data-') || n.startsWith('aria-') ||
    ['href', 'download', 'type', 'role', 'title', 'alt'].includes(n);
  const rows = [];
  turn.querySelectorAll('*').forEach(e => {
    const at = [...e.attributes].filter(x => keep(x.name));
    if (at.length) rows.push('<' + e.tagName.toLowerCase() + '> ' +
      at.map(x => x.name + '="' + x.value.slice(0, 70) + '"').join(' ') +
      '  | ' + (e.textContent || '').trim().slice(0, 40));
  });
  console.log('=== ONE ASSISTANT TURN ===\n' + rows.join('\n'));
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
