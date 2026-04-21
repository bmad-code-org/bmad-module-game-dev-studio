---
name: gds-test-framework
description: 'Initialize game test framework for Unity, Unreal, or Godot. Use when the user says "test framework" or "set up testing"'
---

# Game Test Framework Setup

**Workflow ID**: `gds-test-framework`
**Version**: 1.0 (BMad v6)

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

## Goal

Initialize a production-ready game test framework for Unity, Unreal Engine, or Godot projects. This workflow scaffolds the complete testing infrastructure including unit tests, integration tests, and play mode tests appropriate for the detected game engine.

## Role

You are a Game QA Architect specializing in test infrastructure. You detect the game engine in use, scaffold the appropriate test framework, generate working example tests, and produce documentation — leaving the team with a fully operational testing setup from day one.

---

## WORKFLOW ARCHITECTURE

This workflow detects the game engine and creates all necessary test infrastructure files directly in the game project.

**Primary Output**: `{test_dir}/README.md`

**Supporting Components**:
- Validation: `{installed_path}/checklist.md`

**Knowledge Base References**:
- `knowledge/unity-testing.md`
- `knowledge/unreal-testing.md`
- `knowledge/godot-testing.md`


Load and resolve configuration from `{module_config}`:

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
test_dir: "{project-root}/tests"    # Root test directory
game_engine: "auto"                  # auto | unity | unreal | godot
test_framework: "auto"               # auto | gut (godot) | unity-test-framework | unreal-automation
```

---

## EXECUTION

### Preflight Requirements

**Critical:** Verify these requirements before proceeding. If any fail, HALT and notify the user.

- Game project exists with identifiable engine
- No test framework already configured (check for existing test directories)
- Project structure is accessible

---

### Step 1: Detect Game Engine

#### Actions

1. **Identify Engine Type**

   Look for engine-specific files:
   - **Unity**: `Assets/`, `ProjectSettings/ProjectSettings.asset`, `*.unity` scene files
   - **Unreal**: `*.uproject`, `Source/`, `Config/DefaultEngine.ini`
   - **Godot**: `project.godot`, `*.tscn`, `*.gd` files

2. **Verify Engine Version**
   - Unity: Check `ProjectSettings/ProjectVersion.txt`
   - Unreal: Check `*.uproject` file for `EngineAssociation`
   - Godot: Check `project.godot` for `config_version`

3. **Check for Existing Test Framework**
   - Unity: Check for `Tests/` folder, `*.Tests.asmdef`
   - Unreal: Check for `Tests/` in Source, `*Tests.Build.cs`
   - Godot: Check for `tests/` folder, GUT plugin in `addons/gut/`

**Halt Condition:** If existing framework detected, offer upgrade path or HALT.

---

### Step 2: Scaffold Framework

#### Unity Test Framework

**Knowledge Base Reference**: `knowledge/unity-testing.md`

1. **Create Directory Structure**

   ```
   Assets/
   ├── Tests/
   │   ├── EditMode/
   │   │   ├── EditModeTests.asmdef
   │   │   └── ExampleEditModeTest.cs
   │   └── PlayMode/
   │       ├── PlayModeTests.asmdef
   │       └── ExamplePlayModeTest.cs
   ```

2. **Generate Assembly Definitions**

   `EditModeTests.asmdef`:

   ```json
   {
     "name": "EditModeTests",
     "references": ["<GameAssembly>"],
     "includePlatforms": ["Editor"],
     "defineConstraints": ["UNITY_INCLUDE_TESTS"],
     "optionalUnityReferences": ["TestAssemblies"]
   }
   ```

   `PlayModeTests.asmdef`:

   ```json
   {
     "name": "PlayModeTests",
     "references": ["<GameAssembly>"],
     "includePlatforms": [],
     "defineConstraints": ["UNITY_INCLUDE_TESTS"],
     "optionalUnityReferences": ["TestAssemblies"]
   }
   ```

3. **Generate Sample Tests**

   Edit Mode test example:

   ```csharp
   using NUnit.Framework;

   [TestFixture]
   public class DamageCalculatorTests
   {
       [Test]
       public void Calculate_BaseDamage_ReturnsCorrectValue()
       {
           // Arrange
           var calculator = new DamageCalculator();

           // Act
           float result = calculator.Calculate(100f, 1f);

           // Assert
           Assert.AreEqual(100f, result);
       }
   }
   ```

   Play Mode test example:

   ```csharp
   using System.Collections;
   using NUnit.Framework;
   using UnityEngine;
   using UnityEngine.TestTools;

   public class PlayerMovementTests
   {
       [UnityTest]
       public IEnumerator Player_WhenInputApplied_Moves()
       {
           // Arrange
           var playerGO = new GameObject("Player");
           var controller = playerGO.AddComponent<PlayerController>();

           // Act
           controller.SetMoveInput(Vector2.right);
           yield return new WaitForSeconds(0.5f);

           // Assert
           Assert.Greater(playerGO.transform.position.x, 0f);

           // Cleanup
           Object.Destroy(playerGO);
       }
   }
   ```

---

#### Unreal Engine Automation

**Knowledge Base Reference**: `knowledge/unreal-testing.md`

1. **Create Directory Structure**

   ```
   Source/
   ├── <ProjectName>/
   │   └── ...
   └── <ProjectName>Tests/
       ├── <ProjectName>Tests.Build.cs
       └── Private/
           ├── DamageCalculationTests.cpp
           └── PlayerCombatTests.cpp
   ```

2. **Generate Module Build File**

   `<ProjectName>Tests.Build.cs`:

   ```csharp
   using UnrealBuildTool;

   public class <ProjectName>Tests : ModuleRules
   {
       public <ProjectName>Tests(ReadOnlyTargetRules Target) : base(Target)
       {
           PCHUsage = ModuleRules.PCHUsageMode.UseExplicitOrSharedPCHs;

           PublicDependencyModuleNames.AddRange(new string[] {
               "Core",
               "CoreUObject",
               "Engine",
               "<ProjectName>"
           });

           PrivateDependencyModuleNames.AddRange(new string[] {
               "AutomationController"
           });
       }
   }
   ```

3. **Generate Sample Tests**

   ```cpp
   #include "Misc/AutomationTest.h"

   IMPLEMENT_SIMPLE_AUTOMATION_TEST(
       FDamageCalculationTest,
       "<ProjectName>.Combat.DamageCalculation",
       EAutomationTestFlags::ApplicationContextMask |
       EAutomationTestFlags::ProductFilter
   )

   bool FDamageCalculationTest::RunTest(const FString& Parameters)
   {
       // Arrange
       float BaseDamage = 100.f;
       float CritMultiplier = 2.f;

       // Act
       float Result = UDamageCalculator::Calculate(BaseDamage, CritMultiplier);

       // Assert
       TestEqual("Critical hit doubles damage", Result, 200.f);

       return true;
   }
   ```

---

#### Godot GUT Framework

**Knowledge Base Reference**: `knowledge/godot-testing.md`

1. **Create Directory Structure**

   ```
   project/
   ├── addons/
   │   └── gut/ (plugin files)
   ├── tests/
   │   ├── unit/
   │   │   └── test_damage_calculator.gd
   │   └── integration/
   │       └── test_player_combat.gd
   └── gut_config.json
   ```

2. **Generate GUT Configuration**

   `gut_config.json`:

   ```json
   {
     "dirs": ["res://tests/"],
     "include_subdirs": true,
     "prefix": "test_",
     "suffix": ".gd",
     "should_exit": true,
     "should_exit_on_success": true,
     "log_level": 1,
     "junit_xml_file": "results.xml"
   }
   ```

3. **Generate Sample Tests**

   `tests/unit/test_damage_calculator.gd`:

   ```gdscript
   extends GutTest

   var calculator: DamageCalculator

   func before_each():
       calculator = DamageCalculator.new()

   func after_each():
       calculator.free()

   func test_calculate_base_damage():
       var result = calculator.calculate(100.0, 1.0)
       assert_eq(result, 100.0, "Base damage should equal input")

   func test_calculate_critical_hit():
       var result = calculator.calculate(100.0, 2.0)
       assert_eq(result, 200.0, "Critical hit should double damage")
   ```

---

### Step 3: Generate Documentation

Create `tests/README.md` with:

- Test framework overview for the detected engine
- Directory structure explanation
- Running tests locally
- CI integration commands
- Best practices for game testing
- Links to knowledge base fragments

---

### Step 4: Deliverables

#### Primary Artifacts Created

1. **Directory Structure** — Engine-appropriate test folders
2. **Configuration Files** — Framework-specific config (asmdef, Build.cs, gut_config.json)
3. **Sample Tests** — Working examples for unit and integration tests
4. **Documentation** — `tests/README.md`

---

## Output Summary

After completing this workflow, provide a summary:

```markdown
## Game Test Framework Scaffold Complete

**Engine Detected**: {Unity | Unreal | Godot}
**Framework**: {Unity Test Framework | Unreal Automation | GUT}

**Artifacts Created**:

- Test directory structure
- Framework configuration
- Sample unit tests
- Sample integration/play mode tests
- Documentation

**Next Steps**:

1. Review sample tests and adapt to your game
2. Run initial tests to verify setup
3. Use `test-design` workflow to plan comprehensive test coverage
4. Use `automate` workflow to generate additional tests

**Knowledge Base References Applied**:

- {engine}-testing.md
- qa-automation.md
- test-priorities.md
```

---

## Validation

Refer to `checklist.md` for comprehensive validation criteria.

## On Complete

Run: `python3 {project-root}/_bmad/scripts/resolve_customization.py --skill {skill-root} --key workflow.on_complete`

If the resolved `workflow.on_complete` is non-empty, follow it as the final terminal instruction before exiting.
