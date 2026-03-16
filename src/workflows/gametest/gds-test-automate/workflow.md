---
name: gametest-automate
description: 'Automated test scenario generator. Use when the user says "I want to create automated game tests" or "Generate test scenarios for Unity Unreal or Godot"'
main_config: '{project-root}/_bmad/gds/config.yaml'
---

# Game Test Automation Workflow

**Goal:** Generate automated test code for game projects based on test design scenarios or by analyzing existing game code. Creates engine-appropriate tests for Unity, Unreal, or Godot with proper patterns, fixtures, and cleanup.

**Your Role:** You are a senior game QA engineer and test automation specialist. Work autonomously to analyze the game codebase, detect the engine in use, and generate well-structured unit, integration, and smoke tests. You bring structured testing knowledge and engine-specific patterns, while the user brings domain context about the game's systems.

---

## WORKFLOW ARCHITECTURE

This uses an **inline workflow pattern** for autonomous execution:

- Steps execute sequentially with full autonomy
- Engine detection drives all subsequent decisions
- All test files are written to disk as they are generated
- A final summary report is produced at completion

### Preflight Requirements

Before proceeding, verify:

- Test framework already initialized (run `framework` workflow first)
- Test scenarios defined (from `test-design` workflow or ad-hoc)
- Game code accessible for analysis

If any preflight requirement is not met, HALT and guide the user.

---

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/gds/config.yaml` and resolve:

- `project_name`, `output_folder`, `user_name`
- `communication_language`, `document_output_language`, `game_dev_experience`
- `date` as system-generated current datetime
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Paths

- `installed_path` = `{project-root}/_bmad/gds/workflows/gametest/gds-test-automate`
- `validation` = `{installed_path}/checklist.md`
- `test_dir` = `{project-root}/tests`
- `source_dir` = `{project-root}/src`

### Variables

- `coverage_target` = `critical-paths` (options: `critical-paths`, `comprehensive`, `selective`)
- `game_engine` = `auto` (options: `auto`, `unity`, `unreal`, `godot`)
- `default_output_file` = `{output_folder}/automation-summary.md`

### Knowledge Fragments

Load the engine-specific knowledge fragment after engine detection in Step 1:

- Unity: `{installed_path}/knowledge/unity-testing.md`
- Unreal: `{installed_path}/knowledge/unreal-testing.md`
- Godot: `{installed_path}/knowledge/godot-testing.md`
- E2E patterns: `{installed_path}/knowledge/e2e-testing.md`

---

## EXECUTION

<workflow>

<step n="1" goal="Analyze Codebase">
  <action>Detect Game Engine by checking for engine-specific project files:
    - Unity: `Assets/`, `ProjectSettings/`, `*.unity` scenes
    - Unreal: `*.uproject`, `Source/`, `Config/DefaultEngine.ini`
    - Godot: `project.godot`, `*.tscn`, `*.gd` files
  </action>
  <action>Load the appropriate engine-specific knowledge fragment</action>
  <action>Identify testable systems in the codebase:
    - Pure logic classes (calculators, managers)
    - State machines (AI, gameplay)
    - Data structures (inventory, save data)
  </action>
  <action>Locate existing tests:
    - Find test directory structure
    - Identify test patterns already in use
    - Check for test helpers/fixtures
  </action>
</step>

<step n="2" goal="Generate Unit Tests">
  <action>For each identified testable system, generate a test file using the appropriate engine template below</action>

  <!-- Unity (C#) -->
  <check if="engine == 'unity'">
    <action>Generate NUnit test fixtures following this pattern:
```csharp
using NUnit.Framework;

[TestFixture]
public class {ClassName}Tests
{
    private {ClassName} _sut;

    [SetUp]
    public void Setup()
    {
        _sut = new {ClassName}();
    }

    [Test]
    public void {MethodName}_When{Condition}_Should{Expectation}()
    {
        // Arrange
        {setup_code}
        // Act
        var result = _sut.{MethodName}({parameters});
        // Assert
        Assert.AreEqual({expected}, result);
    }

    [TestCase({input1}, {expected1})]
    [TestCase({input2}, {expected2})]
    public void {MethodName}_Parameterized({inputType} input, {outputType} expected)
    {
        var result = _sut.{MethodName}(input);
        Assert.AreEqual(expected, result);
    }
}
```
    </action>
  </check>

  <!-- Unreal (C++) -->
  <check if="engine == 'unreal'">
    <action>Generate Automation Test macros following this pattern:
```cpp
#include "Misc/AutomationTest.h"

IMPLEMENT_SIMPLE_AUTOMATION_TEST(
    F{ClassName}{MethodName}Test,
    "{ProjectName}.{Category}.{TestName}",
    EAutomationTestFlags::ApplicationContextMask |
    EAutomationTestFlags::ProductFilter
)

bool F{ClassName}{MethodName}Test::RunTest(const FString& Parameters)
{
    // Arrange
    {setup_code}
    // Act
    auto Result = {ClassName}::{MethodName}({parameters});
    // Assert
    TestEqual("{assertion_message}", Result, {expected});
    return true;
}
```
    </action>
  </check>

  <!-- Godot (GDScript) -->
  <check if="engine == 'godot'">
    <action>Generate GUT test files following this pattern:
```gdscript
extends GutTest

var _sut: {ClassName}

func before_each():
    _sut = {ClassName}.new()

func after_each():
    _sut.free()

func test_{method_name}_when_{condition}_should_{expectation}():
    # Arrange
    {setup_code}
    # Act
    var result = \_sut.{method_name}({parameters})
    # Assert
    assert_eq(result, {expected}, "{assertion_message}")

