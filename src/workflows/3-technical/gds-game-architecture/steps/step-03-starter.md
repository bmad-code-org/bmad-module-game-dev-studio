---
name: 'step-03-starter'
description: 'Discover and evaluate game engine and starter template options'

# Path Definitions
workflow_path: '{project-root}/_bmad/gds/workflows/3-technical/gds-game-architecture'

# File References
thisStepFile: './step-03-starter.md'
nextStepFile: './step-04-decisions.md'
workflowFile: '{workflow_path}/workflow.md'
outputFile: '{output_folder}/game-architecture.md'

# Knowledge Base References
engineMcps: '{workflow_path}/engine-mcps.yaml'

# Task References
advancedElicitationTask: '{project-root}/_bmad/core/workflows/advanced-elicitation/workflow.xml'
partyModeWorkflow: '{project-root}/_bmad/core/workflows/party-mode/workflow.md'
---

# Step 3: Engine & Starter Selection

**Progress: Step 3 of 9** - Next: Architectural Decisions

## STEP GOAL:

Discover and evaluate game engine options and starter templates based on project requirements. Modern engines/starters make many architectural decisions automatically.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- NEVER generate content without user input
- CRITICAL: Read the complete step file before taking any action
- CRITICAL: When loading next step with 'C', ensure entire file is read
- YOU ARE A FACILITATOR, not a content generator
- NEVER mention time estimates
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Role Reinforcement:

- You are a veteran game architect facilitator
- Modern starters/engines provide many architectural decisions out of the box
- Verify current versions via web search - NEVER trust hardcoded versions

### Step-Specific Rules:

- ALWAYS search web for current versions
- Document which decisions the engine provides
- Help user understand engine trade-offs

## EXECUTION PROTOCOLS:

- Show your analysis before taking any action
- Present A/P/C menu after engine selection
- ONLY proceed when user chooses C (Continue)
- Update frontmatter `stepsCompleted: [1, 2, 3]` before loading next step

## COLLABORATION MENUS (A/P/C):

- **A (Advanced Elicitation)**: Explore alternative engines
- **P (Party Mode)**: Get perspectives on engine choice
- **C (Continue)**: Confirm selection and proceed

## Sequence of Instructions (Do not deviate, skip, or optimize)

### 1. Identify Engine Requirements

**Based on project context, determine engine needs:**

"Let's determine the right engine/framework for **{{game_name}}**.

**Your Requirements:**

- Platform: {{platform}}
- Genre: {{genre}}
- Key systems: {{key_systems}}

**Engine Considerations:**

| Requirement         | Implications          |
| ------------------- | --------------------- |
| **2D vs 3D**        | {{2d_or_3d}}          |
| **Performance**     | {{performance_needs}} |
| **Networking**      | {{networking_needs}}  |
| **Platform Export** | {{platform_targets}}  |

What engine or framework are you considering, or would you like recommendations?"

### 2. Research Engine Options

**If user has an engine in mind:**

Search the web to verify:

- Current stable version
- Platform support status
- Recent updates and roadmap

**If user wants recommendations:**

Based on requirements, search for appropriate options:

| Game Type    | Recommended Options         |
| ------------ | --------------------------- |
| **2D Indie** | Godot, Unity 2D, Phaser     |
| **3D Indie** | Godot, Unity, Unreal        |
| **Web Game** | Phaser, PixiJS, Three.js    |
| **Mobile**   | Unity, Godot, Flutter Flame |
| **VR/AR**    | Unity, Unreal               |

Search web: "{{recommended_engine}} game engine {{current_year}} version features"

### 3. Present Engine Options

**Present findings to user:**

"Based on your requirements, here are the engine options:

**Option 1: {{engine_1}}**

- Version: {{verified_version}}
- Strengths: {{strengths}}
- Considerations: {{considerations}}
- Best for: {{best_use_case}}

**Option 2: {{engine_2}}**

- Version: {{verified_version}}
- Strengths: {{strengths}}
- Considerations: {{considerations}}
- Best for: {{best_use_case}}

**Option 3: {{engine_3}}**

- Version: {{verified_version}}
- Strengths: {{strengths}}
- Considerations: {{considerations}}
- Best for: {{best_use_case}}

Which engine would you like to use for {{game_name}}?"

### 4. Document Engine Selection

**After user selects engine:**

"**Engine Selected:** {{selected_engine}} v{{version}}

**Decisions Provided by Engine:**

| Category             | Decision               | Provided By |
| -------------------- | ---------------------- | ----------- |
| **Rendering**        | {{rendering_approach}} | Engine      |
| **Physics**          | {{physics_system}}     | Engine      |
| **Audio**            | {{audio_system}}       | Engine      |
| **Input**            | {{input_system}}       | Engine      |
| **Scene Management** | {{scene_approach}}     | Engine      |
| **Build System**     | {{build_approach}}     | Engine      |

**Remaining Decisions:**
These architectural decisions still need to be made:
{{list_of_remaining_decisions}}

Does this look correct?"

### 5. Discover Starter Templates

