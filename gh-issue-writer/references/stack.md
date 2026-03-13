# Chance AI — Stack & Issue Ownership

## The Three Repos

| Repo | Stack | What it owns |
|---|---|---|
| `Chance-Inc/App-by-FlutterFlow` | Flutter / Dart (iOS-first) | All user-facing behavior: UI, local state, camera, history, navigation |
| `Chance-Inc/chance-app-backend-nodejs` | Node.js / TypeScript | API gateway, auth, credits, database (PostgreSQL), chat history persistence |
| `Chance-Inc/dubao-va` | Python / FastAPI | AI inference, LLM streaming, visual agents, per-agent logic |

## Where to file

**Flutter** — file when the symptom is:
- Something visible in the app (layout, navigation, missing UI)
- Local data loss or wrong state (history, cache)
- App lifecycle issues (background, kill, resume)
- Camera, sharing, notifications

**Node.js** — file when the symptom is:
- API errors or wrong responses
- Data not saved / appearing incorrectly in history
- Credit/auth issues
- Slow server responses

**dubao-va** — file when the symptom is:
- Wrong or low-quality AI analysis results
- Specific agent behaving incorrectly (OOTD, skin, snap-to-shop, etc.)
- Inference errors or timeouts from the AI side

**Not sure?** File in the repo closest to where the user feels the pain (usually Flutter), and note: *"Root cause may be in [other repo] — needs investigation."*

## Cross-repo issues

If the symptom clearly involves two layers (e.g. data shown wrong in app AND not saved on server), file in both repos and cross-link:

```
Related: Chance-Inc/chance-app-backend-nodejs#383
```

File the backend issue first when the fix requires an API/contract change — the client issue references it.
