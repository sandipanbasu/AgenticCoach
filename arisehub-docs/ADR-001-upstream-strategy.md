# ADR-001: Upstream Strategy for the DeepTutor Fork

| Field | Value |
|---|---|
| **Status** | Accepted (provisional — review on 2026-10-03) |
| **Date** | 2026-09-05 |
| **Deciders** | Sandipan Basu (AriseHub) |
| **Reviewers** | — |
| **Supersedes** | — |
| **Superseded by** | — |
| **Related** | [Architecture Baseline](./AriseHub_Agent_Playground_Architecture.md) · [V1 Flows and Scope](./v1-flows-and-scope.md) · [Feature Version Matrix](./feature-version-matrix.md) · [Upstream Services Inventory](./upstream-services.md) |

---

## Context

AgenticCoach / AriseHub Agent Playground is a fork of [DeepTutor](https://github.com/sandipanbasu/AgenticCoach), an Apache-2.0 open-source intelligent learning companion. The fork began in August 2026 with branding and deployment customizations layered on top of DeepTutor v1.5.16.

DeepTutor ships releases on a roughly bi-weekly cadence (v1.5.16, v1.6.3, v1.6.4 observed in the recent history). The AriseHub product surface (Playground UX, Agent Library, Run-as-first-class, evaluation) is a substantial addition that lives *on top of* — not *instead of* — the inherited runtime. We must decide how to keep the fork current with DeepTutor without losing our product-layer work, and how to keep AriseHub code mergeable across upstream releases.

### Forces in tension

- **Currency:** We want security patches, model-provider fixes, parser bug fixes, and runtime improvements from upstream.
- **Stability:** We do not want every DeepTutor commit to land in our `main`; continuous upstream churn would destabilize the AriseHub product work.
- **Cost:** Any sync mechanism has an operational cost (merge work, conflict resolution, test cycles).
- **Auditability:** Apache-2.0 obligations (attribution, NOTICE) must be preserved. The fork's git history must clearly show what is DeepTutor-inherited vs. AriseHub-original.
- **Velocity:** AriseHub's product surface is being built at the same time as upstream keeps moving. We need a model that lets both proceed without stepping on each other.

### Options considered

| Option | Mechanism | Cost | Outcome |
|---|---|---|---|
| **A. Cherry-pick only** | Never take a full upstream release; cherry-pick individual bug-fix commits. | Low per-commit, but accumulates debt. Becomes unmaintainable in 3–6 months. | Rejected. |
| **B. Continuous rebase** | Rebase AriseHub customizations on every upstream commit. | High, ongoing. Rewrites AriseHub commit hashes; bisect breaks; merge conflicts are constant because AriseHub code lives inside inherited modules. | Rejected. |
| **C. Merge-on-release, cherry-pick between releases** | Merge each DeepTutor release as a discrete event; cherry-pick only for urgent fixes between releases. | Medium, predictable. Concentrated in merge windows. | **Selected.** |

---

## Decision

**AriseHub Agent Playground adopts a "merge-on-release, cherry-pick between releases, never rebase" upstream strategy, with AriseHub-owned code structurally isolated under a top-level `arisehub/` directory.**

### 1. Merge-on-release

For each DeepTutor release (v1.6.4, v1.6.5, v1.7.0, ...):

1. Add (or refresh) the `upstream` remote pointing at the DeepTutor source repository.
2. Fetch the release tag.
3. Create or fast-forward the integration branch `vendor/upstream-vX.Y.Z`.
4. Merge the upstream tag into that branch. Resolve conflicts; AriseHub-owned files (under `arisehub/`) should not appear in conflicts because they do not exist upstream.
5. Run the full test suite. Add a release-merge smoke test if missing.
6. Fast-forward `main` to the integration branch tip with a merge commit (not rebase).
7. Tag the merge commit: `vendor-upstream-vX.Y.Z-merged-<date>`.
8. Record the merge in `arisehub-docs/upstream-merge-log.md` (one row per merge) with: upstream version, merge SHA, conflict list (should be empty for AriseHub files), test status, and any behavioral changes that affect AriseHub flows.

### 2. Cherry-pick between releases

Between releases, cherry-pick from upstream is permitted **only** for:

- Security fixes (CVEs, prompt-injection mitigations, dependency vulnerabilities).
- Model-provider regressions (e.g. a provider changes its API and DeepTutor ships a fix in `main` before the next release).
- Parser or runtime crashes that affect AriseHub users.

Each cherry-pick commit must:

- Use the prefix `cherry-pick:` in the commit subject.
- Reference the upstream SHA in the commit body: `(cherry-picked from <upstream-sha>)`.
- Be reviewed as a focused change (not bundled with AriseHub feature work).

Cherry-picks that don't fit the above categories are deferred to the next release merge.

### 3. Never rebase

Full-history rebase against upstream is **forbidden**. Force-pushes to any AriseHub-owned branch are forbidden. The reasons are recorded in §"Consequences" below; the operational rule is: **rebase is not a tool we use.**

### 4. AriseHub code isolation

All AriseHub-owned code lives under a top-level `arisehub/` directory:

```text
AgenticCoach/
├── deeptutor/          # Inherited from DeepTutor. Do not modify from AriseHub PRs.
├── deeptutor_cli/      # Inherited.
├── deeptutor_web/      # Inherited.
├── web/                # Inherited Next.js frontend.
│   └── app/
│       └── (arisehub)/ # AriseHub's Next.js route group. The only web/ subdir AriseHub owns.
├── arisehub/           # NEW. AriseHub product layer.
│   ├── runtime/        # Event adapter, run store, agent template resolver.
│   ├── api/            # AriseHub-specific FastAPI routes (sits next to fork's api/).
│   ├── web/            # AriseHub frontend components used from (arisehub) route group.
│   └── docs/           # Architecture, ADRs, capability audit, merge log.
├── arisehub-docs/      # Canonical product/architecture docs (legacy location; see arisehub/docs note).
├── docs/               # Operational runbooks (dev setup, deploy, contribute).
├── tests/              # Inherited + AriseHub-specific (under tests/arisehub/ subdir).
└── ...
```

**Rules:**

- Files under `deeptutor/`, `deeptutor_cli/`, `deeptutor_web/` are **read-only from AriseHub's perspective**. The fork's runtime is the upstream contract; if a runtime change is required, the change is either made via a fork-side adapter (in `arisehub/`) or proposed upstream and merged in.
- Files under `web/` other than `web/app/(arisehub)/` are also off-limits to AriseHub PRs.
- The `arisehub/` directory is structurally invisible to upstream. When upstream merges happen, the diff is contained to the fork's own directories, and AriseHub code never appears in the conflict set.
- AriseHub web components live in `arisehub/web/` and are imported by routes under `web/app/(arisehub)/...`. This is a single integration point: the only file under `web/` that AriseHub owns is the one that mounts the route group.

### 5. Commit convention

To make attribution unambiguous in mixed git history:

- AriseHub commits live under `arisehub/`. The directory is the attribution marker.
- If an AriseHub commit must touch a non-`arisehub/` path (rare exceptions only), the commit subject is prefixed with `arisehub(cross-cutting):` and the body explains why.
- Cherry-picked upstream commits use the `cherry-pick:` prefix (see §2).
- Upstream merge commits are tagged with the `vendor-upstream-vX.Y.Z-merged-<date>` tag (see §1).

### 6. Review cadence

This ADR is **provisional**. It will be reviewed on **2026-10-03** (4 weeks from adoption) with the following data points:

- How many upstream releases were merged in the period?
- How many cherry-picks were needed?
- How many conflicts occurred in `deeptutor/`, `deeptutor_cli/`, `deeptutor_web/`, or non-`(arisehub)` parts of `web/`?
- How many times did an AriseHub PR have to touch a non-`arisehub/` path?
- Cost in engineering hours per merge.

If the data shows the strategy is unsustainable, the strategy will be revised (most likely toward more aggressive cherry-picking and longer release-sync intervals). If the data shows the strategy is working, the "provisional" status is dropped.

---

## Consequences

### Positive

- **Predictable merge cost.** All upstream work is concentrated in one merge event per DeepTutor release, not scattered across arbitrary cherry-picks.
- **Stable AriseHub history.** AriseHub commits are never rewritten. Bisect, blame, and PR links keep working across the fork's lifetime.
- **Conflict-free AriseHub code.** The `arisehub/` directory is structurally invisible to upstream. Upstream merge conflicts are limited to the inherited directories, which AriseHub doesn't touch.
- **Clear attribution.** Anyone reading the git log can immediately tell DeepTutor-inherited code from AriseHub-original code by directory and commit prefix.
- **Apache-2.0 compliance preserved.** No rewriting, no obfuscation, no lost attribution. LICENSE and THIRD_PARTY_NOTICES.md remain accurate.
- **AriseHub can ship independently of upstream cadence.** A DeepTutor release that breaks an AriseHub flow can be partially applied, with the broken parts reverted, without losing AriseHub work.

### Negative

- **Technical debt accumulates against upstream.** We will drift from DeepTutor's mainline. Features, refactors, and architectural improvements in upstream will not flow to us automatically. We accept this in exchange for a mergeable fork.
- **Some upstream improvements require deliberate integration.** A performance improvement in DeepTutor's RAG pipeline, for example, is not auto-ported; we have to identify it, decide it's worth taking, and apply it (either as part of the next release merge or as a focused cherry-pick).
- **First merge is the highest-risk.** The v1.5.16 → v1.6.4 merge is the first sync event under this strategy. Conflicts are possible. The merge should be done carefully, with full test coverage, before any further AriseHub work that touches the inherited directories.
- **Two upgrade windows per quarter, minimum.** If DeepTutor ships a release every 2 weeks, that's ~6 merges per quarter. The discipline of "merge every release" must hold; if it slips, the next merge becomes harder.

### Neutral

- **The strategy is not symmetric.** We do not flow AriseHub code back upstream. This is a one-way fork. If at some future point we want to contribute back, we will need a separate ADR.

---

## Implementation notes

### First sync: v1.5.16 → v1.6.4

1. Add the upstream remote:
   ```bash
   git remote add upstream https://github.com/<upstream-org>/DeepTutor.git
   git fetch upstream --tags
   ```
2. Confirm `v1.6.4` exists upstream and matches the SHA `93df3d48` already in the local object store (observed 2026-09-05).
3. Create the integration branch:
   ```bash
   git checkout -b vendor/upstream-v1.6.4
   git merge v1.6.4
   ```
4. Resolve conflicts. AriseHub-owned files (anything under `arisehub/`, anything added by AriseHub in this fork) should not appear in conflicts because they do not exist upstream. If they do, that's a sign §4 of this ADR was violated and must be remediated before merging.
5. Run the full test suite.
6. Fast-forward `main` to the integration tip.
7. Tag and log per §1.

### What lives in `arisehub/` from day one

This list will grow, but the V1 work — the runtime event adapter, the run store, the agent template resolver, the `/runs` API, the Playground/Library/Run pages — all go under `arisehub/`.

Specifically:

- `arisehub/runtime/run_adapter.py` — wraps `StreamBus` events into AriseHub's normalized `AriseHubRunEvent` schema.
- `arisehub/runtime/run_store.py` — JSONL-backed persistence for runs.
- `arisehub/runtime/agent_templates.py` — resolves starter agent templates (V1: bundled; V1.1: user-clonable).
- `arisehub/api/runs.py` — `/runs` REST API.
- `arisehub/web/playground/` — Playground, Library, Run pages (mounted from `web/app/(arisehub)/playground/`).

### Open questions (for the next ADR)

- **License of AriseHub-original code.** Apache-2.0 inherited code is licensed as Apache-2.0. AriseHub-original code under `arisehub/` could be Apache-2.0 (consistent, low friction) or a more restrictive license (commercial moat). The doc §8 says "prefer permissive OSS" but is silent on AriseHub's own code. Decision needed before any public release of AriseHub-only code.
- **Upstream contribution.** Out of scope for this ADR but should be raised at the 4-week review.
- **When (if ever) to drop the v1.5.16 baseline.** If the v1.6.4 merge succeeds cleanly and the next two releases also merge cleanly, v1.5.16 is effectively retired as a "known artifact." A future ADR can mark the cutover.

---

**End of ADR-001.**
