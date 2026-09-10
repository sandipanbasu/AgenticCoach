# AriseHub Agent Playground Architecture

**Capability Architecture & DeepTutor-Based Implementation Strategy**  
**Architecture Baseline — September 2026**

> **Product principle:** An engineering workbench for building, running, observing and progressively composing AI agents — not merely another chatbot playground.

---

## 0. Document Status

| Field | Value |
|---|---|
| **Document type** | Capability architecture baseline (long-term, aspirational) |
| **Authoritative scope** | AriseHub product surface; DeepTutor runtime is treated as inherited infrastructure |
| **Fork base** | DeepTutor v1.5.16 (inherited, Apache-2.0) — see [ADR-001](./ADR-001-upstream-strategy.md) |
| **Current sync state** | Local `main` at v1.5.16 + 5 AriseHub branding/deploy commits; `origin/main` at upstream v1.6.4 — v1.6.4 merge is the first sync event under ADR-001 |
| **Related documents** | [ADR-001: Upstream Strategy](./ADR-001-upstream-strategy.md) · [V1 Flows and Scope](./v1-flows-and-scope.md) · [Feature Version Matrix](./feature-version-matrix.md) · [Upstream Services Inventory](./upstream-services.md) |
| **Last updated** | 2026-09-05 |
| **Review cadence** | Quarterly, or when DeepTutor ships a major release (v1.7.0+) |

> **Read this doc as a target, not a snapshot.** Capabilities marked "deferred" are intentionally preserved. The Reuse/Extend/Build matrix in §4 must be re-verified against each DeepTutor release merge (see ADR-001).

---

## 1. Purpose and Architectural Position

AriseHub Agent Playground is a standalone AriseHub offering for hands-on agent engineering.

The architecture deliberately separates the **AriseHub product experience** from the underlying agent runtime and infrastructure.

**DeepTutor** is selected as the base because it already provides many of the required primitives under a permissive Apache-2.0 model, while AriseHub retains ownership of the product experience, capability model, extensions, learning experience, and future workflow/evaluation layers.

### Architecture Principles

- Preserve the full capability map even when a capability is deferred from V1.
- Use DeepTutor as a base/runtime, not as the definition of the AriseHub product.
- Prefer extension points and adapters over tightly coupling AriseHub to a single implementation.
- Keep AriseHub-owned schemas for agents, runs, workspaces, evaluations, and future workflows where practical.
- Treat execution, knowledge, model providers, and other infrastructure as replaceable subsystems.
- Keep licensing and third-party attribution as architecture concerns from the beginning.

---

## 2. Logical Architecture

```text
┌──────────────────────────────────────────────────────────────┐
│                  ARISEHUB AGENT PLAYGROUND                   │
│                                                              │
│  Playground • Agent Library • Workspace • Runs/Observability │
│  Workflow Builder (future visual UX)                         │
└──────────────────────────────┬───────────────────────────────┘
                               │
                    AriseHub Product / API Layer
                               │
┌──────────────────────────────▼───────────────────────────────┐
│                       AGENT RUNTIME                          │
│                                                              │
│ Agent Definition • Skills • Tools • Context • Memory • MCP   │
│ Multi-Agent Orchestration                                    │
│                                                              │
│                 DeepTutor base + AriseHub extensions         │
└──────────────┬────────────────┬────────────────┬─────────────┘
               │                │                │
       EXECUTION ENV.     KNOWLEDGE & RAG   MODEL/PROVIDER
       Sandbox/Repo       KB/Retrieval      Gateway/Config
       AriseHub +         DeepTutor base     DeepTutor base
       OpenHands-inspired                   + extensions
               │                │                │
               └────────────────┼────────────────┘
                                │
                    EVALUATION & GOVERNANCE
                       AriseHub extensions
                                │
                     LEARNING EXPERIENCE
                       AriseHub-owned IP
```

---

# 3. Canonical Capability Map

This capability map is the **long-term architecture baseline**.

V1 scope is a selection from this map. Capabilities are not removed merely because they are deferred.

