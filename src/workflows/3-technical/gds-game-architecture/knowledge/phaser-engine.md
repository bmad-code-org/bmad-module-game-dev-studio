# Phaser Engine Architecture Knowledge

## Overview

Phaser is an open-source HTML5 game framework for making 2D browser games with JavaScript or TypeScript. It renders via WebGL (with Canvas fallback), supports Arcade and Matter.js physics, and is ideal for web-first games, game jams, and casual/mobile browser games.

## Architecture Fundamentals

### Scene-Based Architecture

Phaser games are organized into Scenes, each representing a distinct game state (menu, gameplay, pause, game over).

**Key concepts:**

- **Game** — Root container. Holds configuration, manages scenes, owns renderer and systems
- **Scene** — Self-contained game state with its own update loop, physics, cameras, and object management
- **GameObject** — Any visual or interactive entity in a scene (Sprite, Text, Container, etc.)
- **Plugin** — Extends engine functionality globally or per-scene

**Game configuration:**

```typescript
const config: Phaser.Types.Core.GameConfig = {
    type: Phaser.AUTO,              // WebGL with Canvas fallback
    width: 800,
    height: 600,
    parent: 'game-container',
    backgroundColor: '#1a1a2e',
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 300 },
            debug: false
        }
    },
    scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
    scene: [BootScene, PreloadScene, MenuScene, GameScene, UIScene]
};

const game = new Phaser.Game(config);
```

### Scene Lifecycle

Every Scene goes through a defined lifecycle:

```
init(data)        → Called when scene starts (receives data from scene.start/launch)
preload()         → Load assets (images, audio, tilemaps, etc.)
create(data)      → Create game objects, set up physics, input, events
update(time, dt)  → Called every frame (game logic, input polling)
```

**Additional callbacks:**

```typescript
class GameScene extends Phaser.Scene {
    init(data: { level: number }) {
        this.level = data.level;  // Receive data from scene transition
    }

    preload() {
        this.load.image('player', 'assets/player.png');
        this.load.spritesheet('coins', 'assets/coins.png', {
            frameWidth: 32, frameHeight: 32
        });
    }

    create() {
        this.player = this.physics.add.sprite(100, 100, 'player');
        this.setupInput();
        this.setupCollisions();
    }

    update(time: number, delta: number) {
        this.handleMovement(delta);
    }
}
```

### Scene Management Patterns

```typescript
// Switch scenes (stops current, starts target)
this.scene.start('GameScene', { level: 1 });

// Launch scene in parallel (e.g., HUD over gameplay)
this.scene.launch('UIScene');

// Pause/resume
this.scene.pause('GameScene');
this.scene.resume('GameScene');

// Scene stacking for UI layers
this.scene.bringToTop('PauseMenu');
this.scene.sendToBack('GameScene');

// Restart current scene
this.scene.restart();
```

**Typical scene structure:**

```
BootScene       → Minimal setup, load loading bar assets
PreloadScene    → Load all game assets, show progress bar
MenuScene       → Title screen, options
GameScene       → Core gameplay
UIScene         → HUD (launched parallel to GameScene)
PauseScene      → Pause overlay (launched on top)
GameOverScene   → Results, retry
```

### Event System

Phaser has a built-in event emitter system:

```typescript
// Scene-level events
this.events.on('player-died', this.handlePlayerDeath, this);
this.events.emit('player-died', { score: this.score });

// Global events (cross-scene communication)
this.game.events.on('score-changed', this.updateScore, this);
this.game.events.emit('score-changed', newScore);

// Clean up listeners (important to prevent leaks)
this.events.off('player-died', this.handlePlayerDeath, this);

// Or use once for one-time events
this.events.once('level-complete', this.showVictory, this);
```

### Data Registry

Phaser provides a built-in key-value registry for sharing data:

```typescript
// Store data accessible to all scenes
this.registry.set('playerLives', 3);
this.registry.set('highScore', 0);

// Read from any scene
const lives = this.registry.get('playerLives');

// Listen for changes
this.registry.events.on('changedata-playerLives', (parent, value) => {
    this.livesText.setText(`Lives: ${value}`);
});
```

## TypeScript vs JavaScript Decision Guide

