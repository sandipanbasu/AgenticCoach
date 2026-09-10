# V1 Flows and Scope

| Field | Value |
|---|---|
| **Status** | Draft (pending review) |
| **Date** | 2026-09-05 |
| **Author** | Sandipan Basu + ruflo session |
| **Supersedes** | §5 of [AriseHub Agent Playground Architecture](./AriseHub_Agent_Playground_Architecture.md) — *this doc narrows V1 scope; the architecture doc should be updated to match once this is accepted* |
| **Related** | [ADR-001: Upstream Strategy](./ADR-001-upstream-strategy.md) · [Architecture Baseline](./AriseHub_Agent_Playground_Architecture.md) · [Feature Version Matrix](./feature-version-matrix.md) |

---

## Purpose

The architecture baseline ([AriseHub_Agent_Playground_Architecture.md](./AriseHub_Agent_Playground_Architecture.md)) §5 lists eight V1 capabilities. This document argues that list is too large for a single V1, and proposes a sharper V1 / V1.1 / V2 split. It then defines the user-facing and runtime flows that V1 must ship.

**Read this as the V1 contract.** Once accepted, the architecture doc's §5 should be updated to point here rather than re-stating a different scope.

---

## TL;DR — the V1 / V1.1 / V2 split

| Version | Ship window | Capabilities | Differentiator |
|---|---|---|---|
| **V1** | First release | Playground chat, Knowledge base management, Run-as-first-class (read-only), Starter agent library | "An agent playground, not a chatbot" — observability is the load-bearing feature |
| **V1.1** | ~60 days after V1 | Engineering workspace (file upload, basic repo ops), subagent delegation UX, re-run a Run with different model/instructions, user-generated Agent Library, side-by-side run comparison | The workbench: clone, re-run, compare |
| **V2** | After V1.1 | Code sandbox, full repo engineering, visual workflow builder, evaluation framework, guided learning, local desktop, institutional dashboards, advanced routing | The "compose" and "learn" layers from the architecture doc §11 |

The architecture baseline's §9 delivery sequence (Phases 0–5) is consistent with this split, but renames the phases. This document replaces the loose phase numbering with explicit version milestones so the delivery semantics are unambiguous.

---

## V1 Scope — explicit

### In V1 (4 capabilities)

1. **Playground chat** — single-agent chat on a knowledge base. The user can select a model, set instructions, attach a KB, and toggle tools. This is the DeepTutor chat flow, rebranded for AriseHub.

2. **Knowledge base management** — upload documents (PDF, Markdown, DOCX), build a KB, attach it to a chat. Built on top of the fork's `KnowledgeBaseManager` and `add_documents` flow.

3. **Run-as-first-class object (read-only)** — every chat turn becomes a `Run` with:
   - A stable identifier (UUID).
   - A normalized event stream (using the §6 vocabulary from the architecture doc).
   - Persistence (JSONL-backed `run_store`).
   - A Run Inspector page (live and replay modes).
   - **Read-only** in V1: we record and view, but do not yet offer re-run / branch / compare. Those are V1.1.

4. **Starter agent library** — a small set of preconfigured agent templates (Research, Tutor, Code Reader, etc.) bundled with AriseHub. Users can pick a starter, edit its config, and run. User-cloned agents are V1.1.

### Explicitly NOT in V1

The architecture doc §5 lists eight V1 capabilities. We are deferring four to V1.1 or V2:

- **Workspace with file upload and repo operations** → V1.1.
- **Subagent delegation in the chat** (as a user-visible feature) → V1.1. (Subagents may still be used *internally* by `deep_research` etc., but the user does not configure them in V1.)
- **Engineering workspace (sandbox, terminal, isolated execution)** → V2.
- **Re-run a Run with different model / instructions** → V1.1.
- **User-generated Agent Library (clone/edit/share)** → V1.1.
- **Run comparison (side-by-side)** → V1.1.
- **Visual workflow builder, evaluation framework, guided learning** → V2 / deferred.

