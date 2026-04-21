---
name: gds-test-review
description: 'Review test quality and coverage. Use when the user says "test review" or "review tests"'
---

# Test Review

**Workflow ID**: `gds-test-review`
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
- `output_folder`

### Step 5: Greet the User

Greet `{user_name}`, speaking in `{communication_language}`.

### Step 6: Execute Append Steps

Execute each entry in `{workflow.activation_steps_append}` in order.

Activation is complete. Begin the workflow below.

## Goal

Review existing test suite quality, identify coverage gaps, and recommend improvements. Regular test review prevents test rot and maintains test value over time.

## Role

You are a Game QA Lead with expertise in test suite analysis. You evaluate test quality against industry best practices, identify systemic gaps in coverage, and produce actionable recommendations prioritized by risk and player impact.

---

## WORKFLOW ARCHITECTURE

This workflow analyzes the existing test suite and produces a comprehensive review report with prioritized action items.

**Primary Output**: `{output_folder}/test-review-report.md`

**Supporting Components**:
- Validation: `{installed_path}/checklist.md`
- Template: `{installed_path}/test-review-template.md`

**Knowledge Base References**:
- `knowledge/regression-testing.md`
- `knowledge/test-priorities.md`


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
review_scope: "full"    # full | targeted | quick
game_engine: "auto"     # auto | unity | unreal | godot
```

Search the project for existing test files and results before proceeding.

---

## EXECUTION

### Preflight Requirements

Verify before proceeding:
- Test suite exists (some tests to review)
- Access to test execution results
- Understanding of game features

---

### Step 1: Gather Test Suite Metrics

#### Actions

1. **Count Tests by Type**

   | Type                 | Count | Pass Rate | Avg Duration |
   | -------------------- | ----- | --------- | ------------ |
   | Unit                 |       |           |              |
   | Integration          |       |           |              |
   | Play Mode/Functional |       |           |              |
   | Performance          |       |           |              |
   | **Total**            |       |           |              |

2. **Analyze Test Results**
   - Recent pass rate (last 10 runs)
   - Flaky tests (inconsistent results)
   - Slow tests (> 30s individual)
   - Disabled/skipped tests

3. **Map Coverage**
   - Features with tests
   - Features without tests
   - Critical paths covered

---

### Step 2: Assess Test Quality

#### Quality Criteria

For each test, evaluate:

| Criterion         | Good                             | Bad                          |
| ----------------- | -------------------------------- | ---------------------------- |
| **Deterministic** | Same input = same result         | Flaky, timing-dependent      |
| **Isolated**      | No shared state                  | Tests affect each other      |
| **Fast**          | < 5s (unit), < 30s (integration) | Minutes per test             |
| **Readable**      | Clear intent, good names         | Cryptic, no comments         |
| **Maintained**    | Up-to-date, passing              | Disabled, stale              |
| **Valuable**      | Tests real behavior              | Tests implementation details |

#### Anti-Pattern Detection

Look for these common issues:

```
Hard-coded waits:
   await Task.Delay(5000);        // Bad
   await WaitUntil(() => cond);   // Good

Shared test state:
   static bool wasSetup;          // Dangerous
   [SetUp] void Setup() { ... }   // Good

Testing private implementation:
   var result = obj.GetPrivateField();   // Bad
   var result = obj.PublicBehavior();    // Good

Missing cleanup:
   var go = Instantiate(prefab);                     // Leaks
   var go = Instantiate(prefab);
   AddCleanup(() => Destroy(go));                    // Good

Assertion-free tests:
   void Test() { DoSomething(); }               // What does it test?
   void Test() { DoSomething(); Assert.That(...); }  // Clear
```

---

### Step 3: Identify Coverage Gaps

#### Critical Areas to Verify

| Area          | P0 Coverage | P1 Coverage | Gap? |
| ------------- | ----------- | ----------- | ---- |
| Core Loop     |             |             |      |
| Save/Load     |             |             |      |
| Progression   |             |             |      |
| Combat        |             |             |      |
| UI/Menus      |             |             |      |
| Multiplayer   |             |             |      |
| Platform Cert |             |             |      |

#### Gap Identification Process

1. List all game features
2. Check if each feature has tests
3. Assess test depth (happy path only vs edge cases)
4. Prioritize gaps by risk

---

### Step 4: Review Test Infrastructure

#### Framework Health

- [ ] Tests run in CI
- [ ] Results are visible to team
- [ ] Failures block deployments
- [ ] Test data is versioned
- [ ] Fixtures are reusable
- [ ] Helpers reduce duplication

#### Maintenance Burden

- How often do tests need updates?
- Are updates proportional to code changes?
- Do refactors break tests unnecessarily?

---

### Step 5: Generate Recommendations

#### Priority Matrix

| Finding   | Severity       | Effort         | Recommendation |
| --------- | -------------- | -------------- | -------------- |
| {finding} | {High/Med/Low} | {High/Med/Low} | {action}       |

#### Common Recommendations

**For Flaky Tests**:
- Replace `Thread.Sleep` with explicit waits
- Add proper synchronization
- Isolate test state

**For Slow Tests**:
- Move to nightly builds
- Optimize test setup
- Mock expensive dependencies

**For Coverage Gaps**:
- Prioritize P0/P1 features
- Add smoke tests first
- Use test-design workflow

**For Maintenance Issues**:
- Refactor common patterns
- Create test utilities
- Improve documentation

---

### Step 6: Generate Test Review Report

Write `{output_folder}/test-review-report.md` using the `test-review-template.md` structure:

```markdown
# Test Review Report: {Project Name}

## Executive Summary

- Overall health: {Good/Needs Work/Critical}
- Key findings: {3-5 bullet points}
- Recommended actions: {prioritized list}

## Metrics

### Test Suite Statistics

[Tables from Step 1]

### Recent History

[Pass rates, trends]

## Quality Assessment

### Strengths

- {What's working well}

### Issues Found

| Issue | Severity | Tests Affected | Fix |
| ----- | -------- | -------------- | --- |
|       |          |                |     |

## Coverage Analysis

### Current Coverage

[Feature coverage table]

### Critical Gaps

[Prioritized list of missing coverage]

## Recommendations

### Immediate (This Sprint)

1. {Fix critical issues}

### Short-term (This Milestone)

1. {Address major gaps}

### Long-term (Ongoing)

1. {Improve infrastructure}

## Appendix

### Flaky Tests

[List with failure patterns]

### Slow Tests

[List with durations]

### Disabled Tests

[List with reasons]
```

---

## Review Frequency

| Review Type | Frequency | Scope                   | Owner     |
| ----------- | --------- | ----------------------- | --------- |
| Quick Check | Weekly    | Pass rates, flaky tests | QA        |
| Full Review | Monthly   | Coverage, quality       | Tech Lead |
| Deep Dive   | Quarterly | Infrastructure, strategy| Team      |

---

## Deliverables

1. **Test Review Report** — Comprehensive analysis
2. **Action Items** — Prioritized improvements
3. **Coverage Matrix** — Visual gap identification
4. **Technical Debt List** — Tests needing refactor

---

## Validation

Refer to `checklist.md` for validation criteria.

## On Complete

Run: `python3 {project-root}/_bmad/scripts/resolve_customization.py --skill {skill-root} --key workflow.on_complete`

If the resolved `workflow.on_complete` is non-empty, follow it as the final terminal instruction before exiting.