## 3.1 Product Experience

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Playground** | Configure agent; select model; attach tools, skills and context; run task | Strong runtime primitives; tutoring/chat UX needs adaptation | Build AriseHub agent-engineering UX |
| **Agent Library** | Pre-built agents, templates, clone/customize, save/reuse | Agent definitions and reusable configurations available | Extend into first-class library and lifecycle |
| **Workflow Builder** | Multi-agent flows, branching, routing, human approval | Subagents/orchestration primitives exist | Preserve capability; visual builder deferred |
| **Workspace** | Files, repositories, knowledge sources, environment/config | Strong knowledge/files context; code workspace partial | Extend for repositories and execution |
| **Runs & Observability** | Execution steps, tool calls, handoffs, tokens, latency, cost, errors | Trace/activity concepts available | Turn runs into first-class inspectable objects |

## 3.2 Agent Runtime

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Agent Definition** | Instructions, model, tools, skills, memory, policies | Strong | Reuse and extend with AriseHub schema/API |
| **Skills** | Reusable procedural/domain instructions; versioned packages | Strong | Reuse; add library/versioning UX |
| **Tools** | Native tools, APIs, MCP, custom functions | Strong | Reuse; add permissions and curated tool catalog |
| **Context** | Prompt, workspace, retrieved and dynamic context | Strong | Reuse/extend context inspection |
| **Memory** | Session, persistent, project/learner memory | Strong | Reuse; expose memory controls and visibility |
| **Multi-Agent Orchestration** | Delegation, supervisor/worker, parallelism, shared state | Subagent capability provides a base | Extend incrementally; visual workflow not required in V1 |

## 3.3 Execution Environment

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Code Sandbox** | Isolated shell/code execution; language runtimes; packages | Partial / not core strength | AriseHub addition; OpenHands-inspired/adapter |
| **Repository Operations** | Clone, read/modify, diff, branch, commit, tests | Partial | AriseHub addition around workspace/sandbox |
| **Local/Desktop Execution** | Local filesystem and user-machine tools | Not required for base | Deferred; OpenWorker-inspired |
| **Job Runtime** | Background execution, queue, retry, timeout, parallel work | Runtime foundation exists | Harden/extend for production execution |

## 3.4 Knowledge & Retrieval

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Knowledge Base** | PDFs, Markdown, docs, course material, repository docs | Very strong | Reuse strongly |
| **Retrieval** | Chunking, embeddings, search, reranking, context injection | Strong | Reuse/extend as required |
| **Citations / Provenance** | Show sources and trace generated output to context | Strong concepts | Expose clearly in AriseHub run/answer UX |

## 3.5 Model & Provider Layer

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Model Gateway** | OpenAI, Anthropic, Gemini, Kimi/DeepSeek, local models | Multi-provider base | Reuse and extend provider coverage |
| **Model Configuration** | Context limits, reasoning/tool capability, parameters | Supported at runtime level | Build consistent AriseHub configuration UX |
| **Routing** | Fallback, task-based selection, cost-aware routing | Partial/provider dependent | Preserve; later extension |

## 3.6 Evaluation & Governance

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Evaluation** | Task success, tests, LLM judge, rubrics | Learning/evaluation ideas exist | AriseHub evaluation framework |
| **Agent Comparison** | Agent/model/prompt/skill comparisons | Not core | AriseHub addition |
| **Guardrails** | Tool permissions, filesystem/network boundaries, policy | Some runtime/tool boundaries | Strengthen for production |
| **Human-in-the-loop** | Approval for tools/code/output and review gates | Partial patterns | AriseHub extension; later V1+ |

## 3.7 Learning Experience — AriseHub Differentiation

| Capability | Scope | DeepTutor Base | AriseHub Direction |
|---|---|---|---|
| **Guided Labs** | Preconfigured tasks, starter repos, expected outcomes | Tutor concepts can inspire | AriseHub-owned IP |
| **Progressive Complexity** | Single agent → tools → memory → multi-agent → production | Not a product-level progression model | AriseHub-owned IP |
| **Learning Insights** | Explain what agent did, failures, learner changes and improvement | Trace/tutor data can feed this | AriseHub-owned interpretation and UX |
| **Assessment** | Task completion, engineering quality, agent design, human judgment | Some learning evaluation concepts | AriseHub-owned assessment model |

---

# 4. DeepTutor as the Base

DeepTutor should be treated as the **initial runtime foundation rather than the final product boundary**.

Its strengths align particularly well with:

- Agent definitions
- Skills
- Tools and MCP
- Context
- Memory
- Knowledge/RAG
- Model access
- Subagent execution

AriseHub should reshape the user-facing experience around **agent engineering** and add the missing production/workbench capabilities.