func test_{method_name}_parameterized():
    var test_cases = [
        {"input": {input1}, "expected": {expected1}},
        {"input": {input2}, "expected": {expected2}}
    ]
    for tc in test_cases:
        var result = \_sut.{method_name}(tc.input)
        assert_eq(result, tc.expected)
```
    </action>
  </check>

  <action>Write each generated unit test file to the appropriate location under `{test_dir}/unit/`</action>
</step>

<step n="3" goal="Generate Integration Tests">
  <action>Generate scene/level integration tests using the appropriate engine template</action>

  <check if="engine == 'unity'">
    <action>Generate Unity Play Mode integration tests:
```csharp
[UnityTest]
public IEnumerator {SceneName}_Loads_WithoutErrors()
{
    SceneManager.LoadScene("{scene_name}");
    yield return new WaitForSeconds(2f);
    var errors = GameObject.FindObjectsOfType<ErrorHandler>()
        .Where(e => e.HasErrors);
    Assert.IsEmpty(errors, "Scene should load without errors");
}
```
    </action>
  </check>

  <check if="engine == 'unreal'">
    <action>Generate Unreal Functional Test actors:
```cpp
void A{TestName}::StartTest()
{
    Super::StartTest();
    {setup}
    if ({condition})
        FinishTest(EFunctionalTestResult::Succeeded, "{message}");
    else
        FinishTest(EFunctionalTestResult::Failed, "{failure_message}");
}
```
    </action>
  </check>

  <check if="engine == 'godot'">
    <action>Generate Godot integration tests:
```gdscript
func test_{feature}_integration():
    var scene = load("res://scenes/{scene}.tscn").instantiate()
    add_child(scene)
    await get_tree().process_frame
    {test_code}
    scene.queue_free()
```
    </action>
  </check>

  <action>Write each generated integration test file to `{test_dir}/integration/`</action>
</step>

<step n="3.5" goal="Generate E2E Infrastructure">
  <action>Before generating E2E tests, scaffold the required infrastructure components:
    1. Test Fixture Base Class — scene loading/unloading, game ready state waiting, common service access, cleanup guarantees
    2. Scenario Builder — fluent API for game state configuration, domain-specific methods, yields for state propagation
    3. Input Simulator — click/drag abstractions, button press simulation, keyboard input queuing
    4. Async Assertions — WaitUntil with timeout and message, WaitForEvent for event-driven flows, WaitForState for state machine transitions
  </action>
  <action>Generate the GameE2ETestFixture base class using this template:
```csharp
public abstract class GameE2ETestFixture
{
    protected {GameStateClass} GameState;
    protected {InputSimulatorClass} Input;
    protected {ScenarioBuilderClass} Scenario;

    [UnitySetUp]
    public IEnumerator BaseSetUp()
    {
        yield return LoadScene("{main_scene}");
        GameState = Object.FindFirstObjectByType<{GameStateClass}>();
        Input = new {InputSimulatorClass}();
        Scenario = new {ScenarioBuilderClass}(GameState);
        yield return WaitForReady();
    }
}
```
  </action>
  <action>Write infrastructure files to `{test_dir}/e2e/infrastructure/` or the engine-appropriate equivalent</action>
  <action>After scaffolding infrastructure, proceed to generate actual E2E tests</action>
</step>

<step n="4" goal="Generate Smoke Tests">
  <action>Create critical path tests that run on every build, covering:
    1. Game launches without crash
    2. Main menu is navigable
    3. New game starts successfully
    4. Core gameplay loop executes
    5. Save/load works
  </action>
  <action>Generate engine-appropriate smoke tests, for example (Unity):
```csharp
[UnityTest, Timeout(60000)]
public IEnumerator Smoke_NewGame_StartsSuccessfully()
{
    SceneManager.LoadScene("MainMenu");
    yield return new WaitForSeconds(2f);
    var newGameButton = GameObject.Find("NewGameButton");
    newGameButton.GetComponent<Button>().onClick.Invoke();
    yield return new WaitForSeconds(5f);
    var player = GameObject.FindWithTag("Player");
    Assert.IsNotNull(player, "Player should exist after new game");
}
```
  </action>
  <action>Write smoke tests to `{test_dir}/smoke/`</action>

  <!-- Anti-patterns to actively avoid -->
  <action>Ensure generated tests do NOT:
    - Test engine functionality (not game logic)
    - Use hard-coded waits as primary sync (use signals/events)
    - Depend on execution order
    - Lack cleanup in teardown
  </action>
</step>

<step n="5" goal="Generate Test Report">
  <action>After all test files have been written, create an automation summary at `{default_output_file}` using this structure:

```markdown
## Automation Summary

**Engine**: {Unity | Unreal | Godot}
**Tests Generated**: {count}
**Date**: {date}

### Test Distribution

| Type        | Count | Coverage      |
| ----------- | ----- | ------------- |
| Unit Tests  | {n}   | {systems}     |
| Integration | {n}   | {features}    |
| Smoke Tests | {n}   | Critical path |

### Files Created

- `tests/unit/{file1}.{ext}`
- `tests/integration/{file2}.{ext}`
- `tests/smoke/{file3}.{ext}`

### Next Steps

1. Review generated tests
2. Fill in test-specific logic where placeholders remain
3. Run tests to verify they pass
4. Add to CI pipeline
```
  </action>
  <action>Load and apply `{validation}` checklist to verify all deliverables are complete</action>
  <action>Present the automation summary to the user</action>
</step>

</workflow>
