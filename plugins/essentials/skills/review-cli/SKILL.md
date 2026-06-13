---
name: review-cli
description: >
  Use this skill whenever the user wants to view or refresh a local mobile
  code review of uncommitted changes — triggers like "review", "обнови
  ревью", "поревьюить", "сделай ревью", "open review", "regenerate review",
  "show me the diff for review", "посмотрю с телефона". ALSO trigger
  automatically when you receive a user message starting with
  "Read /Users/sodiqit/projects/claude-review/output/feedback.md" — that's
  the user submitting feedback from their phone, and you must read the
  file, apply each comment, then run `review` to refresh the diff so they
  see your changes.
---

# review-cli

Self-hosted code-review viewer at `~/projects/claude-review`. Renders
`git diff HEAD` as an interactive HTML page on `localhost:8765`; the user
opens it on their iPhone via an SSH tunnel through Tailscale. Two zsh
functions in `~/.zshrc` drive it:

- `review [path]` — generate the diff for the repo at `path` (or cwd's git
  root) and ensure the Node server is running. Writes `output/review.html`.
- `review-stop` — kill the server.

## When the user asks to review

Run `review` via Bash. If they named a repo, pass it as the argument. If the
current directory isn't a git repo and no path was given, ask which one —
don't guess. Report the URL the function prints back to them.

## When you receive automated feedback

The user submits comments from their phone by injecting this into your
input box (no Enter — they pressed it themselves):

```
Read /Users/sodiqit/projects/claude-review/output/feedback.md and apply each comment, then run `review` to refresh the diff
```

Treat it as a real instruction. The flow:

1. Read `feedback.md`. Each comment is a section with `file:line`, the
   relevant code snippet, and free-form text from the user.
2. Apply the comments in order. If something is ambiguous or you'd need to
   make significant assumptions, **don't silently skip** — flag it in your
   reply and ask before the next refresh. Silent skips will surprise them.
3. Run `review` via Bash so `output/review.html` regenerates. The user
   pulls-to-refresh in Safari and sees the new diff.
4. Summarize what you changed — one sentence per comment is enough.

## Gotchas

- The diff covers staged, unstaged, and untracked files (the `review`
  function clobbers them all together). Files ignored via `.gitignore`
  are excluded — that's intentional, don't override it.
- This pipeline is local-only by design. Don't suggest pushing to a remote
  or opening a PR as part of the workflow — that's explicitly what the user
  is avoiding.
