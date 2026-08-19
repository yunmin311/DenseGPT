# Selectors

ChatGPT ships Tailwind utility classes that change without notice. This file lists
every hook the style relies on, how stable it is, and what to do when one breaks.

Rule: **no rule may depend on a generated or utility class name unless there is no
structural alternative.** There is currently exactly one such rule (content width),
and it is isolated at the bottom of the stylesheet.

## Inventory

| Hook | Used for | Stability | If it breaks |
| --- | --- | --- | --- |
| `[data-message-author-role="assistant"]` | scope — assistant turns only | High. Semantic data attribute, unchanged across years of redesigns. | Everything stops applying. One-line fix, see below. |
| `.markdown` | scope — the rendered markdown container | High. Named for its purpose, not its styling. | Same as above. |
| `p`, `h1`–`h6`, `ul`, `ol`, `li`, `pre`, `code`, `blockquote`, `table`, `th`, `td`, `hr` | all rhythm rules | Maximum. Plain HTML. | Cannot break. |
| `:not(pre) > code` | separating inline code from code blocks | Maximum. Structural. | Cannot break. |
| `pre div:has(> code)` | code-block inner padding | Medium-high. Structural, but assumes `<code>` sits inside a wrapper `<div>` inside `<pre>`. | Code blocks keep ChatGPT's default padding. Nothing else is affected. |
| `html.dark`, `html[data-theme="dark"]` | dark-mode quote tint | Medium. Tailwind's class strategy; the `data-theme` variant is defensive. | Quote tint falls back to the light value; `prefers-color-scheme` still covers most users. |
| `main article div[class*="thread-content-max-width"]` | content width | **Low.** Depends on a Tailwind arbitrary-value class. | Content width silently does nothing. Nothing else is affected. |

## The one line to fix

If the whole style stops working, ChatGPT renamed the scope. Every rhythm rule is
prefixed with the same pair:

```css
[data-message-author-role="assistant"] .markdown
```

Find and replace that string across `densegpt.user.css` with whatever the new
container is. That is the entire repair.

## Re-verifying after a ChatGPT update

1. Open a conversation, right-click an assistant paragraph → **Inspect**.
2. Walk up the tree until you find the element that holds the whole answer. Confirm
   it has `class="markdown ..."` and that an ancestor has
   `data-message-author-role="assistant"`.
3. In the console, sanity-check each hook:

   ```js
   document.querySelectorAll('[data-message-author-role="assistant"] .markdown').length   // > 0
   document.querySelectorAll('[data-message-author-role="assistant"] .markdown pre div:has(> code)').length
   document.querySelectorAll('main article div[class*="thread-content-max-width"]').length
   ```

   A zero tells you exactly which row of the table above went stale.
4. For content width, also check what actually constrains the column: select the
   message, then walk up until the width stops growing. Read that element's classes.

## Content width, in detail

The rule matches all three Tailwind spellings of the same idea:

```css
main article div[class*="--thread-content-max-width:"]          /* [--thread-content-max-width:32rem] */
main article div[class*="max-w-(--thread-content-max-width)"]   /* Tailwind v4 shorthand */
main article div[class*="max-w-[var(--thread-content-max-width)]"]
```

and sets both the custom property and `max-width`, so it works whether the element
declares the variable or consumes it.

If all three match nothing, uncomment the fallback at the end of the stylesheet:

```css
main article > div > div { max-width: var(--dg-cw) !important; }
```

It is blunt — it hits the turn wrapper directly and will need adjusting if the
nesting depth changes — which is why it ships commented out.

## Deliberate non-targets

Not touched, by design: the sidebar, the composer (except the width it inherits
from `--thread-content-max-width`), the top bar, model picker, message action
buttons, canvas, and user messages. If a rule here ever changes any of those, it is
a bug.

## Status

Selectors are written against ChatGPT's documented-by-observation DOM as of
v1.0.0 and have not yet been re-verified against a live session after the last
update. Run the console checks above before filing a rendering bug — a zero count
is a selector problem, anything else is a styling problem.