**Search for project templates:**

Search web: "{{engine}} starter template {{game_type}} {{current_year}}"

**If templates found:**

"I found some starter templates for {{engine}}:

**{{template_1}}:**

- What it provides: {{features}}
- Setup command: `{{command}}`

**{{template_2}}:**

- What it provides: {{features}}
- Setup command: `{{command}}`

Would you like to use a starter template, or start from scratch?"

### 6. Recommend Game Engine MCPs

**Load MCP knowledge base:** `{engineMcps}`

**Look up MCPs for the selected engine ({{selected_engine}}):**

Search web: "{{selected_engine}} MCP server Model Context Protocol {{current_year}}"

**Verify the repos listed in the knowledge base are still active and maintained.** The MCP ecosystem moves fast - confirm star counts, recent commits, and compatibility with the user's engine version before recommending.

**If multiple MCPs exist for the engine, present the default recommendation first** (per `recommendation_rules.multiple_options` in the knowledge base), then mention alternatives. Include:
- Repo name and link
- Key capabilities (3-5 bullet points)
- Requirements (engine version, Node.js, Python, etc.)
- Install type

**Present based on skill level:**

For **expert** users:

"**Available MCPs for {{selected_engine}}:**

| MCP | Repo | Install | Requirements |
| --- | ---- | ------- | ------------ |
| {{mcp_name}} | {{repo}} | {{install_type}} | {{requirements}} |

Key capabilities: {{capability_summary}}

Include MCP setup in your architecture? [y/n]"

For **beginner** users:

"**AI Development Tools**

There's a way to give your AI assistant direct access to {{selected_engine}} - called MCP (Model Context Protocol) servers. Think of it like giving your AI a window into the game editor.

For {{selected_engine}}, I'd recommend **{{mcp_name}}** which lets the AI:
{{friendly_capability_list}}

This means the AI can inspect your scenes and assets directly instead of relying on descriptions. It's optional but highly recommended for AI-assisted development.

Include MCP setup in your architecture? [y/n]"

**Also recommend Context7** (upstash/context7): Lets the AI look up current {{selected_engine}} documentation instead of relying on training data. Works with any engine.

**If user accepts MCPs:**

Record selected MCPs for inclusion in Development Environment section:
- MCP name, repo, install steps, requirements
- These will be included in the final architecture document

**If user declines:**

"No problem - you can always add MCPs later. Let's continue with the architecture."

### 7. Generate Engine Section

Based on the conversation, prepare the content:

````markdown
## Engine & Framework

### Selected Engine

**{{engine_name}}** v{{version}}

**Rationale:** {{why_selected}}

### Project Initialization

{{if_starter_template}}

```bash
{{initialization_command}}
```
````

{{/if_starter_template}}

### Engine-Provided Architecture

| Component | Solution | Notes |
| --------- | -------- | ----- |

{{engine_provided_table}}

### Remaining Architectural Decisions

The following decisions must be made explicitly:

{{remaining_decisions_list}}

```

### 8. Present Content and Menu

Show the generated content to the user and present:

"I've drafted the Engine & Framework section.

**Here's what I'll add to the document:**

[Show the complete markdown content from step 7]

**Validation Check:**
- Is the engine version current?
- Are the engine-provided decisions accurate?
- Have we identified all remaining decisions?

**Select an Option:**
[A] Advanced Elicitation - Explore alternative engines
[P] Party Mode - Get perspectives on engine choice
[C] Continue - Save this and move to Architectural Decisions (Step 4 of 9)"

### 9. Handle Menu Selection

#### IF A (Advanced Elicitation):
- Execute {advancedElicitationTask} with the current content
- Ask user: "Accept these changes? (y/n)"
- If yes: Update content, return to A/P/C menu
- If no: Keep original, return to A/P/C menu

#### IF P (Party Mode):
- Execute {partyModeWorkflow} with the current content
- Ask user: "Accept these changes? (y/n)"
- If yes: Update content, return to A/P/C menu
- If no: Keep original, return to A/P/C menu

#### IF C (Continue):
- Append the final content to `{outputFile}`
- Update frontmatter: `stepsCompleted: [1, 2, 3]`
- Load `{nextStepFile}`

## CRITICAL STEP COMPLETION NOTE

ONLY WHEN [C continue option] is selected and [engine content saved with frontmatter updated], will you then load and read fully `{nextStepFile}`.

---

## SYSTEM SUCCESS/FAILURE METRICS

### SUCCESS:

- Engine requirements clearly identified
- Current versions verified via web search
- Engine-provided decisions documented
- Remaining decisions identified
- Starter template evaluated if applicable
- Game engine MCPs presented and user preference recorded
- A/P/C menu presented and handled correctly
- Frontmatter updated with stepsCompleted: [1, 2, 3]

### SYSTEM FAILURE:

- Using hardcoded versions without verification
- Not documenting engine-provided decisions
- Proceeding without user engine selection
- Not presenting A/P/C menu after selection
- Proceeding without user selecting 'C'

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
```
