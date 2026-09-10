<div align="center">

<p align="center"><img src="assets/figs/logo/logo.png" alt="AriseHub Agent Playground logo" height="56" style="vertical-align: middle;">&nbsp;<img src="assets/figs/logo/banner.png" alt="AriseHub Agent Playground" height="48" style="vertical-align: middle;"></p>

# AriseHub Agent Playground: Agent-Native AI Workspace

<p align="center">
  <a href="https://arise-hub.github.io/AgenticCoach" target="_blank"><img alt="Docs — arise-hub.github.io/AgenticCoach" src="https://img.shields.io/badge/Docs-arise--hub.github.io%2FAgenticCoach%20%E2%86%97-0A0A0A?style=for-the-badge&labelColor=F5F5F4" height="36"></a>&nbsp;
  <a href="https://github.com/arise-hub/AgenticCoach/discussions" target="_blank"><img alt="Discussions — collaborate with us" src="https://img.shields.io/badge/Discussions-collaborate%20with%20us%20%E2%86%97-0A0A0A?style=for-the-badge&labelColor=F5F5F4" height="36"></a>
</p>

<p align="center">
  <a href="README.md"><img alt="English" height="40" src="https://img.shields.io/badge/English-BCDCF7"></a>&nbsp;
</p>

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/downloads/)
[![Next.js 16](https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=next.js&logoColor=white)](https://nextjs.org/)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue?style=flat-square)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/arise-hub/AgenticCoach?style=flat-square&color=brightgreen)](https://github.com/arise-hub/AgenticCoach/releases)

[![Discord](https://img.shields.io/badge/Discord-Community-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.gg/arisehub)

[Features](#-key-features) · [Get Started](#-get-started) · [Explore](#-explore-arisehub-agent-playground) · [CLI](#%EF%B8%8F-arisehub-agent-playground-cli--agent-native-interface) · [Community](#-community)

</div>

---

> 🤝 **We welcome any kinds of contributing!** Vote on roadmap items or propose new ones at [`Roadmap`](https://github.com/arise-hub/AgenticCoach/issues), and see our [Contributing Guide](CONTRIBUTING.md) for branching strategy, coding standards, and how to get started.

### 📦 Releases

> **[2026.9.x]** [v0.0.1](https://github.com/arise-hub/AgenticCoach/releases/tag/v0.0.1) — **Initial AriseHub Agent Playground release**: Rebranded from DeepTutor, agent-native architecture with Tools + Capabilities plugin model, Chat, Deep Research, Deep Solve, Deep Question, Visualize, Mastery Path, Co-Writer, Knowledge Center, Learning Space, Memory, Partners, and persistent memory.

<details>
<summary><b>Upstream releases (DeepTutor heritage)</b></summary>

> This project is a fork of [DeepTutor](https://github.com/HKUDS/DeepTutor) (HKUDS). See [ADR-001](arisehub-docs/ADR-001-upstream-strategy.md) for our upstream merge strategy.

</details>

### ✨ Key Features

AriseHub Agent Playground is an agent-native workspace that connects chat, problem solving, research, visualization, and mastery practice in one extensible system.

- **One runtime for every mode** — Chat, Deep Research, Deep Solve, Deep Question, Visualize, Mastery Path, and Co-Writer run on the same agent loop, so you switch the objective, not the engine, and context moves with the task.
- **Connected context** — Knowledge bases, Co-Writer drafts, notebooks, question banks, personas, and Memory stay available across every workflow instead of living in isolated tools.
- **Subagents and Partners** — consult a live coding CLI (Claude Code, Codex, Gemini, Kimi, opencode, or MiMo) or a Partner from any turn, and run persistent IM companions on the same brain.
- **Multi-engine knowledge** — versioned RAG libraries across LlamaIndex, PageIndex, GraphRAG, LightRAG, with pluggable document parsing.
- **Extensible tools and skills** — built-in tools, MCP servers, CLI apps, and installable community skills.
- **Inspectable memory** — L1 traces, L2 surface summaries, and L3 synthesis make personalization visible and editable, with a Memory Graph that traces every claim back to its evidence.

---

## 🚀 Get Started

AriseHub Agent Playground ships multiple installation paths. They all share one workspace layout: settings live in `data/user/settings/` under the directory you launch from (or under `ARISEHUB_HOME` / `arisehub start --home` if you set one explicitly).

<details>
<summary><b>Option 1 — Docker (GHCR)</b> · one self-contained container</summary>

Pre-built images on GitHub Container Registry:

- `ghcr.io/arise-hub/arisehub-agent-playground:v0.0.1` — stable release
- `ghcr.io/arise-hub/arisehub-agent-playground:latest` — latest release

```bash
docker run --rm --name arisehub-agent-playground \
  -p 127.0.0.1:3782:3782 \
  -v arisehub-agent-playground-data:/app/data \
  ghcr.io/arise-hub/arisehub-agent-playground:v0.0.1
```

> **Only `3782` needs to be published.** The browser talks exclusively to the frontend origin; the Next.js middleware (`web/proxy.ts`) forwards `/api/*` and `/ws/*` to the FastAPI backend **inside the container**.

Open [http://127.0.0.1:3782](http://127.0.0.1:3782). The container creates `/app/data/user/settings/*.json` on first boot; configure model providers from the Web Settings page. Config, API keys, logs, workspace files, memory, and knowledge bases persist in the `arisehub-agent-playground-data` volume.

</details>

<details>
<summary><b>Option 2 — Install From Source</b> · develop against a checkout</summary>

For development against a checkout. Use **Python 3.11–3.13** and **Node.js 22 LTS** to match CI and Docker.

```bash
git clone https://github.com/arise-hub/AgenticCoach.git
cd AgenticCoach

# Create a venv (macOS/Linux). Windows PowerShell:
#   py -3.11 -m venv .venv ; .\.venv\Scripts\Activate.ps1
python3 -m venv .venv && source .venv/bin/activate
python -m pip install --upgrade pip

# Install backend + frontend deps
python -m pip install -e .
( cd web && npm ci --legacy-peer-deps )

arisehub init
arisehub start --dev
```

`arisehub start` builds the local `web/` frontend for production once and reuses it; `--dev` runs Next.js with HMR. Config layout, ports, and `Ctrl+C` match Option 1.

</details>

<details>
<summary><b>Option 3 — CLI Only</b> · no Web UI, from a source checkout</summary>

When you don't need the Web UI.

```bash
git clone https://github.com/arise-hub/AgenticCoach.git
cd AgenticCoach

python3 -m venv .venv-cli && source .venv-cli/bin/activate
python -m pip install --upgrade pip

python -m pip install -e ./packaging/deeptutor-cli
arisehub init --cli
arisehub chat
```

The local `arisehub-cli` install ships no Web assets or server dependencies. Keep the source checkout around — the editable install points to it.

</details>

<details>
<summary><b>Configuration reference</b> — config files under <code>data/user/settings/</code> (JSON/YAML)</summary>

Everything under `data/user/settings/` is plain JSON/YAML. The **Settings** page in the browser is the recommended editor.

| File | Purpose |
|:---|:---|
| `model_catalog.json` | LLM, embedding, and search provider profiles; API keys; active models |
| `system.json` | Backend/frontend ports, public API base, CORS, SSL verification, attachment directory and upload/extraction limits |
| `auth.json` | Optional auth toggle, username, password hash, token/cookie settings |
| `integrations.json` | Optional PocketBase and sidecar integration settings |
| `interface.json` | UI and model output language / theme / sidebar preferences |
| `main.yaml` | Runtime behavior defaults and path injection |
| `agents.yaml` | Capability/tool temperature and token settings |

Project-root `.env` is **not** read as an application config file. For a minimal model setup, open **Settings → Models**, add an LLM profile (Base URL / API key / model name), and save. Add an embedding profile only if you plan to use Knowledge Base / RAG features.

</details>

## 📖 Explore AriseHub Agent Playground

Start with the main surfaces: Chat, Partners, My Agents, Co-Writer, Knowledge Center, Learning Space, Memory, and Settings.

<details>
<summary><b>💬 Chat — The Agent Loop</b></summary>

Chat is the default capability. A single thread can talk normally, call tools, ground itself in selected knowledge bases, read attachments, generate images, consult subagents, write notebook records, and continue with the same context across turns.

The loop is deliberately simple: the model thinks in rounds, calls tools when useful, observes the results, and finishes with a tool-free message. `ask_user` is special — instead of guessing, the agent can pause the turn, ask a clarifying question, and resume once you answer.

User-toggleable tools: `brainstorm`, `web_search`, `paper_search`, `reason`. Contextual tools such as `rag`, `read_source`, `read_memory`, `write_memory`, `exec`, `web_fetch`, `ask_user`, `list_notebook`, `write_note`, `github`, and `consult_subagent` mount automatically when the turn has the right context.

Chat is also the launch point for deeper capabilities: **Deep Research** for cited reports, **Deep Solve** for worked reasoning, **Deep Question** for question generation, **Visualize** for charts/diagrams/animations, and **Mastery Path** for guided learning flows.

</details>

<details>
<summary><b>🤝 Partners — Persistent Companions</b></summary>

Partners are persistent companions with their own soul, model policy, library, memory, and channels. Every inbound message becomes a normal `ChatOrchestrator` turn inside a partner-scoped workspace.

Each partner has a `SOUL.md`, model selection, channels, tool policy, and assigned library. Knowledge bases, skills, and notebooks are copied into `data/partners/<id>/workspace/`, so the same RAG, skill, notebook, and memory tools work without special cases.

</details>

<details>
<summary><b>🧑‍🚀 My Agents — Consult Other Agents</b></summary>

My Agents turns other agents into context. **Connect a live agent** — a Claude Code, Codex, Gemini, Kimi, opencode, or MiMo Code CLI — and consult it from inside a chat turn via the `consult_subagent` tool. **Import past conversations** — bring in your existing Claude Code and Codex history as named, searchable, resumable agents.

</details>

<details>
<summary><b>✍️ Co-Writer — Selection-Aware Markdown Drafting</b></summary>

Co-Writer is a split-view Markdown workspace for reports, tutorials, notes, and long-form artifacts. Documents autosave and render a live preview (KaTeX math, diagram fences), and can be saved back into notebooks.

Its defining idea is **surgical editing**: select a span and ask the agent to rewrite, expand, or shorten it. The edit agent can ground the change in a knowledge base or web evidence, keeps a trace of its tool calls, and shows every change as an accept/reject diff.

</details>

<details>
<summary><b>📚 Knowledge Center — Multi-Engine RAG</b></summary>

Knowledge bases are the document collections behind RAG. Choice of retrieval engines: **LlamaIndex** (default, local vector + BM25), **PageIndex** (reasoning retrieval), **GraphRAG** and **LightRAG** (knowledge-graph retrieval), **LightRAG Server** (external LightRAG instance).

Creating a KB, you either **create new** (upload documents and build a fresh index) or **link existing** (reuse an index built elsewhere). Re-indexing writes a new flat `version-N` directory and keeps prior ones.

</details>

<details>
<summary><b>🌐 Learning Space — Skills, Personas, Reusable Context</b></summary>

Learning Space holds your chat history, notebooks, question bank, mastery paths, personas, skills (`SKILL.md` playbooks), MCP Services, and CLI Apps. Everything here can be reused from Chat, Partners, Co-Writer.

**Import from EduHub** browses the community catalog and downloads a skill straight into your library through a security gate.

</details>

<details>
<summary><b>🧠 Memory — Inspectable Personalization</b></summary>

Memory is a file-backed, three-layer system you can read, curate, and audit. **L1** is the workspace mirror plus an append-only event trace; **L2** is per-surface curated facts; **L3** is cross-surface synthesis. Because L2 cites L1 and L3 cites L2, nothing in your profile is unaccountable.

The Memory Graph shows the whole pyramid — L3 synthesis at the centre, L2 in the middle ring, L1 traces on the outside — so you can trace any synthesized claim back to the exact raw event behind it.

</details>

<details>
<summary><b>⚙️ Settings — One Control Plane</b></summary>

Settings is the operational control plane with a live status strip and one card per area: **Appearance** (theme, language, code-block styling), **Network** (API base, ports, CORS), **Models** (LLM, Embedding, Search, TTS, STT, Image/Video Generation), **Knowledge Base** (document parsing engine), **Chat** (tools, per-capability parameters), **Partners & Agents**, and **Memory** (consolidator budgets).

Most sections use a draft-and-apply flow. You can also just ask in Chat: the assistant reads the current configuration, applies a change, and says whether it needs a restart or a re-index.

</details>

---

## 🏗️ Architecture

AriseHub Agent Playground follows a two-layer plugin model exposed through three entry points:

```
Entry Points:  CLI (Typer)  |  WebSocket /api/v1/ws  |  Python SDK
                    ↓                   ↓                   ↓
              ┌─────────────────────────────────────────────────┐
              │              ChatOrchestrator                    │
              │   routes UnifiedContext → selected Capability    │
              └──────────┬──────────────┬───────────────────────┘
                         │              │
              ┌──────────▼──┐  ┌────────▼──────────┐
              │ ToolRegistry │  │ CapabilityRegistry │
              │  (Level 1)   │  │   (Level 2)        │
              └──────────────┘  └────────────────────┘
```

- **Level 1 — Tools**: Single-function tools the LLM picks on demand (brainstorm, web_search, paper_search, reason, rag, exec, code_execution, etc.)
- **Level 2 — Capabilities**: Multi-stage pipelines that own the turn (chat, deep_research, deep_solve, deep_question, visualize, mastery_path, math_animator)

All capabilities converge on a shared `StreamBus` for event fan-out. Runtime settings live in `data/user/settings/*.json` — project-root `.env` files are intentionally ignored.

See [Architecture Documentation](arisehub-docs/AriseHub_Agent_Playground_Architecture.md) for details.

---

## 🛠️ CLI — Agent-Native Interface

```bash
# Interactive REPL
arisehub chat

# Run any capability
arisehub run chat "Explain Fourier transform"
arisehub run deep_solve "Solve x^2=4" --tool rag --kb my-kb
arisehub run visualize "Animate sine wave" --config render_mode=manim_video

# Knowledge bases, memory, partners
arisehub kb create my-kb --doc textbook.pdf
arisehub memory show
arisehub partner list

# Server
arisehub serve --port 8001       # API server only
arisehub start                   # backend + frontend together
```

---

## 🤝 Community

- **Discord**: [discord.gg/arisehub](https://discord.gg/arisehub)
- **GitHub Discussions**: [github.com/arise-hub/AgenticCoach/discussions](https://github.com/arise-hub/AgenticCoach/discussions)
- **Issues**: [github.com/arise-hub/AgenticCoach/issues](https://github.com/arise-hub/AgenticCoach/issues)

---

## 📄 License

Apache 2.0 — see [LICENSE](LICENSE) for details.

---

## 🙏 Heritage

AriseHub Agent Playground is a fork of [DeepTutor](https://github.com/HKUDS/DeepTutor) by HKUDS. We follow a **merge-on-release** upstream strategy (see [ADR-001](arisehub-docs/ADR-001-upstream-strategy.md)) — cherry-picking critical fixes and merging on upstream releases, never rebasing. Our AriseHub-specific code lives under `arisehub/` to maintain clean separation.

The original DeepTutor paper is available on [arXiv](https://arxiv.org/abs/2604.26962).