The architecture doc §5 should be edited to match this list. The "deferred" capabilities are not removed from the long-term baseline — they remain in §3 of the architecture doc as part of the canonical capability map.

---

## User-facing flows (V1)

These are the flows the UI must support. Each is described as a step sequence. UI surface names are illustrative; the actual implementation will refine them.

### Flow 1 — Configure and run a starter agent

```text
1. User lands on /playground
2. Picks a starter agent from the library (e.g. "Tutor on documents")
3. Sees the agent config form pre-populated:
   - Model (dropdown from provider catalog)
   - Instructions (text area, editable)
   - Knowledge base (attach from KB list, optional)
   - Tools (toggles for web_search, rag, etc.)
4. Edits any field if they want
5. Types a task in the message box
6. Hits Run
7. Lands on the Run page (Flow 2)
```

**Backend resolution:** when the user picks a starter, the frontend sends `{ agent_template_id: "tutor-on-documents" }`. The backend resolves the template → `{ model, instructions, kb_id, tools }` and synthesizes an agent spec for the existing `ChatOrchestrator` to invoke. The `chat` capability in the fork handles the actual run.

**Starter template format (illustrative):**
```json
{
  "id": "tutor-on-documents",
  "name": "Tutor on documents",
  "description": "Walks a learner through attached documents with Socratic questioning.",
  "default_model": "claude-sonnet-4-5",
  "default_instructions": "You are a Socratic tutor...",
  "default_kb_required": true,
  "default_tools": ["rag", "read_source"],
  "icon": "...",
  "tags": ["learning", "documents"]
}
```

### Flow 2 — Observe a run (the differentiator)

```text
1. Run page (URL: /runs/<run_id>) shows:
   - Run header: id, agent template, model, start time, status
   - Live event stream (left): turns streaming as they happen
   - Run summary (right): tokens, latency, cost so far
   - Events panel (bottom): normalized event log
2. As the run executes, events stream in:
   - RUN_STARTED
   - MODEL_REQUEST / MODEL_RESPONSE (paired, with model + tokens)
   - TOOL_REQUEST / TOOL_RESULT (paired, with args/result preview)
   - CONTEXT_RETRIEVED (when KB hits)
   - KNOWLEDGE_SOURCE_USED (with citation preview)
   - FILE_READ (when a tool reads a workspace file — V1: scoped to KB content)
   - RUN_COMPLETED | RUN_FAILED
3. User can pause / abort the run from a top-right control
4. When complete, the run is saved; user can re-open the same run from /runs
   in replay mode (events load from store, no live streaming)
```

**This is the one flow that defines the product.** If we get this right, the rest of the architecture (eval, comparison, learning) is downstream. If we get it wrong, we're a chatbot with a sidebar.

### Flow 3 — Attach a knowledge base

```text
1. From /playground or the agent config, user picks "Attach knowledge"
2. /knowledge shows existing KBs + a "+ New knowledge base" button
3. Upload (PDF, MD, DOCX) → progress bar → KB appears in list when ready
4. Returns to /playground; KB is now attached
```

This is a thin wrapper over the fork's existing `KnowledgeBaseManager` + `add_documents` flow. Mostly UX work; the runtime already does it.

### Flow 4 — Browse past runs

```text
1. /runs lists all runs (paginated; filter by agent template, date, status)
2. Each row shows: agent template, model, status, token count, duration
3. Click a run → opens the same Run page as Flow 2, in replay mode
```

This is where the "first-class run" claim gets validated. No re-run, no compare, no branch — those are V1.1.

### Flow 5 — Inspect event detail (debug / audit)

```text
1. From the run page, click any event → modal/panel with full payload
2. For MODEL_REQUEST: show the exact prompt
   (system + user + history + retrieved context, each section collapsible)
3. For TOOL_REQUEST: show the full args, schema, and (if available)
   a "what would have happened" preview
4. For TOOL_RESULT: show the truncated result (with expand for full)
5. Modal is keyboard-navigable and accessible
```

