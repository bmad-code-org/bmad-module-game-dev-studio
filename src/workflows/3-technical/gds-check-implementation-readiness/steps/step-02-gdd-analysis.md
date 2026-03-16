---
name: 'step-02-gdd-analysis'
description: 'Read and analyze GDD to extract all FRs and NFRs for coverage validation'

nextStepFile: './step-03-epic-coverage-validation.md'
outputFile: '{planning_artifacts}/implementation-readiness-report-{{date}}.md'
epicsFile: '{planning_artifacts}/*epic*.md' # Will be resolved to actual file
---

# Step 2: GDD Analysis

## STEP GOAL:

To fully read and analyze the GDD document (whole or sharded) to extract all Functional Requirements (FRs) and Non-Functional Requirements (NFRs) for validation against epics coverage.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 🛑 NEVER generate content without user input
- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step with 'C', ensure entire file is read
- 📋 YOU ARE A FACILITATOR, not a content generator
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Role Reinforcement:

- ✅ You are an expert Game Producer and Scrum Master
- ✅ Your expertise is in requirements analysis and traceability
- ✅ You think critically about requirement completeness
- ✅ Success is measured in thorough requirement extraction

### Step-Specific Rules:

- 🎯 Focus ONLY on reading and extracting from GDD
- 🚫 Don't validate files (done in step 1)
- 💬 Read GDD completely - whole or all sharded files
- 🚪 Extract every FR and NFR with numbering

## EXECUTION PROTOCOLS:

- 🎯 Load and completely read the GDD
- 💾 Extract all requirements systematically
- 📖 Document findings in the report
- 🚫 FORBIDDEN to skip or summarize GDD content

## GDD ANALYSIS PROCESS:

### 1. Initialize GDD Analysis

"Beginning **GDD Analysis** to extract all requirements.

I will:

1. Load the GDD document (whole or sharded)
2. Read it completely and thoroughly
3. Extract ALL Functional Requirements (FRs)
4. Extract ALL Non-Functional Requirements (NFRs)
5. Document findings for coverage validation"

### 2. Load and Read GDD

From the document inventory in step 1:

- If whole GDD file exists: Load and read it completely
- If sharded GDD exists: Load and read ALL files in the GDD folder
- Ensure complete coverage - no files skipped

### 3. Extract Functional Requirements (FRs)

Search for and extract:

- Numbered FRs (FR1, FR2, FR3, etc.)
- Requirements labeled "Functional Requirement"
- User stories or use cases that represent functional needs
- Business rules that must be implemented
- Game mechanics and gameplay requirements

Format findings as:

```
## Functional Requirements Extracted

FR1: [Complete requirement text]
FR2: [Complete requirement text]
FR3: [Complete requirement text]
...
Total FRs: [count]
```

### 4. Extract Non-Functional Requirements (NFRs)

Search for and extract:

- Performance requirements (frame rate, load times, throughput)
- Platform requirements (target hardware, OS, resolution)
- Usability requirements (accessibility, ease of use, control schemes)
- Reliability requirements (save system integrity, crash tolerance)
- Scalability requirements (concurrent players, data growth)
- Compliance requirements (ratings, standards, regulations)

Format findings as:

```
## Non-Functional Requirements Extracted

NFR1: [Performance requirement]
NFR2: [Platform requirement]
NFR3: [Usability requirement]
...
Total NFRs: [count]
```

### 5. Document Additional Requirements

Look for:

- Constraints or assumptions
- Technical requirements not labeled as FR/NFR
- Business constraints
- Integration requirements
- Game engine or tool-specific constraints

### 6. Add to Assessment Report

Append to {outputFile}:

```markdown
## GDD Analysis

### Functional Requirements

[Complete FR list from section 3]

### Non-Functional Requirements

[Complete NFR list from section 4]

### Additional Requirements

[Any other requirements or constraints found]

### GDD Completeness Assessment

[Initial assessment of GDD completeness and clarity]
```

### 7. Auto-Proceed to Next Step

After GDD analysis complete, immediately load next step for epic coverage validation.

## PROCEEDING TO EPIC COVERAGE VALIDATION

GDD analysis complete. Loading next step to validate epic coverage.

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- GDD loaded and read completely
- All FRs extracted with full text
- All NFRs identified and documented
- Findings added to assessment report

### ❌ SYSTEM FAILURE:

- Not reading complete GDD (especially sharded versions)
- Missing requirements in extraction
- Summarizing instead of extracting full text
- Not documenting findings in report

**Master Rule:** Complete requirement extraction is essential for traceability validation.
