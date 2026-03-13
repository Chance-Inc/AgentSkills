# Chance AI — Multi-Stack Architecture & Issue Scoping

## The Three-Layer Stack

```
┌─────────────────────────────────────────┐
│  Flutter App (App-by-FlutterFlow)        │  iOS-first cross-platform app
│  lib/chat/, lib/library/, lib/database/ │  FlutterFlow + custom Dart code
└──────────────┬──────────────────────────┘
               │ HTTPS + SSE (text/event-stream)
               │ Auth: signature header + user token
               ▼
┌─────────────────────────────────────────┐
│  Node.js Backend                         │  Express, TypeScript + JS mixed
│  (chance-app-backend-nodejs)             │  Routes: /v1 /v2 /v3 /v4
│  routes/controllers/                     │  Deployed: api.chance.vision
│  routes/services/ChatStreamService.ts   │
│  routes/controllers/                     │
│    unifiedAnalysisController.ts          │
└──────────────┬──────────────────────────┘
               │ HTTP SSE (proxied/forwarded)
               │ Internal service call
               ▼
┌─────────────────────────────────────────┐
│  Python AI Service (dubao-va)            │  FastAPI, Python 3.x
│  agents/agent_chat/router.py            │  Multi-agent platform
│  agents/*/                              │  OpenAI SDK streaming
│  orchestrate/                            │  Deployed separately
└──────────────┬──────────────────────────┘
               │ OpenAI SDK / HTTP
               ▼
         LLM (OpenAI / Gemini)
```

---

## Repo Backgrounds

### Flutter — `Chance-Inc/App-by-FlutterFlow`
- iOS-first cross-platform app built in FlutterFlow with significant custom Dart code
- Core flows: Camera → Chat (inference) → Library (history)
- **Key files for inference flow:**
  - `lib/chat/chat_widget.dart` — main chat screen, SSE consumer, lifecycle hooks
  - `lib/chat/chat_model.dart` — state model for chat
  - `lib/database/history_records_table.dart` — SQLite local history (per-user, soft-delete)
  - `lib/database/chat_details_table.dart` — SQLite cache for chat detail responses
  - `lib/request/services/chat_api.dart` — Retrofit API definitions
  - `lib/request/services/chat_service.dart` — service layer wrapping chat_api
  - `lib/library/library/library_model.dart` — Library page data, history list
- **Local persistence:** SQLite via `HistoryRecordsTable` (history list) + `ChatDetailsTable` (chat detail cache)
- **Remote history:** Synced from Node backend via `GET /v3/chat/:conversationId` and `POST /v1/chat/list`

### Node.js — `Chance-Inc/chance-app-backend-nodejs`
- The main API gateway between Flutter and all AI services
- Mixed TypeScript + JavaScript (actively migrating to TS)
- **Key files for inference flow:**
  - `routes/v3.ts` — v3 router (main production chat endpoint)
  - `routes/controllers/unifiedAnalysisController.ts` — primary inference controller; routes to dubao-va (Python) or Dify depending on agent/version
  - `routes/services/ChatStreamService.ts` — proxies SSE from dubao-va to Flutter, handles `logContext`, parses SSE events
  - `routes/utils/SSEEventParser.ts` — parses dubao-va SSE event stream
  - `routes/models/ChatModel.ts` — PostgreSQL persistence (via Prisma)
  - `routes/controllers/streamingChatController.js` — legacy v2 streaming (Dify-backed, some agents still use this)
- **DB:** PostgreSQL (Prisma ORM) for chat records; Firestore for legacy/some user data
- **Credit system:** Credits checked before inference; deducted after completion

### Python — `Chance-Inc/dubao-va`
- The actual AI inference engine — FastAPI multi-agent platform
- Each agent is a self-contained directory under `agents/`
- **Key files:**
  - `agents/agent_chat/router.py` — `general_lite` and agent_chat endpoint; handles first-call vs follow-up routing
  - `agents/agent_chat/session.py` — SQLAlchemy session management (PostgreSQL); stores conversation history per user
  - `agents/agent_chat/models.py` — SSEEvent model; defines all SSE event types
  - `agents/streaming.py` — concurrent LLM streaming + UI component patching engine
  - `main.py` — FastAPI app entry, registers all agent routers under `/api/v1/` and `/api/v2/`
