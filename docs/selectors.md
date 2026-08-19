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
| `[data-message-author-role="assistant"]` | High. Semantic data attribute, unchanged across years of redesigns. | Everything except width stops applying. One-line fix: find and replace the scope string. |
| `.markdown` | High. Named for its purpose, not its styling. | Same. |
| `p`, `h2`, `h3`, `ul`, `ol`, `li`, `code`, `blockquote`, `em`, `i`, `strong`, `b`, `hr` | Maximum. Plain HTML. | Cannot break. |
| `:not(pre) > code` | Maximum. Structural. | Cannot break. |
| `blockquote::before/::after`, `blockquote p::before/::after` | Maximum. | Cannot break. |

## Content width — one rule

```css
* { --thread-content-max-width: <value> !important; }
```

The universal selector is not a guess about the DOM — it is the only selector that
cannot miss. ChatGPT's own `--thread-content-max-width` is the hook; `!important`
means an element that declares the variable **on itself** cannot shadow it, whether
that declaration comes from a Tailwind arbitrary property
(`[--thread-content-max-width:32rem]`) or from a plain rule in ChatGPT's own
stylesheet. That shadowing is what a `:root`-only override misses.

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
  show('assistant col', a && a.closest('article') && a.closest('article').firstElementChild);
  show('user col', u && u.closest('article') && u.closest('article').firstElementChild);
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

To find out what they actually are, quote a source and attach a file, then paste:

```js
(() => {
  const S = '[data-message-author-role="assistant"]';
  const cls = e => (e.getAttribute('class') || '').slice(0, 120);
  const dump = (label, els) => [...els].slice(0, 6).forEach(e =>
    console.log(label, '<' + e.tagName.toLowerCase() + '>',
      JSON.stringify(Object.fromEntries([...e.attributes].map(a => [a.name, a.value.slice(0, 60)]))),
      '| text:', (e.textContent || '').trim().slice(0, 40)));
  dump('LINK  ', document.querySelectorAll(S + ' .markdown a'));
  dump('CITE? ', document.querySelectorAll(S + ' [class*="citation"], ' + S + ' [data-testid*="cite"], ' + S + ' sup, ' + S + ' cite'));
  dump('CHIP? ', document.querySelectorAll(S + ' [class*="attach"], ' + S + ' [data-testid*="file"], ' + S + ' [class*="chip"]'));
})()
```

Anything that comes back with a stable attribute naming the type is enough to wire
the remaining five tints. Anything that does not is a category ChatGPT does not
model, and the palette entry stays unused rather than being attached to a guess.

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

v1.3.0. Scope and spacing selectors are confirmed by live testing. Two things are
unconfirmed and both are answered by the console check above: the true source of
the blockquote double line, and which columns the width rule reaches.
