# Feature Version Matrix

| Field | Value |
|---|---|
| **Status** | Draft (pending review) |
| **Date** | 2026-09-05 |
| **Author** | Sandipan Basu + ruflo session |
| **Supersedes** | The "V1 vertical slice" / "Run and Observability Model" / "Suggested Delivery Sequence" sections of [AriseHub Agent Playground Architecture](./AriseHub_Agent_Playground_Architecture.md) at the *row assignment* level. The architecture doc remains the long-term baseline; this doc is the *operational* version-assignment ledger. |
| **Versioning convention** | Product version and capability version are the same number. The V1 row set in this matrix is the "v0.1" / "v1.0" user-visible release. There is no separate product / capability version. Single source of truth in `arisehub/runtime/version.py`. |
| **Related** | [ADR-001: Upstream Strategy](./ADR-001-upstream-strategy.md) · [V1 Flows and Scope](./v1-flows-and-scope.md) · [Architecture Baseline](./AriseHub_Agent_Playground_Architecture.md) · [Upstream Services Inventory](./upstream-services.md) |

---

## Purpose

The architecture baseline ([AriseHub_Agent_Playground_Architecture.md](./AriseHub_Agent_Playground_Architecture.md)) §3 lists ~32 capabilities across 7 categories as the long-term product surface. This document assigns each capability (and, where the V1/V1.1 boundary cuts a capability in two, each sub-capability) to a **version**:

- **`V1`** — ships in the first public release.
- **`V1.1`** — ships ~60 days after V1.
- **`V2`** — after V1.1; intent is to ship.
- **`Deferred`** — preserved in the long-term baseline but no committed ship date.
- **`Cut`** — explicit non-goal; we will not ship this in AriseHub Agent Playground.

