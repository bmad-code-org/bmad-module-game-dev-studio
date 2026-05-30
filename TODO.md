# TODO - what's on BMGD's schedule

* Adapt document-project workflow to BMGD
* Add and adapt research agents to pre-production (either to designer agent or new analyst agent)
* Review whether `src/workflows/gds-quick-flow/` parent directory still earns its keep now that it contains only `gds-quick-dev/`. Candidate: promote `gds-quick-dev` up to `src/workflows/4-production/` or similar and retire the `gds-quick-flow/` wrapper.
* Build the research and prototyping skills that were referenced but never existed: `gds-market-research`, `gds-technical-research`, and `gds-quick-prototype`. Their stale `module-help.csv` rows and the `QP` agent-menu entries (designer, dev, solo-dev) were removed in the BMAD-METHOD v6.6.0 sync — rebuild the skills, then restore the catalog rows and menu entries.
* Optionally fork a game-adapted `gds-spec`. Upstream's `bmad-spec` kernel distiller (#2417) is a `core-skills` skill supplied to GDS via the `core` module, so it was not forked during the v6.8.0 sync. If a game-flavored variant is wanted later (game-framed Overview, routing to `gds-gdd` / `gds-prd` / `gds-quick-dev`), fork it under `gds-quick-flow/` or `2-design/`.