This is the "I want to know what the agent actually did" flow. The Run Inspector is the surface; the event-detail panel is the value.

---

## Runtime flows (V1)

These are the things that happen *in code* when the user does the above. The boundary between AriseHub code (under `arisehub/`) and the fork's runtime (under `deeptutor/`) is a key design choice: **the AriseHub layer is an adapter / consumer of the fork's runtime, not a replacement for it.**

### Runtime flow R1 — Run lifecycle (the contract)

```text
RUN_STARTED                    ← AriseHub adapter receives first StreamBus event
  capability_selected          ← which Level-2 capability (chat, deep_solve, …)
  MODEL_REQUEST
  MODEL_RESPONSE
  [TOOL_REQUEST → TOOL_RESULT]×N    ← tool loop, may be 0..N
  [MODEL_REQUEST ↔ MODEL_RESPONSE]   ← continuation turns
  RUN_COMPLETED | RUN_FAILED
```

The orchestrator (`deeptutor/runtime/orchestrator.py:ChatOrchestrator`) already drives this lifecycle via the `StreamBus`. The AriseHub layer wraps it: an `AriseHubRunEventAdapter` consumes `StreamEvent` types from the bus and emits `AriseHubRunEvent` types into the `run_store`.

**Design decision (locked):** the adapter is a *consumer* of the `StreamBus`, not a replacement. We do not change the fork's event types. We translate at the boundary. This keeps the fork's runtime intact and makes the AriseHub event contract independent of the fork's internal representation.

**Future-proofing:** if the fork's `StreamEvent` schema changes in a future upstream release, only the adapter needs updating. The AriseHub event schema and downstream consumers (Run page, Inspector) are insulated.

### Runtime flow R2 — Event adapter

```python
# Pseudocode. Real implementation lives in arisehub/runtime/run_adapter.py.
class AriseHubRunEventAdapter:
    def __init__(self, run_id: str, run_store: "RunStore") -> None:
        self.run_id = run_id
        self.run_store = run_store
        self._pending_tool_requests: dict[str, AriseHubRunEvent] = {}

    async def on_stream_event(self, event: StreamEvent) -> None:
        arise_event = self._translate(event)
        if arise_event is not None:
            await self.run_store.append(self.run_id, arise_event)

    def _translate(self, e: StreamEvent) -> AriseHubRunEvent | None:
        # Map StreamEvent kinds → AriseHubRunEvent kinds
        # MODEL_DELTA → MODEL_RESPONSE (accumulate deltas until final)
        # TOOL_CALL_STARTED → TOOL_REQUEST
        # TOOL_CALL_FINISHED → TOOL_RESULT
        # ... etc
        ...
```

**Key behavior:**

- `MODEL_REQUEST` and `MODEL_RESPONSE` are *paired*. The adapter buffers `MODEL_DELTA` events until a final response arrives, then emits a single `MODEL_RESPONSE` event.
- `TOOL_REQUEST` and `TOOL_RESULT` are paired by `tool_call_id`. The adapter waits for the result before emitting the result event.
- `RUN_STARTED` is emitted on adapter construction.
- `RUN_COMPLETED` / `RUN_FAILED` are emitted on adapter close.
- All events are appended to `run_store` in order. Append-only.

### Runtime flow R3 — Run store

```text
data/
  user/
    runs/
      <run_id>/
        meta.json          # agent template, model, start, end, status, totals
        events.jsonl       # one AriseHubRunEvent per line
```

**Schema (illustrative):**

`meta.json`:
```json
{
  "run_id": "uuid",
  "agent_template_id": "tutor-on-documents",
  "model": "claude-sonnet-4-5",
  "started_at": "iso8601",
  "ended_at": "iso8601 or null",
  "status": "running | completed | failed | aborted",
  "totals": {
    "tokens_in": 0,
    "tokens_out": 0,
    "cost_usd": 0.0,
    "duration_ms": 0
  }
}
```

