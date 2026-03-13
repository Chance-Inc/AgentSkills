---
name: gh-issue-writer
description: >
  Write and file GitHub issues for the Chance AI app. Use when the user wants to report a bug,
  request a feature, or document a problem — even from just 1-2 sentences of context. Triggers
  include "create an issue", "file a bug", "write a ticket", or any request to open a GitHub issue.
---

# gh-issue-writer

File clear, well-scoped GitHub issues from vague symptoms. The goal is to give the dev enough
context to *find* the problem — not to solve it for them.

> **Internal skill for Chance AI** — 3-layer stack:
> Flutter (`App-by-FlutterFlow`) → Node.js (`chance-app-backend-nodejs`) → Python AI (`dubao-va`)
> See `references/stack.md` for repo backgrounds and ownership map.

## What a good issue does

- Describes the **symptom** clearly from the user's perspective
- Captures **when / how to reproduce** it (steps, conditions, frequency)
- States **expected vs actual** behavior
- Points the dev at the **right repo** — nothing more

What a good issue does NOT do:
- Guess at root cause
- Suggest specific file fixes
- Tell the dev how to implement the solution

That's the dev's job. Wrong guesses waste their time.

## Workflow

### 1. Clarify the symptom (if needed)

If the description is too vague to write a useful issue, ask **one focused question**:

| Vague | Ask |
|---|---|
| "it's slow" | Which screen / action feels slow? |
| "it crashes" | What were you doing right before? Does it always happen? |
| "it looks wrong" | What did you expect to see? |
| "it doesn't work" | What happened instead — error, nothing, wrong result? |

Don't ask multiple questions at once.

### 2. Identify the right repo

Use `references/stack.md` ownership table. When in doubt:
- UI / app behavior → Flutter
- API / data / backend logic → Node.js
- AI inference / agent behavior → dubao-va
- Unsure → file in the most likely one, note uncertainty in the issue

No need to clone or read code to pick the repo.

### 3. Write the issue

Keep it tight. Use this structure:

```
## Symptom
[What the user experiences. 1-3 sentences, plain language.]

## How to reproduce
[Steps or conditions. If unknown, say "Reproduction steps unclear — needs investigation."]

## Expected behavior
[What should happen.]

## Actual behavior
[What happens instead.]

## Context
[App version, device, frequency, any other relevant info the user mentioned.]
```

Drop any section the user has no info for. Don't fill in blanks with guesses.

### 4. File it

```bash
gh issue create \
  --repo Chance-Inc/REPO \
  --title "Short verb-led title describing the symptom" \
  --body "..." \
  --label "bug"
```

For issues that clearly span two repos, file in both and cross-link:
```
Related: Chance-Inc/other-repo#N
```

## References

- `references/stack.md` — Repo ownership, what each layer owns, when to file where
