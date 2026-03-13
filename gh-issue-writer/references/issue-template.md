# Issue Body Template

Adapt sections to complexity. Drop sections that don't apply for simple bugs.

---

## Problem

[1–2 sentences. What the user experiences. Plain language, no jargon.]

---

## Root Cause

[The actual code path that causes this. Include file paths and line numbers.]

```language
// file: path/to/file.ext ~L123
relevant code snippet showing the problem
```

[Explain why this code causes the symptom.]

---

## Expected Behavior

[What should happen instead. Be specific — not "it should work" but what the correct behavior looks like.]

---

## Suggested Fix

### 1. [First change — file + what to do]

```language
// Proposed code change
```

### 2. [Second change if needed]

```language
// Proposed code change
```

---

## Affected Files

| File | Change |
|------|--------|
| `path/to/file.ext` | Description of change |

---

## Related
- [Link to related issues in other repos if applicable]

---

## Notes on Depth

**Simple bug** — Keep it to: Problem + Root Cause + one-liner fix. Skip the table.

**Logic/state bug** — Add reproduction steps between Problem and Root Cause:
```
## Steps to Reproduce
1. ...
2. ...
3. Observe: ...
```

**Cross-layer bug** — Add a "Contract" section after Root Cause:
```
## Required Contract Change
[Define the new event/API field/endpoint that both sides must agree on]
```

**Architecture gap** — Add an "Options" section before Suggested Fix:
```
## Options
### Option A — [name] (recommended)
[Tradeoffs]

### Option B — [name]
[Tradeoffs]
```