| Factor | TypeScript | JavaScript |
|---|---|---|
| **Type safety** | Full static typing | None (runtime errors) |
| **IDE support** | Excellent autocomplete | Basic |
| **Phaser API** | Built-in type definitions | No type hints |
| **Build step** | Required (Vite, Webpack) | Optional |
| **Learning curve** | Slight overhead | Immediate |
| **Refactoring** | Safe, compiler-assisted | Error-prone |
| **Team projects** | Strongly recommended | Risky for >1 person |

**Recommendation:** Always use TypeScript for Phaser projects. The type definitions are excellent and catch many bugs at compile time. Use the official Phaser + Vite template as a starter.

## Coding Best Practices

### Scene Class Structure

```typescript
export class GameScene extends Phaser.Scene {
    // Typed properties
    private player!: Phaser.Physics.Arcade.Sprite;
    private enemies!: Phaser.Physics.Arcade.Group;
    private cursors!: Phaser.Types.Input.Keyboard.CursorKeys;
    private score: number = 0;
    private scoreText!: Phaser.GameObjects.Text;

    constructor() {
        super({ key: 'GameScene' });
    }

    create(): void {
        this.createWorld();
        this.createPlayer();
        this.createEnemies();
        this.setupInput();
        this.setupCollisions();
        this.setupUI();
    }

    update(time: number, delta: number): void {
        this.handlePlayerMovement();
        this.updateEnemies(delta);
    }

    // Private methods organized by concern
    private createPlayer(): void { /* ... */ }
    private createEnemies(): void { /* ... */ }
    private setupInput(): void { /* ... */ }
    private setupCollisions(): void { /* ... */ }
    private setupUI(): void { /* ... */ }
    private handlePlayerMovement(): void { /* ... */ }
    private updateEnemies(delta: number): void { /* ... */ }
}
```

### Physics Setup Patterns

**Arcade Physics (simple, fast):**

```typescript
// Create physics-enabled sprites
this.player = this.physics.add.sprite(100, 300, 'player');
this.player.setCollideWorldBounds(true);
this.player.setBounce(0.2);

// Groups for pooling
this.bullets = this.physics.add.group({
    classType: Bullet,
    maxSize: 30,
    runChildUpdate: true
});

// Collisions
this.physics.add.collider(this.player, this.platforms);
this.physics.add.overlap(this.player, this.coins, this.collectCoin, undefined, this);
this.physics.add.collider(this.bullets, this.enemies, this.hitEnemy, undefined, this);
```

**Matter.js Physics (realistic, complex):**

```typescript
// Config
physics: {
    default: 'matter',
    matter: {
        gravity: { y: 1 },
        debug: false
    }
}

// Usage
const ball = this.matter.add.image(400, 100, 'ball', undefined, {
    restitution: 0.8,
    friction: 0.005,
    shape: { type: 'circle', radius: 24 }
});
```

**When to use which:**

| Arcade | Matter.js |
|---|---|
| Platformers, shooters, most 2D games | Physics puzzles, ragdolls, realistic simulation |
| AABB and circle collision only | Complex shapes, joints, constraints |
| Very fast | Slower, more accurate |
| No rotation physics | Full rotation and torque |

### Input Handling

```typescript
// Keyboard
this.cursors = this.input.keyboard!.createCursorKeys();
const wasd = this.input.keyboard!.addKeys('W,A,S,D') as {
    W: Phaser.Input.Keyboard.Key;
    A: Phaser.Input.Keyboard.Key;
    S: Phaser.Input.Keyboard.Key;
    D: Phaser.Input.Keyboard.Key;
};

// In update
if (this.cursors.left.isDown) {
    this.player.setVelocityX(-160);
} else if (this.cursors.right.isDown) {
    this.player.setVelocityX(160);
} else {
    this.player.setVelocityX(0);
}

if (this.cursors.up.isDown && this.player.body!.touching.down) {
    this.player.setVelocityY(-330);
}

// Pointer (mouse/touch)
this.input.on('pointerdown', (pointer: Phaser.Input.Pointer) => {
    this.fireBullet(pointer.worldX, pointer.worldY);
});

// Gamepad
this.input.gamepad?.once('connected', (pad: Phaser.Input.Gamepad.Gamepad) => {
    this.gamepad = pad;
});
```

### Project Structure Conventions