- **DB:** PostgreSQL (`conversation` table, one row per turn)
- **Agents:** Each agent under `agents/<name>/` has its own router, prompt, and tools

---

## How the Three Repos Communicate

### New inference request (first image analysis)

```
Flutter
  POST /v3/chat  { imageUrl, query, language, agentId }
  headers: x-app-version, x-user-uid, x-user-token, x-app-language

Node (unifiedAnalysisController.ts)
  1. Auth + credit check
  2. Calls ChatStreamService.streamChatV2()
     → logContext() → gets conversation_id from dubao-va (or generates one)
     → POST dubao-va /api/v1/general_lite (or agent-specific py_host)
        { image_url, user_id, language, location }
  3. Pipes dubao-va SSE → Flutter SSE (via SSEEventParser)
  4. After full stream: saveAnalysisData() → PostgreSQL

dubao-va (agent_chat/router.py → _handle_first_call)
  1. Generates conversation_id (uuid7)
  2. Calls visual_search (wit_data)
  3. Streams LLM response as SSE content_token events
  4. Saves session to PostgreSQL
  5. Emits: done { status: "completed", conversation_id }
```

### SSE event flow (dubao-va → Node → Flutter)

dubao-va emits these SSE event types:
```
event: cot_step       → forwarded as cot_steps in Node's SSE
event: content_token  → forwarded as message { answer: delta }
event: ui_component   → forwarded as message { ui_component: {...} }
event: done           → triggers Node to emit analysis_complete to Flutter
```

Node wraps and re-emits to Flutter as:
```
data: { "cot_title": "...", "cot_steps": [...] }
data: { "answer": "delta text", "conversation_id": "..." }
data: { "ui_component": {...}, "conversation_id": "..." }
data: { "event": "analysis_complete", "conversation_id": "...", "dialogue_id": "..." }
```

Flutter's `chat_widget.dart` consumes these via `ChatStreamService` / `_chatStreamController`.

### Follow-up (text query on existing conversation)

```
Flutter
  POST /v3/chat  { query, conversation_id }  ← no imageUrl

Node
  → ChatStreamService detects existing conversation_id + no image
  → Routes to handleExistingConversation()
  → POST dubao-va /api/v1/agent_chat  { query, conversation_id, user_id }

dubao-va (_handle_follow_up)
  → Loads session from PostgreSQL by conversation_id
  → Streams LLM response with conversation history context
  → Emits done (no conversation_id in done event for follow-ups)
```

### History retrieval (Library page / returning to a chat)

```
Flutter (library_model.dart)
  1. Reads SQLite HistoryRecordsTable (local cache, instant)
  2. Fetches POST /v1/chat/list from Node → syncs to SQLite

Flutter (chat_widget.dart, paramChatRef set)
  1. Checks in-memory ChatHistoryCache
  2. Falls back to SQLite ChatDetailsTable
  3. Falls back to GET /v3/chat/:conversationId from Node
     → Node queries PostgreSQL ChatModel.getChatDetail()
     → Returns structured messages with cot + structuralResult
```

---

## When to Create Issues in Which Repo

| Root cause | Issue goes in |
|---|---|
| Flutter UI state, local SQLite, lifecycle | `App-by-FlutterFlow` |
| API contract, Node controller logic, DB save timing | `chance-app-backend-nodejs` |
| LLM inference, SSE event format, Python agent logic | `dubao-va` |
| SSE protocol gap between layers | All affected repos, cross-linked; backend defines contract |
| Credit/auth logic | `chance-app-backend-nodejs` |

## Issue ordering for cross-repo bugs

1. **dubao-va** (if SSE event format needs changing — it's the source)
2. **chance-app-backend-nodejs** (if Node proxy logic or DB save needs changing)
3. **App-by-FlutterFlow** (client-side changes, always references backend issue)

## Cross-link format

```markdown
Related: Chance-Inc/chance-app-backend-nodejs#383
Backend change required: See [Chance-Inc/chance-app-backend-nodejs#383](URL)
```
