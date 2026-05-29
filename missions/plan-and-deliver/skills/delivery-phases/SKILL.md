---
name: delivery-phases
description: >-
  Decompose a target Master Plan or Phase plan into delivery phases (mode #2
  of Sedea's New Feature Development Process). Verifies template stage, loads
  development-process § 2 + § 6/§ 5 contents rule, gates Delivery phases vs PR
  breakdown when the dual-title body is _TBD_, then drafts the parent's dual-title
  section as Delivery phases with a numbered child list. Child stubs and Plan:
  links follow **new-plan** indexed spawn; bodies follow **phase-planner**. Target
  resolved per planning-target-resolution. Use under mission dispatch, **delivery-phases**
  protocol branch, or natural language (decompose phases, draft delivery phases).
inputs:
  targetPlanPath:
    type: string
    description: Absolute or workspace-relative path to the Master Plan or Phase plan being decomposed.
    required: true
  targetPlanSlug:
    type: string
    description: Slug for the target plan.
    required: true
  parentAgentRole:
    type: string
    description: Upstream owner that spawned this skill, usually master-plan-agent.
    required: false
  ledgerParent:
    type: string
    description: Slug/path of the ledger parent entry the Squad Leader tracks.
    required: false
  complexityBand:
    type: string
    description: Plan-scope complexity band copied from the upstream plan, when available.
    required: false
  complexityScore:
    type: number
    description: Plan-scope complexity score copied from the upstream plan, when available.
    required: false
  decompositionAssessment:
    type: string
    description: Current Decomposition assessment block from the upstream plan.
    required: false
  routeLock:
    type: string
    description: Optional upstream-selected route. When set to delivery-phases, skip the decision gate.
    required: false
warmUpRules:
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/plan.mdc"
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/skills/README.md"
  - ".sedea/centers/research-and-development/docs/development-process.md"
  - ".sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc"
---

# Delivery phases — mode #2 decomposition

This skill drives **mode #2** (Delivery phases) under Sedea's New Feature Development Process. **Input:** a target **Master Plan** or **Phase plan** whose dual-title section (`Delivery phases | PR breakdown`) is still undecided or is already committed to **`Delivery phases`**. **Output:** that section drafted as a numbered list of child phases; each row is later expanded into its own phase plan via the **`new-plan`** protocol branch (indexed child), then the **`phase-planner`** protocol branch on the new child.

The procedure below is a hard contract — do **not** skip steps, re-order them, or start drafting before stage is verified.

## Trigger

- Mission dispatch or explicit request to run the **`delivery-phases`** protocol branch.
- Natural language: decompose phases, draft delivery phases, phase decomposition.
- After **`planner`** when the developer has already chosen **`Delivery phases`** over **`PR breakdown`** for § 6 — **`planner`** spawns this skill; this skill drafts § 6 and owns indexed phase-child spawning for that branch.

The **developer** picks the next move per **30_planning-target-resolution** § *Sedea input channel*.

## Step 1 — Identify the target plan and verify stage

The skill operates on a **target** `.plan.md` resolved before this skill runs, per [`30_planning-target-resolution.mdc`](../../../../rules/30_planning-target-resolution.mdc) § *Resolution order*. Acknowledge the target slug in one line when this skill starts (e.g. *Target plan: `<slug>` (from prior structured choice).*). Resolve targets from session, snapshot, or explicit path — **planning-target-resolution** is normative. Do **not** infer the target from the IDE’s focused-file list alone.

If there is no resolved target, **stop** and emit a fresh *Where we are now in the plan tree* snapshot (recap). Collect the lane pick via **AskQuestion**, **`MC_PHASED_RESPONSE_V1`**, or **`MC_ASKQUESTION_V1`** per **30_planning-target-resolution** § *Sedea input channel* and **`../README.md`** § *Recap, structured choice, act* — **preferred:** recap + modal in one message; **legacy split:** recap only, then structured choice in the **next** assistant message. Then continue.

Acknowledge in one line: *"Target plan: `<slug>`."*

**Verify the stage** from the plan body and frontmatter (`kind:`), and the sidecar when it helps disambiguate. The target must be a **Master Plan** or **Phase plan**:

- **`kind: roadmap_topic`** or the file is clearly a **roadmap topic** (top-level grouping of Master Plans) → **stop** with: *"This is a roadmap topic. Roadmap topics do not decompose into delivery phases here. Open a child Master Plan under this topic and run **`delivery-phases`** on that plan."*
- Body has **`## Single concern`** (PR plan template) → **stop** with: *"This is a PR plan. PR plans are leaves; they are not decomposed with **`delivery-phases`**. Use **`coding-session`** or **`pr-review`** as appropriate."*
- Master Plan (`## 4. Architectural design` + dual-title `## 6. …`) or Phase plan (`## 1. Background` … `## 5. …` dual-title) → proceed.
- Ambiguous (stub with no distinguishing sections yet) → use **AskQuestion** or **`MC_ASKQUESTION_V1`** with one `option` per stage (Master Plan, Phase plan, PR plan); if not Master or Phase plan, **stop**.

Acknowledge: *"Stage: <Master Plan | Phase plan>; proceeding."*

## Step 2 — Load the development-process doc

Read `.sedea/centers/research-and-development/docs/development-process.md` with the Read tool, **no offset, no limit** (hosting repo root). Acknowledge in one sentence: *"Loaded development-process.md; will follow § 2 Delivery phases + § 6/§ 5 contents rule."*

This is a **standards document**, not an executable plan — its sections describe the process you apply. Re-read on every invocation; do not rely on session memory.

## Step 3 — Read the target plan and locate the dual-title section

Read the target plan in full. Locate the dual-title section — the last numbered section before optional Caveats:

- **Master Plan:** `## 6. Delivery phases | PR breakdown` (or an already-decided heading).
- **Phase plan:** `## 5. Delivery phases | PR breakdown` (or an already-decided heading).

Inspect the section and apply:

| Section state | Meaning | Action |
| --- | --- | --- |
| Heading is `Delivery phases \| PR breakdown` and body is `_TBD_` | Decision pending | Step 4 (decision gate) → Step 5 (draft) |
| Heading is already `Delivery phases` with empty / `_TBD_` body | Decision made, drafting needed | Skip step 4; go to step 5 |
| Heading is already `Delivery phases` with populated body | Already drafted | Step 6 (handoff / iteration menu) |
| Heading is already `PR breakdown` | Wrong skill | **Stop:** *"This plan’s decomposition is **`PR breakdown`**. Use the **`pr-breakdown`** protocol branch on this plan to draft the PR list."* |

Acknowledge the state in one line.

## Step 4 — Decision gate (when section is `_TBD_`)

When the skill was spawned with `routeLock: "delivery-phases"` (or with `parentAgentRole: "master-plan-agent"` after the developer chose **Delivery phases**), the decision is already made upstream. Acknowledge *"Route locked: Delivery phases."* and skip directly to Step 5. Do not ask the developer to choose `Delivery phases` vs `PR breakdown` again.

When no upstream route lock exists, use **AskQuestion** or **`MC_ASKQUESTION_V1`** to ask:

> How does this plan decompose? Most features use a phase layer; small work (on the order of a few PRs) can skip the phase layer and break directly into PRs.

**Options:**

- **Delivery phases** (`id: delivery_phases`) — multi-step decomposition; each child becomes a standalone phase plan.
- **PR breakdown** (`id: pr_breakdown`) — small enough to skip the phase layer; decompose directly into PRs.

If the developer picks **`pr_breakdown`**, **stop** with: *"Use the **`pr-breakdown`** protocol branch on this plan — it sets the heading to **`PR breakdown`** and drafts the set-level PR list per the doc."* Do not draft anything in this skill; do not change the heading here.

If the developer picks **`delivery_phases`**, continue to step 5.

## Step 5 — Draft the Delivery phases numbered list

### 5a — Infer phase boundaries from the parent plan

Read the target plan’s earlier sections:

- **Master Plan:** § 4 Architectural design + § 5 Changes.
- **Phase plan:** § 2 Scope + § 3 Code design + § 4 Changes.

Pick phase boundaries that respect Strategy #6 (single concern per deliverable) and Strategy #4 (small chunks, fast to production) from **development-process.md**:

- A phase is a coherent slice of the parent’s scope to ship or defer together.
- Order phases when sequencing matters (migration before write path, schema before consumers); otherwise prefer delivery-priority (most-blocking value first).
- Avoid mega-phases (roughly more than ~5 PRs of work); split or add sub-phase recursion per mode #2.
- Prefer roughly 2–5 phases. Fewer than 2 often means **`pr-breakdown`** is enough; more than 5 often means a missing decomposition axis.

### 5b — Draft each numbered item per the § 6/§ 5 contents rule

The dev-process **§ 6 / § 5 contents rule** defines the shape. Each numbered item has three sub-bullets:

1. **Decomposition decision** — `Delivery phases` (child decomposes further) or `PR breakdown` (child is PR-ready). Pick the likely value; the developer can correct on iteration.
2. **Scope sentence** — one terse line (proto–§ 2 Scope of the future child phase plan).
3. **Plan link** — a **`Plan:`** line whose placeholder **matches the shape already used in this parent file** when present (see [`new-plan/SKILL.md`](../new-plan/SKILL.md) § *Indexed child spawn* — often `_TBD` with a short hint). If the parent template has no prior shape, use a single `_TBD` line that states the child file is pending after **`new-plan`** indexed spawn for this list item **N**. The relative Markdown link is filled when **`new-plan`** creates the child and updates the parent; **`plan-reconcile`** can repair wiring.

Optional: one short intro paragraph under the heading before the list when the decomposition needs framing; skip when the list is self-explanatory.

### 5c — Write to the parent plan

Use `StrReplace` to mutate **only** the dual-title section:

- Replace the heading `## <N>. Delivery phases | PR breakdown` → `## <N>. Delivery phases` (`<N>` is **6** for Master Plan, **5** for Phase plan).
- Replace body `_TBD_` with the optional intro + numbered list.

**Bold** the phase name on each item’s first line — the **`new-plan`** protocol branch (indexed spawn) derives the child display name from that bold text (see **`new-plan`**). Keep names short (about 2–5 words).

Do **not** modify other sections in the same call. Do **not** add extra `## <N>.` H2 phase headings elsewhere in the parent; the numbered list under **`Delivery phases`** is the primary anchor for indexed spawn.

After writing, read the file back and confirm the section reads as intended.

### 5d — Notify draft (recap)

**Structured choice delivery** per **`.sedea/centers/sedea/rules/2_ask-question-instructions.mdc`** § **Context and structured choice**. Do **not** use implementation labels like “Turn A/B” in developer-facing chat.

After step **5c**, finish the **recap-only** pass with **only**:

1. A **`file://`** link to the target `.plan.md` under `.sedea/operations/.../plans/...`.
2. One line: *Drafted `## <N>. Delivery phases` with **K** child rows — open the plan to review the full section.*

Do **not** mirror the full **`Delivery phases`** body in chat. Count **K** from numbered rows before the approval modal.

Do **not** include **AskQuestion**, **`MC_ASKQUESTION_V1`**, **`AGENT_RESULT_RESPONSE_V1`**, or **`AGENT_RUN_REQUEST_V1`** in this recap-only pass unless you combine recap + approval into one message via **`MC_PHASED_RESPONSE_V1`** (then skip a separate step-5d-only message).

## Step 6 — Hand back with next-move options

**Structured choice** then **act after the developer selects** — see **`../README.md`** § *Recap, structured choice, act (plan-and-deliver)*.

### Structured choice — Approval (interactive)

**Preferred:** **AskQuestion tool** (brief recap allowed in the same message) or **`MC_PHASED_RESPONSE_V1`** with recap in `display.markdown` and options in `askQuestion` — one assistant message.

**Legacy split (when the tool and phased envelope are unavailable):** send the step **5d** recap, then a **separate** message with **AskQuestion** or **sentinel-only** **`MC_ASKQUESTION_V1`** (no prose before the sentinel).

Collect the developer’s choice via **AskQuestion**, **`MC_PHASED_RESPONSE_V1`**, or **`MC_ASKQUESTION_V1`** only in the structured-choice message — not in the same message as spawns or **`AGENT_RESULT_RESPONSE_V1`**.

- When using bare **`MC_ASKQUESTION_V1`** (no phased envelope), the structured-choice message must contain **only** the sentinel line and JSON object — **no** prose, plan recap, or markdown fences before or between the sentinel and JSON.
- Put every choosable path in **`options`** (`id` / `label`). Do **not** duplicate choices as a numbered prose menu in the same turn.

Required **`options`** (adapt labels; keep **K** visible in the **`prompt`** when helpful):

| Option id (illustrative) | Label (brief) |
| --- | --- |
| `approve-spawn` | Approve phase list and spawn children |
| `revise` | Revise phase list first |
| `defer` | Defer child plan creation |
| `abandon` | Abandon this branch |
| `more-details` | More details for option _ |

**Spawned under `planner`:** Structured-choice approval is mandatory before indexed child spawns. Do **not** emit **`AGENT_RESULT_RESPONSE_V1`** in the structured-choice message.

**Standalone:** After structured-choice approval, **stop** and wait for the developer’s next message.

### Act after developer selects

In a **new** assistant turn after the developer selects an option in the approval modal:

| Choice | Action |
| --- | --- |
| **Approve phase list and spawn children** | Emit one **`AGENT_RUN_REQUEST_V1`** per phase row **1…K** for `.sedea/centers/research-and-development/missions/plan-and-deliver/skills/new-plan/SKILL.md`. Record each spawned child in the ledger. Announce waiting for **K** results. Then emit **`AGENT_RESULT_RESPONSE_V1`** with `continuationStatus: "active"` — **not** in the recap-only pass or structured-choice message. |
| **Revise phase list first** | Apply one focused `StrReplace` on the list, then repeat recap → structured choice. |
| **Defer / abandon** | Emit terminal result per labels; do not spawn. |
| **More details for option _** | Elaborate (information-only), then run structured choice again. |

When running as a spawned downstream agent under `planner`, each **`AGENT_RUN_REQUEST_V1`** in the act-after-select message must include `mode: "indexed-child"`, `parentPlanPath`, `parentPlanSlug`, `index`, `childKind: "phase-planner"`, `requestedPopulatorSkill: "phase-planner"`, `ledgerParent`, `upstreamSkill: "delivery-phases"`, and `decompositionKind: "delivery-phases"`. Record each spawned child in the ledger; announce waiting for **K** results.

If **K = 0**, treat that as a drafting failure: do not open structured-choice spawn paths; return failure or partial.

For standalone/non-spawned use, re-offer recap → structured choice after iteration.

## Step 6a — Follow-up turns

When the developer asks to revise the **`Delivery phases`** list, re-read that section, apply edits via `StrReplace`, echo the result, and return to the step 6 menu pattern.

When the developer chooses to spawn or populate a child in standalone use, emit child-spawn requests for **`new-plan`** / **`phase-planner`** instead of impersonating those skills’ full procedures in the same turn. Stop after spawning if the result is needed for the next step.

## Step 6b — Aggregate indexed child results

When Mission Control delivers a child result from a spawned **`new-plan`** lane:

1. Match it to the ledger entry by correlation id first, then by `outputs.parentPlanSlug` + `outputs.parentIndex`.
2. If the result reports a created child plan (`outputs.planPath` / `outputs.planSlug`), add it to `spawnedPlans` and mark that row `created`.
3. If the result reports an active populator lane (`phase-planner`), keep the row open and add the populator lane to `activeLanes`.
4. If the result reports terminal completion with no remaining tasks, close that row as `completed`.
5. If the result is `partial`, keep the row open and copy its `remainingTasks`.
6. If the result is `failure`, `aborted`, or `abandoned`, mark the row blocked and ask the developer whether to retry that row, defer it, accept partial resolution, or abandon the branch.

Only return `continuationStatus: "terminal"` when every row is explicitly `completed`, `deferred`, `abandoned`, or `out_of_scope`, and no active populator lanes remain. Silence or a missing row is not completion.

## One primary choice per turn — surface observations

Match the discipline in **`planner`** and **`phase-planner`**: perform exactly what was chosen; scope stays on the chosen pass. If you notice gaps (diagram vs phase boundary, duplicate wording, phase count vs assessment), list short **numbered observations** in the chat reply (information-only); the developer addresses them on the next turn or folds them into a revise pass. When you need an explicit accept/skip decision on flags, use **AskQuestion** or **`MC_ASKQUESTION_V1`** with one `option` per flag plus **More details for option _**.

## Scope guard

**Owns:** the parent plan’s dual-title **`Delivery phases`** section only (heading + list body for mode #2); decision gate when still `_TBD_`; echo for review.

**Out of scope:** renaming child plans after **`new-plan`** creates them; filling phase bodies inline (**`phase-planner`** owns the body); PR breakdown content (**`pr-breakdown`**); edits outside the dual-title section; extra H2 phase headings in the parent; `git` / commit automation; roadmap topics and PR plans (step 1 stops).

## Completion (spawned)

### Host protocol line (required)

Emit **exactly one** line on its own: `AGENT_RESULT_RESPONSE_V1` immediately followed by a single JSON object on the **same** line. Required keys: `version` (1), `correlationId` (from the spawn request), `status`, `summary`, `outputs`, `errors` (use `[]` when none). Populate `outputs` from the list below. The emitted line must be **valid JSON** (no `{...}` placeholders in the actual output). Re-emit an **updated** line after user-requested follow-up on this lane (same `correlationId`). See **`.sedea/centers/sedea/skills/README.md`** § *Spawned terminal line*.

Required `outputs` fields:

- `outputs.targetPlanPath`, `outputs.targetPlanSlug`
- `outputs.decompositionKind`: `"delivery-phases"`
- `outputs.childCount`, `outputs.developerApprovalStatus`
- `outputs.childRows` — `{index, title, status, planPath?, planSlug?, correlationId?, remainingTasks?}`
- `outputs.spawnedPlans`, `outputs.activeLanes`, `outputs.openLedgerEntries`, `outputs.remainingTasks`
- `outputs.continuationOwner`: `"delivery-phases-agent"`
- `outputs.continuationStatus` — `active` while approval, child creation, or population remains; `terminal` when all child rows are `completed`, `deferred`, `abandoned`, or `out_of_scope` and no active populator lanes remain

Complete the step 6 handoff block (or announce spawn wait) **before** the terminal line. Stop after the terminal line. Do not emit another `AGENT_RUN_REQUEST_V1` or run the next protocol step in the same turn (see **`../README.md`** § *Terminal stop (normative)*).

## Completion (inline)

Report the fields below in prose to the invoker on the **same lane**. Do **not** emit `AGENT_RUN_REQUEST_V1`, `AGENT_RESULT_RESPONSE_V1`, or `MC_DISPATCH_RESOLVED_V1`. Do **not** add a **Host protocol line** under this section (see **`.sedea/centers/sedea/rules/4_mission.mdc`** § *Inline completion* and **`.sedea/centers/sedea/skills/README.md`** § *Completion (inline)*).

Spawned from the **Master Plan agent** or **plan and deliver** decomposition paths in normal flow. If run inline, use the same `outputs` semantics as **`## Completion (spawned)`** in prose only.
