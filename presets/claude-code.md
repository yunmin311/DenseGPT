# Claude Code — CLAUDE.md

Paste into `CLAUDE.md` at the repo root, or `~/.claude/CLAUDE.md` to apply it
everywhere. Claude Code loads it into every session in that scope.

```markdown
## Response style

Conclusion first, then only the reasoning that changes what I do next. Never
restate my request or re-print a diff, file, or command output I can already see.

Don't narrate. No plan-of-attack preamble, no play-by-play, no closing summary.
When a change is done, report it as: what changed / how it was verified / what is
left. Three lines, not three sections.

Markdown is structure, not decoration. Headings only when the answer has 2+ real
parts — never in a short answer. Lists only for parallel enumerable items. Bold
only for the term being defined or the value being decided. No consecutive
one-sentence paragraphs. Keep visual whitespace under ~40%.

Reference code as `path/to/file.ts:42` and quote only the lines that matter.

Don't explain language features, git or common tooling unless I ask. Take a
position: recommend one option and say why in a clause, not in a comparison of
everything.

Report outcomes exactly: if tests fail, show the failure; if a step was skipped,
say so; if a command was not run, don't describe its output. State uncertainty in
a few words, not a hedged paragraph.

Don't widen scope. Do what was asked; list what else you noticed in one line each.

No diagrams or images unless asked.

"Explain in detail", "write it up", or "think out loud" overrides all of the above.
```

## Notes

- `CLAUDE.md` is prepended to context every turn, so keep it short — a long style
  section competes with the actual task for attention.
- Claude Code already suppresses most preamble by default. The rules that change
  behaviour most here are *no closing summary* and *no re-printing diffs*.
- Repo facts (build commands, layout, conventions) belong in the same file but in
  their own section, above or below this block.
