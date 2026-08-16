---
title: "Set up a Godot project with BMGD"
description: Configure a new Godot project for full production game development with BMGD
---

# Set up a Godot project with BMGD

Configure a new Godot project with BMGD workflows for full production game development.

---

## When to Use This Guide

- You're starting a new Godot game project
- You want to use BMGD's Full Production workflow
- You need formal documentation (GDD, architecture, sprint tracking)

---

## When to Skip This

- You just want to prototype quickly — use [Quick Flow](../tutorials/first-game-project.md) instead
- You're using Unity or Unreal — see the setup guides for those engines
- You're prototyping or doing a game jam — Quick Flow is faster

---

## Prerequisites

> **Before starting:**
> - BMad Method installed with BMGD module enabled
> - Godot 4.2+ (recommended) or Godot 3.x installed
> - Basic familiarity with Godot and GDScript
> - A game concept or idea you want to develop

---

## Steps

### Step 1: Create your Godot project

1. Open **Godot Project Manager**
2. Click **New Project**
3. Browse to your desired folder location
4. Choose a renderer:
   - **Forward+** — Modern PBR, best for 3D games
   - **Mobile** — Optimized for mobile platforms
   - **Compatibility** — GLES2, for older hardware
5. Name your project folder
6. Click **Create & Edit**

### Step 2: Generate your project context

BMGD uses a `project-context.md` file to maintain consistency across all workflows.

In your BMad-enabled environment at the project root:

```
/bmgd-generate-project-context
```

This invokes the **Game Architect (Cloud Dragonborn)** to create a `project-context.md` file that includes:
- Project name and description
- Target platforms (PC, mobile, web)
- Engine version and renderer choice
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

Plan your Godot project structure and systems.

```
/bmgd-create-architecture
```

The **Game Architect (Cloud Dragonborn)** creates `architecture.md` with:
- Project structure (scenes, scripts, resources)
- System architecture (game loop, nodes, signals, autoloads)
- Godot-specific patterns (tree organization, resource management)
- Performance budgets and optimization strategy
- Export configurations and platform settings

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
- Implement story tasks with GDScript
- Follow Godot best practices
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
| `architecture.md` | Technical architecture and Godot-specific patterns |
| `sprint-status.yaml` | Sprint tracking with stories and progress |
| `stories/` | Folder containing individual story files |

---

## Godot-Specific Considerations

### Project Structure

BMGD recommends this Godot project structure:

```
res://
├── scenes/
│   ├── levels/
│   ├── characters/
│   ├── ui/
│   └── components/
├── scripts/
│   ├── autoload/
│   └── utils/
├── assets/
│   ├── art/
│   ├── audio/
│   └── data/
└── project.godot
```

### Scene Organization

Godot uses a tree of nodes. Your architecture should define:

- **Autoload Singletons** — Global managers (game state, audio, saves)
- **Scene Composition** — Reusable scenes (player, enemies, pickups)
- **Signal Patterns** — Decoupled communication between nodes

The Game Architect will specify these patterns in your `architecture.md`.

### Performance Budgets

Godot projects typically target:
- **60 FPS** for most games
- **30 FPS** for mobile with heavy computation
- **144+ FPS** for competitive games

Your `architecture.md` will specify frame time budgets.

### Testing Setup

For Godot automated testing, the Game QA agent (GLaDOS) can help:

```
/bmgd-test-framework
```

This sets up GUT (Godot Unit Test) with:
- Unit tests (script logic without scenes)
- Integration tests (scene interactions)
- Test fixtures and test doubles

---

## Tips

> **Best Practice:** Always run `gds-generate-project-context` after creating a new Godot project. The `project-context.md` file is the "single source of truth" for project decisions, and the production workflows read it automatically. The conversational agents pick it up only if you add it to their `persistent_facts`.

> **Avoid:** Don't create deeply nested scene trees. Godot's scene system encourages composition — prefer many small scenes over one large scene.

> **Remember:** Godot uses GDScript (Python-like) by default. Your architecture should specify if you're using C# or GDScript — both are supported but have different workflows.

---

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Skipping project-context generation | Always generate `project-context.md` first — it guides all other workflows |
 Creating monolithic scenes | Break your game into reusable scene components — the Game Architect can help design this |
 Not using autoloads properly | Keep autoloads minimal — only use for true singletons (game state, save system) |
 Mixing GDScript and C# arbitrarily | Choose one primary language and stick with it — mixing adds complexity without benefit |

---

## Next Steps

- **[Quick Flow vs Full Production](../explanation/quick-flow-vs-full.md)** — Understand both development approaches
- **[Set up Unity with BMGD](./setup-unity.md)** — If you're considering Unity instead
- **[Run sprint planning](./sprint-planning.md)** — When you're ready to start building
- **[Agents Reference](../reference/agents.md)** — Learn about all 6 BMGD agents