`events.jsonl` (one event per line, JSON):
```json
{"ts": "iso8601", "kind": "RUN_STARTED", "payload": {...}}
{"ts": "iso8601", "kind": "MODEL_REQUEST", "payload": {...}}
{"ts": "iso8601", "kind": "MODEL_RESPONSE", "payload": {...}}
{"ts": "iso8601", "kind": "TOOL_REQUEST", "payload": {...}}
{"ts": "iso8601", "kind": "TOOL_RESULT", "payload": {...}}
...
{"ts": "iso8601", "kind": "RUN_COMPLETED", "payload": {...}}
```

**Why JSONL for V1:** trivial to append, trivial to replay, no schema migration, no database dependency. Limits: no concurrent writers (single-writer per run, which is fine), no rich queries (V1.1 may want SQLite or DuckDB for filter-by-agent / filter-by-model).

**Replay:** the Run Inspector reads `meta.json` and `events.jsonl` in order. No database required for V1.

### Runtime flow R4 — Starter agent resolution

```text
1. User picks "Tutor on documents" in UI
2. Frontend sends { agent_template_id: "tutor-on-documents" } to backend
3. Backend's AgentTemplateResolver looks up the template (bundled JSON)
4. Resolver returns { model, instructions, kb_id, tools, capability: "chat" }
5. Backend creates a synthetic agent spec on the fly
6. Backend invokes ChatOrchestrator with the agent spec
7. ChatOrchestrator runs the existing chat capability with that spec
```

**The interesting design question here:** does the fork's concept of "agent" map to this? In the fork, a "capability" (`deeptutor/core/capability_protocol.py:BaseCapability`) is a multi-stage turn pipeline. A capability has its own configuration (model, instructions, tools), and the runtime invokes capabilities, not "agents" in the user-facing sense.

