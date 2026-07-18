---
name: delivery-phases
description: >-
 Decompose a target Master Plan or Phase plan into delivery phases (mode #2
 of Sedea's New Feature Development Process). Verifies template stage, loads
 development-process § 2 + § 6/§ 5 contents rule, gates Delivery phases vs PR
 breakdown when the dual-title body is _TBD_, then drafts the parent's dual-title
 section as Delivery phases with a numbered child list. Child stubs and Plan:
 links follow **new-plan** indexed handoff (inline under **master-planner**; spawned when standalone); bodies follow **phase-planner**. Target
 resolved per planning-target-resolution. Use under mission dispatch, **delivery-phases**
 protocol branch, or natural language (decompose phases, draft delivery phases).
designation:
  allowed: Phase decomposition from parent plan; delivery-phases stubs and §5 assessment
  forbidden: Application implementation; worktree ship; spawn coding-session directly
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
    description: Upstream owner that invoked this skill inline, usually master-plan-agent or phase-planner-agent.
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
laneRules:
  - ".sedea/centers/sedea/rules/2_ask-question-instructions.mdc"
  - ".sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc"
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/skills/delivery-phases/SKILL.md"
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/skills/README.md"
warmUpRules:
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/plan.mdc"
  - ".sedea/centers/research-and-development/missions/plan-and-deliver/skills/README.md"
  - ".sedea/centers/research-and-development/docs/development-process.md"
  - ".sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc"
---

# Delivery phases — mode #2 decomposition

This skill drives **mode #2** (Delivery phases) under Sedea's New Feature Development Process. **Input:** a target **Master Plan** or **Phase plan** whose dual-title section (`Delivery phases | PR breakdown`) is still undecided or is already committed to **`Delivery phases`**. **Output:** that section drafted as a numbered list of child phases; each row is expanded **depth-first** via **`new-plan`** (indexed — **inline** when this skill runs under **`master-planner`**) after ship-complete gates, then **`phase-planner`** (spawned) on the child stub.

The procedure below is a hard contract — do **not** skip steps, re-order them, or start drafting before stage is verified.

## Warm-up manifest (spawned)

Per [`.sedea/centers/sedea/docs/lane-manifest-contract.md`](.sedea/centers/sedea/docs/lane-manifest-contract.md) and **`../README.md`** § *Default warm-up*. Often runs **inline** on invoker lane; manifest applies at spawn and warm-up replay. Host merge: `effectiveWarmUp = dedupe(bootstrapRules → laneRules → skillWarmUp)`. **No `alwaysApply` frontmatter flip.**

### `bootstrapRules` — host-resolved (R&D layer)

| Path | Purpose |
|------|---------|
| `.sedea/centers/research-and-development/rules/bootstrap.mdc` | Sole R&D `alwaysApply: true` bootstrap (≤10 KB); host merges when `centerSlug === research-and-development` |

### `skillWarmUp` — frontmatter `warmUpRules`

| Path | Purpose |
|------|---------|
| `.sedea/centers/research-and-development/missions/plan-and-deliver/plan.mdc` | Squad Leader ledger, spawn/wait |
| `.sedea/centers/research-and-development/missions/plan-and-deliver/skills/README.md` | Spawn contracts, terminal stop |
| `.sedea/centers/research-and-development/docs/development-process.md` | NFD process templates |
| `.sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc` | Target resolution, depth-first gates |

### `laneRules` — frontmatter `laneRules`

| Path | Purpose |
|------|---------|
| `.sedea/centers/sedea/rules/2_ask-question-instructions.mdc` | Structured choice, AskQuestion |
| `.sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc` | Planning target resolution (role minimum) |
| `.sedea/centers/research-and-development/missions/plan-and-deliver/skills/delivery-phases/SKILL.md` | This skill procedure |
| `.sedea/centers/research-and-development/missions/plan-and-deliver/skills/README.md` | Spawn preflight, definitive `laneRules` |

## Agent messaging (MCP)

**MCP spawn/result skill.** Parent→child spawn and child terminal result use MCP tools per **`.sedea/centers/sedea/rules/4_mission.mdc`** § *Agent-to-agent spawn protocol*.

| Action | MCP tool |
|--------|----------|
| Parent spawn (when this skill emits a child lane) | **`mission_control_spawn_agent`** |
| **This** spawned lane terminal (and terminal re-emits) | **`mission_control_send_agent_result`** |

**Binding:**

- Run **`../README.md`** § *MCP spawn preflight* (rows M1–M8) before every MCP spawn; **forbidden** host-resolved identity keys in MCP args (`correlationId`, `dispatchId`, `slotId`, … — see README § *Host-resolved identity*).
- Inline skills on this mission stay **inline-only** — no spawn wire change unless the protocol step explicitly spawns a child lane.

## Checkpoint turn UX (skill-local)

Under Checkpoint trust (`trustLevel: checkpoint`), auto-advance scripted happy-path steps; emit structured choice only at **USER_CHECKPOINT** markers in this section, implicit external-wait surfaces, or exception paths. **No cross-skill inheritance** — gate defaults here apply only to **`delivery-phases`**; other planning skills document their own markers.

**Real-dispatch test loop (binding):** After merge, run one full **`delivery-phases`** spawn on a Checkpoint dispatch through Step **6** and collect a developer verdict before the parent phase advances the next **`delivery-phases`** step PR — per **Planning protocol skills UX** § *Single-concern strategy*.

Marker syntax: [`.sedea/centers/sedea/docs/user-checkpoint-marker-syntax.md`](.sedea/centers/sedea/docs/user-checkpoint-marker-syntax.md).

| Step | Checkpoint behavior | Gate |
|------|---------------------|------|
| **1** — Identify target | Auto-advance on spawned handoff with locked `inputs` | exception: wrong template / missing target |
| **2** — Load development-process | Auto-advance | — |
| **3** — Read target / dual-title section | Auto-advance on happy path | — |
| **4** — Decision gate | Auto-advance when `routeLock: delivery-phases` or upstream route already chosen | **Gate** when dual-title body is `_TBD_` and no route lock |
| **5** — Draft list (**5a–5d**) | Auto-advance through write and step **5d** notify recap | — |
| **6** — Approve phase list | **Gate** when **K > 0** — **first developer-pick gate in this calibration PR** | Approve phase list (below) |
| **6a–6b** | Act-after-select; #external-wait on spawned **`new-plan`** / **`phase-planner`** child lanes | Resume modal before turn end |

### Inline invoker lane (binding)

When **`parentAgentRole`** is **`master-plan-agent`** or **`phase-planner-agent`**, this skill runs **inline on the active planner child lane** with **`targetPlanPath`** = the Master Plan or Phase plan under decomposition. The **write target** and **execution lane** align on that parent file.

**Forbidden:**

- Redirecting the developer to another tab or lane to approve the phase list when this skill was invoked inline — Step **6** structured choice runs **on this lane**.
- Emitting **`mission_control_send_agent_result`** when **`parentAgentRole`** is **`master-plan-agent`** or **`phase-planner-agent`** — report **`## Completion (inline)`** to the invoker instead.

**Required:** Merge **`## Completion (inline)`** into **`master-planner`** Step **7** or **`phase-planner`** Step **5** ledger fields per invoker contracts.

## Trigger

- Mission dispatch or explicit request to run the **`delivery-phases`** protocol branch.
- Natural language: decompose phases, draft delivery phases, phase decomposition.
- After **`master-planner`** when the developer has already chosen **`Delivery phases`** over **`PR breakdown`** for § 6 — **`master-planner`** runs this skill **inline** on the same lane; this skill drafts § 6 and owns indexed phase-child creation (**`new-plan`** inline on that lane).
- After **`phase-planner`** when route is **`delivery-phases`** — **`phase-planner`** runs this skill **inline** on the phase-planner lane; same **`new-plan`** inline handoff per row.

The **developer** picks the next move per **30_planning-target-resolution** § *Sedea input channel*.

### Inline handoff — **delivery-phases** → **`new-plan`** (step 6 act-after-select)

When **`parentAgentRole`** is **`master-plan-agent`** or **`phase-planner-agent`** (this skill inline under **`master-planner`** or **`phase-planner`**), run **`new-plan`** **inline on this lane** for **eligible** row index(es) only — **do not** emit **`mission_control_spawn_agent`** for **`new-plan`**. **Depth-first gate:** expand **at most one** phase row per act-after-select pass — the lowest index **N** whose **`Plan:`** is still `_TBD_` and whose prior phase **N−1** is **ship-complete** per **development-process.md** § *Depth-first plan-tree traversal* and **30_planning-target-resolution** § *Depth-first expansion eligibility*. Load `.sedea/centers/research-and-development/missions/plan-and-deliver/skills/new-plan/SKILL.md`, construct inline context per eligible row from the table below, follow that skill’s steps, and merge each **`## Completion (inline)`** into this skill’s ledger (`childRows`, `spawnedPlans`, `activeLanes`, `openLedgerEntries`, `remainingTasks`). Inline **`new-plan`** may still spawn **`phase-planner`** per its contract.

| Inline context field | Value (per row **N**) |
|----------------------|------------------------|
| `mode` | `"indexed-child"` |
| `parentPlanPath` | Absolute path to this skill’s `targetPlanPath` |
| `parentPlanSlug` | This skill’s `targetPlanSlug` |
| `index` | Row number **N** (single eligible index per expand pass for phases) |
| `childKind` | `"phase-planner"` |
| `requestedPopulatorSkill` | `"phase-planner"` |
| `ledgerParent` | `ledgerParent` from this skill’s inputs when present |
| `upstreamSkill` | `"delivery-phases"` |
| `parentAgentRole` | `"delivery-phases-agent"` |
| `decompositionKind` | `"delivery-phases"` |

**Standalone spawned** path: emit **`mission_control_spawn_agent`** per row instead (see step 6 act-after-select).

## Step 1 — Identify the target plan and verify stage

The skill operates on a **target** `.plan.md` resolved before this skill runs, per [`30_planning-target-resolution.mdc`](../../../../rules/30_planning-target-resolution.mdc) § *Resolution order*. Acknowledge the target slug in one line when this skill starts (e.g. *Target plan: `<slug>` (from prior structured choice).*). Resolve targets from session, snapshot, or explicit path — **planning-target-resolution** is normative. Do **not** infer the target from the IDE’s focused-file list alone.

If there is no resolved target, **stop** and emit a fresh *Where we are now in the plan tree* snapshot with **`AskQuestion`** or **`mission_control_present_structured_choice`** in **one turn** per **30_planning-target-resolution** § *Sedea input channel* and **`../README.md`** § *Recap, structured choice, act* (`displayMarkdown` + `askQuestion`). **Obsolete:** recap-only turn without structured choice. Then continue.

Acknowledge in one line: *"Target plan: `<slug>`."*

**Verify the stage** from the plan body and frontmatter (`kind:`), and the sidecar when it helps disambiguate. The target must be a **Master Plan** or **Phase plan**:

- **`kind: roadmap_topic`** or the file is clearly a **roadmap topic** (top-level grouping of Master Plans) → **stop** with: *"This is a roadmap topic. Roadmap topics do not decompose into delivery phases here. Open a child Master Plan under this topic and run **`delivery-phases`** on that plan."*
- Body has **`## Single concern`** (PR plan template) → **stop** with: *"This is a PR plan. PR plans are leaves; they are not decomposed with **`delivery-phases`**. Use **`coding-session`** or **`pr-review`** as appropriate."*
- Master Plan (`## 4. Architectural design` + dual-title `## 6. …`) or Phase plan (`## 1. Background` … `## 5. …` dual-title) → proceed.
- Ambiguous (stub with no distinguishing sections yet) → use **AskQuestion** with one `option` per stage (Master Plan, Phase plan, PR plan); if not Master or Phase plan, **stop**.

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

### Step 4-open-items — Open-item modal contract

Apply the shared planning open-item contract from `../README.md` to every **delivery-phases** gate that can surface multiple unresolved items: route caveats, phase-boundary observations, phase-count concerns, sequencing concerns, list approval caveats, child-row expansion blockers, and depth-first eligibility blockers.

**When open items exist** — use **one modal with multiple `questions[]` entries**:

- **`displayMarkdown`:** numbered list of open items. For each item, include the target section or phase row, the gap/caveat/blocker, why it matters for phase decomposition or depth-first expansion, and the agent's proposed resolution options.
- **`askQuestion.questions`:** one scoped question per open item, with its own stable `id`, `prompt`, and item-only `options` (for example `accept-phase-boundary`, `split-phase`, `merge-phase`, `revise-sequencing`, `defer-row`, `skip-no-change`, `more-details`). **Forbidden:** one combined question whose options mix decisions for several phase rows or concerns.
- **Final question:** always append the terminal delivery-phases gate question last in the array. Use the normal gate for the current step: decomposition route, **Approve phase list**, expand next eligible phase row, revise, defer, or abandon. **Forbidden:** a resolve-only modal that omits list approval or expansion until every item is cleared.
- **Many open items:** batch across turns when needed; each batch still ends with the terminal delivery-phases gate question as the final `questions[]` entry.

**When no open items remain** — use the existing single terminal gate question for Step **4**, Step **6**, or follow-up expansion.

When the skill was spawned with `routeLock: "delivery-phases"` (or with `parentAgentRole: "master-plan-agent"` or `"phase-planner-agent"` after the developer chose **Delivery phases**), the decision is already made upstream. Acknowledge *"Route locked: Delivery phases."* and skip directly to Step 5. Do not ask the developer to choose `Delivery phases` vs `PR breakdown` again.

When no upstream route lock exists, use **AskQuestion** or **`mission_control_present_structured_choice`** to ask:

USER_CHECKPOINT — choose decomposition route (Delivery phases vs PR breakdown).

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

After step **5c**, present step **6** handoff in **one turn** via **`mission_control_present_structured_choice`** or **AskQuestion tool** — put in **`displayMarkdown`** (or brief prose with the tool):

1. A backtick path to the target `.plan.md` (prefer the hosting-absolute path; a `.sedea/operations/…/plans/…` path is also valid). Do **not** use a `file://` Markdown link or put backticks inside a Markdown link label.
2. One line: *Drafted `## <N>. Delivery phases` with **K** child rows — open the plan to review the full section.*

Do **not** mirror the full **`Delivery phases`** body in chat. Count **K** from numbered rows before the approval modal.

- **Next-step resolution:** Auto-advance to Step **6** structured choice after step **5d** recap — no `USER_CHECKPOINT` on substeps **5a–5d**.

**Obsolete:** separate recap-only pass without **`askQuestion`** — step **6** options belong on the **same** turn as the link + one-line summary.

## Step 6 — Hand back with next-move options

**Structured choice** then **act after the developer selects** — see **`../README.md`** § *Recap, structured choice, act (plan-and-deliver)*.

### Structured choice — Approval (interactive)

**Preferred:** **AskQuestion tool** (brief recap allowed in the same message) or **`mission_control_present_structured_choice`** with recap in `displayMarkdown` and options in `askQuestion` — one assistant message.

**Legacy split (when the tool and MCP structured choice are unavailable):** send the step **5d** recap, then a **separate** message with `mission_control_present_structured_choice`** (MCP structured choice; recap in `displayMarkdown` via MCP call).

Collect the developer’s choice via **AskQuestion**, **`mission_control_present_structured_choice`** only in the structured-choice message — not in the same message as spawns or **`mission_control_send_agent_result`**.

- When using split delivery (no AskQuestion tool), call **`mission_control_present_structured_choice`** with valid `displayMarkdown` + `askQuestion` — **no** prose recap in the same message as the MCP call when split per rule **2** priority **3**.
- Put every choosable path in **`options`** (`id` / `label`). Do **not** duplicate choices as a numbered prose menu in the same turn.

USER_CHECKPOINT — approve drafted Delivery phases list before child expansion.

Required **`options`** (adapt labels; keep **K** visible in the **`prompt`** when helpful):

| Option id (illustrative) | Label (brief) |
| --- | --- |
| `approve-list` | Approve phase list — no spawn yet |
| `expand-next-eligible` | Expand next eligible phase row |
| `revise` | Revise phase list first |
| `defer` | Defer child plan creation |
| `abandon` | Abandon this branch |
| `more-details` | More details for option _ |

- When **K > 0** and step **5d** recap is complete → open this gate via **`mission_control_present_structured_choice`** (spawned lanes) or **AskQuestion** per **`.sedea/centers/sedea/rules/2_ask-question-instructions.mdc`**. Apply **Step 4-open-items** when open items exist — this approval question stays last in `questions[]`.
- When **K = 0** → drafting failure; do **not** open this gate.
- **`defaultOptionId: approve-list`** when **K > 0** and no blocking open items remain.

When approval or expansion has open items (phase-count concerns, phase-boundary observations, sequencing caveats, row-specific blockers, or depth-first eligibility blockers), apply **Step 4-open-items**: put one scoped `questions[]` entry per item before this approval/expansion question, and keep this approval/expansion question last in the array.

**Inline under `master-planner` or `phase-planner`:** Structured-choice approval is mandatory before indexed **`new-plan`** handoff. Do **not** emit **`mission_control_send_agent_result`** for this skill when **`parentAgentRole`** is **`master-plan-agent`** or **`phase-planner-agent`** — report **`## Completion (inline)`** to the invoker instead. Run **`new-plan`** **inline** on this lane (no child lanes for **`new-plan`**); **`phase-planner`** child lanes may still open from inline **`new-plan`**.

**Standalone (spawned):** After structured-choice approval, emit **`mission_control_send_agent_result`** with `continuationStatus: "active"` when spawning **`new-plan`** child lanes — **not** in the structured-choice message. On #external-wait while **`new-plan`** or **`phase-planner`** child lanes are in flight (step **6b**), close the turn with structured choice per [`.sedea/centers/sedea/rules/2_ask-question-instructions.mdc`](.sedea/centers/sedea/rules/2_ask-question-instructions.mdc) § **External-wait / next-step modal** — resume paths in **`options`**; do not prose-only announce wait.

### Act after developer selects

In a **new** assistant turn after the developer selects an option in the approval modal:

| Choice | Action |
| --- | --- |
| **Approve phase list — no spawn yet** (`approve-list`) | Record `developerApprovalStatus: "list-approved"`; set each child row to `listed` with `_TBD_` **`Plan:`** links unchanged. Do **not** run **`new-plan`**. Offer **`expand-next-eligible`** on a later turn when the developer is ready (phase **1** is eligible immediately after list approval). |
| **Expand next eligible phase row** (`expand-next-eligible`) | Resolve the lowest index **N** with pending **`Plan:`** whose prior phase is **ship-complete** (index **1** has no prior). **Inline:** run **`new-plan`** **once** for that **N** per [Inline handoff](#inline-handoff--delivery-phases--new-plan-step-6-act-after-select); merge **`## Completion (inline)`**; record **`phase-planner`** spawns in `activeLanes`. **Standalone spawned:** emit one **`mission_control_spawn_agent`** for that index only. If no row is eligible, stop with a one-line reason (prior phase ship incomplete) — do not spawn. |
| **Revise phase list first** | Apply one focused `StrReplace` on the list, then repeat recap → structured choice. |
| **Defer / abandon** | Emit terminal result per labels; do not spawn. |
| **More details for option _** | Elaborate in **`displayMarkdown`** (or brief prose), then **`askQuestion`** again on the **same** turn — no prose-only elaboration handoff. |

When running **standalone spawned** (not inline under **`master-planner`**), each **`mission_control_spawn_agent`** in the expand pass must include the indexed-child inputs listed above for **one** eligible index. Record the spawned child in the ledger; on #external-wait for that child result, close the turn with structured choice per rule **2** § **External-wait / next-step modal** — not prose-only idle.

If **K = 0**, treat that as a drafting failure: do not open structured-choice handoff paths; return failure or partial.

For standalone/non-spawned use, re-offer recap → structured choice after iteration.

## Step 6a — Follow-up turns

When the developer asks to revise the **`Delivery phases`** list, re-read that section, apply edits via `StrReplace`, echo the result, and return to the step 6 menu pattern.

When the developer chooses to hand off or populate a child in standalone use, run **`new-plan`** inline or emit child-spawn requests for **`new-plan`** / **`phase-planner`** instead of impersonating those skills’ full procedures in the same turn. When the handoff ends the assistant turn on #external-wait for a child result, close with structured choice per rule **2** § **External-wait / next-step modal** — do not prose-only stop after handoff.

## Step 6b — Aggregate indexed child results

**Inline `new-plan` under `master-planner` or `phase-planner`:** After each inline **`new-plan`** row completes, merge its **`## Completion (inline)`** into `childRows` and `spawnedPlans`. If inline **`new-plan`** reports an active **`phase-planner`** lane, keep the row open and add it to `activeLanes`. When Mission Control delivers a **`phase-planner`** child result, match by correlation id, then `outputs.parentPlanSlug` + `outputs.parentIndex`.

**Phase-planner child active (inline under `master-plan-agent`):** When **`activeLanes`** includes an open **`phase-planner`** for row **N**, report **`## Completion (inline)`** to **`master-planner`** as **acknowledge only** — one status line naming the phase slug and child lane. Do **not** offer **`expand-next-eligible`** for row **N+1** or phase-scoped decomposition menus on the **Master Plan** lane until **`outputs.phaseShipComplete: true`** for row **N** or the developer explicitly defers/abandons that phase on the **phase-planner** lane (**`phase-planner`** § *Phase delivery ownership*).

**Standalone spawned `new-plan`:** When Mission Control delivers a child result from a spawned **`new-plan`** lane:

1. Match it to the ledger entry by correlation id first, then by `outputs.parentPlanSlug` + `outputs.parentIndex`.
2. If the result reports a created child plan (`outputs.planPath` / `outputs.planSlug`), add it to `spawnedPlans` and mark that row `created`.
3. If the result reports an active **`phase-planner`** lane (spawned from inline or standalone **`new-plan`**), keep the row open and add the lane to `activeLanes`.
4. If the result reports terminal completion with no remaining tasks, close that row as `completed`.
5. If the result is `partial`, keep the row open and copy its `remainingTasks`.
6. If the result is `failure`, `aborted`, or `abandoned`, mark the row blocked and ask the developer whether to retry that row, defer it, accept partial resolution, or abandon the branch.

Only return `continuationStatus: "terminal"` when every row is explicitly `completed`, `deferred`, `abandoned`, or `out_of_scope`, and no active **`phase-planner`** lanes remain for those rows. Silence or a missing row is not completion.

**Ship-complete merge (spawn chain):** When a delivered **`phase-planner`** or inline **`new-plan`** result carries **`outputs.phaseShipComplete: true`** with **`parentIndex`** matching a **`Delivery phases`** row:

1. Set **`childRows[N].status: ship-complete`**.
2. Compute **`expandNextEligibleIndex`** — lowest index with pending **`Plan:`** whose prior phase is ship-complete (index **1** after list approval).
3. Set **`outputs.expandNextEligibleIndex`**; keep **`continuationStatus: active`** when a next phase may expand.
4. **Re-emit updated terminal** (standalone) or **`## Completion (inline)`** (under **`master-planner`**) with merged **`outputs`** (same **`correlationId`**) so **`master-planner`** Step **7b** can offer **`expand-next-eligible`** when spawn-chain **`phaseShipComplete`** is present.
5. On the next structured-choice turn, include **`expand-next-eligible`** when **`expandNextEligibleIndex`** is set.

**Ship-complete vs planning-terminal:** a phase row unlocks the **next** parent index for **`expand-next-eligible`** when **`outputs.phaseShipComplete: true`** arrives on the spawn chain **or** §8 shows all PRs under that phase **`rowStatus: closed`**. Planning-terminal **`phase-planner`** alone does **not** unlock the next phase index.

## One primary choice per turn — surface observations

Match the discipline in **`master-planner`** and **`phase-planner`**: perform exactly what was chosen; scope stays on the chosen pass. If you notice gaps (diagram vs phase boundary, duplicate wording, phase count vs assessment), list short **numbered observations** in **`displayMarkdown`** and apply **Step 4-open-items**: one scoped `questions[]` entry per observation or batch item, then the current terminal delivery-phases gate question last.

## Scope guard

**Owns:** the parent plan’s dual-title **`Delivery phases`** section only (heading + list body for mode #2); decision gate when still `_TBD_`; echo for review.

**Out of scope:** renaming child plans after **`new-plan`** creates them; filling phase bodies inline (**`phase-planner`** owns the body); PR breakdown content (**`pr-breakdown`**); edits outside the dual-title section; extra H2 phase headings in the parent; `git` / commit automation; roadmap topics and PR plans (step 1 stops).

## Completion (spawned)

### MCP result preflight (`mission_control_send_agent_result`)

| Step | Check |
|------|--------|
| R1 | Call **`mission_control_send_agent_result`** with **`status`**, **`summary`**, optional **`outputs`** / **`errors`** |
| R2 | **Forbidden args absent** — no **`correlationId`**, **`dispatchId`**, **`slotId`**, or other host-resolved keys |
| R3 | Populate **`outputs`** from the required field list below |
| R4 | Re-emit updated MCP result after user-requested follow-up on this lane (same spawn session; host resolves **`correlationId`**) |

Required `outputs` fields:

- `outputs.targetPlanPath`, `outputs.targetPlanSlug`
- `outputs.decompositionKind`: `"delivery-phases"`
- `outputs.childCount`, `outputs.developerApprovalStatus`
- `outputs.childRows` — `{index, title, status, planPath?, planSlug?, correlationId?, remainingTasks?, phaseShipComplete?}` — use **`status: ship-complete`** when **`phaseShipComplete`** merged for that index
- `outputs.expandNextEligibleIndex` — one-based phase index eligible for **`expand-next-eligible`** after last ship-complete merge
- `outputs.spawnedPlans`, `outputs.activeLanes`, `outputs.openLedgerEntries`, `outputs.remainingTasks`
- `outputs.continuationOwner`: `"delivery-phases-agent"`
- `outputs.continuationStatus` — `active` while approval, inline **`new-plan`**, **`phase-planner`** child lanes, or population remains; `terminal` when all child rows are `completed`, `deferred`, `abandoned`, or `out_of_scope` and no active **`phase-planner`** lanes remain

Complete the step 6 handoff block (or #external-wait resume modal when child lanes are in flight) **before** the MCP result call when applicable. Stop after the MCP result call. Do not emit another `mission_control_spawn_agent` or run the next protocol step in the same turn (see **`../README.md`** § *Terminal stop (normative)*).

## Completion (inline)

Report the fields below in prose to the invoker on the **same lane**. Do **not** emit `mission_control_spawn_agent`, `mission_control_send_agent_result`, or `mission_control_propose_dispatch_resolution`. Do **not** add a **MCP result** under this section (see **`.sedea/centers/sedea/rules/4_mission.mdc`** § *Inline completion* and **`.sedea/centers/sedea/skills/README.md`** § *Completion (inline)*).

**Primary path:** **`master-planner`** Step 7 or **`phase-planner`** Step 5 runs this skill **inline** (`parentAgentRole: "master-plan-agent"` or `"phase-planner-agent"`). Runs **`new-plan`** **inline** on the same lane. Use the same `outputs` semantics as **`## Completion (spawned)`** in prose only — the invoker lane merges ledger fields. **Standalone** mission dispatch may still spawn this skill on a child lane; then use **`## Completion (spawned)`** and spawn **`new-plan`** child lanes per step 6.
