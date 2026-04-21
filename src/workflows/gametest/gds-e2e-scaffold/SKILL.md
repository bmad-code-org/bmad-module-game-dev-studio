---
name: gds-e2e-scaffold
description: 'Scaffold end-to-end testing infrastructure. Use when the user says "e2e scaffold" or "set up e2e testing"'
---

# E2E Test Infrastructure Scaffold Workflow

**Goal:** Scaffold complete E2E testing infrastructure for an existing game project — creating the foundation required for reliable, maintainable end-to-end tests: test fixtures, scenario builders, input simulators, and async assertion utilities, all tailored to the project's specific architecture.

**Your Role:** You are a senior game QA engineer specializing in E2E test architecture. E2E tests validate complete player journeys. Without proper infrastructure, they become brittle nightmares. Your job is to prevent that by building the right foundation before a single test is written. Work with the user to understand their architecture and generate infrastructure that fits their game's domain.

---

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

### Step 5: Greet the User

Greet `{user_name}`, speaking in `{communication_language}`.

### Step 6: Execute Append Steps

Execute each entry in `{workflow.activation_steps_append}` in order.

Activation is complete. Begin the workflow below.

## WORKFLOW ARCHITECTURE

This uses an **inline workflow pattern** for autonomous execution:

- Steps execute sequentially with critical architecture analysis upfront
- Engine detection and domain discovery drive all generated code
- All infrastructure files are written to disk as they are generated
- A working example test proves the infrastructure functions correctly

### Triggers

- `ES`
- `e2e-scaffold`
- `scaffold e2e`
- `e2e infrastructure`
- `setup e2e`

### Preflight Requirements

**Critical:** Verify these requirements before proceeding. If any fail, HALT and guide the user.

- Test framework already initialized (run `test-framework` workflow first)
- Game has identifiable state manager class
- Main gameplay scene exists and is functional
- No existing E2E infrastructure (check for `Tests/PlayMode/E2E/` or engine equivalent)


### Paths

- `installed_path` = `{skill-root}`
- `validation` = `{installed_path}/checklist.md`

### Inputs (Collect from User or Auto-Detect)

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `game_state_class` | Primary game state manager class name | Yes | — |
| `main_scene` | Scene name where core gameplay occurs | Yes | — |
| `input_system` | Input system in use | No | `auto-detect` |

### Knowledge Fragments

Load `{installed_path}/knowledge/e2e-testing.md` before proceeding. Load the engine-specific fragment after detection in Step 1:

- Unity: `{installed_path}/knowledge/unity-testing.md`
- Unreal: `{installed_path}/knowledge/unreal-testing.md`
- Godot: `{installed_path}/knowledge/godot-testing.md`

---

## EXECUTION

<workflow>

<step n="1" goal="Analyze Game Architecture">
  <action>Detect Game Engine by checking for engine-specific project files:
    - Unity: `Assets/`, `ProjectSettings/`, `*.unity` scenes
    - Unreal: `*.uproject`, `Source/`, `Config/DefaultEngine.ini`
    - Godot: `project.godot`, `*.tscn`, `*.gd` files
  </action>
  <action>Load the appropriate engine-specific knowledge fragment</action>

  <action>Identify core systems:
    1. Game State Manager — the primary class holding game state. Look for: `GameManager`, `GameStateManager`, `GameController`, `GameMode`. Note: initialization method, ready state property, save/load methods
    2. Input Handling — Unity New Input System vs Legacy, Unreal Enhanced Input vs Legacy, Godot built-in Input, or custom abstraction layer
    3. Event/Messaging System — event bus, C# events/delegates, UnityEvents, Godot Signals
    4. Scene Structure — main gameplay scene name, loading approach (additive/single), bootstrap/initialization flow
  </action>

  <action>Identify domain concepts for the ScenarioBuilder:
    - Primary Entities: units, players, items, enemies, etc.
    - State Machine States: turn phases, game modes, player states
    - Spatial System: grid/hex positions, world coordinates, regions
    - Resources: currency, health, mana, ammunition, etc.
  </action>

  <action>Check existing test structure. If `Tests/PlayMode/E2E/` (or engine equivalent) already exists, HALT and ask user how to proceed.</action>
