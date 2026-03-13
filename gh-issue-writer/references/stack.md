# Chance AI — Stack & Issue Ownership

## The Three Repos

| Repo | Stack | What it owns |
|---|---|---|
| `Chance-Inc/App-by-FlutterFlow` | Flutter / Dart (iOS-first) | All user-facing behavior: UI, local state, camera, history, navigation |
| `Chance-Inc/chance-app-backend-nodejs` | Node.js / TypeScript | API gateway, auth, credits, database (PostgreSQL), chat history persistence |
| `Chance-Inc/dubao-va` | Python / FastAPI | AI inference, LLM streaming, visual agents, per-agent logic |

---

## Where to file

**Flutter** — file when the symptom is:
- Something visible in the app (layout, navigation, missing UI)
- Local data loss or wrong state (history, cache)
- App lifecycle issues (background, kill, resume)
- Camera, sharing, notifications

**Node.js** — file when the symptom is:
- API errors or wrong responses
- Data not saved / appearing incorrectly in history
- Credit / auth issues
- Slow server responses

**dubao-va** — file when the symptom is:
- Wrong or low-quality AI analysis results
- A specific agent behaving incorrectly (OOTD, skin, snap-to-shop, palm, etc.)
- Inference errors or timeouts from the AI side

**Not sure?** File in the repo closest to where the user feels the pain (usually Flutter), and note:
*"Root cause may be in [other repo] — needs investigation."*

For issues that clearly span two layers, file in both and cross-link. Backend issue first when an API/contract change is involved.

---

## Key files — quick reference for code pointers

### Flutter (`App-by-FlutterFlow`)

| Area | Files |
|---|---|
| Chat / inference screen | `lib/chat/chat_widget.dart`, `lib/chat/chat_model.dart` |
| History list (Library) | `lib/library/library/library_widget.dart`, `lib/library/library/library_model.dart` |
| Local SQLite history | `lib/database/history_records_table.dart` |
| Local SQLite chat cache | `lib/database/chat_details_table.dart` |
| API definitions | `lib/request/services/chat_api.dart` |
| API service layer | `lib/request/services/chat_service.dart` |
| Camera page | `lib/camera_page/camera_page_widget.dart` |
| Sharing | `lib/sharing/share_executor.dart` |
| App state / navigation | `lib/flutter_flow/nav/nav.dart` |

### Node.js (`chance-app-backend-nodejs`)

| Area | Files |
|---|---|
| Main inference controller | `routes/controllers/unifiedAnalysisController.ts` |
| SSE stream proxy to dubao-va | `routes/services/ChatStreamService.ts` |
| SSE event parser | `routes/utils/SSEEventParser.ts` |
| v3 router (production chat) | `routes/v3.ts` |
| Legacy v2 streaming (Dify) | `routes/controllers/streamingChatController.js` |
| Chat DB model | `routes/models/ChatModel.ts` (Prisma / PostgreSQL) |
| Credit logic | `routes/utils/creditOperations.ts` |
| Auth middleware | `routes/middlewares/authMiddleware.ts` |

### dubao-va

| Area | Files |
|---|---|
| General agent router | `agents/agent_chat/router.py` |
| Session / DB | `agents/agent_chat/session.py` |
| SSE event model | `agents/agent_chat/models.py` |
| Concurrent streaming engine | `agents/streaming.py` |
| Per-agent logic | `agents/<agent-name>/` (ootd, skin, snap_to_shop, palm, nutri_lens, etc.) |
| App entry / route registration | `main.py` |

---

## API endpoints — quick reference

| Endpoint | Repo | Purpose |
|---|---|---|
| `POST /v3/chat` | Node → dubao-va | Main inference (image + query) |
| `GET /v3/chat/:conversationId` | Node | Fetch chat detail for history view |
| `POST /v1/chat/list` | Node | Fetch history list for Library page |
| `DELETE /v1/chat/:conversationId` | Node | Delete a history record |
| `POST /api/v1/agent_chat` | dubao-va | General agent (called by Node internally) |
| `POST /api/v1/<agent>` | dubao-va | Specific agent endpoints |
