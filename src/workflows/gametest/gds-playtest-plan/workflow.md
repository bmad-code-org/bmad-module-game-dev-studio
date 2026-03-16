---
name: gametest-playtest-plan
description: 'Playtest session planner. Use when the user says "lets create a playtest plan" or "I want to schedule gameplay testing"'
main_config: '{project-root}/_bmad/gds/config.yaml'
tags:
  - qa
  - playtesting
  - user-research
  - design-validation
  - feedback
execution_hints:
  interactive: true
  autonomous: false
  iterative: true
---

# Playtest Planning

**Workflow ID**: `gds-playtest-plan`
**Version**: 1.0 (BMad v6)

## Goal

Create structured playtesting sessions to validate gameplay, gather user feedback, and identify issues that automated testing cannot catch. Playtesting validates "feel" and player experience.

## Role

You are a Game QA Specialist with expertise in designing and facilitating playtesting sessions. You help teams create structured, goal-oriented playtest plans that yield actionable insights about player experience, game feel, and design effectiveness.

---

## WORKFLOW ARCHITECTURE

This workflow produces a complete playtesting plan including session structure, observation guides, note-taking templates, and post-session analysis frameworks.

**Primary Output**: `{output_folder}/playtest-plan.md`

**Supporting Components**:
- Validation: `{installed_path}/checklist.md`
- Template: `{installed_path}/playtest-template.md`
- Knowledge Base: `knowledge/playtesting.md`

**Input Files** (auto-located):
- GDD: `{output_folder}/*gdd*.md` or `{output_folder}/*gdd*/*.md` — game mechanics to validate
- Game Brief: `{output_folder}/*brief*.md` — core pillars

---

## INITIALIZATION

Load and resolve configuration from `_bmad/gds/config.yaml`:

```yaml
output_folder: {from config}
user_name: {from config}
communication_language: {from config}
document_output_language: {from config}
game_dev_experience: {from config}
date: {system-generated}
```

Resolve workflow variables:
```yaml
playtest_type: "internal"   # internal | external | focused
session_duration: 60        # minutes
participant_count: 5
```

Greet the user by name (`user_name`) and confirm the playtest type and scope before proceeding.

---

## EXECUTION

### Preflight Requirements

Verify before proceeding:
- Playable build available
- Test objectives defined
- Participant criteria known

---

### Step 1: Define Playtest Objectives

Ask the user (or infer from GDD/game brief):

1. **What are we testing?**
   - Core gameplay loop
   - Specific feature
   - Difficulty curve
   - Tutorial effectiveness
   - Overall experience

2. **What decisions will this inform?**
   - Design changes
   - Difficulty tuning
   - Feature prioritization
   - Ship/no-ship decision

3. **What metrics will we collect?**
   - Completion rates
   - Time-on-task
   - Failure points
   - Player sentiment

---

### Step 2: Choose Playtest Type

Present options and confirm with user:

#### Internal Playtest
**Best for**: Early validation, bug finding, quick iterations

| Aspect       | Details                   |
| ------------ | ------------------------- |
| Participants | Team members, other teams |
| Duration     | 30-60 minutes             |
| Frequency    | Weekly or per-milestone   |
| Setup        | Minimal, informal         |

#### External Playtest
**Best for**: Unbiased feedback, market validation

| Aspect       | Details                           |
| ------------ | --------------------------------- |
| Participants | Target audience, external testers |
| Duration     | 1-2 hours                         |
| Frequency    | Monthly or milestone              |
| Setup        | Formal, NDA if needed             |

#### Focused Playtest
**Best for**: Specific feature validation

| Aspect       | Details                      |
| ------------ | ---------------------------- |
| Participants | Selected for specific traits |
| Duration     | 20-45 minutes                |
| Frequency    | As needed                    |
| Setup        | Specific build/scenario      |

---

### Step 3: Create Session Structure

#### Pre-Session (10-15 min)

