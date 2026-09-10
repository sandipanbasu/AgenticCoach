# Upstream Services Inventory

| Field | Value |
|---|---|
| **Status** | Draft (pending review) |
| **Date** | 2026-09-05 |
| **Author** | Sandipan Basu + ruflo session |
| **Audit method** | Knowledge graph (`codebase-memory-mcp`) cross-referenced with targeted grep; not a full audit |
| **Related** | [ADR-001: Upstream Strategy](./ADR-001-upstream-strategy.md) · [Architecture Baseline §8 Licensing](./AriseHub_Agent_Playground_Architecture.md) · [Feature Version Matrix](./feature-version-matrix.md) · [V1 Flows and Scope](./v1-flows-and-scope.md) |

---

## Purpose

The AriseHub Agent Playground codebase is a fork of DeepTutor (Apache-2.0). Per [ADR-001](./ADR-001-upstream-strategy.md), AriseHub code lives under `arisehub/` and is structurally isolated from the fork's runtime under `deeptutor/`. The fork, however, contains code that makes outbound HTTP calls to **upstream-operated services** — endpoints that the DeepTutor maintainers control, not AriseHub.

This document inventories those upstream services, classifies them by severity, and records what AriseHub users can do about each one. It is the answer to the question *"is there any piece of code which calls home in deeptutor?"* — written down so we don't have to re-derive it each session.

**Scope:** This document covers only **upstream-operated** endpoints — services that the DeepTutor project or its maintainers run, where the user has not opted in by configuring a base URL. User-configured endpoints (arXiv, LLM providers, partner channels, search providers) are not phone-home; they are explicit dependencies the user has chosen. They are listed in §"Out of scope" for completeness.

**Severity taxonomy:**

| Severity | Meaning |
|---|---|
| **P0 — Active phone-home** | Code makes an outbound HTTP request to an upstream-operated service without the user having configured an alternative. Hard to disable. High attention. |
| **P1 — Attribution leak** | Code sends a header or identifier that names the upstream project. The request itself is user-invoked, but third parties can see "this came from a DeepTutor-derivative." |
| **P2 — Default-on dependency** | A user-configurable service that ships with a working upstream default. Not phone-home, but the default endpoint is upstream-operated and many users never change it. |
| **P3 — User-invoked** | The user explicitly runs a command or invokes a tool that hits an upstream-operated service. Documented for completeness; not phone-home. |

---

## Summary of findings

