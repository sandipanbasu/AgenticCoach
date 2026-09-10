# DeepTutor Branding Inventory

| Field | Value |
|---|---|
| **Status** | Draft (pending review) |
| **Date** | 2026-09-06 |
| **Author** | Sandipan Basu + ruflo session |
| **Related** | [Upstream Services Inventory](./upstream-services.md) · [Feature Version Matrix](./feature-version-matrix.md) · [ADR-001 Upstream Strategy](./ADR-001-upstream-strategy.md) |

---

## Summary

The AriseHub Agent Playground codebase is a fork of DeepTutor v1.5.16. The web frontend has **already been rebranded** to "AriseHub Agent Playground" via a central abstraction (`web/lib/brand.ts`). The remaining DeepTutor branding lives in **fork-owned backend/CLI code** that AriseHub cannot modify per ADR-001 §4.

| Classification | Count | What it means |
|---|---|---|
| **Class A — User-visible runtime (fork-owned, V1 must-address)** | 3 | CLI banner, FastAPI OpenAPI title, CLI command name/help |
| **Class B — User-visible static asset** | 0 | Web assets are generic; README assets are docs-only |
| **Class C — External attribution / phone-home** | 4 | Documented in [upstream-services.md](./upstream-services.md) |
| **Class D — Internal / non-user-visible** | 370+ | Module docstrings, internal log lines, test fixtures |
| **Class E — License / NOTICE (preserve per Apache-2.0)** | 2 | LICENSE, THIRD_PARTY_NOTICES.md |
| **Class F — Internal identifier, rename if feasible** | ~200 | Package name `deeptutor`, module paths, npm scope |
| **Files touched** | ~65 | 1 web file (brand abstraction), 4 fork backend/CLI files, 18 asset READMEs, rest internal |

---

## V1 Must-Address (Class A in user-facing surface)

These are the **only** user-visible surfaces that still show "DeepTutor" and matter for V1. All are in `deeptutor/` or `deeptutor_cli/` — **AriseHub cannot modify them directly** (ADR-001 §4).

| File | Location | Line | What shows "DeepTutor" | Mechanism available |
|---|---|---|---|---|
| `deeptutor/runtime/banner.py` | `render_banner()` | 291 | `title="[bold bright_cyan]DeepTutor[/]"` in Rich Panel | Settings seam? Need to check if `LABELS` dict is overridable via `data/user/settings/` |
| `deeptutor/api/main.py` | `FastAPI(...)` | ~347 | `title="DeepTutor API"` in FastAPI constructor | Settings seam? Hardcoded; no settings override today |
| `deeptutor_cli/main.py` | `typer.Typer(...)` | 15–16 | `name="deeptutor"`, `help="DeepTutor CLI — agent-first interface..."` | Package entry point; would need wrapper script or upstream PR |

**Why V1 cares:** When a user runs the CLI (`deeptutor run chat ...`) or visits `/docs` (FastAPI OpenAPI), they see "DeepTutor" instead of "AriseHub Agent Playground." The web UI (the primary V1 surface) is already correct.

---

## V1.1+ / Deferred