1. **Welcome & Context**
   - Brief game description (no spoilers)
   - Session goals (what we're testing)
   - Comfort check (breaks, questions)

2. **Consent & Setup**
   - Recording consent (if applicable)
   - Controller/input preferences
   - Any accessibility needs

3. **Instructions**
   - "Play as you normally would"
   - "Think aloud if comfortable"
   - "There are no wrong answers"

#### Gameplay Session (30-90 min)

1. **Observation Focus Areas**
   - Where do players get stuck?
   - What do they try first?
   - What surprises them?
   - Where do they express frustration/joy?

2. **Note-Taking Template**

   ```
   [TIME] [LOCATION] [OBSERVATION] [PLAYER REACTION]
   0:05   Tutorial    Skipped help text    Seemed impatient
   0:12   Combat      Died to first enemy  Frustrated, retried
   ```

3. **Intervention Rules**
   - Let players struggle (within reason)
   - Note when you want to help
   - Only intervene for:
     - Critical bugs
     - Genuine distress
     - Session time running out

#### Post-Session (10-20 min)

1. **Immediate Reactions**
   - "What was your overall impression?"
   - "What stood out most?"
   - "Would you play again?"

2. **Specific Questions**
   - Feature-specific feedback
   - Difficulty perception
   - Clarity of objectives

3. **Open Feedback**
   - "Anything else?"
   - "Questions for us?"

---

### Step 4: Create Observation Guide

| Category    | Signals                               | Record             |
| ----------- | ------------------------------------- | ------------------ |
| Confusion   | Pausing, wandering, repeating actions | Location, duration |
| Frustration | Sighing, repeated failures, quitting  | Cause, frequency   |
| Engagement  | Leaning in, exclaiming, continuing    | Features that work |
| Boredom     | Checking phone, disengaging           | Drop-off points    |

**Quantitative Metrics**:
- Time to complete tutorial
- Deaths per section
- Items/features discovered
- Session duration
- Completion rate

---

### Step 5: Generate Playtest Plan Document

Write `{output_folder}/playtest-plan.md` using the `playtest-template.md` structure:

```markdown
# Playtest Plan: {Build/Feature Name}

## Overview

- Build version: {version}
- Session date(s): {dates}
- Objective: {primary goal}

## Participant Criteria

- Target: {player type}
- Experience: {gaming background}
- Count: {number}

## Session Structure

### Pre-Session (15 min)

- Welcome and consent
- Setup and preferences
- Brief instructions

### Gameplay (60 min)

- Free play / guided tasks
- Observation focus: {areas}
- Intervention threshold: {criteria}

### Post-Session (15 min)

- Immediate reactions
- Structured questions
- Open feedback

## Observation Guide

{observation_template}

## Data Collection

- Recording: {yes/no}
- Notes template: {attached}
- Metrics: {list}

## Team Roles

- Facilitator: {name}
- Note-taker: {name}
- Technical support: {name}

## Post-Playtest Analysis

- Session debrief: {date}
- Report due: {date}
- Action items review: {date}
```

---

### Step 6: Post-Playtest Analysis Framework

Include in the plan document:

#### Synthesize Findings

1. **Pattern Identification**
   - What issues appeared multiple times?
   - What worked consistently well?

2. **Severity Assessment**
   - Critical: Blocks progression
   - Major: Significantly impacts experience
   - Minor: Noticeable but manageable

3. **Recommendations**
   - Immediate fixes
   - Design considerations
   - Further investigation needed

#### Report Template

```markdown
## Playtest Report: {Session}

### Summary

- Participants: {count}
- Completion rate: {%}
- Overall sentiment: {positive/mixed/negative}

### Key Findings

1. {Finding with evidence}
2. {Finding with evidence}

### Recommendations

| Issue   | Severity | Recommendation | Priority |
| ------- | -------- | -------------- | -------- |
| {issue} | {sev}    | {rec}          | {P0-P3}  |

### Quotes

> "{Notable player quote}" - Participant {N}

### Next Steps

1. {action item}
2. {action item}
```

---

## Deliverables

1. **Playtest Plan Document** — Session structure and logistics
2. **Observation Guide** — What to watch for
3. **Note-Taking Template** — Standardized recording
4. **Report Template** — Post-session analysis format

---

## Validation

Refer to `checklist.md` for validation criteria.
