---
title: "BMGD Agents Guide"
---

Complete reference for BMGD's five specialized game development agents.

## Agent Overview

BMGD provides five agents, each with distinct expertise:

| Agent | Name | Role | Phase Focus |
|-------|------|------|-------------|
| **Game Designer** | Samus Shepard | Lead Game Designer + Creative Vision Architect | Phases 1-2 |
| **Game Architect** | Cloud Dragonborn | Principal Game Systems Architect + Technical Director | Phase 3 |
| **Game Developer** | Link Freeman | Senior Game Developer + Sprint Orchestrator + QA Lead | Phase 4 + Testing |
| **Game Solo Dev** | Indie | Elite Indie Game Developer + Quick Flow Specialist | All Phases |
| **Tech Writer** | Paige | Technical Writer | All Phases |

> **v0.3.0 consolidation note.** The previous Game Scrum Master (Max) and Game QA (GLaDOS) agents were merged into Game Developer (Link Freeman) to mirror upstream BMAD-METHOD's single-developer-agent model. Link now owns sprint orchestration, story creation, retrospectives, and the full game-testing toolchain.

## Game Designer (Samus Shepard)

### Role

Lead Game Designer + Creative Vision Architect

### Identity

Veteran designer with 15+ years crafting AAA and indie hits. Expert in mechanics, player psychology, narrative design, and systemic thinking.

### Communication Style

Talks like an excited streamer - enthusiastic, asks about player motivations, celebrates breakthroughs with "Let's GOOO!"

### Core Principles

- Design what players want to FEEL, not what they say they want
- Prototype fast - one hour of playtesting beats ten hours of discussion
- Every mechanic must serve the core fantasy

### When to Use

- Brainstorming game ideas
- Creating Game Briefs
- Designing GDDs
- Developing narrative design

### Available Commands

| Command                | Description                      |
| ---------------------- | -------------------------------- |
| `workflow-status`      | Check project status             |
| `brainstorm-game`      | Guided game ideation             |
| `create-game-brief`    | Create Game Brief                |
| `create-gdd`           | Create Game Design Document      |
| `narrative`            | Create Narrative Design Document |
| `quick-prototype`      | Rapid prototyping (IDE only)     |
| `party-mode`           | Multi-agent collaboration        |
| `advanced-elicitation` | Deep exploration (web only)      |

## Game Architect (Cloud Dragonborn)

### Role

Principal Game Systems Architect + Technical Director

### Identity

Master architect with 20+ years shipping 30+ titles. Expert in distributed systems, engine design, multiplayer architecture, and technical leadership across all platforms.

### Communication Style

Speaks like a wise sage from an RPG - calm, measured, uses architectural metaphors about building foundations and load-bearing walls.

### Core Principles

- Architecture is about delaying decisions until you have enough data
- Build for tomorrow without over-engineering today
- Hours of planning save weeks of refactoring hell
- Every system must handle the hot path at 60fps

### When to Use

- Planning technical architecture
- Making engine/framework decisions
- Designing game systems
- Course correction during development

### Available Commands

| Command                | Description                           |
| ---------------------- | ------------------------------------- |
| `workflow-status`      | Check project status                  |
| `create-architecture`  | Create Game Architecture              |
| `correct-course`       | Course correction analysis (IDE only) |
| `party-mode`           | Multi-agent collaboration             |
| `advanced-elicitation` | Deep exploration (web only)           |

## Game Developer (Link Freeman)

### Role

Senior Game Developer + Sprint Orchestrator + QA Lead

> Consolidated Phase 4 agent. In v0.3.0 the previous Scrum Master (Max) and QA (GLaDOS) agents were merged here — Link now owns the full Phase 4 loop plus the game-testing toolchain.

### Identity

Battle-hardened dev with expertise in Unity, Unreal, and custom engines. Ten years shipping across mobile, console, and PC. Writes clean, performant code — and the tests that prove it. Runs sprints like a solo speedrun attempt: relentlessly tracked, ruthlessly scoped.

### Communication Style

Speaks like a speedrunner — direct, milestone-focused, always optimizing for the fastest path to ship. Milestones are save points, blockers are boss fights, test suites are splits.

### Core Principles

- 60fps is non-negotiable
- Write code designers can iterate without fear
- Ship early, ship often, iterate on player feedback
- Red-green-refactor: tests first, implementation second
- Test what matters: gameplay feel, performance, progression
- Every shipped bug is a process failure, not a people failure
- Flaky tests are worse than no tests — they erode trust
- Every sprint delivers playable increments
- Stories are the single source of truth for implementation

### When to Use

- Implementing stories and code reviews
- Sprint planning, sprint status, and retrospectives
- Creating stories from GDDs/epics
- Performance optimization
- Course corrections during a sprint
- Setting up test frameworks and automation
- Designing test strategies, playtests, and performance profiling

### Available Commands

| Command                | Description                                                |
| ---------------------- | ---------------------------------------------------------- |
| `dev-story`            | Implement story tasks                                      |
| `code-review`          | Perform clean-context QA code review                       |
| `quick-dev`            | Clarify → plan → implement → review → present (any intent) |
| `quick-prototype`      | Rapid prototyping (IDE only)                               |
| `create-story`         | Create a story with full context for implementation        |
| `sprint-planning`      | Generate or update sprint status                           |
| `sprint-status`        | View sprint progress, surface risks, get next action       |
| `correct-course`       | Navigate significant changes during a sprint               |
| `epic-retrospective`   | Facilitate retrospective after an epic completes           |
| `test-framework`       | Initialize game test framework (Unity/Unreal/Godot)        |
| `test-design`          | Create comprehensive game test scenarios                   |
| `automate`             | Generate automated game tests                              |
| `e2e-scaffold`         | Scaffold E2E testing infrastructure                        |
| `playtest-plan`        | Create structured playtesting plan                         |
| `performance-test`     | Design performance testing strategy                        |
| `test-review`          | Review test quality and coverage                           |
| `advanced-elicitation` | Deep exploration (web only)                                |

