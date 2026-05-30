# Game-Dev Studio Sync Plan — 2026-05-30

Syncing `bmad-module-game-dev-studio` (**v0.5.0**) against `bmad-code-org/BMAD-METHOD` **v6.8.0** (tag `3bcd6c3c`).

> **Supersedes** `SYNC-PLAN.md` (the 2026-05-15 v6.6.0 sync — completed, recorded in `CHANGELOG.md` v0.5.0).
>
> BMGD's content was last synced against upstream **v6.6.0** (tag `e6cdc93b`). This plan covers only the **v6.6.0 → v6.8.0 content delta** — re-porting the PRD overhaul into the already-consolidated `gds-prd`, replacing the UX skill with a spine-based `gds-ux`, streamlining `gds-create-game-brief`, and a batch of content fixes (dev-story baselines, `project_context`, investigate follow-ups, activation guardrails). `bmad-spec` is core-supplied, so no BMGD `gds-spec` is forked — see Decisions Resolved.

---

## DECISIONS RESOLVED (2026-05-30, user-approved)

1. **`gds-spec` — RELY ON CORE.** Do **not** fork a BMGD `gds-spec`. Upstream promoted `bmad-spec` to `core-skills/` with deliberately universal scope; BMGD installs alongside `core`, which supplies it (consistent with `bmad-customize` / `bmad-advanced-elicitation`). **Phase 3 is DROPPED.** Record in CHANGELOG that `bmad-spec` is core-supplied; log an optional game-adapted `gds-spec` variant in `TODO.md` for a possible later sync.

2. **`gds-ux` — KEEP UPSTREAM SPINE.** Harvested game-UX content (diegetic vs. non-diegetic HUD, controller/gamepad input schemes, game-feel/juice, console/handheld form-factors) lands as **game-specific EXPERIENCE.md sections + a game example pair**. DESIGN.md stays the visual-identity model. No third spine file.

3. **`gds-prd` — ADOPT UPSTREAM LAYOUT.** Drop `references/facilitation-guide.md`, `references/validation-render.md`, and `scripts/render-validation-html.py`; add `references/validate.md` + `references/probing.md`. Match upstream for maintainability.

---

## Context — this is a content delta, not an architecture migration

BMGD **v0.5.0** already has the `customize.toml` architecture, the consolidated `gds-prd` / `gds-gdd`, `gds-investigate`, the `team: game-dev` agent roster, and the `preceded-by`/`followed-by` catalog columns. **None of that is re-done here.** This sync ports the *content* that upstream changed between v6.6.0 and v6.8.0.

### Verified BMGD baseline (investigated 2026-05-30)

