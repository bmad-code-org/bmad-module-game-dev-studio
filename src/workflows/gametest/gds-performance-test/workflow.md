---
name: gametest-performance
description: 'Performance test strategy designer. Use when the user says "lets create a performance test plan" or "design game performance testing strategy"'
main_config: '{project-root}/_bmad/gds/config.yaml'
---

# Performance Testing Strategy Workflow

**Goal:** Design a comprehensive performance testing strategy covering frame rate, memory usage, loading times, and platform-specific requirements. Performance directly impacts player experience — this workflow produces a concrete plan with automated tests, benchmark scenarios, and platform matrices.

**Your Role:** You are a senior game performance engineer and QA strategist. Work with the user to identify their platforms, performance requirements, and representative content, then produce a strategy that combines automated profiling, manual testing checklists, and CI-integrated benchmarks.

---

## WORKFLOW ARCHITECTURE

This uses an **inline workflow pattern** for autonomous execution:

- Steps execute sequentially, building toward a complete performance test plan document
- Platform detection and target configuration drive all subsequent decisions
- The final deliverable is a comprehensive Performance Test Plan document
- Knowledge base reference: `knowledge/performance-testing.md`

### Preflight Requirements

Before proceeding, verify:

- Target platforms identified (or discoverable from project files)
- Performance requirements known (target FPS, memory limits), or to be defined in Step 1
- Representative content available for testing
- Profiling tools accessible

---

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/gds/config.yaml` and resolve:

- `project_name`, `output_folder`, `user_name`
- `communication_language`, `document_output_language`, `game_dev_experience`
- `date` as system-generated current datetime
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Paths

- `installed_path` = `{project-root}/_bmad/gds/workflows/gametest/gds-performance-test`
- `validation` = `{installed_path}/checklist.md`
- `template` = `{installed_path}/performance-template.md`
- `default_output_file` = `{output_folder}/performance-test-plan.md`

### Variables

- `target_fps` = `60` (configurable per platform in Step 1)
- `target_platform` = `auto` (options: `auto`, `pc`, `console`, `mobile`)
- `game_engine` = `auto` (options: `auto`, `unity`, `unreal`, `godot`)

---

## EXECUTION

<workflow>

<step n="1" goal="Define Performance Targets">
  <action>Detect game engine and target platforms from project files. If ambiguous, ask the user.</action>
  <action>Establish frame rate targets per platform:

| Platform          | Target FPS | Minimum FPS | Notes              |
| ----------------- | ---------- | ----------- | ------------------ |
| PC (High)         | 60+        | 30          | Uncapped option    |
| PC (Low)          | 30         | 30          | Scalable settings  |
| PS5/Xbox X        | 60         | 60          | Performance mode   |
| PS4/Xbox One      | 30         | 30          | Locked             |
| Switch Docked     | 30         | 30          | Stable             |
| Switch Handheld   | 30         | 25          | Power saving       |
| Mobile (High)     | 60         | 30          | Device dependent   |
| Mobile (Standard) | 30         | 30          | Thermal throttling |

  Filter this table to the user's actual target platforms. Adjust targets based on game genre and user input.
  </action>
  <action>Establish memory budgets per target platform:

| Platform      | Total RAM | Game Budget | Notes               |
| ------------- | --------- | ----------- | ------------------- |
| PC (Min spec) | 8 GB      | 4 GB        | Leave room for OS   |
| PS5           | 16 GB     | 12 GB       | Unified memory      |
| Xbox Series X | 16 GB     | 13 GB       | With Smart Delivery |
| Switch        | 4 GB      | 2.5 GB      | Tight constraints   |
| Mobile        | 4-6 GB    | 1.5-2 GB    | Background apps     |

  </action>
  <action>Establish loading time targets:

| Scenario     | Target | Maximum |
| ------------ | ------ | ------- |
| Initial boot | < 10s  | 30s     |
| Level load   | < 15s  | 30s     |
| Fast travel  | < 5s   | 10s     |
| Respawn      | < 3s   | 5s      |

  Adjust based on genre (e.g., fast travel may not apply to linear games).
  </action>
</step>

<step n="2" goal="Identify Test Scenarios">
  <action>Define stress test scenarios for frame rate validation:
```
SCENARIO: Maximum Entity Count
  GIVEN game level with normal enemy spawn
  WHEN enemy count reaches 50+
  THEN frame rate stays above minimum
  AND no visual artifacts
  AND audio doesn't stutter

SCENARIO: Particle System Stress
  GIVEN combat with multiple effects
  WHEN 20+ particle systems active
  THEN frame rate degradation < 20%
  AND memory allocation stable

SCENARIO: Draw Call Stress
  GIVEN level with maximum visible geometry
  WHEN camera shows worst-case view
  THEN frame rate stays above minimum
  AND no hitching or stuttering
```
  </action>
  <action>Define memory test scenarios:
```
SCENARIO: Extended Play Session
  GIVEN game running for 4+ hours
  WHEN normal gameplay occurs
  THEN memory usage remains stable
  AND no memory leaks detected
  AND no crash from fragmentation

SCENARIO: Level Transition
  GIVEN player completes level
  WHEN transitioning to new level
  THEN previous level fully unloaded
  AND memory baseline returns
  AND no cumulative growth
```
  </action>
  <action>Define loading test scenarios:
```
SCENARIO: Cold Boot
  GIVEN game not in memory
  WHEN launching game
  THEN reaches interactive state in < target
  AND loading feedback shown
  AND no apparent hang

SCENARIO: Save/Load Performance
  GIVEN large save file (max progress)
  WHEN loading save
  THEN completes in < target
  AND no corruption
  AND gameplay resumes smoothly