## Reuse vs Extend vs Build

### Reuse strongly from DeepTutor

- Skills
- Tools
- MCP
- Knowledge/RAG
- Memory
- Core agent runtime
- Provider integration

### Extend

- Agent definition UX/API
- Workspace
- Run model
- Observability
- Permissions
- Subagent orchestration

### Build as AriseHub

- Agent Playground UX
- Agent Library UX
- Code sandbox/repository experience
- Evaluation
- Agent comparison
- Learning experience

### Defer while preserving in architecture

- Visual workflow builder
- Local desktop execution
- Advanced model routing
- Institutional dashboards

---

# 5. V1 Vertical Slice

V1 should validate the Agent Playground thesis without attempting to implement the entire capability map.

The first release should be a coherent vertical slice across the architecture.

> **For the authoritative feature-to-version assignments (which capabilities are V1, V1.1, V2, Deferred, or Cut), see [Feature Version Matrix](./feature-version-matrix.md).** This §5 narrative is a high-level description; the matrix is the source of truth for scope.

### Playground

Create/configure an agent and give it a task.

### Agent Definition

Configure:

- Model
- Instructions
- Skills
- Tools/MCP
- Context
- Memory

### Agent Library

Save, clone, edit, and rerun reusable agents.

### Workspace

Start with:

- Files
- Knowledge sources

Then progressively introduce:

- Git repositories
- Code execution
- Environment configuration

### Run

Task execution becomes a **first-class entity**, rather than merely a chat turn.

### Observability

Inspect:

- Execution steps
- Tool calls
- Agent/subagent activity
- Outputs
- Errors
- Token usage
- Latency
- Cost where available

### Knowledge & Retrieval

Leverage DeepTutor's existing knowledge/RAG strengths.

### Model Providers

Leverage DeepTutor's provider layer and extend only where necessary.

### Subagents

Use DeepTutor's existing delegation patterns without requiring a visual workflow designer.

## Explicitly Not Required for V1

- Visual workflow canvas
- Sophisticated agent comparison
- Full evaluation platform
- Local desktop agent
- Advanced model routing
- Institution dashboards
- Complete guided-learning layer

These remain valid architecture capabilities and can be introduced later without changing the fundamental product model.

---

# 6. Run and Observability Model

A **Run** should become a first-class AriseHub object.

This is central to the difference between an agent engineering workbench and a chat interface.

A normalized event model could include:

```text
RUN_STARTED
RUN_COMPLETED
RUN_FAILED

MODEL_REQUEST
MODEL_RESPONSE

TOOL_REQUEST
TOOL_RESULT

AGENT_DELEGATION
SUBAGENT_RESULT

CONTEXT_RETRIEVED
KNOWLEDGE_SOURCE_USED

FILE_READ
FILE_WRITE

COMMAND_STARTED
COMMAND_FINISHED

HUMAN_APPROVAL_REQUIRED
HUMAN_APPROVAL_RESULT

EVALUATION_STARTED
EVALUATION_COMPLETED
```

The exact vocabulary can evolve.

The architectural objective is to normalize runtime activity into a stable event stream so that the:

- Run Inspector
- Evaluation framework
- Learning Insights
- Future workflow engine
- Analytics

remain decoupled from DeepTutor's internal implementation.

---

# 7. Extension Architecture

```text
AriseHub Web / API
       │
       ▼
AriseHub Capability Contracts
       │
       ├── Agent
       ├── Run
       ├── Workspace
       ├── Tool
       ├── Skill
       └── Memory
       │
       ▼
DeepTutor Adapter / Extension Layer
       │
       ├── DeepTutor Agent Runtime
       ├── Skills / Tools / MCP
       ├── Memory / Context
       ├── Knowledge / Retrieval
       └── Provider Integrations
       │
       ├────────► Sandbox Adapter
       │               │
       │               ▼
       │       OpenHands-style Execution
       │
       └────────► Future Workflow / Evaluation Adapters
```

The adapter/contract layer is not abstraction for its own sake.

It prevents the AriseHub product model from becoming identical to DeepTutor's internal representation and gives AriseHub room to replace or evolve individual subsystems.

---

# 8. Licensing and Source Governance

Because AriseHub Agent Playground is intended as a commercial offering, third-party source usage must be explicit and auditable.