- 33 `SKILL.md` (5 agents + 28 workflow skills).
- `gds-prd` exists, consolidated, but on the **pre-overhaul** content + **old** reference layout (`facilitation-guide.md`, `validation-render.md`, `scripts/render-validation-html.py`).
- `gds-create-ux-design` is the **old scripted-step** UX skill (14 step files: init, continue, discovery, core-experience, emotional-response, inspiration, design-system, defining-experience, visual-foundation, design-directions, user-journeys/Player Journey Flows, component-strategy, ux-patterns, responsive-accessibility, complete). External refs to the name: `module-help.csv` row 11 (menu code **CU**, "Create UX Design", phase 2-design, preceded-by `gds-gdd`, outputs "ux design") and a mention in `gds-prd/assets/prd-template.md`. **It is NOT in any agent's `customize.toml` menu** (the designer menu carries GDD/brainstorm/game-brief/narrative, no UX entry) — so the rewire is mainly `module-help.csv` + deciding which agent should own the UX menu code. **No `gds-ux` exists.**
- **No `gds-spec` exists.**
- `gds-create-game-brief` is the **old scripted-step** brief (8 step files, no intent routing, no `polish_passes`, no reviewer panel) — i.e. it never received the `1c1abaa5` lean-facilitator rework (the prior plan explicitly deferred it).
- `gds-dev-story`, `gds-sprint-planning`, `gds-sprint-status` **do not define `project_context`** (grep count 0) → `bfecb6ee` applies.
- `gds-dev-story` has **no baseline-commit handling** (grep count 0) → `9a2fba97` applies.
- `gds-investigate` was ported at the v6.6.0 sync, **before** the three post-6.6 investigate fixes → `32258a53` / `697d92e3` / `7b590b0a` apply (they touch `SKILL.md` and `references/case-file-template.md`).
- **25 of 33** BMGD skills already carry the activation guardrail sentence "Activation is complete…"; the exact upstream wording "do not begin the main workflow until all activation steps have been completed" is present in **0** — so `43684549` (#2398, ~12 SKILL.md touched upstream incl. `bmad-prd`/`bmad-product-brief`) is a **wording-alignment + coverage** sweep, not a from-scratch add. The 8 skills with no guardrail at all (incl. `gds-create-ux-design`, `gds-create-game-brief`) get it for free when they are rebuilt in Phases 2/4.
- Catalog rename **already done** (`module-help.csv` header cols 9/10 = `preceded-by`,`followed-by`); `team: game-dev` **already on all agents**. Both excluded.
- `CHANGELOG.md` top entry: `[0.5.0] - 2026-05-15`. Version bump target: **0.6.0**.

## Architectural guardrail — preserved

BMGD keeps its top-level organization: `src/workflows/{1-preproduction,2-design,3-technical,4-production,gametest,gds-quick-flow}/` + `src/agents/`. We port content and skill-directory shape **within that tree** — never relocating into `src/bmm-skills/`. All ports get the game-adaptation rewrites: `_bmad/bmm/` → `_bmad/gds/`, `bmad-agent-*` → `gds-agent-*`, bmm/PM terminology → game terminology, and downstream-skill name swaps (`bmad-create-architecture` → `gds-game-architecture`, `bmad-prd` → `gds-prd`, `bmad-quick-dev` → `gds-quick-dev`, etc.).

## Decisions locked for this sync (user-approved)

1. **Replace the UX skill** — `gds-create-ux-design` → game-adapted `gds-ux` modeled on upstream's spine-based `bmad-ux` (DESIGN.md + EXPERIENCE.md, example/asset library). **Harvest the game-specific UX content first.** Update the designer agent menu.
2. **Do not fork `gds-spec`** — upstream's `bmad-spec` kernel distiller is a `core-skills/` skill that BMGD gets via `core` (Decision 1). No BMGD port; an optional game-adapted `gds-spec` variant is logged to `TODO.md` as future work.
3. **Port product-brief streamline** — apply `1c1abaa5` + `c52c9b5b` product-brief updates + `c19f6cd7` into `gds-create-game-brief`, harvesting the game-brief's domain content. (Revisits the v6.6.0 plan's deferral.)

---

## Real content delta — v6.6.0 → v6.8.0