```
  </action>
  <action>Adapt scenario details to match the specific game type and identified systems</action>
</step>

<step n="3" goal="Define Test Methodology">
  <action>Generate automated performance test code for the detected engine</action>

  <check if="engine == 'unity'">
    <action>Generate Unity Performance Test Runner examples:
```csharp
[UnityTest]
public IEnumerator Performance_CombatScene_MaintainsFPS()
{
    using (Measure.ProfilerMarkers(new[] { "Main Thread" }))
    {
        SceneManager.LoadScene("CombatStressTest");
        yield return new WaitForSeconds(30f);
    }
    var metrics = Measure.Custom(new SampleGroupDefinition("FPS"));
    Assert.Greater(metrics.Median, 30, "FPS should stay above 30");
}
```
    </action>
  </check>

  <check if="engine == 'unreal'">
    <action>Generate Unreal Automation test examples:
```cpp
bool FPerformanceTest::RunTest(const FString& Parameters)
{
    float StartTime = FPlatformTime::Seconds();
    for (int i = 0; i < 100; i++)
        GetWorld()->SpawnActor<AStressTestActor>();
    float FrameTime = FApp::GetDeltaTime();
    TestTrue("Frame time under budget", FrameTime < 0.033f);
    return true;
}
```
    </action>
  </check>

  <check if="engine == 'godot'">
    <action>Generate Godot benchmark test examples:
```gdscript
func test_performance_entity_stress():
    var frame_times = []
    for i in range(100):
        var entity = stress_entity.instantiate()
        add_child(entity)
    for i in range(300):
        await get_tree().process_frame
        frame_times.append(Performance.get_monitor(Performance.TIME_PROCESS))
    var avg_frame_time = frame_times.reduce(func(a, b): return a + b) / frame_times.size()
    assert_lt(avg_frame_time, 0.033, "Average frame time under 33ms (30 FPS)")
```
    </action>
  </check>

  <action>Define manual profiling checklists:

**CPU Profiling**
- [ ] Identify hotspots using engine profiler
- [ ] Check GC frequency and allocation pressure
- [ ] Verify multithreading usage and thread contention

**GPU Profiling**
- [ ] Draw call count at target scenes
- [ ] Overdraw analysis on complex areas
- [ ] Shader complexity assessment

**Memory Profiling**
- [ ] Heap allocation patterns over a 30-minute session
- [ ] Asset memory usage by category
- [ ] Leak detection across multiple level loads
  </action>
</step>

<step n="4" goal="Create Benchmark Suite">
  <action>Define the benchmark levels and their purpose:

| Benchmark       | Purpose                  | Duration |
| --------------- | ------------------------ | -------- |
| Combat Stress   | Max entities, effects    | 60s      |
| Open World      | Draw distance, streaming | 120s     |
| Menu Navigation | UI performance           | 30s      |
| Save/Load       | Persistence performance  | 30s      |

  Adapt benchmark names and durations to match the actual game content.
  </action>
  <action>Define baseline capture process:
    1. Run benchmarks on reference hardware (document hardware specs)
    2. Record baseline metrics (avg FPS, P95 frame time, peak memory)
    3. Set regression thresholds (e.g., 10% FPS degradation = fail, 5% memory growth = fail)
    4. Integrate benchmarks into CI pipeline as gated checks
  </action>
</step>

<step n="5" goal="Platform-Specific Testing">
  <action>Define platform-specific testing requirements for each target platform</action>

  <check if="platform includes 'pc'">
    <action>PC testing requirements:
      - Test across min/recommended hardware specs
      - Verify quality settings (Low/Medium/High/Ultra) all perform within budget
      - Check VRAM usage at each quality tier
      - Test at multiple resolutions (1080p, 1440p, 4K)
    </action>
  </check>

  <check if="platform includes 'console'">
    <action>Console testing requirements:
      - Test in both Performance and Quality modes if applicable
      - Verify thermal throttling behavior during extended sessions
      - Check suspend/resume impact on frame rate and memory
      - Test with varying storage speeds (internal SSD vs extended storage)
    </action>
  </check>

  <check if="platform includes 'mobile'">
    <action>Mobile testing requirements:
      - Test on low/mid/high tier representative devices
      - Monitor thermal throttling onset time and severity
      - Measure battery drain per hour of gameplay
      - Test with background apps consuming memory
    </action>
  </check>
</step>

<step n="6" goal="Generate Performance Test Plan">
  <action>Load `{template}` and use it as the structural foundation for the output document</action>
  <action>Compile all information from Steps 1-5 into a comprehensive Performance Test Plan at `{default_output_file}` with this structure:

```markdown
# Performance Test Plan: {project_name}

## Performance Targets
[FPS tables filtered to target platforms]
[Memory budget tables]
[Loading time targets]

## Test Scenarios

### Frame Rate Tests
[Stress test scenarios from Step 2]

### Memory Tests
[Extended play and leak detection scenarios]

### Loading Tests
[Boot, level load, save/load scenarios]

## Methodology

### Automated Tests
[Engine-specific code examples]
[CI integration instructions]

### Manual Profiling
[Checklists from Step 3]
[Tools to use per engine]

## Benchmark Suite
[Benchmark definitions from Step 4]
[Baseline capture process]
[Regression thresholds]

## Platform Matrix
[Platform-specific requirements from Step 5]

## Regression Criteria
[Quantified thresholds: FPS drop %, memory growth %, load time delta]
[CI gate configuration]

## Schedule
[When performance tests run: nightly, per-sprint, pre-release]
[Who reviews results and owns regressions]
```
  </action>
  <action>Load and apply `{validation}` checklist to verify all deliverables are complete</action>
  <action>Present a summary of what was produced and the recommended next steps to the user</action>
</step>

</workflow>