```
project/
├── src/
│   ├── main.ts                  # Game config and entry point
│   ├── scenes/
│   │   ├── BootScene.ts
│   │   ├── PreloadScene.ts
│   │   ├── MenuScene.ts
│   │   ├── GameScene.ts
│   │   ├── UIScene.ts
│   │   └── GameOverScene.ts
│   ├── objects/                  # Game object classes
│   │   ├── Player.ts
│   │   ├── Enemy.ts
│   │   ├── Bullet.ts
│   │   └── Pickup.ts
│   ├── systems/                  # Game systems
│   │   ├── ScoreManager.ts
│   │   ├── AudioManager.ts
│   │   └── SaveManager.ts
│   ├── data/                     # Data definitions
│   │   ├── LevelData.ts
│   │   ├── EnemyTypes.ts
│   │   └── Constants.ts
│   └── utils/                    # Utilities
│       ├── MathUtils.ts
│       └── ObjectPool.ts
├── public/
│   ├── assets/
│   │   ├── images/
│   │   ├── audio/
│   │   ├── tilemaps/
│   │   └── fonts/
│   └── index.html
├── tests/
│   ├── unit/
│   └── e2e/
├── package.json
├── tsconfig.json
└── vite.config.ts
```

**Naming conventions:**

- Files: `PascalCase` for classes (e.g., `GameScene.ts`, `Player.ts`), `camelCase` for utilities
- Classes: `PascalCase`
- Methods and properties: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Scene keys: `PascalCase` strings (e.g., `'GameScene'`, `'MainMenu'`)
- Assets: `kebab-case` (e.g., `player-idle.png`, `jump-sfx.mp3`)

## Performance Optimization

### General Guidelines

- **Target 60fps** — browser games must be smooth; profile with Chrome DevTools
- **Minimize `update()` work** — use events and timers where possible
- **Object pooling is critical** — never create/destroy objects in gameplay loops
- **Limit active game objects** — cull off-screen objects, disable physics on invisible entities

### Rendering Performance

- **Use Texture Atlases** — pack sprites with TexturePacker or free alternatives. Single draw call per atlas
- **Sprite batching** — Phaser auto-batches sprites using the same texture. Keep atlas grouping logical
- **Avoid `Phaser.GameObjects.Graphics`** in gameplay — it regenerates geometry every frame. Use pre-rendered images instead
- **Camera culling** — Phaser only renders objects in camera view by default, but disable offscreen object updates manually
- **Bitmap Text over Dynamic Text** — `BitmapText` is faster than `Text` for frequently changing values (score, timer)
- **Minimize blend modes** — each blend mode change breaks the batch

### Memory and Asset Loading

- **Preload everything in PreloadScene** — avoid loading during gameplay
- **Use audio sprites** — combine SFX into a single file with markers (reduces HTTP requests)
- **Compress textures** — use WebP for modern browsers, PNG for compatibility
- **Limit asset resolution** — browser games don't need 4K textures; 512px-1024px is typical
- **Unload unused scenes:** `this.scene.remove('OldScene')` when no longer needed

### Object Pool Pattern

```typescript
export class BulletPool {
    private pool: Phaser.Physics.Arcade.Group;

    constructor(scene: Phaser.Scene) {
        this.pool = scene.physics.add.group({
            classType: Bullet,
            maxSize: 50,
            runChildUpdate: true,
            createCallback: (obj) => {
                (obj as Bullet).init();
            }
        });
    }

    spawn(x: number, y: number, velocityX: number, velocityY: number): Bullet | null {
        const bullet = this.pool.get(x, y) as Bullet | null;
        if (bullet) {
            bullet.fire(velocityX, velocityY);
        }
        return bullet;
    }
}
```

### JavaScript/TypeScript Performance Tips

- **Avoid object allocation in update loops** — reuse Vector objects, pre-allocate arrays
- **Use `Phaser.Math` utilities** — optimized for game math (distance, angles, interpolation)
- **Minimize DOM interaction** — let Phaser handle rendering, avoid touching the Canvas element
- **Web Workers** for heavy computation (pathfinding, procedural generation) — keep the main thread free

## Popular Third-Party Plugins and Libraries

### Phaser Plugins

| Plugin | Purpose | Source |
|---|---|---|
| **Rex Plugins** | Massive collection (UI, board, behavior trees, CSV parsing, virtual joystick) | GitHub (rexrainbow) |
| **phaser3-rex-notes** | Documentation for Rex plugins | GitHub |
| **phaser-matter-collision-plugin** | Better Matter.js collision callbacks | npm |
| **phaser-navmesh** | 2D navigation mesh pathfinding | npm |
| **phaser3-rex-plugins (Board)** | Board/grid game framework (hex, square grids, pathfinding) | npm |