DeepTutor's Apache-2.0 base is attractive for this reason, but the license of the **exact repository, version/commit, embedded dependencies, and any copied components must be verified before adoption**.

## Rules

- Pin every adopted upstream component to a known version/commit and record its license.
- Maintain `THIRD_PARTY_NOTICES.md`.
- Maintain a machine-readable dependency/license inventory.
- Preserve required copyright and NOTICE obligations.
- Prefer MIT, BSD, and Apache-2.0 dependencies for core commercial product code.
- Require explicit review for GPL/AGPL, source-available, business-source, or custom licenses.
- Do not copy code from Dify, Open WebUI, or other projects merely because they are useful architectural references; verify the exact applicable license first.
- Keep AriseHub-original code and modifications clearly identifiable.
- Run automated license scanning in CI.

> **Note:** This is architecture and engineering guidance, not legal advice. A commercial-release dependency inventory should be reviewed by qualified counsel.

---

# 9. Suggested Delivery Sequence

## Phase 0 — Baseline DeepTutor

- Select and pin the DeepTutor repository/version.
- Run DeepTutor unchanged.
- Map actual DeepTutor modules to the canonical capability model.
- Record licenses and third-party dependencies.
- Identify extension seams.

## Phase 1 — AriseHub Product Shell

Introduce:

- AriseHub product/navigation model
- Agent as a first-class entity
- Workspace as a first-class entity
- Run as a first-class entity

without unnecessarily changing the underlying runtime.

## Phase 2 — Build → Run → Observe

Deliver:

```text
BUILD
  │
  ▼
RUN
  │
  ▼
OBSERVE
```

using DeepTutor's:

- Agent runtime
- Skills
- Tools
- MCP
- Context
- Memory
- Knowledge
- Model providers

## Phase 3 — Engineering Workspace

Add:

- Repository operations
- Git
- Terminal
- Isolated execution
- Tests
- Diffs

through an **AriseHub Sandbox Adapter**.

OpenHands can be used as an architectural/implementation inspiration where licensing and integration are appropriate.

## Phase 4 — Evaluation & Learning

Add:

- Reusable evaluations
- Agent/model comparisons
- Guided labs
- Learning insights
- Assessments

on top of normalized Run data.

## Phase 5 — Compose

Expand:

- Multi-agent orchestration
- Workflow definitions
- Human approval
- Branching/routing

Only when justified, add the **visual workflow builder**.

---

# 10. Architecture Decision Summary

| Decision | Direction |
|---|---|
| **Product** | AriseHub Agent Playground |
| **Core proposition** | Build, run, observe, and progressively compose agents |
| **Base** | DeepTutor runtime and selected capabilities |
| **AriseHub ownership** | Product UX, capability contracts, run model, extensions, evaluation, learning experience |
| **V1** | Playground + Agent Library + Workspace + Runs/Observability with agent/runtime primitives |
| **Workflow** | Capability preserved; visual workflow builder deferred |
| **Engineering execution** | AriseHub sandbox/repository adapter |
| **Knowledge/RAG** | Primarily leverage DeepTutor |
| **Skills/Tools/MCP/Memory** | Primarily leverage DeepTutor |
| **Licensing posture** | Prefer permissive OSS and maintain explicit third-party governance |

---

# 11. Product Capability Summary

The complete AriseHub Agent Playground architecture can be summarized as:

```text
BUILD
 │
 ├── Agent Definition
 ├── Models
 ├── Skills
 ├── Tools / MCP
 ├── Context
 └── Memory
 │
 ▼
RUN
 │
 ├── Workspace
 ├── Knowledge
 ├── Sandbox
 ├── Repository
 └── Agent / Subagents
 │
 ▼
OBSERVE
 │
 ├── Runs
 ├── Traces
 ├── Tool Calls
 ├── Context / Provenance
 ├── Cost / Tokens / Latency
 └── Errors
 │
 ▼
EVALUATE
 │
 ├── Task Success
 ├── Tests
 ├── Rubrics
 └── Comparison
 │
 ▼
COMPOSE
 │
 ├── Multi-Agent
 ├── Delegation
 ├── Routing
 ├── Human-in-the-loop
 └── Future Visual Workflow
 │
 ▼
LEARN
   ├── Guided Labs
   ├── Progressive Complexity
   ├── Learning Insights
   └── Assessment
```

---

**Canonical architecture baseline for subsequent DeepTutor gap analysis, product design, and implementation planning.**