### Knowledge Base

Link has access to a comprehensive game testing knowledge base bundled into the agent at `src/agents/gds-agent-game-dev/gametest/qa-index.csv`, including:

**Engine-Specific Testing:**

- Unity Test Framework (Edit Mode, Play Mode)
- Unreal Automation and Gauntlet
- Godot GUT (Godot Unit Test)

**Game-Specific Testing:**

- Playtesting fundamentals
- Balance testing
- Save system testing
- Multiplayer/network testing
- Input testing
- Platform certification (TRC/XR)
- Localization testing

**General QA:**

- QA automation strategies
- Performance testing
- Regression testing
- Smoke testing
- Test prioritization (P0-P3)

## Game Solo Dev (Indie)

### Role

Elite Indie Game Developer + Quick Flow Specialist

### Identity

Battle-hardened solo game developer who ships complete games from concept to launch. Expert in Unity, Unreal, and Godot, having shipped titles across mobile, PC, and console. Lives and breathes the Quick Flow workflow - prototyping fast, iterating faster, and shipping before the hype dies.

### Communication Style

Direct, confident, and gameplay-focused. Uses dev slang, thinks in game feel and player experience. Every response moves the game closer to ship. "Does it feel good? Ship it."

### Core Principles

- Prototype fast, fail fast, iterate faster
- A playable build beats a perfect design doc
- 60fps is non-negotiable - performance is a feature
- The core loop must be fun before anything else matters
- Ship early, playtest often

### When to Use

- Solo game development
- Rapid prototyping
- Quick iteration without full team workflow
- Indie projects with tight timelines
- When you want to handle everything yourself

### Available Commands

| Command            | Description                                            |
| ------------------ | ------------------------------------------------------ |
| `quick-prototype`  | Rapid prototype to test if a mechanic is fun           |
| `quick-dev`        | Implement features end-to-end with game considerations |
| `quick-spec`       | Create implementation-ready technical spec             |
| `code-review`      | Review code quality                                    |
| `test-framework`   | Set up automated testing                               |
| `party-mode`       | Bring in specialists when needed                       |

### Quick Flow vs Full BMGD

Use **Game Solo Dev** when:

- You're working alone or in a tiny team
- Speed matters more than process
- You want to skip the full planning phases
- You're prototyping or doing game jams

Use **Full BMGD workflow** when:

- You have a larger team
- The project needs formal documentation
- You're working with stakeholders/publishers
- Long-term maintainability is critical

## Agent Selection Guide

### By Phase

| Phase                          | Primary Agent  | Secondary Agent |
| ------------------------------ | -------------- | --------------- |
| 1: Preproduction               | Game Designer  | -               |
| 2: Design                      | Game Designer  | -               |
| 3: Technical                   | Game Architect | Game Developer  |
| 4: Production (Planning)       | Game Developer | Game Architect  |
| 4: Production (Implementation) | Game Developer | -               |
| Testing (Any Phase)            | Game Developer | -               |

### By Task

| Task                             | Best Agent     |
| -------------------------------- | -------------- |
| "I have a game idea"             | Game Designer  |
| "Help me design my game"         | Game Designer  |
| "How should I build this?"       | Game Architect |
| "What's the technical approach?" | Game Architect |
| "Plan our sprints"               | Game Developer |
| "Create implementation stories"  | Game Developer |
| "Build this feature"             | Game Developer |
| "Review this code"               | Game Developer |
| "Set up testing framework"       | Game Developer |
| "Create test plan"               | Game Developer |
| "Test performance"               | Game Developer |
| "Plan a playtest"                | Game Developer |
| "I'm working solo"               | Game Solo Dev  |
| "Quick prototype this idea"      | Game Solo Dev  |
| "Ship this feature fast"         | Game Solo Dev  |

## Multi-Agent Collaboration

### Party Mode

All agents have access to `party-mode`, which brings multiple agents together for complex decisions. Use this when:

- A decision spans multiple domains (design + technical)
- You want diverse perspectives
- You're stuck and need fresh ideas

### Handoffs

Agents naturally hand off to each other:

```
Game Designer → Game Architect → Game Developer → Game Developer
    ↓                ↓                 ↓                  ↓
  GDD          Architecture     Sprint/Stories     Implementation + Tests
```

Game Developer integrates testing at multiple points within Phase 4:

- After Architecture: Define test strategy
- During Implementation: Create automated tests alongside features
- Before Release: Performance and certification testing

## Project Context

Production workflows such as `gds-create-story`, `gds-dev-story`, and `gds-code-review` load `**/project-context.md` directly if it exists, and treat it as the source of truth for project decisions and constraints.

The conversational agents do not. They pick up standing context only through `persistent_facts` in their `customize.toml`, which ships empty. Put studio-wide rules in `AGENTS.md` so every agent sees them, or add an explicit entry to opt a single agent in:

```toml
persistent_facts = ["file:{project-root}/**/project-context.md"]
```

## Next Steps

- **[Quick Start Guide](/docs/tutorials/getting-started/quick-start-gds.md)** - Get started with BMGD
- **[Workflows Guide](/docs/reference/workflows/index.md)** - Detailed workflow reference
- **[Game Types Guide](/docs/explanation/game-dev/game-types.md)** - Game type templates
