---
title: "Set up an Unreal project with BMGD"
description: Configure a new Unreal project for full production game development with BMGD
---

# Set up an Unreal project with BMGD

Configure a new Unreal project with BMGD workflows for full production game development.

---

## When to Use This Guide

- You're starting a new Unreal game project
- You want to use BMGD's Full Production workflow
- You need formal documentation (GDD, architecture, sprint tracking)

---

## When to Skip This

- You just want to prototype quickly — use [Quick Flow](../tutorials/first-game-project.md) instead
- You're using Unity or Godot — see the setup guides for those engines
- You're prototyping or doing a game jam — Quick Flow is faster

---

## Prerequisites

> **Before starting:**
> - BMad Method installed with BMGD module enabled
> - Unreal Engine 5.x installed
> - Basic familiarity with Unreal (Blueprints or C++)
> - A game concept or idea you want to develop

---

## Steps

### Step 1: Create your Unreal project

1. Open the **Epic Games Launcher**
2. Go to **Unreal Engine** tab
3. Click **Launch** to open Unreal Editor
4. In the Project Browser, click **Games** → **Next**
5. Select the appropriate template:
   - **Blank** — Start from scratch (most flexible)
   - **Third Person** — Character-based games
   - **First Person** — FPS/exploration games
   - **Top Down** — Strategy and arcade games
6. Choose **Blueprint** or **C++**
7. Name your project, choose a location, and click **Create**

### Step 2: Generate your project context

BMGD uses a `project-context.md` file to maintain consistency across all workflows.

In your BMad-enabled environment at the project root:

```
/bmgd-generate-project-context
```

This invokes the **Game Architect (Cloud Dragonborn)** to create a `project-context.md` file that includes:
- Project name and description
- Target platforms (PC, console, mobile)
- Engine and framework choices (Blueprint vs C++)
- Performance budgets
- Critical technical decisions

### Step 3: Run the brainstorming workflow

Define your game concept with the Game Designer agent.

```
/bmgd-brainstorm-game
```

The **Game Designer (Samus Shepard)** will guide you through:
- Selecting and combining brainstorming techniques
- Generating and refining game ideas
- Choosing a concept to develop

### Step 4: Create your Game Brief

Capture your vision and positioning.

```
/bmgd-game-brief
```

The Game Designer creates `game-brief.md` with:
- Game vision and elevator pitch
- Target audience and market positioning
- Platform and genre decisions
- Competitive analysis
- Art and audio direction

### Step 5: Design your game (GDD)

Create a comprehensive Game Design Document.

```
/bmgd-create-gdd
```

The Game Designer helps you:
- Select your game type from 24 available templates
- Define core gameplay mechanics
- Design progression systems
- Plan levels and content
- Specify art and audio requirements

Output: `gdd.md`

### Step 6: Create your technical architecture

Plan your Unreal project structure and systems.

```
/bmgd-create-architecture
```

The **Game Architect (Cloud Dragonborn)** creates `architecture.md` with:
- Project structure (Content Browser organization)
- System architecture (game framework, replication, networking)
- Unreal-specific patterns (Components, Game Instances, Gameplay Abilities)
- Performance budgets and optimization strategy
- Asset pipeline and build configuration

### Step 7: Plan your first sprint

Ready to start building? Use the Game Scrum Master to plan your work.

```
/bmgd-sprint-planning
```

The **Game Scrum Master (Max)** creates:
- `sprint-status.yaml` — Your sprint tracking file
- Stories from your GDD and Architecture
- Sprint goals and definition of done

### Step 8: Start implementing

Build features with the Game Developer agent.

```
/bmgd-dev-story [story-name]
```

The **Game Developer (Link Freeman)** helps you:
- Implement story tasks in Blueprints or C++
- Follow Unreal best practices
- Write tests for your features
- Review code before marking complete

---

## What You Get

After completing this setup, you'll have:

| File/Folder | Purpose |
|------------|---------|
| `project-context.md` | AI context for consistency across all BMGD workflows |
| `game-brief.md` | Your game's vision and positioning |
| `gdd.md` | Complete game design document |
| `architecture.md` | Technical architecture and Unreal-specific patterns |
| `sprint-status.yaml` | Sprint tracking with stories and progress |
| `stories/` | Folder containing individual story files |

---

## Unreal-Specific Considerations

### Project Structure

BMGD recommends this Content Browser structure:

```
Content/
├── Game/
│   ├── Blueprints/
│   ├── Materials/
│   ├── Meshes/
│   ├── Textures/
│   ├── Audio/
│   └── UI/
├── Developers/
│   └── [YourName]/
└── Collections/
```

### Blueprint vs C++

Your architecture should specify:

| Approach | When to Use |
|----------|-------------|
| **Blueprints** | Rapid prototyping, gameplay logic, designer-iterable systems |
| **C++** | Performance-critical systems, complex algorithms, platform-specific features |
| **Mixed** | C++ for systems, Blueprints for gameplay (common approach) |

The Game Architect will recommend the right mix for your project.

### Performance Budgets

Unreal projects typically target:
- **60 FPS** for most console/PC games
- **30 FPS** for open-world games with high draw distance
- **120 FPS** for competitive shooters

Your `architecture.md` will specify frame time budgets (ms per frame).

### Testing Setup

For Unreal automated testing, the Game QA agent (GLaDOS) can help:

```
/bmgd-test-framework
```

This sets up Unreal Automation System with:
- Unit tests (C++ and Blueprint function libraries)
- Functional tests (gameplay systems)
- Performance tests (frame rate, memory)

---

## Tips

> **Best Practice:** Always run `gds-generate-project-context` after creating a new Unreal project. The `project-context.md` file is the "single source of truth" for project decisions, and the production workflows read it automatically. The conversational agents pick it up only if you add it to their `persistent_facts`.

> **Avoid:** Don't start with the First Person template if you're making a third-person game. Choose the template closest to your final game — the Game Architect can advise if unsure.

> **Remember:** Unreal projects are larger than Unity projects. Clean up unused content early to keep your project manageable.

---

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Skipping project-context generation | Always generate `project-context.md` first — it guides all other workflows |
 Choosing the wrong template | Consult the Game Architect — starting from Blank is often cleaner than refactoring a template |
 Ignoring Unreal's project structure | Follow Content Browser organization from your architecture — don't create custom folder structures |
 Not using Unreal's built-in systems | Use Gameplay Abilities, Gameplay Tags, and Data Assets — don't reinvent the wheel |

---

## Next Steps

- **[Quick Flow vs Full Production](../explanation/quick-flow-vs-full.md)** — Understand both development approaches
- **[Set up Unity with BMGD](./setup-unity.md)** — If you're considering Unity instead
- **[Run sprint planning](./sprint-planning.md)** — When you're ready to start building
- **[Agents Reference](../reference/agents.md)** — Learn about all 6 BMGD agents
