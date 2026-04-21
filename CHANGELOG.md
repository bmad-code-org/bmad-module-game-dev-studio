# CHANGELOG

## v0.4.0 - Apr 21, 2026 — customize.toml pattern across agents and workflows

### Agent customization surface

* All five agents (`gds-agent-game-architect`, `gds-agent-game-designer`, `gds-agent-game-dev`, `gds-agent-game-solo-dev`, `gds-agent-tech-writer`) adopt the BMAD-METHOD `customize.toml` pattern. Each agent's `SKILL.md` now runs a six-step On Activation block that resolves customization via `resolve_customization.py`, executes prepend/append hook steps, loads `persistent_facts`, reads config from `{project-root}/_bmad/gds/config.yaml`, and greets the user before the menu appears.
* Added `[agent]` namespace in each agent's `customize.toml` exposing `role`, `identity`, `communication_style`, `principles`, `persistent_facts`, `activation_steps_prepend/append`, and the `[[agent.menu]]` entries. Overrides merge per BMad structural rules (scalars override, keyed arrays-of-tables replace-or-append, other arrays append).
* Added an agent roster with essence descriptors in `src/module.yaml` so external skills (party-mode, retrospective, advanced-elicitation, help catalog) can route to, display, and embody GDS agents without reading each agent file.

### Workflow customization surface

* All 31 workflow skills converted from redirect-only `SKILL.md` + `workflow.md` split to a single integrated `SKILL.md`. The standalone `workflow.md` file is removed from every workflow skill.
* Each workflow gains the same six-step On Activation block as agents (resolve customization → prepend → `persistent_facts` → config load → greet → append), plus a `Conventions` block declaring `{skill-root}`, `{project-root}`, and `{skill-name}`.
* Each workflow's terminal step now invokes `resolve_customization.py --key workflow.on_complete` — external step-file workflows (`steps-c/`, `steps-v/`, `steps-e/`, `steps/`) get the hook appended to the final step file; inline workflows get an `<action>` inside the final `<step>`.
* Branching terminals handled: `gds-sprint-status` wires `on_complete` at all three terminal steps (main flow step 5, data mode step 20, validate mode step 30); `gds-document-project` wires it at three terminal points across `instructions.md`, `deep-dive-instructions.md` (Step 13g Finish), and `full-scan-instructions.md` so the hook fires regardless of dispatch path.
* Added `customize.toml` at every workflow skill root with a `[workflow]` namespace exposing `activation_steps_prepend`, `activation_steps_append`, `persistent_facts`, and `on_complete`. Team and per-user overrides merge from `{project-root}/_bmad/custom/{skill-name}.toml` and `{skill-name}.user.toml`.

### Fixes bundled with the rollout

* `gds-e2e-scaffold`, `gds-document-project`: fix `{skill_root}` underscore typo to `{skill-root}` (dash) in `installed_path` declarations so downstream references resolve consistently with the `Conventions` block.

## v0.3.0 - Apr 14, 2026 — sync with BMAD-METHOD v6.3.0

### Phase 4 agent consolidation

* Merged `gds-agent-game-qa` (GLaDOS) and `gds-agent-game-scrum-master` (Max) into `gds-agent-game-dev` (Link Freeman). Mirrors upstream BMAD-METHOD's collapse of QA and Scrum Master agents into a single Developer agent (upstream PRs #2179, #2186). Link now owns all scrum-master, development and QA capabilities — 16 roles in one agent.
* `src/gametest/` (17 testing knowledge fragments + `qa-index.csv`) relocated to `src/agents/gds-agent-game-dev/gametest/`. Path references updated from `{module_root}/gametest/` to `{skill_root}/gametest/`. Reason: `_bmad-output/` is being deprecated as a runtime location requirement; QA knowledge lives with the agent that uses it.

### Quick-dev update

* Updated `gds-quick-dev` to match improvements from `bmad-quick-dev` : a hardened clarify → plan → implement → review → present flow with a `step-oneshot.md` variant. Adds `compile-epic-context.md` and `sync-sprint-status.md` helpers.

### PRD + GDD split into 3-skill structures

