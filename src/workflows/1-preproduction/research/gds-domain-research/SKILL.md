---
name: gds-domain-research
description: 'Conduct game domain and industry research. Use when the user says "lets create a research report on [game domain or industry]"'
---

# Game Domain Research Workflow

**Goal:** Conduct comprehensive game domain/industry research using current web data and verified sources to produce complete research documents with compelling narratives and proper citations.

**Your Role:** You are a game domain research facilitator working with an expert partner. This is a collaboration where you bring research methodology and web search capabilities, while your partner brings game industry knowledge and research direction.

## Conventions

- Bare paths (e.g. `template.md`) resolve from the skill root.
- `{skill-root}` resolves to this skill's installed directory (where `customize.toml` lives).
- `{project-root}`-prefixed paths resolve from the project working directory.
- `{skill-name}` resolves to the skill directory's basename.

## On Activation

### Step 1: Resolve the Workflow Block

Run: `python3 {project-root}/_bmad/scripts/resolve_customization.py --skill {skill-root} --key workflow`

**If the script fails**, resolve the `workflow` block yourself by reading these three files in base → team → user order and applying the same structural merge rules as the resolver:

1. `{skill-root}/customize.toml` — defaults
2. `{project-root}/_bmad/custom/{skill-name}.toml` — team overrides
3. `{project-root}/_bmad/custom/{skill-name}.user.toml` — personal overrides

Any missing file is skipped. Scalars override, tables deep-merge, arrays of tables keyed by `code` or `id` replace matching entries and append new entries, and all other arrays append.

### Step 2: Execute Prepend Steps

Execute each entry in `{workflow.activation_steps_prepend}` in order before proceeding.

### Step 3: Load Persistent Facts

Treat every entry in `{workflow.persistent_facts}` as foundational context you carry for the rest of the workflow run. Entries prefixed `file:` are paths or globs under `{project-root}` — load the referenced contents as facts. All other entries are facts verbatim.

### Step 4: Load Config

Load config from `{project-root}/_bmad/gds/config.yaml` and resolve:

- `user_name`
- `communication_language`
- `planning_artifacts`
- `date` as the system-generated current datetime

### Step 5: Greet the User

Greet `{user_name}`, speaking in `{communication_language}`.

### Step 6: Execute Append Steps

Execute each entry in `{workflow.activation_steps_append}` in order.

Activation is complete. Begin the workflow below.

## PREREQUISITE

**⛔ Web search required.** If unavailable, abort and tell the user.

## CONFIGURATION

Load config from `{module_config}` and resolve:
- `project_name`, `output_folder`, `planning_artifacts`, `user_name`
- `communication_language`, `document_output_language`, `game_dev_experience`
- `date` as a system-generated value

## QUICK TOPIC DISCOVERY

"Welcome {{user_name}}! Let's get started with your **game domain/industry research**.

**What game domain, genre, platform, or sector do you want to research?**

For example:
- 'The battle royale genre on PC and console'
- 'Age ratings and content regulations for games in Europe'
- 'The mobile free-to-play games sector'
- 'Or any other game domain you have in mind...'"

### Topic Clarification

Based on the user's topic, briefly clarify:
1. **Core Domain**: "What specific aspect of [domain] are you most interested in?"
2. **Research Goals**: "What do you hope to achieve with this research?"
3. **Scope**: "Should we focus broadly or dive deep into specific aspects (e.g., particular platforms, regions, or player demographics)?"

## ROUTE TO DOMAIN RESEARCH STEPS

After gathering the topic and goals:

1. Set `research_type = "domain"`
2. Set `research_topic = [discovered topic from discussion]`
3. Set `research_goals = [discovered goals from discussion]`
4. Create the starter output file: `{planning_artifacts}/research/domain-{{research_topic}}-research-{{date}}.md` with exact copy of the `./research.template.md` contents
5. Load: `./domain-steps/step-01-init.md` with topic context

**Note:** The discovered topic from the discussion should be passed to the initialization step, so it doesn't need to ask "What do you want to research?" again - it can focus on refining the scope for game domain research.

**✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`**
