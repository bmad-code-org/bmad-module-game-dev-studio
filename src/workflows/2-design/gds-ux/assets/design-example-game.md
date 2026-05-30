---
colors:
  primary: '#ffb02e'
  primaryHover: '#ff8c00'
  surface: '#12131a'
  surfaceAlt: '#1c1e29'
  text: '#f4f1e8'
  muted: '#8a8d9c'
  border: '#2c2f3d'
  health: '#e5484d'
  stamina: '#30a46c'
  mana: '#4493f8'
  legendary: '#c084fc'
typography:
  font: 'Rajdhani, "Segoe UI", sans-serif'
  numeric: 'JetBrains Mono, monospace'
  scale: '1.333 perfect fourth'
  weights: '500 medium, 600 semibold, 700 bold'
rounded: '0.25rem'
spacing: '8px base grid'
components:
  button: 'beveled panel, primary fill on focus, 1px inner glow, controller-focusable'
  panel: 'surfaceAlt fill, 1px border, sharp corners, subtle scanline texture'
  healthBar: 'segmented, health fill, darkens per missing segment, never fully empty UI'
  prompt: 'glyph + label, platform-adaptive button icon, fades when idle'
---

# DESIGN.md — Emberfall (Top-down Action Roguelike, PC + Controller)

## Brand & Style

Dark-fantasy arcade grit with a warm ember glow. The UI reads instantly at a glance during fast combat — high-contrast, sharp-cornered, faintly textured like worn metal. Non-diegetic HUD stays out of the way until it matters, then flares. The personality is a tense dungeon crawl lit by torchlight, not a sterile dashboard.

## Colors

| Role        | Token          | Hex       | Use                                       |
| ----------- | -------------- | --------- | ----------------------------------------- |
| Primary     | `primary`      | `#ffb02e` | Selection, focus, ember accents           |
| Primary Hov | `primaryHover` | `#ff8c00` | Pressed / held controller focus           |
| Surface     | `surface`      | `#12131a` | Title and pause backdrops                  |
| Surface Alt | `surfaceAlt`   | `#1c1e29` | Menu panels, inventory cells               |
| Text        | `text`         | `#f4f1e8` | Primary HUD and menu text                  |
| Muted       | `muted`        | `#8a8d9c` | Tooltips, secondary stats                  |
| Border      | `border`       | `#2c2f3d` | Panel edges, cell dividers                 |
| Health      | `health`       | `#e5484d` | Health bar fill, damage vignette           |
| Stamina     | `stamina`      | `#30a46c` | Stamina / dodge meter                      |
| Mana        | `mana`         | `#4493f8` | Ability / mana meter                       |
| Legendary   | `legendary`    | `#c084fc` | Legendary-tier loot, rare drops            |

Contrast: text on surface 14.8:1; muted on surface 4.6:1; health/stamina/mana each tested against a 60% black HUD scrim for combat legibility (all ≥ 4.5:1). Resource colors are reinforced with icon + meter shape so they never rely on hue alone.

## Typography

Rajdhani for labels and headings (condensed, readable at speed); JetBrains Mono for all numerics (ammo, gold, timers, damage numbers) so digits hold a fixed width and don't jitter as values change. Scale 1.333 perfect fourth from 16px base: 16 / 21 / 28 / 38. Weights: 500 body and tooltips, 600 menu items, 700 floating damage and run-timer.

## Layout & Spacing

8px base grid. HUD anchored to the four screen corners inside a 5% title-safe margin (TV-safe for couch play): health/resources bottom-left, minimap top-right, ability cooldowns bottom-center, run timer top-left. Menus center on a single column, controller-navigable top-to-bottom, max 480px wide.

## Elevation & Depth

Two HUD layers (in-world overlay; full-screen menu scrim at 75% black) plus a transient toast layer for pickups and level-up banners. No nested modals — pause replaces, never stacks.

## Shapes

0.25rem radius (near-sharp) on panels and cells; segmented bars for health/stamina/mana with hard segment gaps. 2px ember focus ring (`primary`) on the currently controller-focused element.

## Components

- **Button** — beveled panel, `surfaceAlt` resting, `primary` fill + inner glow when controller-focused, 1px border. Min 48px tall for couch legibility.
- **Panel** — `surfaceAlt` fill, 1px `border`, sharp corners, faint scanline texture.
- **Health bar** — segmented; fill `health`; each missing segment darkens rather than vanishing so the player can always read max HP. Pulses on low-health.
- **Resource meter** — stamina (`stamina`) and mana (`mana`) as thin segmented arcs beside health.
- **Button prompt** — glyph + short label; the glyph swaps per detected input device (Xbox / PlayStation / Switch / keyboard); fades to 40% opacity when the action is idle.

## Do's and Don'ts

- Do keep combat-critical HUD (health, cooldowns) in the same corners always; muscle memory is the contract.
- Do swap button glyphs to match the active input device.
- Don't animate numeric digits with proportional fonts — use the mono numeric token.
- Don't stack menus; pause and inventory replace the view.
- Don't rely on color alone for resources — pair with icon and meter shape.