* PRD split: consolidated `create-prd/` (with `workflow-create-prd.md`, `workflow-edit-prd.md`, `workflow-validate-prd.md` and shared `steps-c/`, `steps-e/`, `steps-v/`) split into independent skill dirs — `gds-create-prd`, `gds-edit-prd`, `gds-validate-prd`. Each has its own `SKILL.md` and `workflow.md`. Mirrors upstream's 3-skill PRD layout. Note that creating a PRD in GDS is optional - it's included for compatibility with external tools that expect it.
* GDD split: `gds-create-gdd/steps/` renamed to `steps-c/` for parity. New `gds-edit-gdd` and `gds-validate-gdd` skills added as structural placeholders that delegate to their PRD counterparts pending GDD-specific step-body authoring (tracked in `TODO.md`).

### Phase 4 workflow mirror

* All seven production-phase skills — `gds-code-review`, `gds-correct-course`, `gds-create-story`, `gds-dev-story`, `gds-retrospective`, `gds-sprint-planning`, `gds-sprint-status` — ported from upstream Phase 4 counterparts. `gds-code-review` adopts upstream's step-file architecture (`steps/step-01..04`). Config paths unified to `{module_config}`; all `bmad-agent-dev` references rewritten to `gds-agent-game-dev`.

## v0.2.4 - Apr 1, 2026

### Decouple Skills from \_bmad/ Install Directory

