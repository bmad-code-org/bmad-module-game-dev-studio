# Godot Engine Architecture Knowledge

## Overview

Godot is an open-source game engine using a scene tree architecture where everything is a node. Scenes are reusable node compositions that can be instanced, nested, and inherited. Godot supports GDScript (Python-like), C#, and C++ via GDExtension.

## Architecture Fundamentals

### Scene Tree Model

Godot's architecture is built on a tree of nodes. Every game element — from UI to physics bodies to audio — is a node in the scene tree.

**Key concepts:**

- **Node** — Base building block. Has a name, can have children, receives callbacks (`_ready`, `_process`, `_physics_process`)
- **Scene** — A saved tree of nodes (`.tscn` file). Can be instanced at runtime
- **Autoload** — Singleton nodes loaded before any scene, accessible globally via name
- **Resource** — Data objects (`.tres`) that can be shared across nodes and saved to disk

**Node hierarchy pattern:**

```
Main (Node)
├── World (Node2D / Node3D)
│   ├── Level (Node)
│   │   ├── TileMap / GridMap
│   │   └── SpawnPoints
│   ├── Entities (Node)
│   │   ├── Player (CharacterBody2D/3D)
│   │   └── Enemies (Node)
│   └── Camera2D / Camera3D
├── UI (CanvasLayer)
│   ├── HUD
│   └── Menus
└── Systems (Node)
    ├── AudioManager
    └── GameStateManager
```

### Autoloads vs Dependency Injection

**Autoloads (Singletons):**

Use sparingly for truly global services:

- `GameManager` — Game state, pause, scene transitions
- `AudioManager` — Music and SFX playback
- `SaveManager` — Save/load operations
- `EventBus` — Global signal hub (if using event bus pattern)

**When NOT to use autoloads:**

- Systems only needed in gameplay scenes (use scene composition instead)
- Data that should be passed explicitly (use Resources or method parameters)
- Anything that makes unit testing harder

### Signal Architecture

Signals are Godot's observer pattern implementation. They decouple communication between nodes.

**Best practices:**

```gdscript
# Define signals with typed parameters
signal health_changed(new_health: int, max_health: int)
signal died

# Connect in code (preferred for dynamic connections)
enemy.died.connect(_on_enemy_died)

# Emit with arguments
health_changed.emit(current_health, max_health)
```

**Signal vs direct call guidelines:**

| Use Signals When | Use Direct Calls When |
|---|---|
| Notifying unknown listeners | Calling a known child node |
| Broadcasting state changes | Requesting specific behavior |
| Decoupling siblings | Parent configuring a child |
| Multiple listeners possible | Single target, synchronous |

### Resource-Based Data

Custom Resources are powerful for data-driven design:

```gdscript
# weapon_stats.gd
class_name WeaponStats
extends Resource

@export var damage: float = 10.0
@export var attack_speed: float = 1.0
@export var range: float = 100.0
@export var damage_type: DamageType = DamageType.PHYSICAL

var dps: float:
    get: return damage * attack_speed
```

**Use Resources for:**

- Item definitions, enemy stats, ability data
- Configuration presets
- Shared data between scenes
- Any data that designers should edit in the Inspector

## GDScript vs C# Decision Guide

| Factor | GDScript | C# |
|---|---|---|
| **Learning curve** | Easy, Python-like | Moderate, requires .NET knowledge |
| **Engine integration** | Native, full API access | Full API access, slight overhead |
| **Performance** | Good for gameplay logic | Better for computation-heavy code |
| **Typing** | Optional static typing | Strong static typing |
| **Tooling** | Built-in editor | VS Code, Rider, Visual Studio |
| **Ecosystem** | Godot-native addons | NuGet packages available |
| **Export targets** | All platforms | Most platforms (.NET 6+ required) |
| **Team familiarity** | Godot-specific skill | Transferable C# skill |

**Recommendation:** Use GDScript as default. Use C# when the team has strong C# background or when heavy computation benefits from it. Mixing is supported but adds build complexity.

### GDScript Static Typing

Always use static typing for better performance and error catching:

```gdscript
# Preferred: fully typed
var speed: float = 200.0
var direction: Vector2 = Vector2.ZERO
var inventory: Array[Item] = []
var stats: Dictionary = {}

func take_damage(amount: float, source: Node) -> void:
    health -= amount
    if health <= 0:
        die()

func get_nearest_enemy() -> Enemy:
    # Return type declared
    ...
```