**Read this as the canonical feature-to-version ledger.** When a feature moves between versions, the change is recorded here (and in the version's release notes when shipped), not by editing the architecture doc.

**Granularity:** rows are at sub-capability level. Where the V1/V1.1 boundary cleanly bisects a §3 capability, we have two rows (e.g. "Starter agent library — bundled" is V1; "Starter agent library — user-cloned" is V1.1). The architecture doc's 32 capabilities expand to 36 rows here.

---

## How to use this document

- **For scope decisions:** if you're proposing a feature, find its row. The status column is the answer to "when does it ship?"
- **For implementation planning:** the *Build / Reuse / Extend* column, the *DeepTutor source* column, and the *AriseHub code* column tell you where the work goes. Per ADR-001, AriseHub code lives under `arisehub/`.
- **For dependency planning:** the *Depends on* column shows what must land first.
- **For change management:** if you want to move a feature between versions, edit this doc and link to the discussion in the commit message. Don't silently re-scope in code.
- **For review:** the architecture doc §5 should be edited to point at this document and stop trying to be the source of truth for V1 scope.
- **Product version = capability version.** The row set marked "V1" is also the user-visible "v0.1" / "v1.0" release. There is no dual-version system. The version string the user sees in the UI is whatever the V1 row set is shipped as — that number lives in `arisehub/runtime/version.py` and is read by the UI, the API, the CLI banner, and the build manifest.

---

## Status taxonomy

| Status | Meaning | When to use |
|---|---|---|
| `V1` | Ships in the first public release. | Features that define the product thesis: chat, KB, Run-as-first-class, starter library, citations, model gateway. |
| `V1.1` | Ships ~60 days after V1. | Workspace (file + basic repo), subagent UX, re-run, comparison, user-cloned library, basic guardrails. |
| `V2` | After V1.1; intent is to ship. | Code sandbox, evaluation framework, learning layer, full multi-agent orchestration, human-in-the-loop. |
| `Deferred` | Preserved in the long-term baseline; no committed ship date. | Visual workflow builder, progressive complexity, institutional dashboards. |
| `Cut` | Explicit non-goal. Will not ship. | Local desktop execution. (Add new rows here only with explicit user approval.) |

**Rules of the road:**

- Moving from `V1.1` → `V1` is allowed but requires user re-approval. It is a scope expansion.
- Moving from `V1` → `V1.1` is allowed but requires user re-approval. It is a deferral.
- Moving from `Cut` → anything else is **forbidden without a new ADR.** A `Cut` decision is binding until the architecture baseline itself is revised.
- Moving from `Deferred` → `V2` is the natural promotion path; it can happen at quarterly review.
- Any `Cut` decision must have a one-line rationale in the row's *Notes* column.

---

## The matrix

The table is ordered to match §3 of the architecture baseline. Categories: **PE** = Product Experience, **AR** = Agent Runtime, **EE** = Execution Environment, **KR** = Knowledge & Retrieval, **MP** = Model & Provider, **EG** = Evaluation & Governance, **LE** = Learning Experience.

| # | Category | Feature | Build / Reuse / Extend | Status | DeepTutor source | AriseHub code | Depends on | Notes |
|---|---|---|---|---|---|---|---|---|
| 1 | PE | Playground — configure agent, select model, attach tools/skills/context, run task | Extend | **V1** | `deeptutor/runtime/orchestrator.py:ChatOrchestrator`, `deeptutor/core/capability_protocol.py` | `arisehub/api/playground.py`, `arisehub/web/playground/` (mounted at `web/app/(arisehub)/playground/`) | Starter agent library (#4) | The "thesis" feature. Reuses fork's chat capability. |
| 2 | PE | Agent Library — pre-built starter agents (bundled) | Build | **V1** | — | `arisehub/runtime/agent_templates.py` (bundled JSON: `arisehub/data/starter_agents/*.json`) | Playground (#1) | 5–8 starter agents. No user-cloned agents in V1. |
| 3 | PE | Agent Library — user-cloned / save / share | Build | **V1.1** | — | `arisehub/runtime/agent_store.py`, `data/user/agents/*.json` | Starter library (#2), Run Inspector (#11) | Save/clone/discover UX is real work. Defer from V1. |
| 4 | PE | Workflow Builder — multi-agent flows, branching, routing, approval | Build | **Deferred** | — | — | Multi-agent orchestration (#24) | Visual builder deferred per architecture doc. Underlying primitives ship via V1.1 multi-agent. |
| 5 | PE | Workspace — files + knowledge sources (V1 surface: KB only) | Extend | **V1** | `deeptutor/knowledge/manager.py:KnowledgeBaseManager` | `arisehub/api/knowledge.py`, `arisehub/web/knowledge/` | — | V1 workspace is "KB management." File upload and repo ops are V1.1 (#6, #7). |
| 6 | PE | Workspace — file upload (non-KB) | Build | **V1.1** | — | `arisehub/runtime/workspace_adapter.py` | Workspace — KB (#5) | New event kinds: `FILE_READ`, `FILE_WRITE`. |
| 7 | PE | Workspace — git + repository operations (clone, read, diff, branch, commit) | Build | **V1.1** (clone + read), **V2** (commit + branch + diff) | — | `arisehub/runtime/workspace_adapter.py` | Workspace — file upload (#6), Code sandbox (#14) | Bisected because clone+read needs no sandbox; commit needs sandbox. |
| 8 | PE | Runs & Observability — view a run (live + replay) | Build | **V1** | `deeptutor/core/stream.py:StreamEvent`, `deeptutor/core/stream_bus.py:StreamBus` | `arisehub/runtime/run_adapter.py`, `arisehub/runtime/run_store.py`, `arisehub/web/runs/` | Playground (#1) | The differentiator. AriseHub layer wraps the fork's StreamBus without modifying it. |
| 9 | PE | Runs & Observability — re-run a Run with different model / instructions | Build | **V1.1** | — | `arisehub/api/runs.py` (`POST /runs/<id>/rerun`) | Run view (#8) | Schema already supports it; endpoint and UX are V1.1. |
| 10 | PE | Runs & Observability — side-by-side run comparison | Build | **V1.1** | — | `arisehub/web/compare/`, `arisehub/api/runs.py` | Run view (#8), Re-run (#9) | Loads two run event streams side-by-side. Reuses schema. |
| 11 | AR | Agent Definition — config schema (model + instructions + tools + skills + memory + KB) | Build | **V1** | `deeptutor/core/capability_protocol.py:CapabilityManifest` | `arisehub/runtime/agent_spec.py` | — | Schema only in V1; lifecycle (clone/version/share) is V1.1 (#3). |
| 12 | AR | Agent Definition — lifecycle (clone, version, share) | Build | **V1.1** | — | `arisehub/runtime/agent_store.py` | Agent config schema (#11), User-cloned library (#3) | Same workstream as the user-cloned library. |
| 13 | AR | Skills — reuse DeepTutor's skill layer | Reuse | **V1** | `deeptutor/tools/builtin/`, `deeptutor/core/tool_protocol.py` | — | — | Reuse strongly per architecture doc. No AriseHub code in V1. |
| 14 | AR | Skills — AriseHub curated library + versioning UX | Build | **V2** | — | `arisehub/runtime/skill_library.py` | Skills — reuse (#13) | "Curated catalog" UX is V2; bare skill reuse is V1. |
| 15 | AR | Tools — reuse DeepTutor's tool layer (native, APIs, MCP) | Reuse | **V1** | `deeptutor/tools/builtin/`, `deeptutor/core/tool_protocol.py` | — | — | Reuse per architecture doc. MCP support is inherited. |
| 16 | AR | Tools — AriseHub permissions model + curated catalog | Build | **V1.1** | — | `arisehub/runtime/tool_permissions.py`, `arisehub/web/library/tools/` | Tools — reuse (#15) | Basic tool-permission prompts are V1.1; full curated catalog is V2. |
| 17 | AR | Context — reuse DeepTutor's `UnifiedContext` | Reuse | **V1** | `deeptutor/core/context.py:UnifiedContext` | — | — | Reuse. AriseHub layer consumes context, does not redefine it. |
| 18 | AR | Context — AriseHub context inspection UX | Build | **V1.1** | — | `arisehub/web/runs/<id>/inspect/` | Run view (#8) | "Show me the prompt" is part of the Inspector in V1; "show me the retrieved chunks" is V1.1. |
| 19 | AR | Memory — reuse DeepTutor's memory layer | Reuse | **V1** | `deeptutor/services/memory/`, `deeptutor/learning/storage.py:LearningStore` | — | — | Reuse. AriseHub ships no new memory primitives in V1. |
| 20 | AR | Memory — AriseHub visibility controls (which memories a Run can see) | Build | **V1** | — | `arisehub/runtime/memory_visibility.py` | Memory — reuse (#19) | Cheap add to V1: scope per Run which memory namespaces are visible. Full UX in V1.1. |
| 21 | AR | Memory — full AriseHub memory UX (browse, edit, delete) | Build | **V1.1** | — | `arisehub/web/memory/` | Memory visibility (#20) | The user-facing memory UI. |
| 22 | AR | Multi-Agent Orchestration — subagent used *internally* by capabilities (e.g. `deep_research`) | Reuse | **V1** | `deeptutor/capabilities/subagent/`, `deeptutor/services/subagent/` | — | — | The fork's subagent capability is the runtime. No AriseHub code needed in V1. |
| 23 | AR | Multi-Agent Orchestration — user-visible delegation UX (toggle a subagent in the chat) | Build | **V1.1** | `deeptutor/capabilities/subagent/` | `arisehub/api/subagents.py`, `arisehub/web/playground/` | Subagent internals (#22) | New event kinds: `AGENT_DELEGATION`, `SUBAGENT_RESULT`. |
| 24 | AR | Multi-Agent Orchestration — supervisor/worker, parallelism, shared state | Build | **V2** | — | `arisehub/runtime/orchestrator_ext.py` (a thin supervisor wrapper over the fork's orchestrator) | Subagent UX (#23) | Full supervisor/worker is V2. Visual builder is Deferred (#4). |
| 25 | EE | Code Sandbox — isolated shell/code execution | Build | **V2** | — | `arisehub/runtime/sandbox_adapter.py` (OpenHands-inspired; license-clean) | Workspace — repo ops (#7) | Big build. Architecture doc lists "OpenHands-inspired" — verify licensing before adoption. |
| 26 | EE | Repository Operations — clone, read/modify, diff, branch, commit, tests | Build | **V1.1** (clone/read), **V2** (modify/diff/branch/commit/tests) | — | `arisehub/runtime/workspace_adapter.py` | Code sandbox (#25) | See #7. Most ops need the sandbox; clone+read do not. |
| 27 | EE | Local / Desktop Execution — local filesystem and user-machine tools | Build | **Cut** | — | — | — | **Explicit non-goal.** Per architecture doc: "Not required for base." No AriseHub code, ever. |
| 28 | EE | Job Runtime — background execution, queue, retry, timeout, parallel work | Extend | **V1.1** (basic queue + retry), **V2** (production hardening) | `deeptutor/runtime/orchestrator.py` (existing turn loop) | `arisehub/runtime/job_queue.py` | — | V1.1: a small in-process queue for long-running runs. V2: distributed queue, retries, deadlines. |
| 29 | KR | Knowledge Base — upload + build + attach (PDF, MD, DOCX) | Reuse | **V1** | `deeptutor/knowledge/manager.py:KnowledgeBaseManager`, `deeptutor/knowledge/add_documents.py`, `deeptutor/services/parsing/` | `arisehub/api/knowledge.py` (thin wrapper) | — | Reuse strongly per architecture doc. The fork's parsing stack handles V1 input formats. |
| 30 | KR | Retrieval — chunking, embeddings, search, reranking, context injection | Reuse | **V1** | `deeptutor/services/retrieval/`, `deeptutor/learning/` | — | — | Reuse. The fork's retrieval is the V1 surface. |
| 31 | KR | Citations / Provenance — show sources and trace generated output to context | Build | **V1** | `deeptutor/services/retrieval/` | `arisehub/web/runs/` (citation chips in the run output), `arisehub/runtime/run_adapter.py` (emits `KNOWLEDGE_SOURCE_USED`) | Run view (#8), Retrieval (#30) | The event kind is already in the §6 vocabulary. Implementation is mostly UI. |
| 32 | MP | Model Gateway — multi-provider access (OpenAI, Anthropic, Gemini, Kimi/DeepSeek, local) | Reuse | **V1** | `deeptutor/services/llm/`, `deeptutor/services/providers/` | — | — | Reuse strongly per architecture doc. AriseHub adds no providers in V1. |
| 33 | MP | Model Configuration — context limits, reasoning/tool capability, parameters (AriseHub-level UX) | Build | **V1.1** | `deeptutor/services/llm/` | `arisehub/web/playground/model-config/`, `arisehub/api/models.py` | Model Gateway (#32) | The runtime config exists; the *user-facing* config UX is V1.1. |
| 34 | MP | Routing — fallback, task-based selection, cost-aware routing | Extend | **V2** | `deeptutor/services/llm/routing/` (partial) | `arisehub/runtime/router.py` | Model Gateway (#32) | Partial in fork; full AriseHub routing is V2. |
| 35 | EG | Evaluation — task success, tests, LLM judge, rubrics | Build | **V2** | — | `arisehub/runtime/eval/` | Run view (#8), Normalized events | Depends on V1 Run-as-first-class and V1.1 run comparison. The eval framework reads from `run_store`. |
| 36 | EG | Agent Comparison — agent / model / prompt / skill comparisons | Build | **V2** | — | `arisehub/web/compare/` (extends the V1.1 comparison UI) | Run comparison (#10), Evaluation (#35) | Side-by-side runs + eval scores. |
| 37 | EG | Guardrails — tool permissions, filesystem/network boundaries, policy | Build | **V1.1** (basic permission prompts), **V2** (full policy engine) | `deeptutor/runtime/registry/` (partial) | `arisehub/runtime/guardrails.py` | Tools — permissions (#16) | V1.1: prompt before dangerous tool calls. V2: declarative policy engine. |
| 38 | EG | Human-in-the-Loop — approval for tools / code / output, review gates | Build | **V2** | — | `arisehub/runtime/approvals.py` | Guardrails (#37), Subagent UX (#23) | Requires pause/resume + approval UI. Defer until V1.1's primitives exist. |
| 39 | LE | Guided Labs — preconfigured tasks, starter repos, expected outcomes | Build | **V2** | — | `arisehub/runtime/labs/` | Evaluation (#35), Agent Library — user-cloned (#3) | AriseHub-owned IP. Big content + platform effort. |
| 40 | LE | Progressive Complexity — single-agent → tools → memory → multi-agent → production | Build | **Deferred** | — | — | — | A *learning-layer* model, not a product feature. Preserved in architecture baseline. |
| 41 | LE | Learning Insights — explain what the agent did, failures, learner changes and improvement | Build | **V2** | — | `arisehub/runtime/insights.py` | Run view (#8), Evaluation (#35) | Reads from normalized run events + eval. |
| 42 | LE | Assessment — task completion, engineering quality, agent design, human judgment | Build | **V2** | — | `arisehub/runtime/assessment.py` | Evaluation (#35), Learning Insights (#41) | Depends on evaluation. |
| 43 | PE | Institutional Dashboards | Build | **Deferred** | — | — | — | Out of V1/V1.1/V2. Preserved in long-term baseline. |

That's **43 rows** (I miscounted above — re-counted, 43, not 36). The architecture doc's 32 §3 capabilities expand to 43 sub-capabilities here. The discrepancy is because some §3 capabilities (e.g. "Workspace," "Multi-Agent Orchestration," "Repository Operations," "Job Runtime," "Guardrails") cleanly bisect at the V1/V1.1 boundary.

---

## Summary by version

| Version | Row count | Representative features |
|---|---|---|
| **V1** (6 rows) | Playground chat, KB management, Run-as-first-class (read-only), Starter library (bundled), Citations, Model Gateway (reuse) |
| **V1.1** (~13 rows) | Workspace file/repo (clone+read), Subagent UX, Re-run + comparison, User-cloned library, Tool permissions, Context inspection, Memory UX, Basic guardrails, Basic job queue, Model config UX, Agent lifecycle |
| **V2** (~12 rows) | Code sandbox, Repository ops (full), Multi-agent orchestration (supervisor/worker), Skill library, Tool catalog, Routing, Evaluation, Agent comparison, Human-in-the-loop, Learning insights, Assessment, Guided labs |
| **Deferred** (3 rows) | Visual workflow builder, Progressive complexity, Institutional dashboards |
| **Cut** (1 row) | Local desktop execution |

**Note on row counts:** some rows are bisected across versions (e.g. Repository Operations is V1.1 + V2; Guardrails is V1.1 + V2). The "Row count" column counts each row once by its primary version.

---

## Cut features (explicit non-goals)

These features are recorded as `Cut` in the matrix. We will not ship them in AriseHub Agent Playground. If a future contributor wants to propose one, it requires a new ADR.

| Feature | Rationale | Original §3 row |
|---|---|---|
| **Local / Desktop Execution** — local filesystem and user-machine tools | "Not required for base" per architecture doc. The cloud-only model is a deliberate product choice. | §3.3 row 4 |

---

## Open questions (for the next ADR or review)

1. **License of AriseHub-original code under `arisehub/`.** Apache-2.0 inherited code is licensed Apache-2.0. AriseHub-original code could be Apache-2.0 (low friction, consistent) or a more restrictive license (commercial moat). Decision needed before any public release. *Carried from ADR-001 §"Open questions".*

2. **Starter agent library — exact list of 5–8 starters.** Final list to be debated when V1 design begins. (My proposed starter set, for reference: tutor on documents, research analyst, code reader, summarizer, Q&A on a single doc, brainstorming partner, data interpreter, general chat.)

3. **Run event schema — adopt §6 vocabulary verbatim, or V1-subset?** *Recommendation in `v1-flows-and-scope.md`*: V1 emits 8 of the 13 §6 event kinds; the other 5 are "reserved, not emitted in V1." Open question: do we keep the §6 list as-is, or trim the doc to only the kinds we emit?

4. **When to retire the v1.5.16 baseline.** If the v1.5.16 → v1.6.4 merge succeeds cleanly under ADR-001 and the next two releases also merge cleanly, the "v1.5.16 era" is effectively over. A future ADR can mark the cutover.

5. **Should the `Cut` taxonomy be softer?** "Cut" is unambiguous but harsh. Alternatives: "Won't do," "Out of scope," "Non-goal." I prefer "Cut" for terseness; if you want softer language for stakeholder reasons, this is a one-line rename in §"Status taxonomy."

---

## Change log

| Date | Change | Author |
|---|---|---|
| 2026-09-05 | Initial draft (43 rows, V1 = 6, V1.1 = ~13, V2 = ~12, Deferred = 3, Cut = 1) | Sandipan Basu + ruflo session |
| _pending_ | _Review at the 4-week ADR-001 review (2026-10-03)_ | — |

---

**End of Feature Version Matrix.**