| Upstream commit | Change | BMGD target | Phase |
|---|---|---|---|
| `71136bc6` (#2385) | PRD overhaul: Session Posture / voice rules, `references/probing.md`, intent-detection signals, Discovery 5-section restructure, PRD Discipline 3-cluster regroup, template funnel discipline (FR-N headings, Realizes UJ-X / Validates FR-X cross-refs, §3 Glossary exact-use, named-persona UJs), validation checklist Q-7/S-5/STK-2, reference layout change (drop `facilitation-guide.md`+`validation-render.md`+render script, add `references/validate.md`) | `gds-prd` | 1 |
| `c52c9b5b` (#2378) | PRD `external_sources`/`external_handoffs` customize.toml fields; File-roles constraint (decision-log vs addendum); source-extractor return contract; open-items gate (Finalize step 4); drop PRD distillate + `status: draft`; persona discipline (2-4, research-grounded/`[ILLUSTRATIVE]`); outcome-driven SKILL.md trim + progressive disclosure to `references/` | `gds-prd` (PRD parts); `gds-create-game-brief` (product-brief parts) | 1 / 4 |
| `ee47e30c` (#2413) | Spine-based `bmad-ux`: replaces `bmad-create-ux-design` with DESIGN.md + EXPERIENCE.md two-file spine, design.md-spec reference, example suite (3 DESIGN + 2 EXPERIENCE), creative-tools, named-protagonist journeys, form-factor + surface-closure discovery, reviewer gate | replace `gds-create-ux-design` → `gds-ux` (with harvest) | 2 |
| `aa6dece0` (#2417) | New `bmad-spec` kernel distiller; promoted to **core-skills** | **EXCLUDED** — core-supplied (Decision 1); not forked | — |
| `1c1abaa5` (#2370) | `bmad-product-brief` rewritten as lean intent-routed facilitator: brief-shape detection, intent routing (create/update/validate), config-driven reviewer panel + required-reviewer gate, `polish_passes` (polymorphic skill:/file:/plain entries, parallel subagents, auto-applied), real-time persistence + resume, drop scripted prompt files | `gds-create-game-brief` | 4 |
| `c19f6cd7` | product-brief: Update mode mandatory audit trail + distillate regen (headless); Validate always emits `offer_to_update`; headless validate example | `gds-create-game-brief` | 4 |
| `9a2fba97` (#2403) | dev-story captures baseline commit before implementation work | `gds-dev-story` | 5 |
| `bfecb6ee` (#2422) | define `project_context` in dev-story, sprint-planning, sprint-status | `gds-dev-story`, `gds-sprint-planning`, `gds-sprint-status` | 5 |
| `32258a53` / `697d92e3` / `7b590b0a` | bmad-investigate follow-up fixes (file-path refs + workflow guidance in `SKILL.md`; `references/case-file-template.md`) | `gds-investigate` | 5 |
| `43684549` (#2398) | strengthen activation guardrails to prevent LLM short-circuiting (~12 SKILL.md touched, incl. `bmad-prd`, `bmad-product-brief`) | wording-align the 25 BMGD skills that have a partial guardrail; the 8 with none get it via Phases 2/4 rebuilds | 5 |

### EXCLUDED (and why)

| Upstream commit / area | Why excluded |
|---|---|
| `e36f219c` (#2360) catalog rename `after`/`before` → `preceded-by`/`followed-by` | **Already applied** in BMGD (`bc4d598`); confirmed — `module-help.csv` header cols 9/10 are `preceded-by`,`followed-by`. Do not re-port. |
| `#2286` `team:` on agents | **Already applied** — every agent in `module.yaml` carries `team: game-dev`. |
| `7b31b1ac` (#2062) advanced-elicitation +19 techniques | **core-skills** (`bmad-advanced-elicitation`). BMGD installs alongside `core`, which supplies it. BMGD does not fork it. |
| `1a5df418` (#2402) keep brainstorming collaborative | **core-skills** (`bmad-brainstorming`). BMGD has its own `gds-brainstorm-game` (a distinct game-brainstorm skill, not a fork of core's). The upstream fix targets core's skill; assess at Phase 5 whether the one-line "stay collaborative / don't auto-generate" guidance is worth mirroring into `gds-brainstorm-game`, but do not import core's content. |
| `bmad-distillator` retirement (`aa6dece0`) | core skill; not in BMGD. Ignore. |
| Installer / platform / web-bundles / registry / marketplace / `removals.txt` / docs-site / i18n docs commits in range | BMGD ships **no installer** and consumes the `bmad-method` package. Out of scope. |

---

## Phase 1 — PRD overhaul re-port → `gds-prd`

### Upstream source

`src/bmm-skills/2-plan-workflows/bmad-prd/` at v6.8.0 — current tree:
```
bmad-prd/
├── SKILL.md
├── customize.toml
├── assets/{headless-schemas.md, prd-template.md, prd-validation-checklist.md, validation-report-template.html}
└── references/{headless.md, validate.md}
```
Commits: `71136bc6` (#2385, the overhaul), `c52c9b5b` (#2378, PRD parts), plus the small PRD touches in `ee47e30c` (named-protagonist UJs, drop standalone Primary Persona section, form-factor probe).

### Current BMGD shape

`src/workflows/2-design/gds-prd/` — pre-overhaul content, **old** reference layout:
```
gds-prd/
├── SKILL.md, customize.toml
├── assets/{headless-schemas.md, prd-template.md, prd-validation-checklist.md, validation-report-template.html}
├── references/{facilitation-guide.md, headless.md, validation-render.md}
└── scripts/render-validation-html.py
```
BMGD's `gds-prd/SKILL.md` already has the game-specific framing ("GDD is the primary design document", From-GDD path, `gds-quick-dev` / `gds-gdd` right-skill routing, `_bmad/gds/config.yaml`). **That game adaptation must be preserved through the re-port.**

### Plan

1. Diff upstream `bmad-prd@v6.8.0` against the BMGD `gds-prd` (file by file) to isolate the overhaul deltas. The overhaul is *content*, not structure — re-apply it into BMGD's existing files.
2. **SKILL.md**: fold in the overhaul's Session Posture (voice prohibitions, record-as-you-go, anti-caving, register-matching), the Discovery 5-sub-section restructure (Posture / Brain dump / Four-dimension read / Right-skill check / Working mode — BMGD already has most of these), the PRD Discipline 3-cluster regroup (Artifact shape / Substance / Honesty about scope), intent-detection signals on activation, the inline conflict-detection procedure for Update, and the activation-step-1 fallback (read `customize.toml` directly instead of halting on resolver failure). **Preserve BMGD's game framing and the From-GDD path throughout.** Keep `_bmad/gds/config.yaml`; do not let the upstream `_bmad/bmm/config.yaml` text creep back in.
3. **references/**: adopt upstream's new layout (Decision 3) — add `references/probing.md` (7 probing categories, 6 critical assumptions, PRD/solution-design boundary) and `references/validate.md`; **remove** `references/facilitation-guide.md`, `references/validation-render.md`, and `scripts/render-validation-html.py`. Re-point any SKILL.md/customize.toml references accordingly.
4. **assets/prd-template.md**: apply funnel discipline — §3 Glossary exact-use enforcement; §4 Features `#### FR-N: Name` headings with `Realizes UJ-X` cross-refs, testable consequences, optional per-FR Out of Scope; §7 Success Metrics `SM-N`/`SM-CN` numbering with `Validates FR-X`; §2.4 named-persona mini-flow UJs; drop the standalone Primary Persona section; drop `status: draft` frontmatter; strip the "job of UX / not this PRD" gatekeeping. Game-adapt persona/journey examples to player-facing language.
5. **assets/prd-validation-checklist.md**: add Q-3 traceability tightening, Q-7 (FR testability), S-1 glossary integrity expansion, S-2 (SM in ID continuity), S-5 (UJ persona linkage), STK-2 (UJ density gate).
6. **customize.toml**: add `external_sources` + `external_handoffs` fields (game-context comments); drop the PRD distillate output config + `bmad-distillator` finalize step; reconcile reference/script path keys with the new `references/` layout.
7. Em-dash strip pass on prose; preserve `[v2 — out of MVP]` callout convention.

### Phase 1 verification

- `gds-prd/SKILL.md` retains game framing (GDD-primary, From-GDD, `gds-gdd`/`gds-quick-dev` routing) AND the overhaul content (Session Posture, probing reference, FR-N template, new checklist items).
- Zero references to deleted files (`facilitation-guide.md`, `validation-render.md`, `render-validation-html.py`) remain in `src/workflows/2-design/gds-prd/`.
- Zero `_bmad/bmm/` paths in `gds-prd`.
- `npm run lint:md` clean.

---

## Phase 2 — UX replacement → `gds-ux` (with harvest)

### Upstream source

`src/bmm-skills/2-plan-workflows/bmad-ux/` (commit `ee47e30c`):
```
bmad-ux/
├── SKILL.md, customize.toml
├── assets/{color-themes.md, design-directions.md, design-example-{editorial,mobile,shadcn}.md,
│           excalidraw-wireframe.md, experience-example-{mobile,shadcn}.md, headless-schemas.md,
│           key-screens.md, validation-report-template.html}
└── references/{creative-tools.md, design-md-spec.md, headless.md, validate.md}
```
Two-file spine: **DESIGN.md** (visual identity, Google Labs design.md spec) + **EXPERIENCE.md** (Foundation, IA, Voice&Tone, Component Patterns, State Patterns, Interaction Primitives, Accessibility Floor, Key Flows). EXPERIENCE.md references DESIGN.md tokens via `{path.to.token}`. Named-protagonist journeys, form-factor resolution, surface-closure, opt-in reviewer gate.

### Current BMGD shape (to be replaced)

`src/workflows/2-design/gds-create-ux-design/` — old scripted-step skill, 14 steps. The game-UX adaptation is **lighter than expected**: heavy on accessibility (×56 across steps, incl. a dedicated `step-13-responsive-accessibility`), with a `step-10-user-journeys` titled "Player Journey Flows", emotional-response and core-experience steps, and only scattered explicit game vocabulary (HUD ×1, controller ×1, no diegetic/juice). **The real harvest is the game *framing*** (game-developer stakeholder, player journeys, emotional response/core-experience, game accessibility) rather than a deep diegetic-HUD/input-scheme corpus — set expectations accordingly. External name refs: `module-help.csv` row 11 (code **CU**) + a mention in `gds-prd/assets/prd-template.md`. Not in any agent menu.

### Plan

1. **Harvest first (do not lose game UX content).** Before deleting, extract from `gds-create-ux-design/` the game framing actually present: the game-developer-stakeholder facilitation posture, "Player Journey Flows" (step-10), the emotional-response (step-04) and core-experience (step-03/07) framing, and the substantial game accessibility material (step-13 responsive+accessibility, ×56 mentions — remappable controls, colorblind modes, subtitle/caption standards). Augment with the game-UX concepts the old skill *underweighted* but a game UX skill should cover: diegetic vs. non-diegetic HUD, controller/gamepad/touch input schemes, game-feel/juice, console/handheld/PC/VR form-factors, menu/navigation flow. Stage these as harvested notes for re-insertion.
2. Create `src/workflows/2-design/gds-ux/` mirroring upstream `bmad-ux`'s compact shape: `SKILL.md` + `customize.toml` + `assets/` + `references/`.
3. Port upstream's spine framing (DESIGN.md + EXPERIENCE.md, `design-md-spec.md`, creative-tools, named-protagonist journeys, form-factor, surface-closure, reviewer gate). Game-adapt: form-factor list includes console/handheld/PC/mobile/VR; named-protagonist journeys are *player* sessions; the misroute scan routes PRD→`gds-prd`, architecture→`gds-game-architecture`, narrative→`gds-create-narrative`, GDD→`gds-gdd`.
4. **Fold harvested game UX into EXPERIENCE.md** as game-specific sections (HUD/diegetic UI, input schemes, game feel, game accessibility floor) and provide at least one **game example pair** (a DESIGN.md + EXPERIENCE.md example for a game UI) alongside or replacing upstream's web/app examples. Per Decision 2, the harvested game-UX content lands as game-specific EXPERIENCE.md sections plus this game example pair; the upstream DESIGN.md + EXPERIENCE.md spine is kept and DESIGN.md stays the visual-identity model (no third spine file).
5. Adaptation rewrites: `_bmad/bmm/` → `_bmad/gds/`; `{planning_artifacts}` already game-configured in `module.yaml`; downstream "common next" pointers → `gds-game-architecture`, `gds-create-epics-and-stories`, `gds-dev-story`.
6. **Delete** `src/workflows/2-design/gds-create-ux-design/` entirely.
7. **Rewire**: update `module-help.csv` row 11 (skill `gds-create-ux-design` → `gds-ux`; keep menu code **CU** or pick a clearer one; refresh display-name/description). Update the `gds-prd/assets/prd-template.md` mention. **Decide UX menu ownership**: the UX skill is currently not in any agent menu — wire `gds-ux` into `gds-agent-game-designer/customize.toml` (the natural owner, code **CU**) so it is actually reachable. Confirm no stale `gds-create-ux-design` string survives anywhere.

### Phase 2 verification

- `gds-ux` resolves; DESIGN.md + EXPERIENCE.md spine reachable; game example pair present.
- Harvested game-UX terms (controller/gamepad, HUD/diegetic, game feel/juice, game accessibility) appear in `gds-ux` content — grep confirms nothing of value was lost.
- Zero references to `gds-create-ux-design` remain anywhere in `src/` (grep).
- `gds-agent-game-designer` menu renders the UX entry pointing at `gds-ux`.
- `npm run lint:md` clean.

---

## Phase 3 — DROPPED (`gds-spec` is core-supplied)

**Decision 1 (resolved):** Do not fork a BMGD `gds-spec`. Upstream's `bmad-spec` is now a `core-skills/` skill with deliberately universal scope; BMGD installs alongside `core`, which supplies it — consistent with how the v6.6.0 plan excluded `bmad-advanced-elicitation` and `bmad-customize`. No skill is created in this phase. Actions instead:

- `CHANGELOG.md` (Phase 6) notes that `bmad-spec` is available to BMGD users via core, no BMGD port required.
- `TODO.md` (Phase 6) records an optional game-adapted `gds-spec` variant as possible future work.

Phases below retain their original numbers for traceability; there is no work item here.

---

## Phase 4 — Product-brief streamline → `gds-create-game-brief`

### Upstream source

`src/bmm-skills/1-analysis/bmad-product-brief/` (commits `1c1abaa5`, product-brief parts of `c52c9b5b`/`ee47e30c`, `c19f6cd7`):
```
bmad-product-brief/
├── SKILL.md, customize.toml
└── assets/brief-template.md
```
Lean intent-routed facilitator: brief-shape detection, intent routing (create/update/validate), **config-driven reviewer panel** (`finalize_reviewers`/required-reviewer gate, confirmed `reviewer` ×6 in `customize.toml`), **`polish_passes`** (polymorphic `skill:`/`file:`/plain entries, parallel subagents, auto-applied so the user sees a polished draft), real-time persistence + cross-session resume, `persistent_facts` default empty with opt-in examples, form-factor surfaced in Discovery. Removed the scripted prompt files and the research/skeptic/web-researcher agents. `c19f6cd7`: Update mode mandatory audit trail (decision-log + addendum) + distillate regen in headless; Validate always emits `offer_to_update`.

### Current BMGD shape (to be reworked)

`src/workflows/1-preproduction/gds-create-game-brief/` — old scripted-step skill: 8 step files (init, continue, vision, market, fundamentals, scope, references, content, complete), `templates/game-brief-template.md`, `checklist.md`. **No intent routing, no `polish_passes`, no reviewer panel** (grep counts 0). This skill never received the lean-facilitator rework (prior plan deferred it).

### Plan — harvest vs. adopt

**Adopt from upstream (the lean-facilitator pattern):**
- Intent routing (create/update/validate) in `SKILL.md`, replacing the scripted step-file orchestration.
- Config-driven reviewer panel + required-reviewer gate in `customize.toml`.
- `polish_passes` (polymorphic, parallel, auto-applied).
- Real-time persistence + cross-session resume; `decision-log.md` as canonical memory.
- `persistent_facts` default-empty with opt-in examples.
- `c19f6cd7`: Update-mode mandatory audit trail + distillate regen (headless); Validate emits `offer_to_update`.
- Single `assets/` template instead of embedded resources.

**Harvest / preserve from the game brief (do not lose game-domain content):**
- The game-brief's **domain sections**: vision, market/competitive (game market), core fundamentals (genre, core loop, pillars), scope (game scope/MVP), references (mood/inspiration games), content (assets/levels/narrative breadth). These become the game `brief-template.md` body and the Discovery topic list — NOT generic product-brief sections.
- The game-specific Discovery framing (target players, fantasy/feel, comparable titles).
- `checklist.md` game-brief checks (fold into the validation/reviewer path).
- `module.yaml` game-brief vars and `{planning_artifacts}` output location.

**Adaptation rewrites:** `_bmad/bmm/` → `_bmad/gds/`; `bmad-product-brief` self-references → `gds-create-game-brief`; downstream "common next" → `gds-gdd` (primary design doc), then `gds-prd`. Keep the skill **name and directory** `gds-create-game-brief` (do not rename to mirror `bmad-product-brief`).

### Phase 4 verification

- `gds-create-game-brief/SKILL.md` does intent routing (create/update/validate); `polish_passes` + reviewer panel present in `customize.toml`.
- Game-domain brief content (genre, core loop, pillars, comparable titles, game scope) is preserved in the template + Discovery topics.
- Old scripted step files removed; no dangling references to them (grep).
- Zero `_bmad/bmm/` paths; downstream pointer is `gds-gdd`.
- `npm test` (lint + lint:md + format:check) passes.

---

## Phase 5 — Content fixes

For each, diff the upstream commit and port the *content* delta into BMGD's counterpart with game-adaptation rewrites.

1. **`gds-dev-story`** ← `9a2fba97` (#2403): capture the **baseline commit** (current git HEAD / state) before implementation work begins, so the story records its starting point. (BMGD currently has no baseline handling — grep 0.)
2. **`gds-dev-story` + `gds-sprint-planning` + `gds-sprint-status`** ← `bfecb6ee` (#2422): **define `project_context`** in each (the variable is referenced but undefined in BMGD — grep 0). Mirror upstream's definition, pointing at BMGD's `{project_knowledge}` / `gds-generate-project-context` output as appropriate.
3. **`gds-investigate`** ← `32258a53` + `697d92e3` + `7b590b0a`: port the three follow-up fixes (file-path reference corrections + workflow guidance in `SKILL.md`; `references/case-file-template.md` corrections). BMGD ported investigate at v6.6.0 before these landed. Game-adapt paths (`_bmad/gds/`, `investigations` subdir) as already established.
4. **Activation guardrails** ← `43684549` (#2398): 25 of 33 BMGD skills already carry a partial "Activation is complete…" guardrail, but **none** carry the strengthened upstream tail ("…confirm every entry was executed in order before proceeding. Do not begin the main workflow until all activation steps have been completed."). Align the wording in those 25 SKILL.md activation sections, and add it to any of the 8 currently-unguarded skills not already rebuilt in Phases 2/4. This is a sweep, not a blind paste: confirm each skill's activation section is shaped to accept it. Prioritize high-traffic skills (gds-prd, gds-gdd, gds-ux, gds-dev-story, sprint-*, gds-game-architecture, gds-create-epics-and-stories, gds-create-story).
5. **`gds-brainstorm-game`** ← `1a5df418` (#2402) **assessment only**: core's `bmad-brainstorming` got a "stay collaborative / don't auto-generate ideas" fix. BMGD's `gds-brainstorm-game` is its own skill; if it has the same auto-generation failure mode, mirror the one-line collaborative-posture guidance. Do NOT import core content. If not applicable, note and skip.

### Phase 5 verification

- `project_context` is defined in `gds-dev-story`, `gds-sprint-planning`, `gds-sprint-status` (grep > 0 each).
- `gds-dev-story` records a baseline commit before implementation.
- `gds-investigate` SKILL.md + case-file-template reflect the three fixes.
- Guardrail coverage rises from 4/33 toward ~33/33 (grep count).
- `npm test` passes.

---

## Phase 6 — Catalog / config housekeeping + release

1. **`module-help.csv`**: row 11 `gds-create-ux-design` → `gds-ux`; confirm no `gds-create-ux-design` row remains; verify the skill column matches the on-disk `SKILL.md` directory set exactly (no phantoms — the prior plan removed the old phantoms; re-verify). Header already `preceded-by`/`followed-by` (no change). No `gds-spec` row (Decision 1 — core-supplied).
2. **Agents**: confirm `gds-agent-game-designer` now exposes `gds-ux` (code **CU**); confirm no orphaned menu codes.
3. **`module.yaml`**: no roster changes expected (`team: game-dev` already present); confirm.
4. **Validation**: `npm test` in BMGD (lint + lint:md + format:check) clean; run BMAD-METHOD `tools/validate-skills.js` against BMGD `src/` (path-adapted) → exit 0; install smoke test (`npx bmad-method install` into a scratch dir) → confirm `gds-prd` and `gds-ux` install and resolve, and that core supplies `bmad-spec`/`bmad-advanced-elicitation` alongside (validates Decision 1's "core already supplies it" premise).
5. **Release**: `CHANGELOG.md` entry (incl. note that `bmad-spec` is core-supplied — no BMGD port); version bump `0.5.0` → **`0.6.0`** (`module.yaml` `module_version` + `package.json`).
6. **Docs**: update root `CLAUDE.md` "Last reviewed" note (post upstream sync to v6.8.0 `3bcd6c3c`; BMGD v0.6.0), the 2-design description (`gds-ux` replaces `gds-create-ux-design`), and the streamlined `gds-create-game-brief`. Update `TODO.md` with the optional `gds-spec` fork and any UX-harvest follow-ups.

### Phase 6 verification

- `module-help.csv` skill column == set of on-disk `SKILL.md` dirs; header unchanged.
- `npm test` + `validate-skills.js` + install smoke test all green.
- Version is `0.6.0` in `module.yaml` and `package.json`; `CHANGELOG.md` has the `[0.6.0]` entry; `CLAUDE.md` "Last reviewed" updated.

---

## Execution order + checkpoints

Phase-by-phase, **commit between each phase**, each independently revertible via `git revert`.

1. Phase 1 — PRD overhaul re-port → `gds-prd` — commit
2. Phase 2 — UX replacement → `gds-ux` (harvest, delete old, rewire) — commit
3. Phase 3 — DROPPED (`gds-spec` core-supplied; no commit)
4. Phase 4 — product-brief streamline → `gds-create-game-brief` — commit
5. Phase 5 — content fixes (dev-story baseline, `project_context`, investigate follow-ups, activation guardrails sweep, brainstorm assessment) — commit
6. Phase 6 — catalog/config housekeeping + full `npm test` + validate-skills + install smoke → CHANGELOG + version bump 0.5.0→0.6.0 + CLAUDE.md update → commit → push

Conventional Commits throughout (`feat:`, `refactor:`, `fix:`, `chore:`, `docs:`), subject < 72 chars.

## Out of scope (deferred / excluded)

- **Installer / platform / web-bundles / registry / marketplace / docs-site / i18n-docs** changes in the v6.6.0→v6.8.0 range — BMGD ships no installer; consumed via the `bmad-method` package.
- **core-supplied skills**: `bmad-advanced-elicitation` (#2062), `bmad-brainstorming` content (#2402 — only a posture-assessment of BMGD's own `gds-brainstorm-game`), `bmad-distillator` (retired), and **`bmad-spec`** (#2417 — Decision 1: core-supplied, not forked; optional `gds-spec` logged to `TODO.md`).
- **Catalog rename** (#2360) and **agent `team:`** (#2286) — already applied in BMGD.
- **Party-mode redesign** (`fae70152`, #2441) and **skill-metadata doc refresh** (`e74dd804`, #2439) — core/docs, not BMGD content.
- Collapsing the `gds-quick-flow/` wrapper directory (pre-existing `TODO.md` item).