## Coding Best Practices

### Node References

```gdscript
# Use @onready for child references (resolved after _ready)
@onready var sprite: Sprite2D = $Sprite2D
@onready var collision: CollisionShape2D = $CollisionShape2D
@onready var anim_player: AnimationPlayer = $AnimationPlayer

# Use %UniqueNodeName for unique nodes in scene
@onready var health_bar: ProgressBar = %HealthBar

# For nodes in other parts of tree, use get_node or signals
# NEVER use absolute paths like /root/Main/World/Player
```

### Export Variables

```gdscript
# Use @export for Inspector-configurable values
@export var move_speed: float = 200.0
@export var jump_force: float = -400.0
@export var max_health: int = 100

# Export groups for organization
@export_group("Movement")
@export var acceleration: float = 500.0
@export var friction: float = 700.0

@export_group("Combat")
@export var weapon: WeaponStats
@export var armor_rating: int = 0

# Export enums
@export_enum("Warrior", "Mage", "Rogue") var character_class: String

# Export ranges
@export_range(0, 100, 1) var volume: int = 80
```

### State Machine Pattern

```gdscript
# state_machine.gd
class_name StateMachine
extends Node

@export var initial_state: State
var current_state: State

func _ready() -> void:
    for child in get_children():
        if child is State:
            child.state_machine = self
    current_state = initial_state
    current_state.enter()

func _process(delta: float) -> void:
    current_state.update(delta)

func _physics_process(delta: float) -> void:
    current_state.physics_update(delta)

func transition_to(target_state: State) -> void:
    current_state.exit()
    current_state = target_state
    current_state.enter()

# state.gd
class_name State
extends Node

var state_machine: StateMachine

func enter() -> void: pass
func exit() -> void: pass
func update(_delta: float) -> void: pass
func physics_update(_delta: float) -> void: pass
```

### Scene Organization Conventions

```
project/
├── addons/                  # Third-party plugins
├── assets/                  # Raw art, audio, fonts
│   ├── sprites/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   ├── fonts/
│   └── shaders/
├── scenes/                  # .tscn scene files
│   ├── main/
│   ├── levels/
│   ├── ui/
│   └── entities/
├── scripts/                 # .gd script files
│   ├── autoloads/
│   ├── components/
│   ├── resources/
│   ├── state_machines/
│   └── utils/
├── data/                    # .tres resource files, JSON configs
│   ├── items/
│   ├── enemies/
│   └── levels/
└── tests/                   # GUT test files
    ├── unit/
    ├── integration/
    └── e2e/
```

**Naming conventions:**

- Files and folders: `snake_case` (e.g., `player_controller.gd`, `main_menu.tscn`)
- Classes: `PascalCase` (e.g., `class_name PlayerController`)
- Variables and functions: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Signals: `snake_case` past tense (e.g., `health_changed`, `item_collected`)
- Private members: prefix with `_` (e.g., `_internal_state`)

## Performance Optimization

### General Guidelines

- **Use `_physics_process` for movement/physics**, `_process` for visuals and input
- **Avoid `_process` on nodes that don't need per-frame updates** — use signals or timers instead
- **Object pooling** for frequently spawned objects (bullets, particles, enemies)
- **Use `call_deferred`** for operations that shouldn't happen mid-frame

### Rendering Performance

- **Use visibility notifiers** (`VisibleOnScreenNotifier2D/3D`) to disable processing off-screen
- **Limit draw calls** — use texture atlases, merge static geometry
- **For 2D:** Use `CanvasGroup` for batch rendering, minimize `CanvasLayer` count
- **For 3D:** Use LOD (MultiMeshInstance3D), occlusion culling, and the appropriate renderer:
  - **Forward+** — Best for most 3D games
  - **Mobile** — Lower overhead, fewer features
  - **Compatibility (OpenGL)** — Web export, older hardware

### Physics Performance

- **Use `Area2D/3D`** for triggers instead of physics bodies when no collision response needed
- **Simplify collision shapes** — circles/spheres over polygons where possible
- **Collision layers and masks** — configure strictly, don't leave defaults
- **For many static bodies** use `StaticBody` not `RigidBody` with `freeze`