| # | Service | Severity | Where in fork | User-disablable? | V1 impact? |
|---|---|---|---|---|---|
| 1 | **Skill Hub defaults** (clawhub.ai, eduhub.deeptutor.info) | **P0** | `deeptutor/services/skill/hub.py:68-78` | Yes — `data/user/settings/skill_hubs.json` | None (V1 doesn't use skills) |
| 2 | **Web fetch User-Agent** (`DeepTutor/1.0 (+https://hkuds.dev/deeptutor)`) | P1 | `deeptutor/tools/web_fetch.py:41` | No | None (V1 doesn't use web_fetch) |
| 3 | **OpenRouter HTTP-Referer** (`https://github.com/HKUDS/DeepTutor`) | P1 | `deeptutor/services/llm/provider_core/openai_compat_provider.py:55` | No (header is hardcoded) | None (V1 uses model gateway but can avoid OpenRouter) |
| 4 | **MCP OAuth CLIENT_URI** (`https://deeptutor.info`) | P1 | `deeptutor/services/mcp/oauth.py:61` | No (string is hardcoded) | None (V1 doesn't use MCP) |
| 5 | **mineru.net parser default** | P2 | `deeptutor/api/routers/settings.py:248`, `deeptutor/services/config/runtime_settings.py:131` | Yes — settings override | None (V1 uses local parsers by default; mineru.net is opt-in) |
| 6 | **arXiv LaTeX downloader** | P3 | `deeptutor/tools/tex_downloader.py:79,246` | n/a (tool is user-invoked) | None (V1 doesn't use paper_search) |
| 7 | **Skill publish flow** (uses skill_hub_auth.json) | P3 (depends on #1) | `deeptutor/services/skill/credentials.py:6-103` | Same as #1 | None (V1 doesn't publish skills) |
| 8 | **Skill format docs link** (CLI help text) | P1 (link only) | `deeptutor_cli/skill.py:295` | n/a | None |

---

## Detailed findings

### 1. Skill Hub defaults — P0 (active phone-home) ⛔

**File:** `deeptutor/services/skill/hub.py:68-78`

```python
_CLAWHUB_BASE_URL = "https://clawhub.ai/api/v1"
_EDUHUB_BASE_URL = "https://eduhub.deeptutor.info/api/v1"

_BUILTIN_HUBS: dict[str, dict[str, str]] = {
    "clawhub": {"type": "clawhub", "base_url": _CLAWHUB_BASE_URL},
    "eduhub":  {"type": "clawhub", "base_url": _EDUHUB_BASE_URL},
}
```

**What happens:** Any user who runs `deeptutor skill install <ref>` or `deeptutor skill publish` (or the AriseHub equivalents, once they exist) makes an HTTPS request to `clawhub.ai` or `eduhub.deeptutor.info` — both **upstream-operated services under DeepTutor maintainers' control**. Bearer tokens are persisted to `data/user/settings/skill_hub_auth.json` (per `deeptutor/services/skill/credentials.py`).

**Why this is the highest-priority finding:**
- The endpoint is hardcoded in the fork. Per ADR-001 §4, AriseHub code cannot modify files under `deeptutor/`, so we cannot just remove these lines.
- The user is *not* explicitly told their first `skill install` will hit upstream services. It just works.
- The packages downloaded from these hubs are *executable code* that runs in the user's environment. This is a trust-boundary issue, not just data exfiltration.
- The `skill_publish` flow would upload user-authored skills to upstream servers, again with a bearer token in settings.

**How a user can disable / override it (today):**
- Create `data/user/settings/skill_hubs.json` with custom entries that override `clawhub` and `eduhub`. The loader at `deeptutor/services/skill/hub.py:71-73` says: *"A user's `skill_hubs.json` is layered on top and may override any entry (e.g. point `eduhub` at a local dev server)."*
- The override can point at a local mirror, a different community hub, or just block the calls.

**What AriseHub should do (recommended, not yet done):**
- **V1.1 (or earlier):** Ship a default `data/user/settings/skill_hubs.json` that points the built-in hubs at AriseHub-operated endpoints *or* at neutral placeholders. The user would need to opt-in to actually use skills from the upstream hubs.
- **V1.1:** Document this in the AriseHub README so the user knows before they run `skill install`.
- **Future (V2+):** If AriseHub wants its own skill ecosystem, an AriseHub-operated hub would replace these. License-clean.

**Why V1 is unaffected:** V1 (per the matrix) does not use the skill system. The `Skills — Reuse` row (#13) is the runtime ability to *use* skills; the *Skill Hub* distribution channel is a different concept and is not in V1 scope. Reconfirm at V1.1 audit.

---

### 2. Web fetch User-Agent — P1 (attribution leak)

**File:** `deeptutor/tools/web_fetch.py:41`

```python
DEFAULT_USER_AGENT = "DeepTutor/1.0 (+https://hkuds.dev/deeptutor)"
```

**What happens:** When the `web_fetch` tool fetches a URL on the user's behalf (user-invoked, not silent), it identifies itself in the `User-Agent` header as `DeepTutor/1.0` with a project URL. Any web server that logs `User-Agent` (the vast majority) will see this string.

**Why this matters for AriseHub:**
- It is an *attribution signal* in third-party logs. Operators of public websites, CDNs, and security scanners can see traffic labelled `DeepTutor/1.0` from AriseHub users.
- It is *not* a privacy violation in the conventional sense (the request is user-invoked) and it is *not* a phone-home (no upstream service is being called).
- It is, however, a small but real signal that AriseHub is a DeepTutor derivative.

**How a user can change it (today):**
- The constant is hardcoded. The user cannot override without forking.
- AriseHub code cannot modify this file (ADR-001 §4).

**What AriseHub should do (recommended, not yet done):**
- **At first upstream merge under ADR-001 (v1.6.4):** propose a change to upstream to make `DEFAULT_USER_AGENT` configurable via settings. If upstream accepts, this becomes a setting override. If upstream declines, document the limitation.
- **Short-term:** add a note to the AriseHub README that `web_fetch` identifies itself as DeepTutor in HTTP headers.

**Why V1 is unaffected:** V1 (per the matrix) does not include the `web_fetch` tool. The `Tools — Reuse` row (#15) is reuse of the tool *layer*; specific tools like `web_fetch` are user-toggled and not in the V1 starter agent set.

---

### 3. OpenRouter HTTP-Referer — P1 (attribution leak)

**File:** `deeptutor/services/llm/provider_core/openai_compat_provider.py:55`

```python
"HTTP-Referer": "https://github.com/HKUDS/DeepTutor",
```

**What happens:** When the fork calls OpenRouter, it includes a `HTTP-Referer` header naming the DeepTutor GitHub repository. OpenRouter (and any HTTP intermediary) sees this.

**Why this matters for AriseHub:**
- Same as #2: it is a third-party-log attribution signal that AriseHub is a DeepTutor derivative.
- The request is user-invoked (the user configured OpenRouter as their provider).
- Not a phone-home in the sense of *calling a DeepTutor-operated service*, but it is a public attribution signal.

**How a user can change it (today):**
- The header is hardcoded. The user cannot override without forking.
- AriseHub code cannot modify this file (ADR-001 §4).

**What AriseHub should do (recommended, not yet done):**
- **At first upstream merge under ADR-001 (v1.6.4):** propose making this header configurable via provider settings.
- **Short-term alternative:** AriseHub users can avoid OpenRouter entirely and use a different provider from the model's V1 surface (per matrix row #32). OpenRouter is one of many; nothing forces V1 users to use it.

**Why V1 is unaffected:** V1 reuses the model gateway. The OpenRouter attribution leak is a known issue for users who choose OpenRouter; users who choose other providers do not see it.

---

### 4. MCP OAuth CLIENT_URI — P1 (attribution leak)

**File:** `deeptutor/services/mcp/oauth.py:61`

```python
CLIENT_URI = "https://deeptutor.info"
```

**What happens:** When the fork initiates an OAuth flow with an MCP (Model Context Protocol) server, the `client_uri` parameter identifies the client as `deeptutor.info`. The OAuth server logs and stores this.

**Why this matters for AriseHub:**
- Most MCP OAuth servers do not act on `client_uri` — it is informational.
- Some servers *do* surface it to the user during the consent screen ("Client deeptutor.info is requesting access to your data"). For an AriseHub user, that text is wrong.
- It is a settings string, not a phone-home (the OAuth flow is initiated by the user).

**How a user can change it (today):**
- Hardcoded. Cannot override without forking.
- AriseHub code cannot modify this file (ADR-001 §4).

**What AriseHub should do (recommended, not yet done):**
- **At first upstream merge under ADR-001 (v1.6.4):** propose making `CLIENT_URI` configurable via settings.
- **V1.1+ (when MCP ships in AriseHub per matrix row #15):** ensure the OAuth consent screen shows an AriseHub-consistent name.

**Why V1 is unaffected:** V1 (per the matrix) does not use MCP. `Tools — Reuse` row (#15) reuses the tool layer; the `MCP` capability ships in V1.1 or later when user-visible MCP tooling is in scope.

---

### 5. mineru.net parser default — P2 (default-on dependency)

**Files:**
- `deeptutor/api/routers/settings.py:248, 1236`
- `deeptutor/services/config/runtime_settings.py:131, 932`

```python
api_base_url: str = "https://mineru.net"
```

**What happens:** When a user configures a document parser, the default endpoint is `https://mineru.net`. Until the user re-configures, every parsed document makes a request to `mineru.net`. The DeepTutor maintainers do not operate this service (it is operated by MinerU's authors), so this is *not* a DeepTutor phone-home — it is a third-party service that ships as the default.

**Why this matters for AriseHub:**
- This is a *default-on* dependency, not a phone-home. The user is not calling DeepTutor infrastructure; they are calling a third-party parser.
- It is, however, an externally-operated service that AriseHub users will hit by default, which the user may not realize.

**How a user can change it (today):**
- `data/user/settings/` overrides work via the runtime-settings loader. The user can set their own `api_base_url` for the parser.

**What AriseHub should do (recommended, not yet done):**
- **V1:** ship a default that points at a local parser (no network) or a clearly-marked AriseHub-neutral default. Document the choice in the README.
- **V1.1:** if MinerU is still the recommended parser, document the data flow ("documents you upload are sent to mineru.net for parsing unless you change this in settings").

**Why V1 is unaffected:** V1 reuses the fork's knowledge base management (matrix row #29) which already includes the parser stack. The user can configure a local parser from day one; nothing in V1 forces a `mineru.net` call.

---

### 6. arXiv LaTeX downloader — P3 (user-invoked)

**File:** `deeptutor/tools/tex_downloader.py:79, 246`

```python
source_url = f"https://arxiv.org/e-print/{arxiv_id}"
response = requests.get(source_url, timeout=30)
```

**What happens:** When a user runs the `paper_search` tool and selects a paper, the tool fetches the LaTeX source from arXiv. arXiv is operated by Cornell University, not by DeepTutor.

**Severity:** P3 — this is a user-invoked call to a public scholarly archive, not a phone-home to upstream DeepTutor infrastructure. Documented for completeness.

**Why V1 is unaffected:** V1 (per the matrix) does not include the `paper_search` tool. Listed for awareness when V1.1 adds it.

---

### 7. Skill publish credentials — P3 (depends on #1)

**File:** `deeptutor/services/skill/credentials.py:6-103`

Bears are stored in `data/user/settings/skill_hub_auth.json` with `0600` perms. The flow is: user runs `deeptutor skills login` (browser-based OAuth), receives a bearer token, the token is persisted. Subsequent `skill publish` / `skill update` calls send the bearer token to whichever hub the user is publishing to.

**Severity:** P3 — the user explicitly initiated the login flow. Not silent. But: if the user is using a built-in hub (clawhub.ai or eduhub.deeptutor.info), they are sending an authenticated request to an upstream-operated service. The token stays local; the *request* is outbound.

**Why V1 is unaffected:** V1 doesn't ship skill publishing.

---

### 8. Skill format docs link — P1 (link only, but still attribution)

**File:** `deeptutor_cli/skill.py:295`

```python
console.print(
    f"[dim]格式规范见 https://eduhub.deeptutor.info/skill-format.md[/]"
)
```

**What happens:** When a user runs `deeptutor skill register` (or similar) and the format precheck fails, the CLI helpfully prints a link to the skill format docs on `eduhub.deeptutor.info`.

**Severity:** P1 — it's a documentation link, not a phone-home, but the user is being directed to an upstream-operated site for help text. AriseHub users seeing this link will notice it.

**Why V1 is unaffected:** V1 doesn't use the skill CLI. Listed for awareness.

---

## Out of scope (explicitly not phone-home)

These are *user-configured* external services that the fork talks to. They are not phone-home because the user explicitly configured them (or the user explicitly chose to use a tool that calls them). They are listed here so future audits don't flag them as findings.

| Service | File | Why out of scope |
|---|---|---|
| LLM provider endpoints (OpenAI, Anthropic, Cohere, OpenRouter, Groq, SiliconFlow, Volces, etc.) | `deeptutor/services/config/provider_runtime.py` | User configures the API key and base URL. |
| arXiv (paper search tool) | `deeptutor/tools/tex_downloader.py` | User-invoked via `paper_search`. |
| Search providers (Tavily, Brave, Jina, Bocha, Serper, etc.) | `deeptutor/services/search/providers/*.py` | User configures the API key. |
| Document parser endpoints (mineru.net) | `deeptutor/api/routers/settings.py` | See finding #5 — user-configurable. |
| Partner channel endpoints (Matrix homeserver, Discord, Weixin, DingTalk, MSTeams, Mochat, Feishu, Napcat, etc.) | `deeptutor/partners/channels/*.py` | User explicitly configured the channel integration. |
| Codebuddy credentials endpoints | `deeptutor/services/codebuddy_credentials.py` | User explicitly logged in to the Codebuddy service. |
| External user-configured HTTP base URLs | various | All overridable via settings. |
| `urlopen` health-checks in launcher | `deeptutor/runtime/launcher.py:475,485` | Probes URLs the caller constructed. No upstream call. |

---

## Cross-references to the matrix

The following matrix rows should be read with the findings in this document as footnotes:

| Matrix row | Finding to footnote |
|---|---|
| #2 (Starter agent library — V1) | None directly, but starter agents should not by default invoke tools that hit upstream services (e.g. `web_fetch`, `paper_search`, `skill install`). |
| #5 (Workspace — V1) | See #5 (mineru.net default). |
| #13 (Skills — Reuse, V1) | **See #1 (Skill Hub defaults — P0).** The skill *system* is reusable; the skill *hub* is upstream-operated. |
| #15 (Tools — Reuse, V1) | **See #2, #3, #4 (attribution leaks).** Tools that fetch URLs (web_fetch) or talk to LLM providers (OpenRouter) leak upstream attribution. |
| #29 (Knowledge Base — V1) | **See #5 (mineru.net default).** Document parsing default is upstream. |
| #32 (Model Gateway — V1) | **See #3 (OpenRouter referer).** Provider-layer attribution leak. |

---

## Recommended actions (proposed, not yet done)

In rough priority order, with proposed effort:

| Action | Priority | Proposed version | Proposed mechanism |
|---|---|---|---|
| Document skill-hub override path in the AriseHub README so users know how to neutralize finding #1. | High | V1.1 | README addition; no code change. |
| Ship a default `data/user/settings/skill_hubs.json` that overrides the built-in hubs to neutral endpoints (or comments them out). | High | V1.1 | Settings file in `arisehub/data/user_settings_seed/skill_hubs.json`, copied on first run. |
| Propose upstream changes (under ADR-001) for #2, #3, #4 to make attribution strings configurable. | Medium | First v1.6.4 merge (or later) | Upstream PRs. |
| Document the data flow for `mineru.net` (finding #5) in the AriseHub README. | Medium | V1.1 | README addition. |
| Decide on an AriseHub-operated skill hub (or a clear decision not to operate one). | Strategic, deferred | V2+ | Separate ADR when this becomes a real product question. |

---

## Audit method and limitations

- **What was searched:** The full `deeptutor/` and `deeptutor_cli/` Python codebase, via the knowledge graph (`codebase-memory-mcp`) cross-referenced with targeted grep.
- **What was NOT searched:** JavaScript / TypeScript code under `web/` (the frontend). The frontend does not make outbound calls itself; it proxies through the backend. A separate frontend audit could be done at V1.1.
- **What was NOT analyzed:** Embedded binaries, compiled assets, fonts, or icons — all believed to be static and offline.
- **What may be missed:** Any code that constructs URLs at runtime (string concatenation, format strings) rather than referencing a constant. A `SearchCode` pass for `f"https://` and similar string-formatting patterns could surface these. Recommended as a follow-up.
- **What may be transient:** The `clawhub.ai` and `eduhub.deeptutor.info` endpoints could change ownership or be redirected at any time. This document is a snapshot as of 2026-09-05 against the fork at v1.5.16 + 5 AriseHub branding/deploy commits.

---

## Change log

| Date | Change | Author |
|---|---|---|
| 2026-09-05 | Initial draft (8 findings, 1 P0, 4 P1, 1 P2, 2 P3) | Sandipan Basu + ruflo session |
| _pending_ | Re-audit after v1.6.4 merge (per ADR-001). | — |
| _pending_ | Frontend audit (web/) before V1.1. | — |

---

**End of Upstream Services Inventory.**