### Companion Libraries

| Library | Purpose | Source |
|---|---|---|
| **Tiled** | Level editor (TMX/JSON tilemaps) | mapeditor.org (free) |
| **TexturePacker** | Sprite atlas packing | texturepacker.com (free/paid) |
| **Howler.js** | Advanced audio (alternative to Phaser audio) | npm |
| **Colyseus** | Multiplayer game server (Node.js) | colyseus.io |
| **Socket.IO** | WebSocket-based multiplayer | npm |
| **LDtk** | Level designer toolkit (alternative to Tiled) | ldtk.io (free) |

### Build and Deploy

| Tool | Purpose | Source |
|---|---|---|
| **Vite** | Fast build tool (recommended bundler for Phaser) | npm |
| **Capacitor** | Wrap web game as native mobile app | npm |
| **Electron** | Wrap web game as desktop app | npm |
| **itch.io** | Free game hosting and distribution | itch.io |
| **CrazyGames / Poki** | Browser game portals (monetization) | Developer programs |

## Common Architectural Patterns

### Custom GameObject Class

```typescript
export class Player extends Phaser.Physics.Arcade.Sprite {
    private health: number = 100;
    private speed: number = 200;
    private cursors!: Phaser.Types.Input.Keyboard.CursorKeys;

    constructor(scene: Phaser.Scene, x: number, y: number) {
        super(scene, x, y, 'player');

        scene.add.existing(this);
        scene.physics.add.existing(this);

        this.setCollideWorldBounds(true);
        this.cursors = scene.input.keyboard!.createCursorKeys();
    }

    update(): void {
        // Called if group has runChildUpdate = true
        if (this.cursors.left.isDown) {
            this.setVelocityX(-this.speed);
            this.setFlipX(true);
        } else if (this.cursors.right.isDown) {
            this.setVelocityX(this.speed);
            this.setFlipX(false);
        } else {
            this.setVelocityX(0);
        }

        if (this.cursors.up.isDown && this.body!.touching.down) {
            this.setVelocityY(-400);
        }
    }

    takeDamage(amount: number): void {
        this.health -= amount;
        this.scene.cameras.main.shake(100, 0.01);
        if (this.health <= 0) {
            this.scene.events.emit('player-died');
        }
    }
}
```

### Scene Communication Pattern

```typescript
// GameScene launches UIScene in parallel
class GameScene extends Phaser.Scene {
    create(): void {
        this.scene.launch('UIScene');

        // Send data to UI
        this.events.on('score-changed', (score: number) => {
            this.scene.get('UIScene').events.emit('update-score', score);
        });
    }
}

// UIScene listens
class UIScene extends Phaser.Scene {
    create(): void {
        this.scoreText = this.add.text(10, 10, 'Score: 0');

        const gameScene = this.scene.get('GameScene');
        gameScene.events.on('score-changed', (score: number) => {
            this.scoreText.setText(`Score: ${score}`);
        });
    }
}
```

### State Machine for Entities

```typescript
interface State {
    enter(): void;
    update(delta: number): void;
    exit(): void;
}

class StateMachine {
    private currentState: State | null = null;
    private states = new Map<string, State>();

    addState(name: string, state: State): void {
        this.states.set(name, state);
    }

    setState(name: string): void {
        this.currentState?.exit();
        this.currentState = this.states.get(name) ?? null;
        this.currentState?.enter();
    }

    update(delta: number): void {
        this.currentState?.update(delta);
    }
}
```

### Responsive Scaling

```typescript
const config: Phaser.Types.Core.GameConfig = {
    scale: {
        mode: Phaser.Scale.FIT,          // Fit to container, preserve ratio
        autoCenter: Phaser.Scale.CENTER_BOTH,
        width: 800,
        height: 600,
        min: { width: 400, height: 300 },
        max: { width: 1600, height: 1200 }
    },
    // OR for mobile-first pixel art:
    scale: {
        mode: Phaser.Scale.RESIZE,       // Dynamic resize
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
    render: {
        pixelArt: true,                  // Disable anti-aliasing for pixel art
        antialias: false
    }
};
```

## Browser and Platform Considerations

### Browser Compatibility

- **WebGL** — supported by all modern browsers. Phaser uses it by default
- **Canvas fallback** — `Phaser.AUTO` falls back to Canvas if WebGL unavailable
- **Audio autoplay** — browsers block audio until user interaction. Use `this.sound.unlock()` or gate audio behind a "Click to Start" screen
- **Fullscreen** — use `this.scale.startFullscreen()` only inside a user-gesture handler