All skill file references migrated away from hardcoded `_bmad/` paths to forward-compatible patterns, anticipating the BMAD-METHOD installer change that stops copying skill directories into `_bmad/` (BMAD-METHOD PR #2182).

* Core skill references (party-mode, advanced-elicitation, brainstorming, adversarial-review) converted to `skill:bmad-*` invocations across 68+ files
* GDS self-references converted: step `workflow_path` → `{installed_path}`, cross-workflow handoffs → `skill:gds-*`, config → `{module_config}`, installed paths → `{skill_root}`
* Next-step navigation converted to relative paths (`./step-NN-*.md`)
* Data file references (CSVs, templates) converted to relative paths (`../data/`, `../templates/`)
* Fixed 31 files using incorrect `bmad-party-mode` path (installer strips `bmad-` prefix, correct installed name is `party-mode`)

### Fix project-context.md Not Applied During Story Creation and Development

* Added `project-context.md` to `gds-create-story` Input Files table so `discover-inputs.md` actually loads it
* Added project-context analysis step that extracts third-party frameworks, MCP configs, and conventions into story Dev Notes
* Added "Project Context Rules" section to story template
* Updated `gds-dev-story` to extract and actively apply project-context rules during implementation (Steps 2 and 5)

## v0.2.3 - Apr 1, 2026

### Opencode Compatibility Fix

* Changed SKILL.md workflow references from markdown links (`[workflow.md](workflow.md)`) to bare paths (`./workflow.md`) across all 28 workflow skills, matching the BMAD-METHOD convention. Opencode does not follow markdown-style links when resolving skill workflow files.

## v0.2.2 - Mar 16, 2026

### Agent Skill Conversion

All 7 agent YAML files converted to native skill format (`gds-agent-*` directories with SKILL.md and bmad-skill-manifest.yaml). Agents are now invocable as skills rather than parsed from YAML definitions.

* Cloud Dragonborn (Game Architect), Samus Shepard (Game Designer), Link Freeman (Game Developer), Indie (Game Solo Dev), Max (Scrum Master), GLaDOS (Game QA), Paige (Technical Writer)
* Tech Writer agent now includes prompt files for each capability (write-document, validate-doc, mermaid-gen, explain-concept, update-standards)

### Complete gds- Prefix Rename

All remaining workflow directories renamed to use `gds-` prefix for multi-module coexistence. This completes the rename started in v0.2.1.

* 22 workflow directories renamed (production, technical, gametest, design, preproduction, document-project, quick-spec)
* All step file `workflow_path` references updated to match new directory names
* SKILL.md passthrough files added to all renamed directories

### Other Changes

* module-help.csv updated to use `skill:` references for all workflows
* Removed `src/teams/` folder (default-party.csv, team-gamedev.yaml)
* Removed agent YAML schema test infrastructure (fixtures, schema, test scripts)
* Simplified all workflow bmad-skill-manifest.yaml to `type: skill`

## v0.2.1 - Mar 13, 2026

* Fix: Rename all skill directories to use `gds-` prefix for multi-module coexistence
* Fix: Remove accidentally committed `.idea/` directory

## v0.2.0 - Mar 13, 2026

### Skill Format Migration

All workflows converted to the unified skill format. This aligns with BMAD-METHOD Beta 7 conventions and enables consistent skill-based invocation across all agentic tools.

**Phase 3 (3-technical) - 2 new workflows:**

* NEW: check-implementation-readiness - ported from BMAD-METHOD, adapted for GDD-based validation
* NEW: create-epics-and-stories - ported from BMAD-METHOD, adapted for game design requirements

**Phase 2 (2-design) - 2 new workflows:**

* NEW: create-ux-design - ported from BMAD-METHOD, adapted for game UI/HUD design with player-centric framing
* NEW: create-prd - optional workflow for generating PRDs from GDD, primarily for external tool compatibility (bmad-assist)

**Phase 1 (1-preproduction) - 1 new workflow suite:**

* NEW: research suite (market, domain, technical) - ported from BMAD-METHOD with game industry context (player demographics, ESRB/PEGI ratings, game engine research, middleware evaluation)

**Quick Flow - 1 new workflow:**

* NEW: quick-dev-new-preview - unified quick flow (experimental), ported from BMAD-METHOD

### Agent Updates

* All 7 agent files updated to use new skill format
* game-architect: added check-implementation-readiness menu item
* game-solo-dev: added quick-dev-new-preview menu item

### Help System

* bmad-help fully updated with correct workflow paths, new workflows, and skill references

## v0.1.10 - Feb 28, 2026

* Knowledge base added for the 4 most popular game development engines (Unity, Unreal, Godot and Phaser) to inform the game architecture design workflow.

## v0.1.9 - Feb 23, 2026

* Fix workflow YAML quoting to prevent installation breakage

## v0.1.8 - Feb 22, 2026

* Standardize all 21 workflow descriptions to follow consistent format for improved AI skill invocation accuracy
  - Add short human-readable prefixes to descriptions
  - Use explicit trigger phrases with intent markers
  - Limit to max 2 phrases per workflow to reduce false positives
  - Fix substring matching issues with questions

## v0.1.7 - Feb 10, 2026

* Removed incorrect obsolete references to "validate-story" in the scrum-master agent and the help system.
* Minor changes from recent BMM versions backported to production workflows.
* Help files are in place. They are still somewhat placeholdery and will be improved soon.

## v0.1.5 - Feb 1, 2026

* Improve module-help.csv descriptions with "Use when..." clauses for better LLM comprehension
* Update all 26 workflow descriptions with '[Action]. Use [when].' pattern
* Move Correct Course workflow from anytime to 4-production phase (sequence 55)
* Remove sequence numbers from all anytime items
* Change "AI agent" to "chosen agentic tool" for vendor neutrality
* Properly order anytime items at top before phased workflows

## v0.1.4 - Jan 27, 2026

* The architecture creation process has been revamped and now has significantly more content relevant to game development. When creating your architecture document, you'll now make decisions on such things as rendering pipelines, physics systems, anti-cheat libraries, dialogue systems, and more. You'll be prompted for common starter templates and useful MCPs that interact with game engines.
* Fixed bug in multiple workflows that were using BMM's user_skill_level instead of GDS' game_dev_experience.

## v0.1.3 - Jan 26, 2026

* Help system update

## v0.1.2 - Jan 22, 2026

* Workflow init replaced with new help system
* Changing all references of BMGD to GDS

## v0.1.1 - Jan 21, 2026

NOTE: This is in preparation for BMAD's Beta 6 launch. It is a *significant* update and recommended for all BMGD users as it integrates all the new BMAD Beta 6 work done in the past month. Documentation is still incomplete and will come shortly. Thank you for your patience.

* Update of all shared BMM code to current version
  * Brownfield workflow init
  * Greenfield workflow init
  * Project context file is now created as part of both brownfield and greenfield workflow inits
  * quick-flow massively updated (quick-dev, quick-spec updated, quick-prototype removed) to match recent BMM Beta 6 changes
  * All workflows in 4-production changed to match recent BMM Beta 6 changes
  * Retrospective now uses actual BMGD agents, not BMM agents. Now GLaDOS will pat you on the head and Samus will randomly run around the table to work off energy. (Not really, but you get the idea.)
* Adding technical writer agent
* Adding document project workflow (used by technical writer agent and when beginning a brownfield (pre-existing codebase) project). Unmodified for now so will have many sections not typically used by games (such as API documentation)