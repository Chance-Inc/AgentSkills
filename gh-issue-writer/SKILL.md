---
name: gh-issue-writer
description: >
  Write and file GitHub issues for the Chance AI app. Use when the user wants to report a bug,
  request a feature, or document a problem — even from just 1-2 sentences of context. Triggers
  include "create an issue", "file a bug", "write a ticket", or any request to open a GitHub issue.
---

# gh-issue-writer

File clear, well-scoped GitHub issues from vague symptoms. The goal: give the dev enough context
to find and investigate the problem quickly — symptom, reproduction, and code pointers.

> **Internal skill for Chance AI** — 3-layer stack:
> Flutter (`App-by-FlutterFlow`) → Node.js (`chance-app-backend-nodejs`) → Python AI (`dubao-va`)
> See `references/stack.md` for repo backgrounds, key files, and ownership map.

## What a good issue includes

- **Symptom** — what the user experiences, in plain language
- **Reproduction** — when/how it happens, conditions, frequency
- **Expected vs actual** behavior
- **Code pointers** — relevant module, API endpoint, screen name, or file; enough for the dev to know where to start looking

What a good issue does NOT do:
- Assert the root cause
- Prescribe the fix
- Suggest specific implementation changes

Code pointers are hints, not conclusions. Frame them as *"probably relevant"*, not *"the bug is here"*.

## Workflow

### 1. Clarify the symptom (if needed)

If the description is too vague, ask **one focused question**:

| Vague | Ask |
|---|---|
| "it's slow" | Which screen / action feels slow? |
| "it crashes" | What were you doing right before? Does it always happen? |
| "it looks wrong" | What did you expect to see? |
| "it doesn't work" | What happened instead — error, nothing, wrong result? |

### 2. Identify the right repo(s)

Use `references/stack.md` ownership table. For cross-layer symptoms, file in both repos and cross-link.

### 3. Do a lightweight code scan

Clone shallow and grep for keywords related to the symptom. The goal is **pointers, not analysis**:

```bash
git clone --depth=1 https://github.com/Chance-Inc/REPO /tmp/repo-name
```

Look for:
- The screen/widget/controller that handles the relevant user action
- The API endpoint being called (method + path)
- The relevant service, model, or module name
- Any obviously related file names

**Stop when you have 2–4 pointers.** Don't trace the full call chain. Don't form conclusions.

### 4. Write the issue

```
## Symptom
[What the user experiences. 1-3 sentences, plain language.]

## How to reproduce
[Steps or conditions. If unknown: "Reproduction steps unclear — needs investigation."]

## Expected behavior
[What should happen.]

## Actual behavior
[What happens instead.]

## Potentially relevant code
[2–4 pointers: file paths, API endpoints, module/class names. 
 Frame as hints: "This flow likely goes through X" or "The relevant API is Y".]

## Context
[Device, frequency, any other info the user mentioned.]
```

Drop any section you have no real info for. Don't fill blanks with guesses.

### 5. File it

```bash
gh issue create \
  --repo Chance-Inc/REPO \
  --title "Short verb-led title describing the symptom" \
  --body "..." \
  --label "bug"
```

For cross-repo issues, create backend issue first (it owns the contract), then client issue referencing it.

## References

- `references/stack.md` — Repo ownership, key files per layer, when to file where