</step>

<step n="2" goal="Generate Infrastructure">
  <action>Create the E2E directory structure:
```
Tests/PlayMode/E2E/          (Unity)
├── E2E.asmdef
├── Infrastructure/
│   ├── GameE2ETestFixture.cs
│   ├── ScenarioBuilder.cs
│   ├── InputSimulator.cs
│   └── AsyncAssert.cs
├── Scenarios/
│   └── (empty - user will add tests here)
├── TestData/
│   └── (empty - user will add fixtures here)
└── README.md
```
  </action>

  <!-- Unity-specific infrastructure -->
  <check if="engine == 'unity'">
    <action>Generate Assembly Definition `E2E.asmdef`:
```json
{
  "name": "E2E",
  "rootNamespace": "{ProjectNamespace}.Tests.E2E",
  "references": ["{GameAssemblyName}", "Unity.InputSystem", "Unity.InputSystem.TestFramework"],
  "includePlatforms": [],
  "excludePlatforms": [],
  "allowUnsafeCode": false,
  "overrideReferences": true,
  "precompiledReferences": ["nunit.framework.dll", "UnityEngine.TestRunner.dll", "UnityEditor.TestRunner.dll"],
  "autoReferenced": false,
  "defineConstraints": ["UNITY_INCLUDE_TESTS"],
  "versionDefines": [],
  "noEngineReferences": false
}
```
    Replace `{ProjectNamespace}` with detected project namespace and `{GameAssemblyName}` with main game assembly. Include `Unity.InputSystem` references only if Input System package detected.
    </action>

    <action>Generate `GameE2ETestFixture.cs` base class. Customize placeholders:
      - `{Namespace}` = detected project namespace
      - `{MainSceneName}` = detected main gameplay scene
      - `{GameStateClass}` = identified game state manager class
      - `{IsReadyProperty}` = property indicating game is initialized (e.g., `IsReady`, `IsInitialized`)
    The fixture must handle: scene loading/unloading, game ready state waiting, access to GameState/Input/Scenario, cleanup guarantees, and failure screenshot capture.</action>

    <action>Generate `ScenarioBuilder.cs` with fluent API. Analyze the game's domain model from Step 1 and add 3-5 concrete configuration methods based on identified entities. Include `FromSaveFile(string fileName)` as base method. Add domain-specific methods in the `#region State Configuration` block with TODO comments documenting the pattern.</action>

    <action>Generate `InputSimulator.cs`. If New Input System detected:
      - `ClickWorldPosition(Vector3)`, `ClickScreenPosition(Vector2)`, `ClickButton(string)`, `DragFromTo(Vector3, Vector3, float)` using `InputState.Change` and `StateEvent.From`
      - `PressKey(Key)`, `HoldKey(Key, float)` for keyboard
      - `Reset()` and `RefreshCamera()` utility methods
    If Legacy Input detected, generate simpler version using UI event triggering.</action>

    <action>Generate `AsyncAssert.cs` static utility class with:
      - `WaitUntil(Func<bool>, string, float)` — core wait-for-condition
      - `WaitUntilVerbose(...)` — with periodic debug logging
      - `WaitForValue<T>(...)` — wait for specific value (exact equality)
      - `WaitForValueApprox(...)` — float/double with tolerance
      - `WaitForValueNot<T>(...)` — wait for value to change
      - `WaitForNotNull<T>(...)` and `WaitForUnityObject<T>(...)`
      - `AssertNeverTrue(...)` — assert something doesn't happen
      - `WaitFrames(int)` and `WaitForPhysics(int)` utility methods
    </action>
  </check>

  <!-- Unreal-specific infrastructure -->
  <check if="engine == 'unreal'">
    <action>Generate equivalent infrastructure files under `Source/{ProjectName}/Tests/E2E/`:
      - `GameE2ETestBase.h/.cpp` — base test class
      - `ScenarioBuilder.h/.cpp` — fluent scenario configuration
      - `InputSimulator.h/.cpp` — input abstraction
      - `AsyncAssert.h` — wait-for-condition utilities
      - `{ProjectName}E2ETests.Build.cs` — build configuration
    </action>
  </check>

  <!-- Godot-specific infrastructure -->
  <check if="engine == 'godot'">
    <action>Generate equivalent infrastructure files under `tests/e2e/infrastructure/`:
      - `game_e2e_test_fixture.gd`
      - `scenario_builder.gd`
      - `input_simulator.gd`
      - `async_assert.gd`
    </action>
  </check>

  <action>Write all infrastructure files to disk</action>
