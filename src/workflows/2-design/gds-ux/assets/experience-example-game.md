---
title: 'EXPERIENCE.md — Emberfall (Top-down Action Roguelike)'
status: example
form_factor: PC + controller (couch and desk)
---

# EXPERIENCE.md — Emberfall (Top-down Action Roguelike)

## Foundation

PC title, shipping on Steam with first-class controller support and Steam Deck verification as a goal. Two postures: desk (keyboard + mouse, monitor) and couch (controller, TV across the room). Engine UI: Unity UI Toolkit; HUD elements are custom render-layer overlays. Visual identity: see DESIGN.md. Foundation must hold for both input schemes and both viewing distances.

## Information Architecture

Flat, fast, controller-first:

- **Title** → New Run · Continue · Loadout · Options · Quit.
- **In-Run HUD** (no navigation — always-on overlay): health/resources bottom-left, ability cooldowns bottom-center, minimap top-right, run timer + depth top-left.
- **Pause** → Resume · Inventory · Map · Options · Abandon Run. Replaces the view; never stacks.
- **Inventory / Loadout** → grid of items, controller cursor wraps; compare-on-focus.

Nothing is deeper than two levels from the HUD. Pause is one button from anywhere.

## Voice and Tone

Terse, evocative, dungeon-flavored. "The torch gutters." not "Low light warning." Pickups read as discoveries ("Ember-forged Blade"), not inventory line-items. Death screens are grim but inviting a retry ("Fallen at Depth 4. The dark remembers. Again?").

## Component Patterns

- **Menu item** — controller D-pad / stick navigates; A/Enter confirms; B/Esc backs out one level. Focus state per DESIGN.md `{colors.primary}`. Behavior; visual spec in DESIGN.md.Components.
- **Inventory cell** — focus shows a compare tooltip vs. the equipped item; A equips, X drops, Y inspects.
- **HUD meters** — health/stamina/mana update every frame; never animate the digit font (use the mono numeric token).
- **Button prompt** — context-sensitive; glyph swaps to match the active input device (see Input Schemes).

## State Patterns

- **Idle in-combat** — HUD at full opacity; idle prompts fade to 40%.
- **Low health** — health bar pulses, screen-edge damage vignette in `{colors.health}`, low-health audio sting.
- **Loading between rooms** — brief diegetic transition (descending stairs), never a spinner.
- **Death** — run summary overlay (depth, time, kills, loot), single "Again?" focus default.
- **Disconnect (controller)** — auto-pause, "Reconnect controller" prompt; mid-combat never punishes a dropped pad.

## Interaction Primitives

- Move (stick / WASD), dodge (button / Space — costs stamina), attack, ability ×2, interact.
- Hold-to-confirm on destructive actions (Abandon Run, Drop legendary).
- Rumble/haptic on hit landed, hit taken, and pickup (light/medium/heavy impact).
- Mouse hover = controller focus equivalent; both drive the same focus state.

## HUD & Diegetic UI

The HUD is **non-diegetic** by default — a clean overlay the character is not aware of. Two deliberate **diegetic** exceptions: the torch flame in the world dims as a light-radius timer (replacing a bare meter), and boss health renders as cracks spreading across the boss model in addition to the non-diegetic bar. HUD **information hierarchy**: health and cooldowns are always full-opacity (combat-critical); minimap and run timer are secondary; pickup toasts are transient. During cutscenes and safe rooms, non-critical HUD fades out; health stays. Nothing combat-critical ever hides during active play.

## Input Schemes

- **Controller (primary)** — full play on a gamepad; left stick move, right trigger attack, face buttons abilities, bumpers swap loadout. All actions remappable.
- **Keyboard + mouse** — WASD + mouse-aim; every binding remappable; mouse also drives menu focus.
- **Steam Deck** — treated as controller; verify text legibility at 7-inch handheld size (min 24px effective for combat-critical labels).
- **Context-sensitive prompts** — the interact glyph appears only near interactables; glyph art swaps per detected device (Xbox / PlayStation / Switch Pro / keyboard).
- **Remapping** — a full bindings screen in Options; conflict detection; reset-to-default per scheme.

## Game Feel & Juice

The felt responsiveness of the UI is part of UX, not just combat. On hit landed: brief hit-stop (≈60ms), small directional screen shake, a floating damage number (mono font, `{colors.primary}`), and a controller rumble pulse. On level-up: time briefly dilates, a banner sweeps in, the ability bar flashes. On legendary drop: the toast uses `{colors.legendary}`, a distinct chime, and a heavier rumble. Juice is **tunable and respects the reduced-motion / reduced-shake accessibility toggle** — when set, shake and hit-stop drop to zero while damage numbers and audio cues remain.

## Accessibility Floor

Targets WCAG-AA-equivalent plus game-specific standards (informed by the AbleGamers / Xbox accessibility guidelines):

- **Remappable controls** for every action, both schemes — non-negotiable.
- **Colorblind modes** (protanopia / deuteranopia / tritanopia) plus the rule that resources never rely on hue alone (icon + meter shape carry the signal).
- **Subtitles and captions** on by default; speaker labels; size and background-opacity sliders; captions for important non-speech audio (boss roar, trap arming).
- **Reduced-motion / reduced-shake** toggle (drives Game Feel & Juice above).
- **HUD scale slider** and a TV-safe 5% margin so corner HUD is never clipped on overscan displays.
- **Hold-vs-toggle** option for held actions (sprint, aim) for limited-dexterity players.
- Combat-critical labels legible at handheld (Steam Deck) distance and couch (TV) distance.

## Responsive & Platform

- **Desk (KB+M, monitor)** — denser tooltips, hover affordances, mouse menu navigation.
- **Couch (controller, TV)** — larger focus rings, 5% title-safe HUD margin, hover replaced by focus, bigger combat-critical type.
- **Handheld (Steam Deck)** — controller scheme, 7-inch legibility pass, battery-friendly effects.
- Layout adapts by input device and viewing distance, not just resolution; the HUD anchor corners stay fixed across all three for muscle memory.

## Key Flows

### Rosa clears the Depth-4 boss on her third run

Rosa, a speedrunner, knows the layout cold and is hunting a sub-12-minute clear on her Steam Deck.

1. She drops into Depth 4; the run timer (top-left, mono font) reads 9:48.
2. The boss arena seals — a diegetic stone door slams; the boss bar fades in bottom-center.
3. She weaves dodges; her stamina arc (`{colors.stamina}`) empties and she waits a beat for one segment to refill — the segmented meter tells her exactly how much she has.
4. At 20% boss HP the model starts visibly cracking (diegetic) and her own health bar pulses red with an edge vignette — she's one hit from death.
5. **Climax:** she lands the killing blow. Hit-stop snaps, a heavy rumble fires, the screen flashes, a legendary drops in `{colors.legendary}` with its chime, and the run timer freezes at 11:31 on the victory banner. Sub-12. She feels it in her hands before she reads it.
6. The "Again?" prompt is pre-focused; one button restarts.

### Kenji plays on the couch with a controller, first time

Kenji, new to the genre, is on the TV across the room with a PlayStation pad. The interact prompt shows a PlayStation cross glyph (device-detected), HUD type is large enough to read from the sofa, and when he wanders near a shrine the context prompt fades in only then. He opens Options and finds reduced-shake and a HUD-scale slider in the first screen — he bumps both, and the game stops making him queasy without hiding anything he needs.
