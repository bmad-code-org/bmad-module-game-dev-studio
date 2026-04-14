---
name: gds-create-ux-design
description: 'Plan UX patterns and design specifications for game UI/HUD elements. Use when the user says "lets create UX design" or "create UX specifications" or "help me plan the game UX"'
---

## Available Scripts

- **`scripts/resolve-customization.py`** -- Resolves customization from three-layer TOML merge (user > team > defaults). Outputs JSON.

## Resolve Customization

Resolve `inject` and `additional_resources` from customization:
Run: `python3 scripts/resolve-customization.py gds-create-ux-design --key inject --key additional_resources`
Use the JSON output as resolved values.

1. **Inject before** -- If `inject.before` resolved to a non-empty value, prepend it to your active instructions and follow it.
2. **Available resources** -- Note the `additional_resources` list. Do not read these files now; they are available for the injected prompt or workflow steps to reference when needed.

Follow the instructions in ./workflow.md.

## Post-Workflow Customization

After the workflow completes, resolve `inject.after` from customization:
Run: `python3 scripts/resolve-customization.py gds-create-ux-design --key inject.after`

If resolved `inject.after` is not empty, append it to your active instructions and follow it.