</step>

<step n="3" goal="Generate Example Test">
  <action>Create a working E2E test that exercises the infrastructure and proves it works</action>

  <check if="engine == 'unity'">
    <action>Generate `ExampleE2ETest.cs` with three tests:
      1. `Infrastructure_GameLoadsAndReachesReadyState` — verifies base fixture, GameState, Input, Scenario are all non-null and game reaches ready state
      2. `Infrastructure_InputSimulatorCanClickButtons` — demonstrates input simulation pattern with commented example for the user to customize
      3. `Infrastructure_ScenarioBuilderCanConfigureState` — demonstrates ScenarioBuilder usage with commented domain-specific example
    Apply `[Category("E2E")]` attribute to the class.
    </action>
  </check>

  <check if="engine in ('unreal', 'godot')">
    <action>Generate equivalent example test in the engine-appropriate location and language, covering the same three verification patterns</action>
  </check>

  <action>Write example test file to disk</action>
</step>

<step n="4" goal="Generate Documentation">
  <action>Create `README.md` in the E2E root directory covering:
    - Quick Start (inherit from GameE2ETestFixture, use Scenario/Input/AsyncAssert)
    - Example test using Given-When-Then structure
    - Component documentation for GameE2ETestFixture, ScenarioBuilder, InputSimulator, AsyncAssert
    - Directory structure explanation
    - Running tests (Editor UI and command line)
    - Best practices (wait for conditions not time, one journey per test, descriptive assertions)
    - Extension guide (adding Scenario methods, adding Input methods)
    - Troubleshooting table for common issues
  </action>
  <action>Write README.md to disk</action>
</step>

<step n="5" goal="Output Summary">
  <action>Load and apply `{validation}` checklist to verify all deliverables are complete</action>
  <action>Present a summary to the user:

```markdown
## E2E Infrastructure Scaffold Complete

**Engine**: {Unity | Unreal | Godot}
**Version**: {detected_version}

### Files Created

[Directory tree of all created files]

### Configuration

| Setting | Value |
|---------|-------|
| Game State Class | `{GameStateClass}` |
| Main Scene | `{MainSceneName}` |
| Input System | `{InputSystemType}` |
| Ready Property | `{IsReadyProperty}` |

### Customization Required

1. ScenarioBuilder: Add domain-specific setup methods for your game entities
2. InputSimulator: Add game-specific input methods (e.g., hex clicking, gesture shortcuts)
3. ExampleE2ETest: Modify example tests to use your actual UI elements

### Next Steps

1. Run `Infrastructure_GameLoadsAndReachesReadyState` to verify setup works
2. Extend `ScenarioBuilder` with your domain methods
3. Extend `InputSimulator` with game-specific input helpers
4. Use `test-design` workflow to identify E2E scenarios
5. Use `automate` workflow to generate E2E tests from scenarios
```
  </action>
<action>Run: `python3 {project-root}/_bmad/scripts/resolve_customization.py --skill {skill-root} --key workflow.on_complete` — if the resolved value is non-empty, follow it as the final terminal instruction before exiting.</action>
</step>

</workflow>