### Memory and Loading

- **Preload vs load:** `preload()` for small, always-needed assets; `load()` or `ResourceLoader.load_threaded_request()` for large or conditional assets
- **Free nodes explicitly** when done: `queue_free()` for deferred, `free()` for immediate
- **Use `ResourceLoader` threaded loading** for seamless scene transitions:

```gdscript
func change_scene_async(path: String) -> void:
    ResourceLoader.load_threaded_request(path)
    # Show loading screen
    while ResourceLoader.load_threaded_get_status(path) != ResourceLoader.THREAD_LOAD_LOADED:
        await get_tree().process_frame
    var scene = ResourceLoader.load_threaded_get(path)
    get_tree().change_scene_to_packed(scene)
```

### GDScript Performance Tips

- **Use static typing everywhere** — typed code runs significantly faster
- **Avoid `get_node()` in loops** — cache references with `@onready`
- **Use `Array.map()`, `filter()`, `reduce()`** carefully — they create closures
- **For hot paths**, consider moving to C# or GDExtension (C++)

## Popular Third-Party Plugins

### Essential Development

| Plugin | Purpose | Source |
|---|---|---|
| **GUT** (Godot Unit Test) | Testing framework | AssetLib or git submodule |
| **Phantom Camera** | Advanced camera system (2D/3D) | AssetLib |
| **Limbo AI** | Behavior trees and state machines for AI | AssetLib or GitHub |
| **Dialogic 2** | Dialogue and timeline system | AssetLib |
| **Terrain3D** | Large-scale 3D terrain editing | GitHub releases |

### Gameplay Systems

| Plugin | Purpose | Source |
|---|---|---|
| **GodotSteam** | Steamworks SDK integration | GitHub / GDExtension |
| **Godot Jolt** | Jolt Physics replacement (faster, more stable 3D) | AssetLib |
| **SmartShape2D** | Procedural 2D terrain/shapes | AssetLib |
| **Maaack's Game Template** | Project template with menus, settings, save system | GitHub / AssetLib |
| **Quest Manager** | Quest/objective tracking | AssetLib |

### Visual and Audio

| Plugin | Purpose | Source |
|---|---|---|
| **Godot Shaders** | Curated shader collection | GitHub |
| **Sprite3D Billboard** | 2D sprites in 3D world | AssetLib |
| **SFXPlayer** | Advanced SFX management with pooling | AssetLib |

### Networking

| Plugin | Purpose | Source |
|---|---|---|
| **Godot Steam Multiplayer Peer** | Steam networking transport | GitHub |
| **Nakama** | Open-source game server (auth, matchmaking, realtime) | Docker + GDScript SDK |
| **WebRTC** | Browser-compatible P2P networking | Built-in module |

### Export and Platform

| Plugin | Purpose | Source |
|---|---|---|
| **Godot Google Play** | Google Play services integration | GitHub |
| **Godot Admob** | Ad monetization for mobile | GitHub |

## Common Architectural Patterns

### Component Pattern (Composition over Inheritance)

```gdscript
# Instead of deep inheritance, compose nodes:
# Player (CharacterBody2D)
#   ├── HealthComponent
#   ├── HitboxComponent
#   ├── HurtboxComponent
#   ├── StateMachine
#   │   ├── IdleState
#   │   ├── RunState
#   │   └── JumpState
#   └── InteractionComponent

# health_component.gd
class_name HealthComponent
extends Node

signal health_changed(current: int, maximum: int)
signal died

@export var max_health: int = 100
var current_health: int

func _ready() -> void:
    current_health = max_health

func take_damage(amount: int) -> void:
    current_health = max(0, current_health - amount)
    health_changed.emit(current_health, max_health)
    if current_health == 0:
        died.emit()

func heal(amount: int) -> void:
    current_health = min(max_health, current_health + amount)
    health_changed.emit(current_health, max_health)
```

### Event Bus Pattern

```gdscript
# autoloads/event_bus.gd
extends Node

# Game flow
signal game_started
signal game_paused
signal game_over(score: int)

# Gameplay
signal enemy_defeated(enemy_type: String, position: Vector2)
signal item_collected(item: Item)
signal level_completed(level_id: String)

# UI
signal show_dialogue(text: String, speaker: String)
signal notification_requested(message: String)
```