**AriseHub's "agent" is configuration, not code.** An AriseHub agent is a bundle of:
- A capability (from the fork's set: `chat`, `deep_solve`, `deep_question`, `deep_research`, `visualize`, `math_animator`, `mastery_path`).
- A model.
- Instructions.
- Tools.
- (Optionally) a knowledge base.
- (Optionally) memory settings.

The fork's capabilities are the runtime code; AriseHub's agents are user-facing configurations that select and parameterize capabilities. This is the correct mental model and the one the architecture doc §3.2 row "Agent Definition" implies.

**V1 implementation:** a starter agent resolves to `(capability=chat, model=X, instructions=Y, kb=K, tools=[...])` at runtime. The `chat` capability in the fork handles the actual turn loop. AriseHub code does not need to write a new orchestrator — it writes the resolution layer and the run-event adapter.

**V1.1 implementation:** user-cloned agents are stored as JSON in `data/user/agents/<agent_id>.json` and follow the same resolution path.

---

## Where the architecture doc and this doc disagree

These are intentional disagreements, captured here so they can be debated and reconciled.

1. **§5 lists 8 V1 capabilities; this doc ships 4.** The doc over-scopes V1. The four deferred (workspace, subagent, re-run, comparison) are real and important, but not V1-blocking.

2. **§5 puts "Subagents" in V1; this doc defers to V1.1.** The fork's `subagent` capability works, but the user-facing UX for "delegate to a subagent" is a real design effort and shouldn't be in the first release. Subagents are still used *internally* by `deep_research` and similar capabilities.

3. **§6 proposes a normalized event vocabulary but does not specify ownership.** This doc specifies: the `run_store` owns the events, the adapter produces them, the orchestrator is unaware. This is the auditable "Run-as-first-class" claim.

4. **§5 mentions the "Run Inspector" without defining it.** Flows 2 and 5 above are the Run Inspector's definition. If we agree on these, the doc can be tightened.

5. **§5 puts "Agent Library" in V1 as a clone/edit/reuse experience; this doc ships it as a starter-only library.** The user-generated library is a V1.1 capability because the UX (save, share, version, discover) is non-trivial.

---

## Open decisions for V1

These need to be settled before implementation begins.

1. **Run event schema — should we adopt the §6 vocabulary verbatim, or extend it?**
   - The current §6 list has 13 event kinds. V1 needs at most 8 of them. The other 5 (HUMAN_APPROVAL_*, EVALUATION_*, AGENT_DELEGATION, SUBAGENT_RESULT) are V1.1+ concerns.
   - **Recommendation:** ship a V1 subset of 8 event kinds, named exactly per §6. Document the rest as "reserved, not emitted in V1." When V1.1 lands, no event-kind renames are needed — just new emissions.

2. **Starter agent library — how many, and which?**
   - **Recommendation:** ship 5–8 starter agents covering: (a) tutor on documents, (b) research analyst, (c) code reader, (d) summarizer, (e) Q&A on a single doc, (f) brainstorming partner, (g) data interpreter, (h) general chat. Final list to be debated when V1 design begins.

3. **Run store — JSONL or SQLite from day one?**
   - **Recommendation:** JSONL for V1 (simpler, no migrations). Plan a SQLite migration in V1.1 if query-by-field becomes a UX requirement (e.g. "show me all runs with this model" filter).

4. **Where do AriseHub components mount in the fork's frontend?**
   - **Recommendation:** Next.js route group `web/app/(arisehub)/...` per ADR-001 §4. The fork's existing routes (e.g. `/chat`, `/knowledge`) keep working unchanged; AriseHub adds `/playground`, `/runs`, `/runs/<id>`, `/library` under the route group.

5. **Model catalog — re-use the fork's provider layer, or introduce an AriseHub-level abstraction?**
   - **Recommendation:** re-use the fork's provider layer for V1. The AriseHub-level abstraction (e.g. "AriseHub model catalog" with AriseHub-specific config UX) is a V1.1+ concern.

6. **What license applies to AriseHub-original code under `arisehub/`?**
   - This is the same open question from ADR-001. Decision needed before any public release.

---

## Migration path to V1.1

The V1 architecture is designed so V1.1 additions are additive, not rewrite-from-scratch:

- **Workspace / file upload:** add a new adapter (`arisehub/runtime/workspace_adapter.py`) and a new event kind (`FILE_READ`, `FILE_WRITE`) to the schema. The Run page gains a workspace panel. No change to existing event consumers.

- **Subagent delegation in chat:** add a new event kind (`AGENT_DELEGATION`, `SUBAGENT_RESULT`). Expose a "delegate to subagent" toggle in the Playground UI. The fork's existing `subagent` capability is used as the runtime; AriseHub adds the UX.

- **Re-run a Run:** add a new endpoint (`POST /runs/<id>/rerun`) that creates a new run with the same initial config. The Run page gains a "re-run" button. No schema change.

- **User-generated Agent Library:** add a new route (`/library`) and persistence (`data/user/agents/*.json`). The AgentTemplateResolver gains a fallback to user-defined agents.

- **Run comparison:** add a new page (`/compare?runs=<id1>,<id2>`) that loads two run event streams side-by-side. Reuses the existing event schema and Run page components.

This additive property is why the V1 architecture deliberately keeps the event schema and run store simple. The complexity of V1.1 features lives in the UI and the resolution layer, not in the event model.

---

## End of V1 Flows and Scope

Once accepted, this document becomes the source of truth for V1 scope. The architecture doc's §5 should be edited to either match this scope or to point at this document and explain the relationship.

> **For the full feature-to-version ledger (all 43 sub-capabilities across V1, V1.1, V2, Deferred, Cut), see [Feature Version Matrix](./feature-version-matrix.md).** This doc covers V1 in detail and the V1 → V1.1 → V2 transition. The matrix covers every feature in the long-term baseline.
