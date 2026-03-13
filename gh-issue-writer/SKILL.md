---
name: gh-issue-writer
description: >
  Write high-quality GitHub issues from vague symptoms or short descriptions, especially for
  multi-repo/multi-stack apps. Use when the user wants to file a bug report, feature request,
  or engineering ticket — even with only 1-2 sentences of context. Triggers include "create an
  issue", "file a bug", "write a ticket", "open an issue in [repo]", or any request to document
  a problem or feature in GitHub.
---

# gh-issue-writer

Turn vague symptoms into actionable, well-researched GitHub issues.

> **Internal skill for Chance AI** — a cross-platform iOS-first app with a 3-layer stack:
> Flutter (`App-by-FlutterFlow`) → Node.js (`chance-app-backend-nodejs`) → Python AI (`dubao-va`)
> See `references/multi-stack.md` for full architecture, repo backgrounds, and communication flow.

## Core Principle

**Research before writing.** Clone the relevant repo(s) and trace the code before drafting anything. Ten minutes of reading beats hours of dev back-and-forth.

## Workflow

### 1. Extract the signal

From the user's description, identify:
- The **symptom** (what the user experiences)
- The **trigger** (what action causes it)
- The **affected repo(s)** — ask if unclear

### 2. Research the code

```bash
git clone --depth=1 https://github.com/ORG/REPO /tmp/repo-name
```

Then trace the call chain relevant to the symptom:
- Find the UI entry point (widget, screen, button)
- Follow through to the API call
- Find the backend controller/handler
- Check DB writes, SSE events, lifecycle hooks
- For multi-stack apps, trace all the way through (e.g. Flutter → Node → Python → LLM)

Use targeted searches:
```bash
grep -rn "keyword" /tmp/repo-name/lib --include="*.dart" | head -30
find /tmp/repo-name -name "*.ts" | xargs grep -l "relevant_function"
```

### 3. Identify the layer that owns the root cause

See `references/multi-stack.md` for how to assign issues across repos.

### 4. Draft the issue

Use the template in `references/issue-template.md`. Adapt depth to complexity:
- Simple bug → Symptom + root cause + one-file fix
- Logic/state bug → Full code path + reproduction conditions
- Cross-layer bug → Root cause per layer + contract definition + cross-links

### 5. Create the issue

```bash
gh issue create \
  --repo ORG/REPO \
  --title "Short, action-oriented title" \
  --body "$(cat /tmp/issue-body.md)" \
  --label "bug"
```

For cross-repo issues: create backend issue first (it defines the contract), then Flutter/client issue referencing it.

Always cross-link related issues across repos.

## Issue Quality Checklist

Before filing, verify:
```
☐ Root cause identified with file + line reference
☐ Expected vs actual behavior clearly stated
☐ Fix is concrete enough to start coding without questions
☐ Cross-repo dependencies linked
☐ Contract between layers explicitly defined (if applicable)
☐ Each issue is independently closeable
```

## References

- `references/issue-template.md` — Issue body template with section guidance
- `references/multi-stack.md` — How to scope and split issues across multiple repos/services