| Category | Files | Why deferred |
|---|---|---|
| Partner channels (Discord, Matrix, etc.) | `deeptutor/partners/channels/*.py` | Not in V1 scope per matrix |
| Immersive reading / Book engine | `deeptutor/book/`, `deeptutor/capabilities/reading/` | V1.1+ feature |
| Math animator (Manim) | `deeptutor/capabilities/math_animator/` | V2 feature |
| Skill hub defaults | `deeptutor/services/skill/hub.py` | P0 phone-home, V1.1 (see upstream-services.md #1) |
| mineru.net default | `deeptutor/api/routers/settings.py` | P2 default-on, V1.1 |
| CLI help texts (subcommands) | `deeptutor_cli/*.py` | Cosmetic; user sees main command first |
| Internal identifiers (`deeptutor` package name) | Throughout | F — rename only if feasible, not V1 |

---

## Findings by File (User-Visible Only)

### `deeptutor/runtime/banner.py` (fork backend, **cannot modify**)
- **L291** [A] `title="[bold bright_cyan]DeepTutor[/]"` — Rich Panel title in CLI banner
- **L248-259** [A] `LABELS` dict contains `"lab": "DeepTutor Lab"` (en/zh)
- **L266-299** Full `render_banner()` function builds the startup banner
- **Called from:** `deeptutor/runtime/launcher.py` (startup), `deeptutor_cli/init_cmd.py` (init wizard)
- **V1 action:** Check if `LABELS` is overridable via runtime settings. If not, this requires upstream PR or wrapper.

### `deeptutor/api/main.py` (fork backend, **cannot modify**)
- **L347** [A] `title="DeepTutor API"` — FastAPI constructor, appears in `/docs` and `/openapi.json`
- **L5** [D] Module docstring: "DeepTutor API"
- **V1 action:** FastAPI `title` is hardcoded. Could be made configurable via `get_system_settings()` or similar. Upstream PR under ADR-001 cherry-pick rules.

### `deeptutor_cli/main.py` (fork CLI, **cannot modify**)
- **L15** [A] `name="deeptutor"` — Typer app name, determines binary name (`deeptutor run ...`)
- **L16** [A] `help="DeepTutor CLI — agent-first interface..."` — Help text
- **L140-150** [A] `serve` command help: "Start the DeepTutor API server"
- **L158-168** [A] `run` command help mentions "DeepTutor" capability names
- **V1 action:** Binary name is a packaging concern. Could ship an `arisehub` wrapper script that invokes `deeptutor` internally. Help texts require upstream PR.

### `web/lib/brand.ts` (AriseHub-owned, **already correct**)
- **L2** `export const BRAND_NAME = "AriseHub Agent Playground";`
- **L3** `export const BRAND_TAGLINE = "${BRAND_NAME} · Agent-Native Learning";`
- **Used in:** `web/app/layout.tsx` (page title), `web/app/(auth)/login|register/page.tsx`, `web/components/sidebar/SidebarShell.tsx`, `web/components/layout/AppShell.tsx`
- **V1 status:** ✅ Done. No action needed.

### `assets/README/README_*.md` (9 languages, **marketing docs, not product UI**)
- Each file has ~70 occurrences of "DeepTutor" in headings, body text
- **Class B** (static asset) but **not user-facing in the product** — these are GitHub/docs assets
- **V1 action:** Optional — update when AriseHub publishes its own docs. Not a V1 blocker.

### `deeptutor/runtime/launcher.py` (fork backend)
- **L1053** [A] `console.print(f"[bold cyan]DeepTutor[/] backend started...")` — startup log
- **L740,777** [D] Internal variable names containing "deeptutor"
- **V1 action:** Log line is user-visible during `deeptutor start`. Settings seam possible.

### `deeptutor/services/llm/provider_core/openai_compat_provider.py` (fork backend)
- **L55** [C] `"HTTP-Referer": "https://github.com/HKUDS/DeepTutor"` — OpenRouter header
- **Already in:** [upstream-services.md](./upstream-services.md) finding #3 (P1 attribution leak)

### `deeptutor/services/mcp/oauth.py` (fork backend)
- **L61** [C] `CLIENT_URI = "https://deeptutor.info"` — OAuth client metadata
- **Already in:** [upstream-services.md](./upstream-services.md) finding #4 (P1 attribution leak)

### `deeptutor/tools/web_fetch.py` (fork backend)
- **L41** [C] `DEFAULT_USER_AGENT = "DeepTutor/1.0 (+https://hkuds.dev/deeptutor)"`
- **Already in:** [upstream-services.md](./upstream-services.md) finding #2 (P1 attribution leak)

### `deeptutor/services/skill/hub.py` (fork backend)
- **L68-69** [C] `_CLAWHUB_BASE_URL`, `_EDUHUB_BASE_URL` — skill hub defaults
- **Already in:** [upstream-services.md](./upstream-services.md) finding #1 (P0 active phone-home)

### `deeptutor/services/config/loader.py` (fork backend)
- **L6** [D] Module docstring: "Unified configuration loading for all DeepTutor modules."

### License / NOTICE files (Class E — **MUST PRESERVE**)
- `LICENSE` — Apache-2.0 license text, copyright "The DeepTutor Authors"
- `THIRD_PARTY_NOTICES.md` — Third-party license inventory
- **V1 action:** DO NOT MODIFY. Apache-2.0 §4 requires preserving copyright/license/notice. AriseHub modifications should add their own copyright notice, not remove upstream's.

---

## Recommended Action Plan for V1

| # | Action | File | Class | Mechanism | Effort |
|---|---|---|---|---|---|
| 1 | **Verify if CLI banner `LABELS` dict is overridable via runtime settings** | `deeptutor/runtime/banner.py` | A | Check if `data/user/settings/` can inject `LABELS`. If yes, ship AriseHub defaults in `arisehub/data/user_settings_seed/`. If no, upstream PR. | 1h investigation |
| 2 | **Make FastAPI `title` configurable via system settings** | `deeptutor/api/main.py` | A | Upstream PR (cherry-pick under ADR-001) to read title from `load_system_settings()["brand_name"]`. Ship AriseHub default in settings seed. | 2h (PR + test) |
| 3 | **Ship an `arisehub` CLI wrapper script** | New: `arisehub/cli/arisehub` | A | Wrapper that invokes `deeptutor` internally but presents as `arisehub run ...`. Adds to PATH via pip entry point. Bypasses binary name issue. | 2h |
| 4 | **Document V1 branding status in README** | `README.md` (top-level) | Meta | Add "Branding" section: web is AriseHub; CLI/API show DeepTutor (upstream) — will be AriseHub in V1.1 after upstream merges. | 15min |
| 5 | **Audit browser surfaces against deployed V1** | (browser audit) | A/B | Use Chrome MCP against running VM to confirm web UI shows zero DeepTutor. Cross-check with this inventory. | 30min |

---

## What Is Preserved (Per Apache-2.0)

| File | Must preserve |
|---|---|
| `LICENSE` | Full Apache-2.0 text, upstream copyright |
| `THIRD_PARTY_NOTICES.md` | Full third-party inventory |
| Any `NOTICE` file in source distributions | All attribution lines |

**AriseHub additions** should add their own copyright notice (e.g. "Copyright 2026 AriseHub Contributors") alongside, not instead of, the upstream notices.

---

## Browser Audit Checklist (Post-Deploy)

When the VM is up and running V1, verify these surfaces show **only** "AriseHub Agent Playground" or the product version:

- [ ] Page title (browser tab) — `web/app/layout.tsx` → `BRAND_NAME`
- [ ] Login page — `web/app/(auth)/login/page.tsx` → `BRAND_NAME`
- [ ] Register page — `web/app/(auth)/register/page.tsx` → `BRAND_NAME`
- [ ] Sidebar logo/aria-label — `web/components/sidebar/SidebarShell.tsx` → `BRAND_NAME`
- [ ] App shell header — `web/components/layout/AppShell.tsx` → `BRAND_NAME`
- [ ] Favicon / apple-touch-icon — generic PNGs (no text)
- [ ] OpenAPI `/docs` — **currently shows "DeepTutor API"** (action #2 above)
- [ ] CLI banner on `arisehub run ...` — **currently shows "DeepTutor"** (actions #1, #3)
- [ ] CLI help text `arisehub --help` — **currently shows "deeptutor"** (action #3)
- [ ] Error pages (500, 404) — check for any DeepTutor text
- [ ] WebSocket connection logs — check for any DeepTutor identifiers

---

## Cross-Reference to Upstream Services

The following Class C items are **external attribution leaks** (phone-home or third-party-log signals) and are fully documented in [upstream-services.md](./upstream-services.md):

| Finding | upstream-services.md # | Severity |
|---|---|---|
| Skill hub defaults → `clawhub.ai`, `eduhub.deeptutor.info` | #1 | P0 |
| `web_fetch` User-Agent | #2 | P1 |
| OpenRouter `HTTP-Referer` | #3 | P1 |
| MCP OAuth `CLIENT_URI` | #4 | P1 |

These are **not** branding changes — they are upstream service dependencies. The branding inventory only covers what a *user sees in the product UI*.

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-06 | Initial inventory. Web is already rebranded; 3 fork-owned V1 gaps identified. |

---

**End of DeepTutor Branding Inventory.**