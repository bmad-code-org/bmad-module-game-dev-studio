---
title: "Set up a Unity project with BMGD"
description: Configure a new Unity project for full production game development with BMGD
---

# Set up a Unity project with BMGD

Configure a new Unity project with BMGD workflows for full production game development.

---

## When to Use This Guide

- You're starting a new Unity game project
- You want to use BMGD's Full Production workflow
- You need formal documentation (GDD, architecture, sprint tracking)

---

## When to Skip This

- You just want to prototype quickly — use [Quick Flow](../tutorials/first-game-project.md) instead
- You're using Unreal or Godot — see the setup guides for those engines
- You're prototyping or doing a game jam — Quick Flow is faster

---

## Prerequisites

> **Before starting:**
> - BMad Method installed with BMGD module enabled
> - Unity Hub and Unity 2022 LTS (or later) installed
> - Basic familiarity with Unity and C#
> - A game concept or idea you want to develop

---

## Steps

### Step 1: Create your Unity project

1. Open Unity Hub
2. Click **New Project**
3. Select the appropriate template for your game type:
   - **2D** → Core 2D
   - **3D** → 3D Core
   - **URP** → Universal Render Pipeline (recommended for most games)
   - **HDRP** → High Definition Render Pipeline (high-end visuals)
4. Name your project and choose a location
5. Click **Create Project**

### Step 2: Generate your project context

BMGD uses a `project-context.md` file to maintain consistency across all workflows.

In your BMad-enabled environment at the project root:

```
/bmgd-generate-project-context
```

This invokes the **Game Architect (Cloud Dragonborn)** to create a `project-context.md` file that includes:
- Project name and description
- Target platforms
- Engine and framework choices
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

Plan your Unity project structure and systems.

```
/bmgd-create-architecture
```

The **Game Architect (Cloud Dragonborn)** creates `architecture.md` with:
- Project structure (folders, naming conventions)
- System architecture (game loop, input, physics, networking)
- Unity-specific patterns (ScriptableObjects, events, object pooling)
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
- Implement story tasks with C# scripts
- Follow Unity best practices
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
| `architecture.md` | Technical architecture and Unity-specific patterns |
| `sprint-status.yaml` | Sprint tracking with stories and progress |
| `stories/` | Folder containing individual story files |

---

## Unity-Specific Considerations

### Project Structure

BMGD recommends this Unity project structure:

```
Assets/
├── _Project/
│   ├── Scripts/
│   ├── Art/
│   ├── Audio/
│   └── Data/
├── Packages/
└── ProjectSettings/
```

### ScriptableObjects

Use ScriptableObjects for game data — the Game Architect will include this in your architecture:

- Game configuration
- Character stats
- Item definitions
- Level data

### Performance Budgets

Unity projects typically target:
- **60 FPS** for most platforms
- **30 FPS** for mobile (if targeting battery life)
- **120+ FPS** for VR/high-refresh gaming

Your `architecture.md` will specify your targets.

### Testing Setup

For Unity automated testing, the Game QA agent (GLaDOS) can help:

```
/bmgd-test-framework
```

This sets up Unity Test Framework with:
- Edit Mode tests (logic without running the game)
- Play Mode tests (gameplay systems)
- Test assembly structure

---

## Tips

> **Best Practice:** Always run `gds-generate-project-context` after creating a new Unity project. The `project-context.md` file is the "single source of truth" for project decisions, and the production workflows read it automatically. The conversational agents pick it up only if you add it to their `persistent_facts`.

> **Avoid:** Don't manually organize your Assets folder before running `bmgd-create-architecture`. Let the Game Architect define the structure first, then follow it consistently.

> **Remember:** Unity projects can get large quickly. Use the architecture document to keep your project organized as it grows.

---

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Skipping project-context generation | Always generate `project-context.md` first — it guides all other workflows |
 Choosing the wrong Unity template | Consult the Game Architect if unsure — your engine choice affects architecture |
 Starting implementation before GDD | Complete the Design phase first — changes are cheaper before code is written |
 Ignoring sprint planning | Even solo projects benefit from story tracking — it keeps you focused on ship |

---

## Next Steps

- **[Quick Flow vs Full Production](../explanation/quick-flow-vs-full.md)** — Understand both development approaches
- **[Set up Unreal with BMGD](./setup-unreal.md)** — If you're considering Unreal instead
- **[Run sprint planning](./sprint-planning.md)** — When you're ready to start building
- **[Agents Reference](../reference/agents.md)** — Learn about all 6 BMGD agents
