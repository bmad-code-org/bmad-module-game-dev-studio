# Validate Reference

The Validate intent playbook: spawn a validator subagent, run the checklist against the PRD, render a report. Standalone — this intent critiques an existing PRD without changing it and ends after the user has seen the report; it does not run Finalize. The render pipeline below is also reused for mid-session report requests during Create/Update.

## Orient

Source-extract against `decision-log.md`, any original inputs (including the GDD), and the PRD/addendum themselves. Delegate to subagents per PRD Discipline → "Extract, don't ingest" (in SKILL.md); the parent assembles from extracts.

## Run the validator

Spawn the validator subagent with: the full text of `prd.md` (and `addendum.md` if present), the checklist at `{workflow.validation_checklist}`, and the project-context extracts. Instruct it to evaluate every checklist item and write `{doc_workspace}/validation-findings.json`:

```json
{
  "prd_name": "Hollow Tide",
  "prd_path": "{doc_workspace}/prd.md",
  "checklist_path": "{workflow.validation_checklist}",
  "timestamp": "2026-05-30T09:14:00",
  "overall_synthesis": "2-3 sentences of judgment about the PRD's overall state — what holds up, what's at risk. Written by the subagent.",
  "findings": [
    {
      "id": "Q-7",
      "category": "Quality",
      "title": "FR testability",
      "status": "warn",
      "severity": "medium",
      "location": "§4.2 Combat, FR-9",
      "note": "FR-9 says the boss 'feels fair' but states no observable acceptance condition.",
      "suggested_fix": "Add a testable consequence, e.g. 'telegraph windows are at least 0.4s before any one-shot attack.'"
    }
  ]
}
```

Per-finding fields: `id` (checklist item ID), `category` (optional), `title`, `status` (`pass` | `warn` | `fail` | `n/a`), `severity` (`low` | `medium` | `high` | `critical`), `location` (cite specifics), `note`, `suggested_fix`.

## Render the report

After the subagent writes findings, the parent fills `{workflow.validation_report_template}` directly — read the findings JSON, populate the skeleton's placeholders (header with PRD name/path, overall synthesis, findings grouped by category, pass/warn/fail counts, a derived grade), and write the result to `{doc_workspace}/validation-report.html`. Write a markdown twin to `{doc_workspace}/validation-report.md` (same content, grouped by severity — this is the canonical form Update mode reads when rolling findings into a revision).

Grade derivation: *Excellent* = no fails, no high/critical findings · *Good* = no critical findings, at most minor fails · *Fair* = any high finding or several fails · *Poor* = any critical finding.

For interactive runs, open the HTML:

```bash
python3 -c "import webbrowser, pathlib; webbrowser.open(pathlib.Path('{doc_workspace}/validation-report.html').resolve().as_uri())"
```

Skip the open step in headless mode (see `references/headless.md`). Re-running validation overwrites the report in place.

## Close

Surface findings tiered, never dumped: lead with a one-sentence verdict, walk critical and high findings, roll medium/low into a tail. The rendered HTML/markdown is the persistent artifact. Always offer to roll findings into an Update.
