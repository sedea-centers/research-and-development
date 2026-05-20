# plan-and-deliver — spawn contracts

Planning skills for **`plan and deliver`** follow the dual-mode shape in **`.sedea/centers/sedea/skills/README.md`**. On this mission, Squad Leader steps **§3** and **§5** and downstream decomposition agents run these skills **spawned** only; each skill file has **`## Completion (spawned)`** and **`## Completion (inline)`**.

Parent resume behavior for the **Squad Leader** is in **`../plan.mdc`** § **Spawn, wait, and parent resume**. Host spawn/result protocol is in **`.sedea/centers/sedea/rules/4_mission.mdc`**.

## Spawn index

| Skill | Typical spawner | Squad Leader ledger |
|-------|-----------------|---------------------|
| `ad-hoc-prd` | Squad Leader §3 | `prdRef` → developer approval before §4; no child lanes |
| `master-plan` | Squad Leader §5 | Seed ledger; §6 ack when `continuationOwner: master-plan-agent` |
| `delivery-phases` | Master Plan agent | Merge `childRows`, `spawnedPlans`, `activeLanes` in §7 |
| `pr-breakdown` | Master Plan agent | Same as delivery-phases |
| `new-plan` | decomposition agents | Register child plan path/slug per row index |
| `phase-plan` | `new-plan` / decomposition | Populator lane; route fields for next branch |
| `pr-plan` | `new-plan` / decomposition | `readyForImplementation`; implementation handoff pending |

## Required terminal line

Every spawned child ends with exactly one:

```text
AGENT_RESULT_RESPONSE_V1 {"version":1,"correlationId":"<uuid>","status":"<success|partial|failure|aborted|abandoned>","summary":"<1-3 sentences>","outputs":{...},"errors":[...]}
```

Field-level requirements are listed under each skill’s **`## Completion (spawned)`**.

## Default warm-up (skill frontmatter)

Spawned children merge skill `warmUpRules` with run-request paths. Default pack on planning skills:

- `.sedea/centers/research-and-development/docs/development-process.md`
- `.sedea/centers/research-and-development/rules/30_planning-target-resolution.mdc`

The invoker should also pass this mission **`plan.mdc`** and Sedea always-apply rules via Mission Control warm-up and optional run-request `warmUpRules`.
