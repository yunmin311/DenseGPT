# Selectors

ChatGPT ships Tailwind utility classes that change without notice. Ten selectors
exist in the whole stylesheet. This is all of them.

## Spacing — seven rules

Every one is the same scope plus a plain HTML element:

```css
[data-message-author-role="assistant"] .markdown
```

| Hook | Stability | If it breaks |
| --- | --- | --- |
| `[data-message-author-role="assistant"]` | High. Semantic data attribute, unchanged across years of redesigns. | All spacing stops applying. One-line fix, below. |
| `.markdown` | High. Named for its purpose, not its styling. | Same. |
| `p`, `h2`, `h3`, `ul`, `ol`, `li`, `code`, `blockquote` | Maximum. Plain HTML. | Cannot break. |
| `:not(pre) > code` | Maximum. Structural. | Cannot break. |

If the whole style stops working, ChatGPT renamed the scope. Find and replace that
one string across `densegpt.user.css`. That is the entire repair.

## Content width — three rules

```css
:root                                              { --thread-content-max-width: … }
[class*="--thread-content-max-width:"]             { --thread-content-max-width: … }
[class*="max-w-(--thread-content-max-width)"],
[class*="max-w-[var(--thread-content-max-width)]"] { max-width: … }
```

The primary hook is ChatGPT's own `--thread-content-max-width` custom property, set
at the document root so it inherits down to everything that consumes it — message
column and composer alike. That is the stable half.

The other two exist because inheritance alone is not enough:

- An element that declares the variable **on itself** (Tailwind arbitrary property,
  `[--thread-content-max-width:32rem]`) shadows the inherited value. Rule 2
  overrides that declaration where it is made.
- An element that reads the variable into `max-width` needs the value applied
  directly if it computed before ours landed. Rule 3 covers that.

Both are matched by the **variable name**, not by a styling class, so they survive
Tailwind churn as long as the variable itself keeps its name.

**Status: unverified against a live DOM.** Confirm what they actually hit:

```js
// 1. does ChatGPT define the variable at all?
getComputedStyle(document.documentElement).getPropertyValue('--thread-content-max-width')

// 2. which elements declare or consume it?
document.querySelectorAll('[class*="thread-content-max-width"]').length

// 3. what is the message column and the composer actually clamped to?
getComputedStyle(document.querySelector('[data-message-author-role="assistant"]').closest('article').firstElementChild).maxWidth
getComputedStyle(document.querySelector('main form')).maxWidth
```

A zero on 2 with a value on 1 means the variable is set somewhere other than the
element carrying the class — report the numbers rather than guessing at new
selectors.

## No theme hooks

Deliberately none. No `html.dark`, no `[data-theme]`, no `prefers-color-scheme`, no
`color-mix`, no `currentColor` arithmetic. Both tints are literal `rgba()` values
chosen to read over either background.

The single `:root` rule in the file sets **one custom property** — the content
width — and nothing else. Grep before every release:

```
prefers-color-scheme     →  no matches
color-scheme\s*:         →  no matches
html\.|data-theme        →  no matches
currentColor|color-mix   →  no matches
^\s*(html|body)          →  no matches
^\s*:root                →  exactly one, the width variable
```

## Deliberate non-targets

Not touched: sidebar, top bar, model picker, message action buttons, canvas, user
messages, and fenced code blocks (background, colours, font size and syntax
highlighting all stay ChatGPT's). The composer is affected only by content width,
which is the point — a wide message column above a narrow composer looks broken.

## Status

v1.2.0. Scope and element selectors held up in live testing. The three width rules
are new and unconfirmed; run the console checks above before filing a width bug.
