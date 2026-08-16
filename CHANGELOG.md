# CHANGELOG

## v0.7.1 - Aug 16, 2026 — `persistent_facts` ships empty

### Fixes

- **Workflows and agents no longer load `project-context.md` by default** (#34). Thirty-three `customize.toml` files shipped with `persistent_facts = ["file:{project-root}/**/project-context.md"]` pre-seeded, which made loading that file an opt-out default baked into every skill rather than a customization you choose. All thirty-three now ship `persistent_facts = []`.

  Repository-wide context belongs in `AGENTS.md`, which every skill already sees. `persistent_facts` is for context that only one workflow or agent needs, loaded when it runs instead of carried as constant memory. To opt a skill back in, add the entry to your team or user override TOML:

  ```toml
  persistent_facts = ["file:{project-root}/**/project-context.md"]
  ```

- **Docs no longer overstate what loads automatically** (#34). This is the part worth reading if you rely on `project-context.md`, because the two halves of the module behave differently.

  Seven production workflows — including `gds-create-story`, `gds-dev-story`, and `gds-code-review` — declare `project_context = **/project-context.md (load if exists)` as their own workflow variable. They never went through `persistent_facts` and are completely unaffected: they still read the file automatically.

  The five conversational agents picked it up *only* through `persistent_facts`, so they are the population this change touches. `docs/reference/agents.md` claimed "all agents share the principle" of treating `project-context.md` as the source of truth, and `gds-generate-project-context` told you on completion that "AI agents will automatically read this file when implementing". Both now draw the line where it actually falls, as do the Godot, Unity, and Unreal setup guides, which carried the same claim.

- **Setup guides name the right skill** (#34). The three engine guides told you to run `bmgd-generate-project-context`, a prefix that no longer exists. Corrected to `gds-generate-project-context`.

## v0.7.0 - Aug 9, 2026 — skills stop assuming a system Python

### Fixes

- **All Python invocations go through `uv run`** (#30). Sixty-one call sites shelled out to a bare `python3` to run `_bmad/scripts/resolve_customization.py`, plus one for `render-validation-html.py`. The resolver declares `requires-python = ">=3.11"` and hard-exits below it, because `tomllib` is a 3.11 stdlib addition. On macOS without Homebrew or Ubuntu 22.04, where `python3` is 3.10, activation fell through to the "if the script fails" path and hand-merged the TOML layers in-context — no error surfaced. `uv run` reads each script's own `requires-python` and provisions a matching interpreter.
- **Browser openers no longer spawn a Python interpreter** (#30). Four sites ran `python3 -c "import webbrowser, pathlib; webbrowser.open(...)"` purely to open an HTML file. They now use the platform opener — `open` on macOS, `xdg-open` on Linux, `start ""` on Windows — and hand you the file path if that fails instead of leaving you waiting on a window that never appears. These were the last place the module needed a system Python at all, and they failed silently on a machine that has none.
- **README Python badge corrected to `>=3.11`** (#30). It advertised `>=3.10`, a floor that cannot run the shared resolver.
- **The published plugin pointed at a skill that doesn't exist.** `.claude-plugin/marketplace.json` still listed `gds-create-ux-design`, which v0.6.0 replaced with `gds-ux` — and `gds-ux` itself was never added. Anyone installing the plugin got a dead path and no UX skill. Swapped; all 33 listed paths now resolve.
- **Orphaned legacy research workflow files removed** (#29).
- **Roblox listed under Supported Engines in the README** (#28). The engine landed in v0.6.0 but the README table hadn't caught up.

### Requirements

- **`uv` is what you need; a system Python is not.** `src/` no longer contains a bare Python invocation. Install [`uv`](https://docs.astral.sh/uv/) and it provisions the right Python per script.

## v0.6.0 - May 30, 2026 — sync with BMAD-METHOD v6.8.0

Syncs game-content and skill changes from BMAD-METHOD v6.6.0 → v6.8.0, and adds Roblox as a first-class engine.

### Engine support

- **Roblox / Luau is now a first-class engine** in `gds-game-architecture`, on par with Unity, Unreal, Godot, and Phaser (#27). Adds a Luau-aware engine knowledge fragment (DataModel and service tree, script run contexts, the replication model, DataStore persistence with session locking, RemoteEvent networking, Parallel Luau, UI architecture, and the Rojo/Wally/TestEZ tooling ecosystem); the official Roblox Studio MCP; Roblox entries across all decision-catalog categories plus a `roblox_experience` stack and `roblox_new_place` starter; and Roblox rows in the engine-recommendation and data-access pattern tables.

### New skill

- `gds-ux` — spine-based UX skill ported from upstream `bmad-ux`. Produces two peer contracts: `DESIGN.md` (visual identity, per the Google Labs design.md spec) and `EXPERIENCE.md` (information architecture, menu/HUD behavior, states, interactions, player journeys). Carries game-specific EXPERIENCE.md sections (HUD/diegetic UI, controller/gamepad/touch input schemes, game feel/juice, console/handheld/PC/VR form-factors) and a game example pair. Replaces the former `gds-create-ux-design` and is wired into the designer agent menu (code `CU`).

### Changed

- `gds-prd` — re-ported the upstream PRD overhaul (#2385, #2378): Session Posture and voice rules, intent-detection signals, the Discovery restructure, PRD Discipline regroup, FR-N template funnel discipline (testable FRs, `Realizes UJ-X` / `Validates FR-X` cross-references, glossary exact-use, named-persona journeys), an expanded validation checklist, and `external_sources` / `external_handoffs` config. Adopted upstream's reference layout — added `references/validate.md` and `references/probing.md`; removed `facilitation-guide.md`, `validation-render.md`, and `scripts/render-validation-html.py`. The GDD-primary framing and From-GDD path are preserved.
- `gds-create-game-brief` — streamlined into a lean intent-routed facilitator (create / update / validate) with a config-driven reviewer panel, `polish_passes`, real-time persistence, and a single `assets/brief-template.md`, replacing the 8-step scripted workflow (upstream #2370 / #2378 / `c19f6cd7`). All game-domain brief content (vision, genre, core loop, design pillars, scope/MVP, comparable titles, references, content) is preserved.
- `gds-dev-story` — captures a baseline git commit before implementation begins (upstream #2403).
- Activation guardrails strengthened across 24 workflow skills to prevent short-circuiting before activation completes (upstream #2398).
- `gds-brainstorm-game` — ideation now develops ideas collaboratively with the user rather than batch-generating (mirrors upstream #2402).

### Notes

- `bmad-spec` (upstream #2417) is a `core-skills` skill supplied to GDS through the `core` module; it is not forked here. An optional game-adapted `gds-spec` is logged in `TODO.md` for a possible later sync.

## v0.5.0 - May 15, 2026 — sync with BMAD-METHOD v6.6.0

Syncs game-content and skill changes from BMAD-METHOD v6.3.0 → v6.6.0. The `customize.toml` architecture was already adopted in v0.4.0, so this release ports content, consolidates the PRD and GDD skill trios, and adds a new investigation skill.

### PRD and GDD consolidation

- The `gds-create-prd` / `gds-edit-prd` / `gds-validate-prd` trio is replaced by a single `gds-prd` skill that detects intent (create / update / validate) from one entry point, mirroring upstream BMAD-METHOD's consolidated `bmad-prd`. `gds-prd` ships a compact facilitator `SKILL.md`, a PRD template, a validation checklist, an HTML report renderer, and a From-GDD discovery path that builds the PRD from an existing GDD.
- The `gds-create-gdd` / `gds-edit-gdd` / `gds-validate-gdd` trio is replaced by a single `gds-gdd` skill on the same pattern. The 24 genre guides, `game-types.csv`, and `genre-complexity.csv` move into `gds-gdd/assets/`; the former 13-step validation pipeline is distilled into `gdd-validation-checklist.md`, preserving the genre-compliance and game-type checks. `gds-gdd` is the canonical primary design document for GDS.

### New skill

- `gds-investigate` — forensic, evidence-graded case-file investigation ported from upstream `bmad-investigate`. Traces bugs, reconstructs incidents, and builds mental models of unfamiliar code. Added to the `gds-agent-game-dev` menu (code `IN`).

### Content ports

- `gds-create-epics-and-stories`: brownfield epic-scoping (upstream #1826) — an Implementation Efficiency principle, a file-churn example pair, a design-completeness assessment, a Review for File Overlap step, and a File Churn Check in final validation. Reduces unnecessary file churn when epics target the same core files.
- `gds-create-story`: the workflow now reads every UPDATE-marked file before generating dev notes (upstream #2274), so the dev agent knows the current state of code it will modify.

### Catalog and cleanup

- `module-help.csv`: the `after` / `before` columns are renamed `preceded-by` / `followed-by`, matching the upstream catalog schema (#2360).
- Removed three skills that were catalogued but never implemented — `gds-market-research`, `gds-technical-research`, `gds-quick-prototype` — and the `QP` menu entries pointing at the missing prototype skill. Rebuilding them is tracked in `TODO.md`.

### Engine MCP catalog

- Added an AI-Assisted Development section to the Unity and Godot architecture knowledge files, and three MCP server entries to `engine-mcps.yaml`: Unity's official MCP (noting its paid Unity AI subscription, so the free open-source servers stay preferred) plus the GoPeak and `tugcantopaloglu/godot-mcp` Godot servers. GoPeak is the new default Godot recommendation.

## v0.4.0 - Apr 21, 2026 — customize.toml pattern across agents and workflows

### Agent customization surface

- All five agents (`gds-agent-game-architect`, `gds-agent-game-designer`, `gds-agent-game-dev`, `gds-agent-game-solo-dev`, `gds-agent-tech-writer`) adopt the BMAD-METHOD `customize.toml` pattern. Each agent's `SKILL.md` now runs a six-step On Activation block that resolves customization via `resolve_customization.py`, executes prepend/append hook steps, loads `persistent_facts`, reads config from `{project-root}/_bmad/gds/config.yaml`, and greets the user before the menu appears.
- Added `[agent]` namespace in each agent's `customize.toml` exposing `role`, `identity`, `communication_style`, `principles`, `persistent_facts`, `activation_steps_prepend/append`, and the `[[agent.menu]]` entries. Overrides merge per BMad structural rules (scalars override, keyed arrays-of-tables replace-or-append, other arrays append).
- Added an agent roster with essence descriptors in `src/module.yaml` so external skills (party-mode, retrospective, advanced-elicitation, help catalog) can route to, display, and embody GDS agents without reading each agent file.

### Workflow customization surface

- All 31 workflow skills converted from redirect-only `SKILL.md` + `workflow.md` split to a single integrated `SKILL.md`. The standalone `workflow.md` file is removed from every workflow skill.
- Each workflow gains the same six-step On Activation block as agents (resolve customization → prepend → `persistent_facts` → config load → greet → append), plus a `Conventions` block declaring `{skill-root}`, `{project-root}`, and `{skill-name}`.
- Each workflow's terminal step now invokes `resolve_customization.py --key workflow.on_complete` — external step-file workflows (`steps-c/`, `steps-v/`, `steps-e/`, `steps/`) get the hook appended to the final step file; inline workflows get an `<action>` inside the final `<step>`.
- Branching terminals handled: `gds-sprint-status` wires `on_complete` at all three terminal steps (main flow step 5, data mode step 20, validate mode step 30); `gds-document-project` wires it at three terminal points across `instructions.md`, `deep-dive-instructions.md` (Step 13g Finish), and `full-scan-instructions.md` so the hook fires regardless of dispatch path.
- Added `customize.toml` at every workflow skill root with a `[workflow]` namespace exposing `activation_steps_prepend`, `activation_steps_append`, `persistent_facts`, and `on_complete`. Team and per-user overrides merge from `{project-root}/_bmad/custom/{skill-name}.toml` and `{skill-name}.user.toml`.

### Fixes bundled with the rollout

- `gds-e2e-scaffold`, `gds-document-project`: fix `{skill_root}` underscore typo to `{skill-root}` (dash) in `installed_path` declarations so downstream references resolve consistently with the `Conventions` block.

## v0.3.0 - Apr 14, 2026 — sync with BMAD-METHOD v6.3.0

### Phase 4 agent consolidation

- Merged `gds-agent-game-qa` (GLaDOS) and `gds-agent-game-scrum-master` (Max) into `gds-agent-game-dev` (Link Freeman). Mirrors upstream BMAD-METHOD's collapse of QA and Scrum Master agents into a single Developer agent (upstream PRs #2179, #2186). Link now owns all scrum-master, development and QA capabilities — 16 roles in one agent.
- `src/gametest/` (17 testing knowledge fragments + `qa-index.csv`) relocated to `src/agents/gds-agent-game-dev/gametest/`. Path references updated from `{module_root}/gametest/` to `{skill_root}/gametest/`. Reason: `_bmad-output/` is being deprecated as a runtime location requirement; QA knowledge lives with the agent that uses it.

### Quick-dev update

- Updated `gds-quick-dev` to match improvements from `bmad-quick-dev` : a hardened clarify → plan → implement → review → present flow with a `step-oneshot.md` variant. Adds `compile-epic-context.md` and `sync-sprint-status.md` helpers.

### PRD + GDD split into 3-skill structures

- PRD split: consolidated `create-prd/` (with `workflow-create-prd.md`, `workflow-edit-prd.md`, `workflow-validate-prd.md` and shared `steps-c/`, `steps-e/`, `steps-v/`) split into independent skill dirs — `gds-create-prd`, `gds-edit-prd`, `gds-validate-prd`. Each has its own `SKILL.md` and `workflow.md`. Mirrors upstream's 3-skill PRD layout. Note that creating a PRD in GDS is optional - it's included for compatibility with external tools that expect it.
- GDD split: `gds-create-gdd/steps/` renamed to `steps-c/` for parity. New `gds-edit-gdd` and `gds-validate-gdd` skills added as structural placeholders that delegate to their PRD counterparts pending GDD-specific step-body authoring (tracked in `TODO.md`).

### Phase 4 workflow mirror

- All seven production-phase skills — `gds-code-review`, `gds-correct-course`, `gds-create-story`, `gds-dev-story`, `gds-retrospective`, `gds-sprint-planning`, `gds-sprint-status` — ported from upstream Phase 4 counterparts. `gds-code-review` adopts upstream's step-file architecture (`steps/step-01..04`). Config paths unified to `{module_config}`; all `bmad-agent-dev` references rewritten to `gds-agent-game-dev`.

## v0.2.4 - Apr 1, 2026

### Decouple Skills from \_bmad/ Install Directory

All skill file references migrated away from hardcoded `_bmad/` paths to forward-compatible patterns, anticipating the BMAD-METHOD installer change that stops copying skill directories into `_bmad/` (BMAD-METHOD PR #2182).

- Core skill references (party-mode, advanced-elicitation, brainstorming, adversarial-review) converted to `skill:bmad-*` invocations across 68+ files
- GDS self-references converted: step `workflow_path` → `{installed_path}`, cross-workflow handoffs → `skill:gds-*`, config → `{module_config}`, installed paths → `{skill_root}`
- Next-step navigation converted to relative paths (`./step-NN-*.md`)
- Data file references (CSVs, templates) converted to relative paths (`../data/`, `../templates/`)
- Fixed 31 files using incorrect `bmad-party-mode` path (installer strips `bmad-` prefix, correct installed name is `party-mode`)

### Fix project-context.md Not Applied During Story Creation and Development

- Added `project-context.md` to `gds-create-story` Input Files table so `discover-inputs.md` actually loads it
- Added project-context analysis step that extracts third-party frameworks, MCP configs, and conventions into story Dev Notes
- Added "Project Context Rules" section to story template
- Updated `gds-dev-story` to extract and actively apply project-context rules during implementation (Steps 2 and 5)

## v0.2.3 - Apr 1, 2026

### Opencode Compatibility Fix

- Changed SKILL.md workflow references from markdown links (`[workflow.md](workflow.md)`) to bare paths (`./workflow.md`) across all 28 workflow skills, matching the BMAD-METHOD convention. Opencode does not follow markdown-style links when resolving skill workflow files.

## v0.2.2 - Mar 16, 2026

### Agent Skill Conversion

All 7 agent YAML files converted to native skill format (`gds-agent-*` directories with SKILL.md and bmad-skill-manifest.yaml). Agents are now invocable as skills rather than parsed from YAML definitions.

- Cloud Dragonborn (Game Architect), Samus Shepard (Game Designer), Link Freeman (Game Developer), Indie (Game Solo Dev), Max (Scrum Master), GLaDOS (Game QA), Paige (Technical Writer)
- Tech Writer agent now includes prompt files for each capability (write-document, validate-doc, mermaid-gen, explain-concept, update-standards)

### Complete gds- Prefix Rename

All remaining workflow directories renamed to use `gds-` prefix for multi-module coexistence. This completes the rename started in v0.2.1.

- 22 workflow directories renamed (production, technical, gametest, design, preproduction, document-project, quick-spec)
- All step file `workflow_path` references updated to match new directory names
- SKILL.md passthrough files added to all renamed directories

### Other Changes

- module-help.csv updated to use `skill:` references for all workflows
- Removed `src/teams/` folder (default-party.csv, team-gamedev.yaml)
- Removed agent YAML schema test infrastructure (fixtures, schema, test scripts)
- Simplified all workflow bmad-skill-manifest.yaml to `type: skill`

## v0.2.1 - Mar 13, 2026

- Fix: Rename all skill directories to use `gds-` prefix for multi-module coexistence
- Fix: Remove accidentally committed `.idea/` directory

## v0.2.0 - Mar 13, 2026

### Skill Format Migration

All workflows converted to the unified skill format. This aligns with BMAD-METHOD Beta 7 conventions and enables consistent skill-based invocation across all agentic tools.

**Phase 3 (3-technical) - 2 new workflows:**

- NEW: check-implementation-readiness - ported from BMAD-METHOD, adapted for GDD-based validation
- NEW: create-epics-and-stories - ported from BMAD-METHOD, adapted for game design requirements

**Phase 2 (2-design) - 2 new workflows:**

- NEW: create-ux-design - ported from BMAD-METHOD, adapted for game UI/HUD design with player-centric framing
- NEW: create-prd - optional workflow for generating PRDs from GDD, primarily for external tool compatibility (bmad-assist)

**Phase 1 (1-preproduction) - 1 new workflow suite:**

- NEW: research suite (market, domain, technical) - ported from BMAD-METHOD with game industry context (player demographics, ESRB/PEGI ratings, game engine research, middleware evaluation)

**Quick Flow - 1 new workflow:**

- NEW: quick-dev-new-preview - unified quick flow (experimental), ported from BMAD-METHOD

### Agent Updates

- All 7 agent files updated to use new skill format
- game-architect: added check-implementation-readiness menu item
- game-solo-dev: added quick-dev-new-preview menu item

### Help System

- bmad-help fully updated with correct workflow paths, new workflows, and skill references

## v0.1.10 - Feb 28, 2026

- Knowledge base added for the 4 most popular game development engines (Unity, Unreal, Godot and Phaser) to inform the game architecture design workflow.

## v0.1.9 - Feb 23, 2026

- Fix workflow YAML quoting to prevent installation breakage

## v0.1.8 - Feb 22, 2026

- Standardize all 21 workflow descriptions to follow consistent format for improved AI skill invocation accuracy
  - Add short human-readable prefixes to descriptions
  - Use explicit trigger phrases with intent markers
  - Limit to max 2 phrases per workflow to reduce false positives
  - Fix substring matching issues with questions

## v0.1.7 - Feb 10, 2026

- Removed incorrect obsolete references to "validate-story" in the scrum-master agent and the help system.
- Minor changes from recent BMM versions backported to production workflows.
- Help files are in place. They are still somewhat placeholdery and will be improved soon.

## v0.1.5 - Feb 1, 2026

- Improve module-help.csv descriptions with "Use when..." clauses for better LLM comprehension
- Update all 26 workflow descriptions with '[Action]. Use [when].' pattern
- Move Correct Course workflow from anytime to 4-production phase (sequence 55)
- Remove sequence numbers from all anytime items
- Change "AI agent" to "chosen agentic tool" for vendor neutrality
- Properly order anytime items at top before phased workflows

## v0.1.4 - Jan 27, 2026

- The architecture creation process has been revamped and now has significantly more content relevant to game development. When creating your architecture document, you'll now make decisions on such things as rendering pipelines, physics systems, anti-cheat libraries, dialogue systems, and more. You'll be prompted for common starter templates and useful MCPs that interact with game engines.
- Fixed bug in multiple workflows that were using BMM's user_skill_level instead of GDS' game_dev_experience.

## v0.1.3 - Jan 26, 2026

- Help system update

## v0.1.2 - Jan 22, 2026

- Workflow init replaced with new help system
- Changing all references of BMGD to GDS

## v0.1.1 - Jan 21, 2026

NOTE: This is in preparation for BMAD's Beta 6 launch. It is a _significant_ update and recommended for all BMGD users as it integrates all the new BMAD Beta 6 work done in the past month. Documentation is still incomplete and will come shortly. Thank you for your patience.

- Update of all shared BMM code to current version
  - Brownfield workflow init
  - Greenfield workflow init
  - Project context file is now created as part of both brownfield and greenfield workflow inits
  - quick-flow massively updated (quick-dev, quick-spec updated, quick-prototype removed) to match recent BMM Beta 6 changes
  - All workflows in 4-production changed to match recent BMM Beta 6 changes
  - Retrospective now uses actual BMGD agents, not BMM agents. Now GLaDOS will pat you on the head and Samus will randomly run around the table to work off energy. (Not really, but you get the idea.)
- Adding technical writer agent
- Adding document project workflow (used by technical writer agent and when beginning a brownfield (pre-existing codebase) project). Unmodified for now so will have many sections not typically used by games (such as API documentation)