### Scene Transition Manager

```gdscript
# autoloads/scene_manager.gd
extends Node

signal scene_changed(scene_name: String)
signal loading_progress(progress: float)

var _current_scene: Node
var _loading_screen: Control

func change_scene(path: String, transition: String = "fade") -> void:
    _show_transition(transition)
    await _transition_midpoint
    _load_scene_async(path)
```

## Godot 4 vs Godot 3 Key Differences

If architecture docs or tutorials reference Godot 3, note these changes:

| Godot 3 | Godot 4 |
|---|---|
| `yield()` | `await` |
| `KinematicBody2D` | `CharacterBody2D` |
| `move_and_slide(velocity)` returns velocity | Set `self.velocity = ...` then call `move_and_slide()` (no args, returns void) |
| `export var` | `@export var` |
| `onready var` | `@onready var` |
| `connect("signal", obj, "method")` | `signal.connect(callable)` |
| `tween = create_tween()` | Same, but `Tween` is refcounted |
| GDNative (C++) | GDExtension (C++) |
| `PoolStringArray` | `PackedStringArray` |

## Licensing and Cost

Godot is licensed under the **MIT License** — fully free and open source with no royalties, no per-seat fees, and no revenue share. You can modify the engine source, use it commercially, and ship on any platform without paying Godot anything.

**Cost implications:**

- Engine: Free forever, no revenue thresholds
- Console export: Requires third-party porting services (see below) — paid
- Plugins: Most community plugins are MIT/free; some Asset Store items are paid
- No vendor lock-in: you own your export templates and can fork the engine

## When NOT to Use Godot

Godot may be the wrong choice when:

- **AAA-quality 3D visuals are required** — Godot's 3D renderer is improving rapidly but does not yet match Unreal's Nanite/Lumen or Unity's HDRP for high-fidelity visuals
- **Console launch is a hard requirement** — Console export requires contracting with specific porting companies (W4 Games, Lone Wolf Technology, Pineapple Works) under NDA, adds cost, and limits release timing. This is a significant business risk for console-first projects
- **Large team with C# expertise and no GDScript willingness** — While Godot supports C#, the ecosystem (plugins, tutorials, community) is overwhelmingly GDScript-oriented
- **Proven production pipeline for 100+ person teams** — Godot's tooling for large-scale team workflows (asset pipelines, custom editors, build systems) is less mature than Unity or Unreal
- **VR/XR is the primary platform** — Godot has OpenXR support but the ecosystem (interaction toolkits, platform SDKs) is substantially less mature than Unity XR Toolkit or Unreal OpenXR

## Console Export Reality

Console export for Godot is substantially more complex than desktop/mobile/web:

- **Not available through the open-source engine** — Console SDKs (Nintendo, Sony, PlayStation) are under NDA and cannot be distributed with Godot
- **Requires a third-party porting company** with console SDK access:
  - **W4 Games** — Founded by Godot core contributors, offers official console porting
  - **Lone Wolf Technology** — Provides Godot console export solutions
  - **Pineapple Works** — Another porting option
- **Cost and timeline:** Porting contracts add development cost and can introduce delays. Budget this early in planning
- **Developer program membership required** — You still need Nintendo Developer, PlayStation Partners, or Xbox ID@Xbox registration independently
- **Testing and certification** — Console certification requirements add QA overhead regardless of engine

**If console is a target:** Factor porting cost and timeline into the architecture decision. Consider whether Unity or Unreal's built-in console export support would reduce risk for your project.

## Architecture Decision Checklist

When choosing Godot for a project, verify:

- [ ] Target platforms supported by Godot export (web, mobile, desktop, consoles via third-party)
- [ ] GDScript vs C# decision based on team skills
- [ ] 2D vs 3D renderer selection (Forward+, Mobile, Compatibility)
- [ ] Physics engine: default Godot Physics or Jolt plugin for 3D
- [ ] Networking approach if multiplayer (built-in MultiplayerAPI, Steam, Nakama)
- [ ] Audio middleware needed or engine-native sufficient
- [ ] Key plugins identified and verified as Godot 4 compatible
- [ ] Console export strategy evaluated (see Console Export Reality section — requires porting company, adds cost and timeline)