### Mobile Browser Optimization

- **Touch input** — Phaser handles touch as pointer events automatically
- **Virtual joystick** — use Rex Virtual Joystick plugin for mobile controls
- **Performance** — target 30fps on low-end mobile; reduce particle counts, simplify physics
- **PWA** — Phaser games can be wrapped as Progressive Web Apps for installability
- **Viewport meta tag:**

```html
<meta name="viewport" content="width=device-width, initial-scale=1,
      maximum-scale=1, user-scalable=no, viewport-fit=cover">
```

### Wrapping as Native App

- **Mobile:** Capacitor (recommended) or Cordova
- **Desktop:** Electron or Tauri (smaller binary)
- **Steam:** Electron + Steamworks.js for Steam API integration

## Licensing and Cost

Phaser is licensed under the **MIT License** — fully free and open source with no royalties, no fees, and no restrictions on commercial use.

**Cost implications:**

- Engine: Free forever, no revenue thresholds
- Hosting: Web games can be hosted anywhere (itch.io free, own server, game portals)
- Monetization: CrazyGames and Poki have revenue-sharing models for browser game portals
- Tooling: Vite/TypeScript toolchain is free. TexturePacker has a free tier. Tiled is free
- No vendor lock-in: standard web technologies throughout

## When NOT to Use Phaser

Phaser is the wrong choice when:

- **3D graphics are needed** — Phaser is 2D only. Use Unity, Godot, or Unreal for 3D
- **Native mobile/desktop is the primary target** — While Capacitor/Electron wrapping works, native engines provide better performance, platform integration, and app store compliance
- **Complex physics simulation** — Arcade physics is limited to AABB/circle. Matter.js handles more but is slow for many bodies. Native engines have far superior physics
- **CPU-intensive game logic** — JavaScript/TypeScript has a performance ceiling. For complex AI, large-scale simulation, or heavy pathfinding, the single-threaded JS runtime becomes a bottleneck (see Performance Ceiling section)
- **Team has no web development experience** — Phaser assumes familiarity with npm, bundlers, TypeScript, and browser APIs. Game-engine-only developers may struggle
- **Multiplayer with complex server authority** — Phaser has no built-in networking. You must build or integrate a separate server (Colyseus, custom Node.js)

## Performance Ceiling and WebAssembly

JavaScript/TypeScript performance hits a ceiling for computationally intensive games:

- **Single-threaded by default** — the main thread handles rendering, physics, and game logic. Heavy computation blocks frame rendering
- **Web Workers** can offload computation but cannot access the DOM, Canvas, or Phaser APIs directly. Communication is via message passing (serialization overhead)
- **WebAssembly (Wasm)** provides near-native performance for CPU-heavy logic:
  - **AssemblyScript** — TypeScript-like syntax compiling to Wasm (lowest friction for Phaser devs)
  - **Rust + wasm-bindgen** — Best Wasm performance, steeper learning curve
  - **Emscripten (C/C++)** — Compile existing C libraries to Wasm
  - Wasm modules handle computation; Phaser handles rendering and input
- **Architectural impact:** If your game needs heavy pathfinding, procedural generation, or physics beyond Arcade/Matter, plan for a Wasm module boundary early. This changes your build pipeline and data flow architecture

**Rule of thumb:** If your game would run fine as a mobile app on a mid-range phone, Phaser can handle it. If it needs the kind of computation that would stress a native app, consider a native engine or plan for Wasm from the start.

## Architecture Decision Checklist

When choosing Phaser for a project, verify:

- [ ] Game is 2D and targeting web browsers as primary platform
- [ ] TypeScript + Vite build pipeline configured
- [ ] Physics engine selected: Arcade (simple) or Matter.js (realistic)
- [ ] Asset pipeline: atlas packing tool, audio sprite tool
- [ ] Responsive scaling strategy for target screen sizes
- [ ] Mobile controls plan if targeting mobile browsers (virtual joystick, touch zones)
- [ ] Multiplayer approach if needed: Colyseus, Socket.IO, or custom WebSocket
- [ ] Monetization/distribution: itch.io, CrazyGames, Poki, or self-hosted
- [ ] Native wrapping needed: Capacitor for mobile, Electron/Tauri for desktop
- [ ] Audio autoplay handling for browser restrictions
